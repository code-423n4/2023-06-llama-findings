# [G-01] USE NESTED IF STATEMENT FOR SAVING GAS

When using multiple && conditions we should consider nesting IF statements for saving gas.

``` solidity
if (
      msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
    ) revert UnauthorizedCaller();
```

##### Reference

[LlamaPolicyMetadataParamRegistry.sol#L20-22](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#LL20C5-L22C35)