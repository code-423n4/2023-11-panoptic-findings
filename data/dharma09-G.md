## Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
File: /tokens/ERC1155Minimal.sol
178: function balanceOfBatch(
179:        address[] calldata owners,
180:        uint256[] calldata ids
181:    ) public view returns (uint256[] memory balances) {
182:        balances = new uint256[](owners.length);

        // Unchecked because the only math done is incrementing
        // the array index counter which cannot possibly overflow.
186:        unchecked {
187:            for (uint256 i = 0; i < owners.length; ++i) {
188:                balances[i] = balanceOf[owners[i]][ids[i]];
189:            }
190:        }
191:    }
```

> Gas Diff
> 

```diff
diff --git a/contracts/tokens/ERC1155Minimal.sol b/contracts/tokens/ERC1155Minimal.sol
index c9e69d8..7bf5955 100644
--- a/contracts/tokens/ERC1155Minimal.sol
+++ b/contracts/tokens/ERC1155Minimal.sol
@@ -187,8 +187,11 @@ abstract contract ERC1155 {
+             do{
+             //for (uint256 i = 0; i < owners.length; ++i) {
                 balances[i] = balanceOf[owners[i]][ids[i]];
+            
+            unchecked {
+                ++i;
             }
-        }
+        } while(i< owners.length);
     }
```

```diff
| Function Name    | min             | avg   | median | max    | # calls |
- | balanceOfBatch | 8292            | 8292  | 8292   | 8292   | 3       |
+ | balanceOfBatch | 8125            | 8125  | 8125   | 8125   | 3       |
```

### Instead of using separate state variables for multiple boolean flags,use a single `uint256`

- Instead of using separate state variables for multiple boolean flags, you can use a single `uint256` and manipulate its bits to represent multiple flags. This will reduce the storage cost. This is completely safe as long as you manage the bits carefully and avoid overflows.
- Extra context: The bot report mentions `using uint256(1)/uint256(2) instead of true/false to save gas`, which is a similar concept to this one but not identical, therefore this is a unique gas optimization and is not present on the bot report.

After Optimization:

```
uint256 public flags;

function setFlag(uint256 _flag, bool _value) internal {
    if (_value) {
        flags |= _flag;
    } else {
        flags &= ~_flag;
    }
}

function getFlag(uint256 _flag) internal view returns (bool) {
    return (flags & _flag) != 0;
}
```

- Estimated gas saved = Approximately 15,000 - 20,000 gas per `flag` operation.

Avoids a Gsset (20000 gas) when changing from false to true, after having been true in the past. Since most of the bools aren't changed twice in one transaction.

```solidity
File: contracts/SemiFungiblePositionManager.sol
127: bool internal constant MINT = false; //@audit
128: bool internal constant BURN = true;
```

## Possible Optimizations in s_accountLiquidity[positionKey_to]

By using assembly, we can directly interact with the storage slot of the stored variable, allowing us to efficiently write and read address values in storage.

```solidity
File:
626: //update+store liquidity and fee values between accounts
627:            s_accountLiquidity[positionKey_to] = fromLiq; //@audit
628:            s_accountLiquidity[positionKey_from] = 0;
629:
630:            s_accountFeesBase[positionKey_to] = fromBase; //@audit
631:            s_accountFeesBase[positionKey_from] = 0;
632:            unchecked {
633:                ++leg;
634:            }
```