## Unauthorized Caller Check in LlamaPolicyMetadataParamRegistry Contract

In the `onlyAuthorized` modifier, the comparison between `msg.sender` and `address(ROOT_LLAMA_EXECUTOR)` may fail if `ROOT_LLAMA_EXECUTOR` is uninitialized (i.e., its value is `address(0)`). To prevent this, add a check to ensure that `ROOT_LLAMA_EXECUTOR` is not equal to the zero address.

```solidity
require(ROOT_LLAMA_EXECUTOR != address(0), "Root Llama executor not set");
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L19-L24