 
# Gas Optimizations Report

This report focuses on Llama Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Llama protocol are categorised into 10 main areas; with further multiple instances in each of the category.


# Summary
[G-01] Immutable has more gas efficiency than constant (04 Instances)
[G-02] Bytes constant are more efficient than strings constant (04 Instances)
[G-03] Multiple mappings can be combine into a single one (14 Instances)
[G-04] Use hardcode address instead address(this) (07 Instances)
[G-05] Use != 0 instead of > 0 for unsigned integer comparison (11 Instances)
[G-06] Use uint256(1)/uint256(2) instead for true and false boolean states (09 Instances)
[G-07] Use assembly to check for address(0) (13 Instances)
[G 08] String literals passed to abi.encode()should not be split by commas (09 Instances)
[G-09] Public functions to external instead (07 Instances)
[G-10] Use calldata instead of memory for function parameters (17 Instances)



# [G-01] Immutable has more gas efficiency than constant (04 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L99
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L68
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L105
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L111


# [G-02] Bytes constant are more efficient than string constant (04 Instances)

Bytes constant are more gas efficient in storing text data than string constant.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L61
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L62
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L118
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L183



# [G-03] Multiple mappings can be combine into a single one (17 Instances)
When multiple mappings are used in same function, it’s better to combined them into a single mapping using a struct.
Combined mapping reduces storage slot per mapping and also are cheaper in terms of associated atack operations calculation carried out.

Link to the Code:
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



# [G-04] Use hardcode address instead address(this) (07 Instances)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

Link to the code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L223
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L200
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L249
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L258

5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708


# [G-05] Use != 0 instead of > 0 for unsigned integer comparison (11 Instances)
	
Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L242
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L307
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L313
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L320
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L326
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L425
8.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L444
9.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L445
10.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629
11.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L649


# [G-06] Use uint256(1)/uint256(2) instead for true and false boolean states (09 Instances)

Boolean for storage if not used, saves Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L181
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L189
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L268
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L274
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L326
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L356
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L571
8.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L582
9.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L640


# [G-07] Use assembly to check for address(0) (13 Instances)
	
Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L148
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L166
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L199
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L247
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L256
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L181
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L405
8.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298
9.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L330
10.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L339
11.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L389
12.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L422
13.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L550


# [G 08] String literals passed to abi.encode()/abi.encodePacked() should not be split by commas (09 Instances)

String literals can be split into multiple parts within quotes and still be considered as a single string literal.
EACH new comma costs 21 gas due to stack operations and separate MSTOREs.
	
Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L242
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L687
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L741
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L753
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L763
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L775
8.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L785
9.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L791


# [G-09] Public functions to external instead (07 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:
1.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L104
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L287
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L262
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L337
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L359
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L367
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L375


# [G-10] Use calldata instead of memory for function parameters (17 Instances)

Using calldata in external function does not require data to be stored, which reduced the process time as compared to memory. This in return saves gas during calling the data.

Link to the Code:
1.	 https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L82
2.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L90
3.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L17
4.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L104
5.	https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156
6.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L155
7.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L206
8.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L218
9.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L228
10.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L285
11.	https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130
12.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L265
13.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L291
14.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L523
15.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L565
16.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L576
17.	https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L721
