## [L-01] State variable visibility is not set in the Errors contract
## Impact
It is best practice to set the visibility of state variables explicitly. The default visibility for "CastingError" is internal. Other possible visibility settings are public and private.  Please see the underlying source.

It is best practice to set the visibility of state variables explicitly. The default visibility for "InvalidTick" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "InvalidUniswapCallback" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "InvalidTokenIdParameter" is internal. Other possible visibility settings are public and private.
It is best practice to set the visibility of state variables explicitly. The default visibility for "parameterType" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "LeftRightInputError" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "NoLegsExercisable" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "PositionTooLarge" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "NotEnoughLiquidity" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "OptionsBalanceZero" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "PriceBoundFail" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "ReentrantCall" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "TransferFailed" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "TicksNotInitializable" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "UnderOverFlow" is internal. Other possible visibility settings are public and private.

It is best practice to set the visibility of state variables explicitly. The default visibility for "UniswapPoolNotInitialized" is internal. Other possible visibility settings are public and private.

## Proof of Concept
The contract reference is ```https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol```
The line numbers are
```sol
// LINE 11
    error CastingError();
// LINE 14
    error InvalidTick();
// LINE 17
    error InvalidUniswapCallback();
// LINE 21
    error InvalidTokenIdParameter(uint256 parameterType);
// LINE 24
    error LeftRightInputError();
// LINE 27
    error NoLegsExercisable();
// LINE 30
    error PositionTooLarge();
// LINE 33
    error NotEnoughLiquidity();
// LINE 36
    error OptionsBalanceZero();
// LINE 39
    error PriceBoundFail();
// LINE 42
    error ReentrantCall();
// LINE 45
    error TransferFailed();
// LINE 49
    error TicksNotInitializable();
// LINE 52
    error UnderOverFlow();
// LINE 55
    error UniswapPoolNotInitialized();
```
## Tools Used
VS CODE.
## Recommended Mitigation Steps
Variables can be specified as being public, internal or private. Explicitly define visibility for all state variables.
