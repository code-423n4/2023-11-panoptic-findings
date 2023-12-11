### All users should be required to transfer funds with a value greater than 0
In ERC1155Minimal.sol, in the function "safeTransferFrom" there is no check for the transferred funds which means that 0 funds can be transferred from one user to another which will lead to redundant gas usage. Consider approving only transfers with amount that is greater that 0

require(amount > 0, "Amount must be greater than zero");
