** L-01 No deadline check in the mint and burn can cause unintended bad trades
Minting and Burning positions does not have a deadline check. Without this check, there could be situations where a user 

** L-02 No check for active sequencer on L2s 
The protocol does not check whether the L2 sequencer is active this can result in outdated trades and incorrect premia calculations 

** L-03 No enforcement of fee range 
The readme states that only certain fee ranges are supported. However this is not enforced through the code. 

** L-04 User can front run a transfer by transfering their own options
A bad actor can front run any transfer of options by transferring an option of his own 

** L-05 