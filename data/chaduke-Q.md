QA. LeftRight.toLeftSlot() fails to detect that int128(left) might overflow. As a result, the function might be return wrong values sometimes and lead to loss fundings during trading.

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
