
# Low Risk and Non-Critical Issues

# [L-01] Hash Collision with encodePacked (06 Instances)
abi.encodePacked can result in hash collision if arguments are passed with similar values. Refer to this article (https://coinsbench.com/solidity-25-keccak256-50776e91eae6).

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L250
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L253
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L687
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L741
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L763
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L785


# [L-02] Use a modifier for access control (10 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:
1.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/Router.sol#L71
2.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/Router.sol#L76
3.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/ARM.sol#L202
4.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/ARM.sol#L299
5.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/PriceRegistry.sol#L63
6.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/CommitStore.sol#L123
7.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/CommitStore.sol#L138
8.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/onRamp/EVM2EVMOnRamp.sol#L333
9.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/offRamp/EVM2EVMOffRamp.sol#L273
10.	https://github.com/code-423n4/2023-05-chainlink/blob/main/contracts/offRamp/EVM2EVMOffRamp.sol#L475


# [L 03] Missing initializer modifier on constructor (03 Instances)
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. 
If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol#L15
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L57
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L102



# [L-04] Add a timelock to critical functions (07 Instances)

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). 

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol#L33
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L82
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L90
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L154
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L62
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L430
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L436


# [L-05] unsafe downcasting can cause unexpected overflows when casting from higher to lower type int/uint values (04 Instances)

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L23
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L249
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L228
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L534

# [L-06] Missing checks for address(0x0) when assigning values to address state variables (05 Instances)

Zero-address check should be used, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L246
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L253
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L269
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L288
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L300

# [N-01] According to the syntax rules, use  => mapping ( instead of  => mapping( using spaces as keyword (17 Instances)
Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L47
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L50
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L130
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L133
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L136
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L139
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L86
8.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L89
9.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L100
10.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L114
11.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L119
12.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L169
13.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L189
14.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L192
15.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L195
16.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L198
17.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L203


# [N-02] Inconsistent solidity version
Inconsistent solidity versions 0.8.0 and 0.8.19 are used throughout the codebase in scope that can lead to (u)known error and bugs both at compiling and after deployment.

# [N-03] Empty blocks should be removed (06 Instances)

Avoid using code blocks or use them for some process like emitting events.

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L143
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L364
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L372
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L380
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L383
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L386


# [N-04] Function writing that does not comply with the Solidity Style Guide

All Contracts
Order of functions should help readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered as mentioned in the article i.e.;
constructor
external
public
internal
private
within a grouping, place the view and pure functions last

