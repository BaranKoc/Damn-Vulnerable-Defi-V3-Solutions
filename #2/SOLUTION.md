## What is our mission
There’s a pool with 1000 ETH in balance, offering flash loans. It has a fixed fee of 1 ETH.

A user has deployed a contract with 10 ETH in balance. It’s capable of interacting with the pool and receiving flash loans of ETH.

Take all ETH out of the user’s contract. If possible, in a single transaction.

<br/>

## What are the smart contracts 
- **`NaiveReceiverLenderPool.sol`**: smart contract that includes **`flashLoan(...)`** function
- **`FlashLoanReceiver.sol`**: smart contract that includes **`onFlashLoan(...)`** function

<br/>


## Solution 

**1.** **`NaiveReceiverLenderPool.sol`** contract's **`flashLoan(...)`** function
``` solidity
// NaiveReceiverLenderPool.sol

function flashLoan(
        IERC3156FlashBorrower receiver,
        address token,
        uint256 amount,
        bytes calldata data
    ) external returns (bool) {
        if (token != ETH)
            revert UnsupportedCurrency();
        
        uint256 balanceBefore = address(this).balance;

        // Transfer ETH and handle control to receiver
        SafeTransferLib.safeTransferETH(address(receiver), amount);
        if(receiver.onFlashLoan(
            msg.sender,
            ETH,
            amount,
            FIXED_FEE,
            data
        ) != CALLBACK_SUCCESS) {
            revert CallbackFailed();
        }

        if (address(this).balance < balanceBefore + FIXED_FEE)
            revert RepayFailed();

        return true;
    }
```

**2.** **`FlashLoanReceiver.sol`** contract's **`onFlashLoan(...)`** function

``` solidity
// FlashLoanReceiver.sol

function onFlashLoan(
        address,
        address token,
        uint256 amount,
        uint256 fee,
        bytes calldata
    ) external returns (bytes32) {
        assembly { // gas savings
            if iszero(eq(sload(pool.slot), caller())) {
                mstore(0x00, 0x48f5c3ed)
                revert(0x1c, 0x04)
            }
        }
        
        if (token != ETH)
            revert UnsupportedCurrency();
        
        uint256 amountToBeRepaid;
        unchecked {
            amountToBeRepaid = amount + fee;
        }

        _executeActionDuringFlashLoan();

        // Return funds to pool
        SafeTransferLib.safeTransferETH(pool, amountToBeRepaid);

        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }

    // Internal function where the funds received would be used
    function _executeActionDuringFlashLoan() internal { }
```

<br/>

The vulnarability cames from,  **`FlashLoanReceiver.sol`** contract.

**`onFlashLoan(...)`** function is triggered from `NaiveReceiverLenderPool.sol` when **`flashLoan(...)`** is called.



⚠️ But **`onFlashLoan(...)`** function doesn't check Who has called **`flashLoan(...)`** function.

This means anybody who call **`flashLoan(...)`** function can pass  **`FlashLoanReceiver.sol`** contract as a `receiver` paremether to **`flashLoan(...)`** function. This will trigger **`onFlashLoan(...)`** function inside **`FlashLoanReceiver.sol`**.

And **`FlashLoanReceiver`** will pay transection fee until be out of money. 😥😥😥

[**(Basic Solution Test File)**](Basic.Solution.md)

<br/>

But we have send 10 transection. If you want to do this challenge **in single transection** fallow this,

[**(Single transection Test File)**](Single.Transection.Solution.md)


<br/>

## How could this exploit be prevented ?
We want the **sender of the transection** to be equal to **users address**.

first we have to create new variable inside **`FlashLoanReceiver.sol`** contract named **`owner`**

``` solidity
address private owner;

// Modify the constructor
constructor(address _pool, address _owner) {
        pool = _pool;
        owner = _owner;
    }
```
Than inside **`FlashLoanReceiver.sol`** contract's **`onFlashLoan(...)`** function we will require **`tx.origin`** to be equal to **`owner`**.

``` solidity
function onFlashLoan(...) external returns (bytes32) {

        require(tx.origin != owner, "Wrong sender");

        /...
        /...
        /...
    }
```

<br/>


## How to test it
got to ➡️ [**Damn Vulnerable Defi - Chalange #2**](https://www.damnvulnerabledefi.xyz/challenges/2.html). Clone the repository as mentioned and,

put [**basic solution code**](Basic.Solution.md) to his place 

/or

put [**single transection solution code**](Single.Transection.Solution.md) to his place

Then you can just type **`npm run naive-receiver`** 