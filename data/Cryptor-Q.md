## L-01 No deadline check in the mint and burn can cause unintended bad trades
Minting and Burning positions does not have a deadline check. Without this check, there could be situations where a transaction is lingering in the mempool and a user forgets about it, resulting in unintended bad trades

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L510-L518

## L-02 No check for active sequencer on L2s 
The protocol does not check whether the L2 sequencer is active when minting or burning positions. This can result in outdated trades and incorrect premia calculations and exercising options that unexpectedly becomes out of the money 

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L510-L518

## L-03 No enforcement of fee range 
The readme states that only certain fee ranges are supported (50, 200 ,and 200 bp). However this is not enforced through the code. 

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L479-L480





## L-04 User can front run a transfer by transfering their own options
A bad actor can front run any transfer of options by transferring an option of his own since there is a rule that you cannot transfer an option to someone that already has one 


## L-05 Using sqrtPriceX96 could result in partial swaps in some situations

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L775




## L-06 Return Values from Uniswap callback not checked 


## L-07 Having Vegoid as a constant makes no sense 
The comments describe Vegoid as a way to measure the sensitivity of an option. However, the volatility of an option can change whenever there is a change in price
This is shown in the function _getpremiumdeltas


## L-07 minting positions violates CEI code pattern
Minting positions violates the checks-effects-interaction code pattern used to prevent reentrancy

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L519-L533



