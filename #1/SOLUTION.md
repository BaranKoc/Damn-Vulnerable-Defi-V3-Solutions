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

Here **`ReceiverUnstoppable.sol`** contract's **`onFlashLoan(...)`** function

([**ReceiverUnstoppable.sol**](contracts/ReceiverUnstoppable.md))

<br/>

Here **`UnstoppableVault.sol`** contract's the **`flashloan(...)`** function

([**UnstoppableVault.sol**](contracts/UnstoppableVault.md))

<br/>

The vulnarability cames from, **`UnstoppableVault.sol`** contract's the **`flashloan(...)`** function. 

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

        /*´:°•.°+.*•´.*:˚.°*.˚•´.°:°•.°•.*•´.*:˚.°*.˚•´.°:°•.°+.*•´.*:*/
        /*            THIS IS WHERE THE PROBLEM CAMES FROM            */
        /*.•°:°.´+˚.*°.˚:*.´•*.+°.•°:´*.´•*.•°.•°:°.´:•˚°.*°.˚:*.´+°.•*/ 
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

[**(Test File)**](Solution.md)

<br/>


## How could this exploit be prevented ?
I think instead of using **`convertToShares(totalSupply) != balanceBefore`**,  

**`convertToShares(totalSupply) >= balanceBefore`** should be used inside **`UnstoppableVault.sol`** contract's **`onFlashLoan(...)`** function.



<br/>


## How to test it
got to ➡️ [**Damn Vulnerable Defi - Chalange #1**](https://www.damnvulnerabledefi.xyz/challenges/1.html). Clone the repository as mentioned and put [**solution code**](Solution.md) to his place.

Then you can just type **`npm run unstoppable`**


