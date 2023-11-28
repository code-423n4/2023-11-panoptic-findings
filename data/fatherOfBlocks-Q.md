**contracts/types/TokenId.sol**
- L337/338/339/340/341/342/343/344/345/346/353 - Multiple numbers 2 ** 64/112/160/208 are used and the optionRatios value is set with a number between 0 and 4 , but they are not defined with a name that explains their meaning, this is important to increase the understanding of the code.

- L377/385/386 - A division is made by an input that is int24 tickSpacing, therefore it could be zero and generate an error, it should be previously validated and an appropriate exception thrown.


**contracts/libraries/Math.sol**
- L112 - A division is made by the lowPriceX96 variable, which is a by-product of the liquidityChunk input, therefore it could be worth zero and generate an error, it should be previously validated and an appropriate exception thrown.


**contracts/libraries/CallbackLib.sol**
- L39/40 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/libraries/PanopticMath.sol**
- L57 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/SemiFungiblePositionManager.sol**
- L594/595/603/604/1104/1105/1353/1380/1446 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.