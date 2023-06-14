## [N-01] Differentiate constants from immutables

Constants can be kept all capital letters while immutable variables can be named as i_variable for ease of understanding while reading the codebase.

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L71

```solidity
File: src/LlamaFactory.sol
71: LlamaCore public immutable LLAMA_CORE_LOGIC;
```

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L74

```solidity
File: src/LlamaFactory.sol
74: LlamaPolicy public immutable LLAMA_POLICY_LOGIC;
```

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L77

```solidity
File: src/LlamaFactory.sol
77: LlamaPolicyMetadataParamRegistry public immutable LLAMA_POLICY_METADATA_PARAM_REGISTRY;
```

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L80

```solidity
File: src/LlamaFactory.sol
80: LlamaExecutor public immutable ROOT_LLAMA_EXECUTOR;
```

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#LL83C46-L83C46

```solidity
File: src/LlamaFactory.sol
83: LlamaCore public immutable ROOT_LLAMA_CORE;
```

## [N-02] No event emitted after execution
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaExecutor.sol#L34
On success or failure of execution, an event should be emitted to show status of execution.
```solidity
File: src/LlamaExecutor.sol
29: function execute(address target, uint256 value, bool isScript, bytes calldata data)
30:     external
31:     returns (bool success, bytes memory result)
32:   {
33:     if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
34:     (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
        ///@audit emission over here after success or failure
35:  }
```



