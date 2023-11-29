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

QA4. LeftRight.toRightSlot() fails to check that that there might be an overflow. As a result, the function might return wrong value and some users might lost funds during trading. 

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L75-L80](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L75-L80)

Mitigation: 
```diff
 function toRightSlot(int256 self, int128 right) internal pure returns (int256) {
        int128  rightSum = int128(self >> 128) + right; // overflow will be detected here out of unchecked

        // bit mask needed in case rightHalfBitPattern < 0 due to 2's complement
        unchecked {
            return self + (int256(right) & RIGHT_HALF_BIT_MASK);
        }
    }
```

QA5.  LeftRight.toRightSlot() fails to check that that there might be an overflow. As a result, the function might return wrong value and some users might lose funds during trading. 

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L65-L69](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L65-L69)

Mitigation: 

```diff
 function toRightSlot(int256 self, uint128 right) internal pure returns (int256) {
+       if(right > uint128(type(int128).max)) revert Errors.UnderOverFlow();
+       int128 rightsum = int128(self >> 128) + int128(right); // overflow/underflow will be detected here

        unchecked {
            return self + int256(uint256(right));
        }
    }
```

QA6. A second LeftRight.toRightSlot() fails to check that that there might be an overflow. As a result, the function might return wrong value and some users might lose funds during trading. 

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L54-L59](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L54-L59)

Mitigation: 

```diff
 function toRightSlot(uint256 self, int128 right) internal pure returns (uint256) {
        if (right < 0) revert Errors.LeftRightInputError();
        unchecked {
-            return self + uint256(int256(right));
+            uint256 z = self + uint256(int256(right));
+            if(z >> 128 < uint128(right)) revert Errors.UnderOverFlow();
        }

    }
```

QA7.  A third LeftRight.toRightSlot() fails to check that that there might be an overflow. As a result, the function might return wrong value and some users might lose funds during trading. 

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L44-L48](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L44-L48)

Mitigation: 
```diff
 function toRightSlot(uint256 self, uint128 right) internal pure returns (uint256) {
        unchecked {
            return self + uint256(right);
        }
+       if(uint128(self >> 128)  < right) revert Errors.UnderOverFlow();
    }
```
