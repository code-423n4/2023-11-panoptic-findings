https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L342-L343

Missing check your zero address when assign the uniswapv3Factory address in the constructor.


The constructor sets the address of the `uniswapv3Factory` contract 

```
    constructor(IUniswapV3Factory _factory) {
        FACTORY = _factory;
    }
```
While assign the address to the variable there's no check of the address is a zero address or not in which case if it is a zero address the assignment should revert.

Allowing the assignment without checking if the address is a valid address will lead to the contract address of the `uniswapV3Factory ` point to the wrong address or a zero address this can happen due I human error or some unforeseen issues.

When assign a new address with a constructor it is advisable to include a zero address check.

## FIX 
Add a require check if the `_factory` is a zero address and revert if true.