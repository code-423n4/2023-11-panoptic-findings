QA1. LeftRight.toLeftSlot() fails to detect that int128(left) might overflow. As a result, the function might be return wrong values sometimes and lead to loss fundings during trading.

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L118-L122](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L118-L122)

Mitigation: check wether int128(left) overflows since ``left`` might be too large to be contained in int128

```diff
function toLeftSlot(int256 self, uint128 left) internal pure returns (int256) {
 +     if(left > uint128(type(int128).max) revert Errors.CastingError(); 
       unchecked {
            return self + (int256(int128(left)) << 128);
        }
    }
``

QA2. Another LeftRight.toLeftSlot() fails to check that that there might be an overflow. As a result, the function might return wrong value and some users might lost funds during trading. 

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L108-L112](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L108-L112)

```diff
  function toLeftSlot(uint256 self, uint128 left) internal pure returns (uint256) {
        unchecked {
-            return self + (uint256(left) << 128);
+            uint256 z = self + (uint256(left) << 128);
+            if(uint128(z >> 128) < left) revert Errors.UnderOverFlow();

        }
    }
```

QA3. A third  LeftRight.toLeftSlot() fails to check that that there might be an overflow. As a result, the function might return wrong value and some users might lost funds during trading. 

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L128C14-L132](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L128C14-L132)

Mitigation:

```diff
 function toLeftSlot(int256 self, int128 left) internal pure returns (int256) {
        unchecked {
            return self + (int256(left) << 128);
        }
        int128 leftSum = int128(self >> 128) + left; // overflow will be caught out of unchecked 
    }
```