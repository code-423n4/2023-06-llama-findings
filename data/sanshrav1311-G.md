# Gas Optimization Report
## [01] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol

	File: /src/LlamaCore.sol
	
	224:     string memory _name,

	264:     string memory description

	332:     (bool success, bytes memory result) =

	470:   function getAction(uint256 actionId) external view returns (Action memory) {

	522:     string memory description

	527:     PermissionData memory permission = PermissionData(target, bytes4(data), strategy);

	543:     ActionInfo memory actionInfo = ActionInfo(actionId, policyholder, role, strategy, target, value, data);

	564:   function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)

	574:   function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)

	719:     string memory description
	
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol

	File: /src/LlamaExecutor.sol
	31:     returns (bool success, bytes memory result)

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

	File: /src/LlamaFactory.sol

	108:     string memory name,
	109:     bytes[] memory initialStrategies,
	110:     bytes[] memory initialAccounts,
	111:     RoleDescription[] memory initialRoleDescriptions,
	112:     RoleHolderData[] memory initialRoleHolders,
	113:     RolePermissionData[] memory initialRolePermissions
	114:   ) {

	155: tring memory name,

	158:     bytes[] memory initialStrategies,
	159:     bytes[] memory initialAccounts,
	160:     RoleDescription[] memory initialRoleDescriptions,
	161:     RoleHolderData[] memory initialRoleHolders,
	162:     RolePermissionData[] memory initialRolePermissions,
	163:     string memory color,
	164:     string memory logo

	206: function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)
	207:     external
	208:     view
	209:     returns (string memory)
	210:   {
	211:     (string memory color, string memory logo) = LLAMA_POLICY_METADATA_PARAM_REGISTRY.getMetadata(llamaExecutor);
	212:     return llamaPolicyMetadata.tokenURI(name, tokenId, color, logo);
	213:   }

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

	File: /src/LlamaPolicy.sol
	349: RI(uint256 tokenId) public view override returns (string memory) {
	350:     return factory.tokenURI(LlamaExecutor(llamaExecutor), name, tokenId);
	351:   }

	355: view returns (string memory) {
	356:     return factory.contractURI(name);
	357:   }

## [02] Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =). consider replacing >= with the strict counterpart > and add - 1 to the second side

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L636

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L656

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L237

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L151

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L155

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L155

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L161


