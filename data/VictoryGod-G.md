## Summary
### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |Calldata for data parameter| LlamaExecutor.sol |
| [G-02] |External visibility for getMetadata| LlamaPolicyMetadataParamRegistry.sol |
| [G-03] |Remove unnecessary storage writes| LlamaPolicyMetadataParamRegistry.sol |
| [G-04] |Use immutable for constant variables| strategies/LlamaAbsoluteQuorum.sol |
| [G-05] |Minimize external calls| strategies/LlamaAbsoluteQuorum.sol |
| [G-06] |Use immutable for constant variables| strategies/LlamaAbsolutePeerReview.sol |
| [G-07] |Minimize external calls| strategies/LlamaAbsolutePeerReview.sol |
| [G-08] |Use abi.encodePacked for concatenation| strategies/LlamaAbsolutePeerReview.sol |
| [G-09] |Use bytes for concatenation | strategies/LlamaAbsolutePeerReview.sol |
| [G-10] |Use constants for static parts| LlamaPolicyMetadata.sol |
| [G-11] |Cache block.timestamp| LlamaPolicyMetadata.sol |
| [G-12] |Use mapping for role management| LlamaPolicyMetadata.sol |
| [G-13] |Immutable for authorized mappings| LlamaPolicyMetadata.sol |
| [G-14] |Optimize event emission| LlamaPolicyMetadata.sol |
| [G-15] |Optimize loops in batch functions| LlamaFactory.sol |
| [G-16] |Reduce function visibility| LlamaFactory.sol |
| [G-17] |Remove redundant zero address checks| llama-scripts/LlamaGovernanceScript.sol |
| [G-18] |Single transfer function for ERC20 tokens| accounts/LlamaAccount.sol |
| [G-19] |Single approve function for ERC20 tokens| accounts/LlamaAccount.sol |
| [G-20] |Public vs external visibility| accounts/LlamaAccount.sol |
| [G-21] |Gas optimization in loops| accounts/LlamaAccount.sol |
| [G-22] |Use immutable for constant values| LlamaCore.sol |
| [G-23] |Use keccak256 for signature verification| LlamaCore.sol |
| [G-24] |Avoid string types for events| LlamaCore.sol |
| [G-25] |Use calldata for function parameters| LlamaCore.sol |
| [G-26] |Avoid string data type for descriptions| LlamaCore.sol |
| [G-27] |Immutable for type hashes| LlamaCore.sol |
| [G-28] |Calldata for function arguments| LlamaCore.sol |
| [G-29] |Unchecked blocks for arithmetic operations| LlamaCore.sol |
| [G-30] |Replace uncheckedIncrement with direct increment| LlamaCore.sol |
| [G-31] |Use bytes32 for salt| LlamaCore.sol |

Total 31 issues


## File -> LlamaExecutor.sol
### [G-01] Use calldata for the data parameter: 
Since the data parameter is only read and not modified in the execute function, you can use the calldata data location instead of memory. This can help reduce gas usage for the function call:

```
   function execute(address target, uint256 value, bool isScript, bytes calldata data)
     external
     returns (bool success, bytes memory result)
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaExecutor.sol#L29

## File -> LlamaPolicyMetadataParamRegistry.sol
### [G-02] Use the external visibility instead of public for the getMetadata function. 
This change will make the function slightly more gas-efficient when called from outside the contract since it avoids copying the arguments to memory. The function signature should be updated as follows:
```
function getMetadata(LlamaExecutor llamaExecutor) external view returns (string memory _color, string memory _logo)
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L74

### [G-03] Remove the unnecessary storage write in the constructor when setting the color and logo for the root Llama Executor.
Instead, you can directly initialize the mapping in the constructor, like this:
```
constructor(LlamaExecutor rootLlamaExecutor) {
  ROOT_LLAMA_EXECUTOR = rootLlamaExecutor;
  LLAMA_FACTORY = msg.sender;

  string memory rootColor = "#6A45EC";
  string memory rootLogo = '<g><path fill="#fff" d="M91.749 ... Z"/></g></g>';

  color[ROOT_LLAMA_EXECUTOR] = rootColor;
  logo[ROOT_LLAMA_EXECUTOR] = rootLogo;

  emit ColorSet(ROOT_LLAMA_EXECUTOR, rootColor);
  emit LogoSet(ROOT_LLAMA_EXECUTOR, rootLogo);
}
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L57

## File -> strategies/LlamaAbsoluteQuorum.sol
### [G-04] Use immutable for variables that don't change after deployment.
In the current code, LlamaPolicy llamaPolicy = policy; is used to reduce SLOADs. However, you can make the policy variable immutable to save even more gas. This will ensure that the value is stored in the contract bytecode itself and not in storage. You can change the declaration of the policy variable in LlamaAbsoluteStrategyBase to:
```
LlamaPolicy immutable public policy;
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaAbsoluteQuorum.sol#L28

### [G-05] Minimize the number of external calls.
In the validateActionCreation function, there are two calls to llamaPolicy.getRoleSupplyAsQuantitySum(). Consider passing the approval and disapproval policy supplies as arguments to the function, minimizing the number of external calls. This will reduce gas consumption.
```
function validateActionCreation(
  ActionInfo calldata /* actionInfo */,
  uint256 approvalPolicySupply,
  uint256 disapprovalPolicySupply
) external view override {
  // ...
}
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaAbsoluteQuorum.sol#L27

## File -> strategies/LlamaAbsolutePeerReview.sol
### [G-06] Use immutable for variables that don't change after deployment.
As mentioned previously, you can make the policy variable immutable in the LlamaAbsoluteStrategyBase contract to save gas.
```
LlamaPolicy immutable public policy;
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaAbsolutePeerReview.sol#L41

### [G-07] Minimize the number of external calls.
In the validateActionCreation function, there are multiple calls to llamaPolicy.getQuantity() and llamaPolicy.getRoleSupplyAsQuantitySum(). Consider passing the required values as arguments to the function, minimizing the number of external calls. This will reduce gas consumption.
```
function validateActionCreation(
  ActionInfo calldata actionInfo,
  uint256 approvalPolicySupply,
  uint256 disapprovalPolicySupply,
  uint256 actionCreatorApprovalRoleQty,
  uint256 actionCreatorDisapprovalRoleQty
) external view override {
  // ...
}
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaAbsolutePeerReview.sol#L40

## File -> LlamaPolicyMetadata.sol
### [G-08] Use abi.encodePacked instead of string.concat:
The string.concat function from @solady/utils/LibString.sol is being used multiple times to concatenate strings. It can be replaced with abi.encodePacked, which is a native Solidity function and is more gas-efficient.

```
replace:

string memory output1 = string.concat(parts[0], parts[1], parts[2], parts[3], parts[4], parts[5], parts[6], parts[7], parts[8]);
with:

string memory output1 = string(abi.encodePacked(parts[0], parts[1], parts[2], parts[3], parts[4], parts[5], parts[6], parts[7], parts[8]));

```
### [G-09] Use bytes instead of string for concatenation:
Concatenating string variables can be costly in terms of gas. Instead, you can use bytes for concatenation and then convert the result back to a string.

```
replace:

parts[3] = string.concat(LibString.slice(policyholder, 0, 6), "...", LibString.slice(policyholder, 38, 42));
with:

parts[3] = string(abi.encodePacked(bytes(LibString.slice(policyholder, 0, 6)), "...", bytes(LibString.slice(policyholder, 38, 42))));
```


### [G-10] Use constants for static parts:
Some parts of the SVG are static and don't change. You can declare these parts as constant variables to avoid the need for memory allocation at runtime.
```

string public constant PART_0 = '<svg xmlns="http://www.w3.org/2000/svg" width="390" height="500" fill="none">...';
Then, use these constants in the tokenURI function:

string memory output1 = string(abi.encodePacked(PART_0, parts[1], PART_1, parts[3], PART_2, parts[5], PART_3, parts[7], PART_4));
```
### [G-11] caching block.timestamp:
The block.timestamp is used multiple times in functions isActive, isActionExpired, and approvalEndTime. Accessing block.timestamp multiple times consumes more gas. To optimize gas usage, consider caching the block.timestamp value in a local variable and reusing it. 
```

   uint256 currentTimestamp = block.timestamp;

```
### [G-12] use of mapping for forceApprovalRole and forceDisapprovalRole:
The contract uses mapping(uint8 => bool) for forceApprovalRole and forceDisapprovalRole. This can be gas inefficient when iterating through the mapping. Consider using an array of uint8 instead, which can be more gas-efficient when iterating and checking for the existence of a role.


## File -> LlamaFactory.sol

### [G-13] Use immutable for authorizedStrategyLogics and authorizedAccountLogics mappings: 
Since the mappings are only being updated through the _authorizeStrategyLogic() and _authorizeAccountLogic() functions, you can make the mappings immutable. This would save gas costs during contract deployment and execution.
```
mapping(ILlamaStrategy => bool) public immutable authorizedStrategyLogics;
mapping(ILlamaAccount => bool) public immutable authorizedAccountLogics;
```
### [G-14] Optimize event emission: 
In the _deploy() function, the LlamaInstanceCreated event is emitted with the block.chainid parameter. This parameter is not necessary since events are specific to the blockchain they are emitted on. Removing this parameter would save gas costs.
```
emit LlamaInstanceCreated(
  llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy)
);
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L260
## File ->llama-scripts/LlamaGovernanceScript.sol

### [G-15] The loops in the batch functions such as initializeRoles, setRoleHolders, setRolePermissions, revokePolicies, and updateRoleDescriptions can be optimized by using a local variable for the length of the array. 
This would save gas by reducing the number of times the length is accessed from storage.
```
uint256 length = description.length;
for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  ...
}
```
## File -> accounts/LlamaAccount.sol

### [G-16] Reduce function visibility: 
Some functions are marked as public, but they can be changed to external if they are only called from external contracts. Functions like transferNativeToken, transferERC20, approveERC20, transferERC721, approveERC721, approveOperatorERC721, and approveOperatorERC1155 can be changed to external. This will save some gas as external functions do not need to copy arguments to memory.


### [G-17] Remove redundant zero address checks: 
The transferERC20, transferERC721, transferERC1155, and batchTransferSingleERC1155 functions check if the recipient is a zero address. However, the underlying OpenZeppelin safeTransfer, transferFrom, and safeTransferFrom functions already include these checks. Removing the redundant checks can save some gas.


### [G-18] Use a single transfer function for ERC20 tokens: 
Instead of having separate transferERC20 and batchTransferERC20 functions, you can use a single transferERC20 function that accepts an array of ERC20Data and loop through them to transfer tokens. This will reduce the code size and save some gas.
```
function transferERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
  uint256 length = erc20Data.length;
  for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
    if (erc20Data[i].recipient == address(0)) revert ZeroAddressNotAllowed();
    erc20Data[i].token.safeTransfer(erc20Data[i].recipient, erc20Data[i].amount);
  }
}
```
### [G-19]  Use a single approve function for ERC20 tokens: 
instead of having separate approveERC20 and batchApproveERC20 functions, you can use a single approveERC20 function that accepts an array of ERC20Data and loop through them to approve allowances. This will reduce the code size and save some gas.
```
function approveERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
  uint256 length = erc20Data.length;
  for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
    erc20Data[i].token.safeApprove(erc20Data[i].recipient, erc20Data[i].amount);
  }
}
```

## File -> LlamaCore.sol


### [G-20]  Use of public instead of external visibility for functions:
Functions like initialize, getAction, and other functions that are only meant to be called externally could be marked as external instead of public. This can result in gas savings, as external functions are slightly more gas-efficient when called externally. For example, change the visibility of the initialize function to external:
```
   function initialize(...) external initializer returns (bytes32 bootstrapPermissionId) {...}
```
### [G-21]  Gas optimization in loops:

In the _deployStrategies and _deployAccounts functions (not shown in the provided code), if you are using loops to deploy multiple strategies or accounts, you can optimize gas usage by minimizing the number of storage writes. One way to do this is by using a temporary local variable to store intermediate values and only writing the final value to storage at the end of the loop.

### [G-22]  Use of immutable keyword for constant values:
The EIP-712 typehash constants (e.g., EIP712_DOMAIN_TYPEHASH, CREATE_ACTION_TYPEHASH, etc.) can be declared as immutable instead of constant. This will store their values directly in the contract bytecode, resulting in gas savings during contract deployment and execution.
```
   bytes32 internal immutable EIP712_DOMAIN_TYPEHASH = keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
```
### [G-23]  Use keccak256 instead of ecrecover for signature verification. 
Using keccak256 is more gas-efficient compared to ecrecover. You can follow the EIP-712 standard for creating and verifying typed data signatures.
For example, you can change the _getCreateActionTypedDataHash and _getCastApprovalTypedDataHash functions to use keccak256 for signature verification.

### [G-24]  Avoid using string types for events if possible. 
Using string types in events can increase gas costs. Consider changing the reason and description parameters in the events to bytes32 if the length of the strings is limited or use a hash of the original string value.
```
   event ActionQueued(uint256 indexed id, address indexed queueCaller, ILlamaStrategy indexed strategy, address creator, uint64 minExecutionTime);
```
### [G-25]  Use calldata instead of memory for function parameters. 
Using calldata is more gas-efficient than memory because it does not require copying the data to memory. For example, change the string memory description parameter in the createAction function to string calldata description.
```
   function createAction(
       uint8 role,
       ILlamaStrategy strategy,
       address target,
       uint256 value,
       bytes calldata data,
       string calldata description
   ) external returns (uint256 actionId) {
       // ...
   }
```
### [G-26]  Avoid using the string data type for the description and reason parameters.
Instead, use bytes32 or other fixed-length types to reduce gas costs. If you still need to store the original string value, consider using a separate mapping to store the string data and reference it via a unique identifier.

### [G-27]  Use immutable for EIP712_DOMAIN_TYPEHASH, CREATE_ACTION_TYPEHASH, CAST_APPROVAL_TYPEHASH, CAST_DISAPPROVAL_TYPEHASH, and ACTION_INFO_TYPEHASH

These constants can be declared as immutable to save gas on each function call that uses them. 
```
bytes32 private constant immutable EIP712_DOMAIN_TYPEHASH = keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
```
### [G-28]  Use calldata for function arguments
Using calldata for function arguments allows you to save on gas costs, as calldata is cheaper than memory. For example, change string memory description to string calldata description.

### [G-29]  Use unchecked blocks for arithmetic operations
To avoid unnecessary checks for arithmetic overflow and underflow, you can use unchecked blocks to perform arithmetic operations. 
```
unchecked {
  action.totalApprovals = action.totalApprovals + quantity;
}
```
### [G-30]  Replace LlamaUtils.uncheckedIncrement with direct increment
The provided code uses a custom function, LlamaUtils.uncheckedIncrement, to increment uint256 values. However, it is more gas-efficient to simply increment the values directly using the ++ operator.
```
actionsCount++;
```
### [G-31]  Use bytes32 for salt: In the _deployStrategies and _deployAccounts functions
you use bytes32 for the salt variable. Instead of converting the keccak256 hash to bytes32, you can directly declare the salt variable as bytes32.
```
bytes32 salt = keccak256(abi.encodePacked(strategyConfigs[i]));
```