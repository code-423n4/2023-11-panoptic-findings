`%` can be replace by bitwise `&` to reduce 40 ~ 50 gas

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol

### Math
`x % (2**n)` is the same as `x & ((2 << n) - 1)`

### Results

`forge test --match-path test/foundry/poc/token_id.test.sol --gas-report --fuzz-runs 100`

```
| test/foundry/poc/token_id.test.sol:TokenIdBit contract |                 |     |        |     |         |
|--------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                        | Deployment Size |     |        |     |         |
| 81729                                                  | 440             |     |        |     |         |
| Function Name                                          | min             | avg | median | max | # calls |
| asset                                                  | 348             | 348 | 348    | 348 | 1       |
| isLong                                                 | 327             | 327 | 327    | 327 | 1       |
| optionRatio                                            | 305             | 305 | 305    | 305 | 1       |
| riskPartner                                            | 349             | 349 | 349    | 349 | 1       |
| tokenType                                              | 326             | 326 | 326    | 326 | 1       |
| width                                                  | 329             | 329 | 329    | 329 | 1       |


| test/foundry/poc/token_id.test.sol:TokenIdMod contract |                 |     |        |     |         |
|--------------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                        | Deployment Size |     |        |     |         |
| 95741                                                  | 510             |     |        |     |         |
| Function Name                                          | min             | avg | median | max | # calls |
| asset                                                  | 395             | 395 | 395    | 395 | 1       |
| isLong                                                 | 374             | 374 | 374    | 374 | 1       |
| optionRatio                                            | 341             | 341 | 341    | 341 | 1       |
| riskPartner                                            | 396             | 396 | 396    | 396 | 1       |
| tokenType                                              | 373             | 373 | 373    | 373 | 1       |
| width                                                  | 376             | 376 | 376    | 376 | 1       |

```

### Code

```solidity
pragma solidity ^0.8.0;

import "forge-std/Test.sol";

contract ModGasTest is Test {
    function test_asset(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48)) % 2);
        }
    }

    function test_optionRatio(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 1)) % 128);
        }
    }

    function test_isLong(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 8)) % 2);
        }
    }

    function test_tokenType(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 9)) % 2);
        }
    }

    function test_riskPartner(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 10)) % 4);
        }
    }

    function test_width(uint256 self, uint256 legIndex) public pure returns (int24) {
        unchecked {
            return int24(int256((self >> (64 + legIndex * 48 + 36)) % 4096));
        } // "% 4096" = take last (2 ** 12 = 4096) 12 bits
    }
}

contract BitGasTest is Test {
    function test_asset(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48)) & 1);
        }
    }

    function test_optionRatio(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 1)) & 127);
        }
    }

    function test_isLong(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 8)) & 1);
        }
    }

    function test_tokenType(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 9)) & 1);
        }
    }

    function test_riskPartner(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 10)) & 3);
        }
    }

    function test_width(uint256 self, uint256 legIndex) public pure returns (int24) {
        unchecked {
            return int24(int256((self >> (64 + legIndex * 48 + 36)) & 4095));
        } // "% 4096" = take last (2 ** 12 = 4096) 12 bits
    }
}

contract TokenIdMod {
    function asset(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48)) % 2);
        }
    }

    function optionRatio(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 1)) % 128);
        }
    }

    function isLong(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 8)) % 2);
        }
    }

    function tokenType(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 9)) % 2);
        }
    }

    function riskPartner(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 10)) % 4);
        }
    }

    function width(uint256 self, uint256 legIndex) public pure returns (int24) {
        unchecked {
            return int24(int256((self >> (64 + legIndex * 48 + 36)) % 4096));
        } // "% 4096" = take last (2 ** 12 = 4096) 12 bits
    }
}

contract TokenIdBit {
    function asset(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48)) & 1);
        }
    }

    function optionRatio(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 1)) & 127);
        }
    }

    function isLong(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 8)) & 1);
        }
    }

    function tokenType(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 9)) & 1);
        }
    }

    function riskPartner(uint256 self, uint256 legIndex) public pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 10)) & 3);
        }
    }

    function width(uint256 self, uint256 legIndex) public pure returns (int24) {
        unchecked {
            return int24(int256((self >> (64 + legIndex * 48 + 36)) & 4095));
        } // "% 4096" = take last (2 ** 12 = 4096) 12 bits
    }
}

contract CompareTest is Test {
    TokenIdBit private bit;
    TokenIdMod private mod;

    function setUp() public {
        bit = new TokenIdBit();
        mod = new TokenIdMod();
    }

    function test_asset(uint256 self, uint256 legIndex) public {
        assertEq(bit.asset(self, legIndex), mod.asset(self, legIndex));
    }

    function test_optionRatio(uint256 self, uint256 legIndex) public {
        assertEq(bit.optionRatio(self, legIndex), mod.optionRatio(self, legIndex));
    }

    function test_isLong(uint256 self, uint256 legIndex) public {
        assertEq(bit.isLong(self, legIndex), mod.isLong(self, legIndex));
    }

    function test_tokenType(uint256 self, uint256 legIndex) public {
        assertEq(bit.tokenType(self, legIndex), mod.tokenType(self, legIndex));
    }

    function test_riskPartner(uint256 self, uint256 legIndex) public {
        assertEq(bit.riskPartner(self, legIndex), mod.riskPartner(self, legIndex));
    }

    function test_width(uint256 self, uint256 legIndex) public {
        assertEq(bit.width(self, legIndex), mod.width(self, legIndex));
    }
}

```