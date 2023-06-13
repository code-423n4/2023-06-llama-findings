# Summary

| Id | Title |
| --- | --- |
| L-1 | Unchecked `minDisapprovalPct` during initialization |
| L-2 | `guard` may changed after `executeAction()` |
| L-3 | Prefer `safeTransferFrom()` for ERC721 transfers |
| NC-1 | Deprecated Natspec comment |

# L-1. Unchecked `minDisapprovalPct` during initialization

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L167

## Detail
In the `LlamaRelativeQuorum.sol` file at line 167, there is an issue where the `minDisapprovalPct` is not checked during initialization. Both `minApprovalPct` and `minDisapprovalPct` represent the minimum percentages required for an action to be approved or disapproved, respectively. Both values should be smaller than `ONE_HUNDRED_IN_BPS`. However, in the `initialize()` function, only `minApprovalPct` is checked, while `minDisapprovalPct` is not.
```solidity
if (strategyConfig.minApprovalPct > ONE_HUNDRED_IN_BPS) revert InvalidMinApprovalPct(minApprovalPct);
minApprovalPct = strategyConfig.minApprovalPct;
minDisapprovalPct = strategyConfig.minDisapprovalPct; // @audit not check minDisapproval
```

## Recommendation
Consider adding check for `minDisapprovalPct`.

# L-2. `guard` may changed after `executeAction()`

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L340

## Detail
During the execution of an action, there is a pre-execution guard check and a post-execution guard check. However, the `guard` address can be changed during action execution. This can lead to the post-execution check referencing the previous guard address, potentially resulting in incorrect behavior.

```solidity
// Check pre-execution action guard.
ILlamaActionGuard guard = actionGuard[actionInfo.target][bytes4(actionInfo.data)];
if (guard != ILlamaActionGuard(address(0))) guard.validatePreActionExecution(actionInfo);

// Execute action.
(bool success, bytes memory result) =
  executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);

if (!success) revert FailedActionExecution(result);

// Check post-execution action guard.

// @audit guard might already changed ?
if (guard != ILlamaActionGuard(address(0))) guard.validatePostActionExecution(actionInfo);
```

## Recommendation
Review the logic to ensure the guard address is handled correctly during action execution.

# L-3. Prefer `safeTransferFrom()` for ERC721 transfers

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#L200

## Detail

The `transferFrom()`` method is used instead of safeTransferFrom()`, presumably to save gas. I however argue that this isn’t recommended because:

* [OpenZeppelin’s documentation](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-) discourages the use of `transferFrom()`, use `safeTransferFrom()` whenever possible
* Given that any NFT can be used in LlamaAccount, there are a few NFTs (here’s an [example](https://github.com/sz-piotr/eth-card-game/blob/master/src/ethereum/contracts/ERC721Market.sol#L20-L31)) that have logic in the `onERC721Received()` function, which is only triggered in the `safeTransferFrom()` function and not in `transferFrom()`


## Recommendation
Consider using `safeTransferFrom()` when transferring ERC721

# NC-1. Deprecated Natspec comments

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L175

## Detail
In LlamaCore, there is a comment said
> /// @notice The ERC721 contract that defines the policies for this Llama instance.
  /// @dev We intentionally put this first so it's packed with the `Initializable` storage // @audit deprecated Natspec
  /// comment
  // variables, which are the key variables we want to check before and after a delegatecall.
  LlamaPolicy public policy;
  
This is no longer relevant in the current implementation of LlamaCore since the `delegatecall` is moved to `LlamaExecutor`.

## Recommendation
Consider removing this Natspec comment.
