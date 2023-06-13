# Findings Summary

| ID     | Title                                                | Severity |
| ------ | ---------------------------------------------------- | -------- |
| [L-01] | Clones.cloneDeterministic can be frontrun            | Low      |
| [L-02] | Malicious guard can break protocol                   | Low      |

# Detailed Findings

# [L-01] Clones.cloneDeterministic can be frontrun

## Link

- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L249-L255

## Description

If the initialization parameters and salt are in sync, frontrun can do nothing.   
But in factory, salt relies only on name, and if a name has value, confusing, a malicious user may frontrun to register the name to confuse user to arbitrage.   

## Recommendations

Keep initialization parameters and salt are in sync

# [L-02] Malicious guard can break protocol

## Link

- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L330
- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L339
- https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L550

## Description

A malicious user may deploy guard through an upgradable contract or create2 + selfdestruct.     
After action approved, the protocol can be corrupted by upgrading guard, especially those function with onlyLlama modifier, and the malicious guard will stop the protocol until guard is modified to a different address.    
In particular, for `LlamaCore.setGuard.selector`, if is a malicious guard that prevents the guard to modify again.   

## Recommendations

Consider prevent setGuard for `LlamaCore.setGuard.selector`