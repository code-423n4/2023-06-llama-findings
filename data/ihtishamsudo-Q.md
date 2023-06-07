# [L-01] IMMUTABLE ADDRESSES LACK ZERO ADDRESS CHECK

Constructors should check the address written in an immutable address variable is not the zero address.

##### Reference
[LlamaExecutor.sol#L12](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaExecutor.sol#L12)
[LlamaExecutor.sol#L16](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaExecutor.sol#L16)
[LlamaPolicyMetadataParamRegistry.sol#l43](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaPolicyMetadataParamRegistry.sol#L44-L45)
##### Mitigation Step
Add a zero address check for `LLAMA_CORE`

# [N-01] IT IS NOT GOOD PRACTICE TO USE COMMENTS BETWEEN NETSPAC COMMENTS

##### Reference

[LlamaExecutor.sol#L21-21](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaExecutor.sol#L21-L22)

##### Mitigation Step 

Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction)