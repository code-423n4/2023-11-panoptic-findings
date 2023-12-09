## L-01 No deadline check in the mint and burn can cause unintended bad trades
Minting and Burning positions does not have a deadline check. Without this check, there could be situations where a transaction is lingering in the mempool and a user forgets about it, resulting in unintended bad trades

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L510-L518

## L-02 No check for active sequencer on L2s 
The protocol does not check whether the L2 sequencer is active this can result in outdated trades and incorrect premia calculations 

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L510-L518

## L-03 No enforcement of fee range 
The readme states that only certain fee ranges are supported (50, 200 ,and 200 bp). However this is not enforced through the code. 

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L479-L480





## L-04 User can front run a transfer by transfering their own options
A bad actor can front run any transfer of options by transferring an option of his own 

## L-05 No check for sqrtPriceLimitX96 < slot0Start.sqrtPriceX96


## L-06 