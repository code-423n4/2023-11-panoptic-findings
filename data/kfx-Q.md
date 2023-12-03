The authors should consider adding functions for rescuing stuck ERC-20 tokens and Ether in case they end up in the manager's contract. (At least one its function is `payable` - specifically, the `multicall` function).

The functions could be similar to the `sweepToken` and `refundETH` functions in Uniswap's periphery contracts.
https://docs.uniswap.org/contracts/v3/reference/periphery/base/PeripheryPayments