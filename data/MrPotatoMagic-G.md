## [G-01] uint8 can be made uint256 to save gas
https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaFactory.sol#L68
Before VS After:
Deployment cost: 6571396 - 6562384 = 9012 gas saved
```solidity
File: src/LlamaFactory.sol
68: uint8 public constant BOOTSTRAP_ROLE = 1;
```

## [G-02] Extra memory variable created can be removed to save gas
https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaPolicyMetadataParamRegistry.sol#LL61C5-L61C5
Before VS After:
Deployment cost: 2388056 - 2388043 = 13 gas saved 

This line can be removed:
```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol
61: string memory rootColor = "#6A45EC";
```
Instead, we can hardcode rootColor in this way:
```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol
64: setColor(ROOT_LLAMA_EXECUTOR, "#6A45EC");
```
We could also hardcode rootLogo on line 62 to save gas but that would affect the readability of the code if entered as a parameter directly on Line 65.

### Total deployment cost: 9025 gas saved