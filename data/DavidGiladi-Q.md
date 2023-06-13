# Please note: I reported issues that were overlooked by the winning bot. I reported only on instances that were missed


## Else block not required
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies unnecessary else blocks in the code. When an if-statement block ends with a return statement, any subsequent else block becomes superfluous and can be eliminated to reduce code complexity. "

Note that this check also applies to single-line else conditions. For instance, in 'return a > b ? a : b', the else condition is not needed.


<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: src/lib/Checkpoints.sol
```
 
Line: 85          return pos == 0 ? 0 : _unsafeAccess(self._checkpoints, pos - 1).quantity
```
Should not be inside else block.
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L85


- File: src/lib/Checkpoints.sol
```
 
Line: 148          return (0, quantity)
```
Should not be inside else block.
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L148


</details>

# 


## Function Length Too Long
- Severity: Non-Critical
- Confidence: High

### Description
Identifies functions that exceed 50 lines of code. Long functions can be harder to understand and maintain. It is often beneficial to split these into smaller, more manageable functions.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: src/LlamaPolicy.sol
```
 
Line: 431          function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal 
```
_setRoleHolder has more than 50 lines. Start line 431, end line 487.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L431-L487


- File: src/LlamaPolicyMetadata.sol
```
 
Line: 17          function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)

    external

    pure

    returns (string memory)

  
```
tokenURI has more than 50 lines. Start line 17, end line 100.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L17-L100


- File: src/lib/Checkpoints.sol
```
 
Line: 223          function sqrt(uint256 x) internal pure returns (uint256 z) 
```
sqrt has more than 50 lines. Start line 223, end line 285.
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L223-L285


</details>

# 

 


## Missing Underscore Prefix on Large Number Literals
- Severity: Non-Critical
- Confidence: High

### Description
In Solidity, it is common to work with large number literals to represent values such as timestamps, amounts, or other numerical data. However, when dealing with large numbers, readability can be improved by explicitly indicating the magnitude of the value.

By prefixing large number literals with an underscore, it provides a visual indication to developers and auditors that the value is intentionally large and not a typographical error. It helps prevent misinterpretation or confusion regarding the intended value and promotes code comprehension.

For example, instead of writing uint256 balance = 1000000000000000000;, the convention suggests writing uint256 balance = 1_000_000_000_000_000_000;. The underscores act as separators and improve readability without affecting the actual value.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/lib/Checkpoints.sol
```
 
Line: 267          z := shr(18, mul(z, add(y, 65536)))
```
 use `_` for 65536 as 65_536

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L267


</details>

# 
## Setter No Initial Value Check Detection
- Severity: Non-Critical
- Confidence: Medium

### Description
This detector flags setter functions that lack an initial value check. Lack of such a check can lead to assigning wrong values to the variable, causing unexpected contract behavior.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: src/LlamaFactory.sol
```
 
Line: 279          function _setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) internal 
```
 Setter lacks an initial value check
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L279-L282


</details>

# 

## Events are missing sender information
- Severity: Non-Critical
- Confidence: High

### Description
When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender in the events of these types of action will make events much more useful to end users, especially when msg.sender is not tx.origin.

###
- File: src/LlamaPolicy.sol
```
 
Line: 395          RoleInitialized(numRoles, description)
```
  The event `RoleInitialized` emits an event without including the sender's address:

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L395


- File: src/LlamaPolicy.sol
```
 
Line: 493          RolePermissionAssigned(role, permissionId, hasPermission)
```
  The event `RolePermissionAssigned` emits an event without including the sender's address:

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L493
#



## Unused Event Field Indexing in Solidity Contracts
- Severity: Non-Critical
- Confidence: High

### Description
Indexing event fields enhances the accessibility and efficiency of event data for off-chain tools that parse events. Indexing is particularly beneficial for filtering events based on specific criteria, such as address filtering. However, it's important to consider the additional gas cost associated with indexing. Indexing all fields may not be the best approach if gas usage is a concern. To properly index event fields, the following guidelines are recommended:

 - If an event has three or more applicable fields and gas usage is not a significant concern for the events in question, it is recommended to index all three applicable fields. This ensures efficient filtering based on multiple criteria.

 - If an event has fewer than three applicable fields, all of the applicable fields should be indexed to provide accessibility for filtering, even if there are fewer fields to consider.

By following these guidelines, you can optimize the accessibility and usability of your contract's events while considering the associated gas costs.

<details>

<summary>
There are 18 instances of this issue:

</summary>

###
- File: src/LlamaCore.sol
```
 
Line: 103          event ActionCanceled(uint256 id);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L103


- File: src/LlamaCore.sol
```
 
Line: 106          event ActionGuardSet(address indexed target, bytes4 indexed selector, ILlamaActionGuard actionGuard);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L106


- File: src/LlamaCore.sol
```
 
Line: 123          event ApprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L123


- File: src/LlamaCore.sol
```
 
Line: 126          event DisapprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L126


- File: src/LlamaCore.sol
```
 
Line: 129          event StrategyCreated(ILlamaStrategy strategy, ILlamaStrategy indexed strategyLogic, bytes initializationData);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L129


- File: src/LlamaCore.sol
```
 
Line: 132          event AccountCreated(ILlamaAccount account, ILlamaAccount indexed accountLogic, bytes initializationData);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L132


- File: src/LlamaPolicy.sol
```
 
Line: 85          event RoleAssigned(address indexed policyholder, uint8 indexed role, uint64 expiration, uint128 quantity);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L85


- File: src/LlamaPolicy.sol
```
 
Line: 88          event RoleInitialized(uint8 indexed role, RoleDescription description);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L88


- File: src/LlamaPolicy.sol
```
 
Line: 91          event RolePermissionAssigned(uint8 indexed role, bytes32 indexed permissionId, bool hasPermission);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L91


- File: src/LlamaPolicyMetadataParamRegistry.sol
```
 
Line: 31          event ColorSet(LlamaExecutor indexed llamaExecutor, string color);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L31


- File: src/LlamaPolicyMetadataParamRegistry.sol
```
 
Line: 34          event LogoSet(LlamaExecutor indexed llamaExecutor, string logo);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L34


- File: src/lib/ERC721NonTransferableMinimalProxy.sol
```
 
Line: 20          event ApprovalForAll(address indexed owner, address indexed operator, bool approved);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L20


- File: src/strategies/LlamaAbsoluteStrategyBase.sol
```
 
Line: 83          event ForceApprovalRoleAdded(uint8 role);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L83


- File: src/strategies/LlamaAbsoluteStrategyBase.sol
```
 
Line: 87          event ForceDisapprovalRoleAdded(uint8 role);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L87


- File: src/strategies/LlamaAbsoluteStrategyBase.sol
```
 
Line: 90          event StrategyCreated(LlamaCore llamaCore, LlamaPolicy policy);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L90


- File: src/strategies/LlamaRelativeQuorum.sol
```
 
Line: 75          event ForceApprovalRoleAdded(uint8 role);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L75


- File: src/strategies/LlamaRelativeQuorum.sol
```
 
Line: 79          event ForceDisapprovalRoleAdded(uint8 role);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L79


- File: src/strategies/LlamaRelativeQuorum.sol
```
 
Line: 82          event StrategyCreated(LlamaCore llamaCore, LlamaPolicy policy);
```
event is not indexed properly.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L82


</details>

# 


## Contracts that lock Ether
- Severity: Non-Critical
- Confidence: High

### Description
Contract with a `payable` function, but without a withdrawal capacity.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Contract locking ether found:
	Contract File: src/LlamaCore.sol
```
 
Line: 20          contract LlamaCore is Initializable 
```
 has payable functions:
	 - File: src/LlamaCore.sol
```
 
Line: 317          function executeAction(ActionInfo calldata actionInfo) external payable 
```

	But does not have a function to withdraw the ether

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L20-L803


</details>

# 