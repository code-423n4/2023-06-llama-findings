## Avoid extra computation
1. First checking, prevent doing extra for loop.

[https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L318-L324](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L318-L324)
```
+    if (msg.value != actionInfo.value) revert IncorrectMsgValue();
     Action storage action = actions[actionInfo.id];
+    if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();
     ActionState currentState = getActionState(actionInfo);
	 if (currentState != ActionState.Queued) revert InvalidActionState(currentState);
-    if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();
-    if (msg.value != actionInfo.value) revert IncorrectMsgValue();
```
 
2. use OR instead of and

 [https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L21](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L21)
``` 
   modifier onlyAuthorized(LlamaExecutor llamaExecutor) {
-    if ( 
-      msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
 -	   ) revert UnauthorizedCaller();
    _;

+    if (
+     msg.sender == address(llamaExecutor) || ...
+   )
 ```

3. do check2 before check1
[https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L262-L270](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L262-L270)

```
    // Check 2.
    if (caller != actionInfo.creator) revert OnlyActionCreator();
    
    // Check 1.
    ActionState state = llamaCore.getActionState(actionInfo);
    if (
      state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
        || state == ActionState.Failed
    ) revert CannotCancelInState(state);
```

