# The protocol's technical implementation doesn't match the one described inside of the whitepaper
## ERC1155
### Whitepaper
The whitepaper describes the ERC1155 `tokenId` as having the following the structure:
The first 160 bits are the 4 legs with each having the following structure:
- width (12 bits)
- strike (24 bits)
- defined risk partner (2 bits)
- put/call (1 bit)
- long/short (1 bit)

Overall each leg is 40 bits.

Then the ratio comes which is 16 bits where each 4 bits describe the nth leg's ratio.
Then it's the uniswap pool id which is 80 bits
### Actual code
In the actual code and code documentation the `tokenId` has the following structure:
The first 192 bits are the 4 legs with each having the following structure:
- asset (1 bit)
- optionRatio (7 bits)
- isLong (1 bit)
- tokenType (1 bit)
- riskPartner (2 bits)
- strike (24 bits)
- width (12 bits)

Overall each leg is 48 bits

Then it's the uniswap pool id which is 64 bits

## Recommendations
While the code is fully functional because everywhere where the `tokenId` is used, it's used in the way that is described inside of the code documentation but it's still advised to update the whitepaper or the code so they have the same implementation details so if any 3rd party decides to implement the `tokenId` in some way or another in their own protocol they would be able to do so by either looking at whitepaper or by looking at the code and documentation. Or if the users try to write their own smart contract implementing the `tokenId` it would work regardless of the source.