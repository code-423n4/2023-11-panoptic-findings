Unbounded loop vulnerabilities by DOS attack in SemiFungiblePositionManager smart contract:

The SemiFungiblePositionManager smart contract contains an instance of while loop that execute external calls. However, the number of iterations in these loops is not bounded and instead depends on the number of existing univ3pools. This poses a security risk, as executing a large number of external calls could potentially exceed the available gas limit and cause the initializeAMMPool function to revert and then DOS.

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L370C1-L372C10

Recommendation:

To mitigate this vulnerability, it is recommended to implement a limit on the maximum number of univ3pools that can be added to the SemiFungiblePositionManager contract. This would prevent the loops from becoming unbounded and help to ensure the smooth functioning of the initializeAMMPool functions.