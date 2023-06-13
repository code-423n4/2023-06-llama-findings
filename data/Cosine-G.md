## Duplicated condition checks should be refactored to a modifier or function

The following condition appears three times in LlamaPolicy.sol and should therefore be reused with a modifier or function to save deployment gas:

```solidity
if (role > numRoles) revert RoleNotInitialized(role);
```

Appearances within the code:

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L237

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L414

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L491