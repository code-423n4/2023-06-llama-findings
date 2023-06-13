# Low

## 1. Missing sanity checks for multiple parameters in `LlamaRelativeQuorum` `initialize` function allows for invalid strategy to be created in multiple ways.

###  no minimum `queuingPeriod` 

This will allow an action to be immediately executed once it has entered `queued` state, without any oppurtunity for another user to disapprove the action. 

The function `minExecutionTime` returns `block.timestamp + queuingPeriod`, so as `queuingPeriod` can be 0 this means in `LlamaCore#QueueAction`, `minExecutionTime` will == `block.timestamp`.

[LlamaCore#QueueAction](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L309-L311)
```
uint64 minExecutionTime = actionInfo.strategy.minExecutionTime(actionInfo);
if (minExecutionTime < block.timestamp) revert MinExecutionTimeCannotBeInThePast();
action.minExecutionTime = minExecutionTime;       
```

This revert check doesn't check if `minExecutionTime` == `block.timestamp`. Also in `LlamaCore#ExecuteAction` the following check is done: 

[LlamaCore#ExecuteAction](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L323)
```
if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();
```

This check will be bypassed and doesn't match the natspec for this function which states 
```
/// @notice Execute an action by its `actionInfo` struct if it's in Queued state and `minExecutionTime` has passed.
```

### `minExpirationPeriod` can be 0

This will mean an action that has entered the `queued` state will enter the `expired` state after 1 second, preventing any user from approving the action.

[LlamaRelativeQuorum#isActionExpired](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L295-L298)

```
return block.timestamp > action.minExecutionTime + expirationPeriod;
```

If an action is created while `expirationPeriod` == 0 and `minExecutionTime` == `block.timestamp` when `queueAction` was called (if queuingPeriod == 0), 

then when `executeAction` was called, `block.timestamp` will always be greater than `action.minExecutionTime + expirationPeriod` meaning `isActionExpired` will always return true.

This means the `getActionState` will always return the `expired` state [LlamaCore#getActionState](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L505)

So `executeAction` will revert [LlamaCore#executeAction](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L322)

```
if (currentState != ActionState.Queued) revert InvalidActionState(currentState);
```