### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Code changes |

| Total Found Issues | 6 |
|:--:|:--:|

### [Low Risk](#low-risk) 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01]| `LlamaAccount.batchApproveERC20` may revert if one of the ERC20 tokens current approval is not zero | 1 |
| [L-02] |  Approval race conditions in `LlamaAccount.sol` | 1 |
| [L-03] | Action can be executed when expiration period is reached | 3 |

| Total Low Risk Issues | 3 |
|:--:|:--:|

### [Non-Critical](#non-critical) 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [NC-01] | No function to remove `forceApproval/forceDisapproval` role except by revoking policies| 1 |
| [NC-02] | Default enum variable is Active instead of Canceled in `ActionState` | 1 |

| Total Non-Critical Issues | 2 |
|:--:|:--:|

### [Refactor](#refactor) 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01] |Refactor `LlamaCore._preCastAssertions()` | 1 |

| Total Refactor Issues | 1 |
|:--:|:--:|

## Low Risk

## [L-01] `LlamaAccount.batchApproveERC20` may revert if one of the ERC20 tokens current approval is not zero

## Impact
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L187-L192

Calling `approve() without first calling approve(0) `if the current approval is non-zero will revert with some tokens, such as Tether (USDT). While Tether is known to do this, it applies to other tokens as well. 

In `LlamaAccount.batchApproveERC20`, if one of the native tokens is USDT has current approval that is not zero, whole function will revert.

## Recommendation
Always reset the approval to zero in `approveERC20()` before changing it to a new value, or use `safeIncreaseAllowance()/safeDecreaseAllowance()`

## [L-02] Approval race conditions in `LlamaAccount.sol`

## Impact
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L181-L183

Due to the implementation of the `safeApprove()` function in `LlamaAccount.sol`, it’s possible for a user to over spend their allowance in certain situations.

Consider the following scenario:

1. Alice approves an allowance of 5000 USDC to Bob.
2. Alice attempts to lower the allowance to 2500 USDC.
3. Bob notices the transaction in the mempool and front-runs it by using up the full allowance with a `transferFrom` call.
4. Alice’s lowered allowance is confirmed and Bob now has an allowance of 2500 USDC, which can be spent further for a total of 7500 USDC.

## Recommendation
Consider using `increaseAllowance` and `decreaseAllowance` instead of `safeApprove`. Also not that `safeApprove` is [deprecated.](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45)


## [L-03] Action can be executed when expiration period is reached

## Impact
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L286

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L297

From the different execution contracts, the `isActionExpired()` still allows execution of action when expiration period is reached. If execution is performed on the dot of expiry date, actions can be executed when they are not supposed to.

## Recommendation
Use `>=` instead of `>` in `isActionExpired`
```solidity
  function isActionExpired(ActionInfo calldata actionInfo) external view virtual returns (bool) {
    Action memory action = llamaCore.getAction(actionInfo.id);
--  return block.timestamp > action.minExecutionTime + expirationPeriod;
++  return block.timestamp >= action.minExecutionTime + expirationPeriod;
  }
```

## Non Critical

## [NC-01] No function to remove `forceApproval/forceDisapproval` role except by revoking policies

## Impact
Currently, there is no function to remove `forceApproval/forceDisapproval` role except by revoking policies

Although protocol owners can adjust role to have 0 quantity to prevent approval/disapproval casting, policyholders still retain `forceApproval/forceDisapproval role` forever unless another strategy is deployed, which can cause confusion.

## Recommendation
Consider adding a new `onlyLlama` function to change the `forceApprovalRole/forceDispprovalRole` mapping.


## [NC-02] Default enum variable is Active instead of Canceled in `ActionState`

## Impact
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Enums.sol#L5

Currently, the default value for action state is `Active` that may open a possible vulnerability that allow actions that do not exist to be queued for approval/disapproval

## Recommendation
Consider putting `Canceled` as the default action state in `ActionState` struct

## Refactor

## [R-01] Refactor `LlamaCore._preCastAssertions()`

## Details
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L607

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L611

In `LlamaCore._preCastAssertions()`, the zero quantity check for quantity is not needed since `hasRole()` already ensures `quantity > 0` if not it will revert.

Furthermore, `isApprovalEnabled()/isDisapprovalEnabled()` already checks that role is `approvalRole/disApprovalRole`, so `getApprovalQuantityAt` will never return 0 regardless if its approval/disapproval. 

`castApproval()` gas savings based on `LlamaCore.t.sol` test:
| Function Name  | min | avg | median  | max | # calls |
|:------: | :--- | :------- | :----- | :----- |  :----- | 
| castApproval                         | 99870           | 99870   | 99870   | 99870   | 67      |
| castApproval                         | 99841           | 99841   | 99841   | 99841   | 67      |

## Recommendation
Remove the `if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);` check in `_preCastAssertion()`



