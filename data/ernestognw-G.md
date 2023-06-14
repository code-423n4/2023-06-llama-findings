## LlamaCore.sol `_preCastAssertions` has a duplicated check

### Summary

In LlamaCore.sol, there's a check at [line 607](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L607) that is also done at [line 611](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L611).

The check can be changed by a single instruction at the end of both branches, reducing the bytecode size and also reducing the deployment costs while only increasing the average runtime cost by 1 gas unit:

```
diff --git a/.gas-report b/.gas-report
index 590577b..e7c8bfb 100644
--- a/.gas-report
+++ b/.gas-report
@@ -1,7 +1,7 @@
 | src/LlamaCore.sol:LlamaCore contract |                 |                     |         |                     |         |
 |--------------------------------------|-----------------|---------------------|---------|---------------------|---------|
 | Deployment Cost                      | Deployment Size |                     |         |                     |         |
-| 5636388                              | 28512           |                     |         |                     |         |
+| 5618569                              | 28423           |                     |         |                     |         |
 | Function Name                        | min             | avg                 | median  | max                 | # calls |
 | actionGuard                          | 1358            | 1358                | 1358    | 1358                | 1       |
 | actionsCount                         | 519             | 519                 | 519     | 519                 | 4       |
@@ -9,7 +9,7 @@
 | authorizeScript                      | 2454            | 14067               | 9067    | 28967               | 14      |
 | authorizedScripts                    | 912             | 912                 | 912     | 912                 | 1       |
 | cancelAction                         | 6309            | 39527               | 43334   | 66563               | 10      |
-| castApproval                         | 17611           | 89361               | 99870   | 112364              | 240     |
+| castApproval                         | 17611           | 89362               | 99870   | 112701              | 240     |
 | castApprovalBySig                    | 12372           | 75654               | 80831   | 127391              | 8       |
 | castDisapproval                      | 21384           | 86986               | 102725  | 104725              | 13      |
 | castDisapprovalBySig                 | 12350           | 87477               | 136111  | 138111              | 9       |
```

### Recommendation

Apply the following diff to LlamaCore:

```
diff --git a/src/LlamaCore.sol b/src/LlamaCore.sol
index 89d60de..c4eeebd 100644
--- a/src/LlamaCore.sol
+++ b/src/LlamaCore.sol
@@ -604,12 +604,11 @@ contract LlamaCore is Initializable {
     if (isApproval) {
       actionInfo.strategy.isApprovalEnabled(actionInfo, policyholder, role);
       quantity = actionInfo.strategy.getApprovalQuantityAt(policyholder, role, action.creationTime);
-      if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);
     } else {
       actionInfo.strategy.isDisapprovalEnabled(actionInfo, policyholder, role);
       quantity = actionInfo.strategy.getDisapprovalQuantityAt(policyholder, role, action.creationTime);
-      if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);
     }
+    if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);
   }
```