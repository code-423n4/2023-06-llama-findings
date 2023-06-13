# Please note: I reported issues that were overlooked by the winning bot. I reported only on instances that were missed

## Unused Named Return Variables
- Severity: Gas Optimization
- Confidence: High

### Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.
```
Line: 92          function latestCheckpoint(History storage self)

       internal

        view

        returns (

            bool exists,

            uint64 timestamp,

            uint64 expiration,

            uint128 quantity

        )
    
```
there is not use of this variables:
@ **exists**
@ **timestamp**
@ **expiration**
@ **quantity**

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L92-L109
#

## Optimal State Variable Order
- Severity: Gas Optimization
- Confidence: High

### Description
Detect optimal variable order in contract storage layouts to decrease the number of slots used.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Optimization opportunity in storage variable layout of contract File: src/LlamaPolicy.sol
```
 
Line: 20          contract LlamaPolicy is ERC721NonTransferableMinimalProxy 
```

- original variable order (count: 6 slots)
    - mapping(uint256 => mapping(uint8 => Checkpoints.History)) roleBalanceCkpts
    - uint8 ALL_HOLDERS_ROLE
    - uint8 BOOTSTRAP_ROLE
    - mapping(uint8 => mapping(bytes32 => bool)) canCreateAction
    - mapping(uint8 => LlamaPolicy.RoleSupply) roleSupply
    - uint8 numRoles
    - address llamaExecutor
    - LlamaFactory factory


 - optimized variable order (count: 5 slots)
    - address llamaExecutor
    - uint8 ALL_HOLDERS_ROLE
    - uint8 BOOTSTRAP_ROLE
    - uint8 numRoles
    - LlamaFactory factory
    - mapping(uint256 => mapping(uint8 => Checkpoints.History)) roleBalanceCkpts
    - mapping(uint8 => mapping(bytes32 => bool)) canCreateAction
    - mapping(uint8 => LlamaPolicy.RoleSupply) roleSupply



Recomended optimizations will use 1 less slots.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L20-L536


</details>

# 

## Optimizing Small Data Storage with Bytes32
- Severity: Gas Optimization
- Confidence: Medium

### Description
This flags contracts that inefficiently use bytes and strings for small data that could fit into bytes32. Using bytes32 is cheaper in Solidity when the data can fit into 32 bytes.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/LlamaPolicyMetadataParamRegistry.sol
```
 
Line: 61          string memory rootColor = "#6A45EC"
```
 should be convert to `bytes32`
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L61


</details>

# 


## Potential Optimization by Combining Multiple Mappings into a Struct
- Severity: Gas Optimization
- Confidence: High

### Description
This identifies instances where multiple mappings of an address or ID could be combined into a single mapping to a struct. This optimisation not only saves a storage slot for each mapping that is combined, but also makes the contract more efficient in terms of gas usage. Depending on the circumstances and sizes of types, it can avoid a 20000 gas cost (Gsset) per combined mapping. Also, it can make reads and subsequent writes cheaper when a function requires both values and they both fit in the same storage slot. Lastly, if both fields are accessed in the same function, it can save approximately 42 gas per access due to not having to recalculate the key's keccak256 hash and its associated stack operations.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: src/LlamaCore.sol
```
 
Line: 169          mapping(uint256 => Action) internal actions
```
Consider to combine with <br>-    ```Line: 189 mapping(uint256 => mapping(address => bool)) public approvals```<br>-    ```Line: 192 mapping(uint256 => mapping(address => bool)) public disapprovals```<br>-    ```Line: 203 mapping(address => mapping(bytes4 => uint256)) public nonces```<br>-    ```Line: 206 mapping(address target => mapping(bytes4 selector => ILlamaActionGuard)) public actionGuard```<br><br><br>
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L169


- File: src/LlamaPolicy.sol
```
 
Line: 114          mapping(uint8 role => mapping(bytes32 permissionId => bool)) public canCreateAction
```
Consider to combine with <br>-    ```Line: 119 mapping(uint8 role => RoleSupply) public roleSupply```<br><br><br>
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L114


- File: src/LlamaPolicyMetadataParamRegistry.sol
```
 
Line: 47          mapping(LlamaExecutor => string) public color
```
Consider to combine with <br>-    ```Line: 50 mapping(LlamaExecutor => string) public logo```<br><br><br>
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L47


- File: src/lib/ERC721NonTransferableMinimalProxy.sol
```
 
Line: 36          mapping(uint256 => address) internal _ownerOf
```
Consider to combine with <br>-    ```Line: 54 mapping(uint256 => address) public getApproved```<br>-    ```Line: 56 mapping(address => mapping(address => bool)) public isApprovedForAll```<br><br><br>
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L36


- File: src/strategies/LlamaAbsoluteStrategyBase.sol
```
 
Line: 133          mapping(uint8 => bool) public forceApprovalRole
```
Consider to combine with <br>-    ```Line: 136 mapping(uint8 => bool) public forceDisapprovalRole```<br><br><br>
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L133


</details>

# 

### Description
This flags contracts that inefficiently use bytes and strings for small data that could fit into bytes32. Using bytes32 is cheaper in Solidity when the data can fit into 32 bytes.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/LlamaPolicyMetadataParamRegistry.sol
```
 
Line: 61          string memory rootColor = "#6A45EC"
```
 should be convert to `bytes32`
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L61


</details>

# 
