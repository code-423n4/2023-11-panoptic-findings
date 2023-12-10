# L1 Incorrect/misleading comment on TokenId example
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L52
``` solidity
///  - the underlying strike price of the 2nd leg (leg index = 1) in this option position starts at bit index  (64 + 12 + 48 * (leg index=1))=123
```
By specification the strike price of the second leg will commence at bit index 124
``` solidity
--- ///  - the underlying strike price of the 2nd leg (leg index = 1) in this option position starts at bit index  (64 + 12 + 48 * (leg index=1))=123
+++ ///  - the underlying strike price of the 2nd leg (leg index = 1) in this option position starts at bit index  (64 + 12 + 48 * (leg index=1))=124 
```

#L2  Redundant casting
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1381
The variable came in form of an address, so there is no need to cast it in an address type
``` solidity 
        bytes32 positionKey = keccak256(
          ---  abi.encodePacked(address(univ3pool), owner, tokenType, tickLower, tickUpper)
          +++  abi.encodePacked(univ3pool, owner, tokenType, tickLower, tickUpper)
        );
```