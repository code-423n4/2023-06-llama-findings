# Findings Summary

| ID     | Title                                                | Severity |
| ------ | ---------------------------------------------------- | -------- |
| [G-01] | _validateActionInfoHash is no need                   | Gas      |

# Detailed Findings

# [G-01] _validateActionInfoHash is no need

## Link

- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L665-L675
- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L350
- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L491

## Description

The elements that make up `infoHash` do not change, so don't need to check the frontrun with `_validateActionInfoHash`

## Recommendations

remove _validateActionInfoHash
