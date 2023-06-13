## [L-01] Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

```solidity
File: src/LlamaFactory.sol

154  function deploy(
155    string memory name,
156    ILlamaStrategy strategyLogic,
157    ILlamaAccount accountLogic,
158    bytes[] memory initialStrategies,
159    bytes[] memory initialAccounts,
160    RoleDescription[] memory initialRoleDescriptions,
161    RoleHolderData[] memory initialRoleHolders,
162    RolePermissionData[] memory initialRolePermissions,
163    string memory color,
164    string memory logo
165  ) external onlyRootLlama returns (LlamaExecutor executor, LlamaCore core) {


227  function _deploy(
228    string memory name,
229    ILlamaStrategy strategyLogic,
230    ILlamaAccount accountLogic,
231    bytes[] memory initialStrategies,
232    bytes[] memory initialAccounts,
233    RoleDescription[] memory initialRoleDescriptions,
234    RoleHolderData[] memory initialRoleHolders,
235    RolePermissionData[] memory initialRolePermissions
236  ) internal returns (LlamaExecutor llamaExecutor, LlamaCore llamaCore) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

62  function aggregate(address[] calldata targets, bytes[] calldata data)



85  function initializeRolesAndSetRoleHolders(
86    RoleDescription[] calldata description,
87    RoleHolderData[] calldata _setRoleHolders
88  ) external onlyDelegateCall {


93  function initializeRolesAndSetRolePermissions(
94    RoleDescription[] calldata description,
95    RolePermissionData[] calldata _setRolePermissions
96  ) external onlyDelegateCall {


101  function initializeRolesAndSetRoleHoldersAndSetRolePermissions(
102    RoleDescription[] calldata description,
103    RoleHolderData[] calldata _setRoleHolders,
104    RolePermissionData[] calldata _setRolePermissions
105  ) external onlyDelegateCall {


111  function createNewStrategiesAndSetRoleHolders(
112    CreateStrategies calldata _createStrategies,
113    RoleHolderData[] calldata _setRoleHolders
114  ) external onlyDelegateCall {


120  function createNewStrategiesAndInitializeRolesAndSetRoleHolders(
121    CreateStrategies calldata _createStrategies,
122    RoleDescription[] calldata description,
123    RoleHolderData[] calldata _setRoleHolders
124  ) external onlyDelegateCall {


131  function createNewStrategiesAndSetRolePermissions(
132    CreateStrategies calldata _createStrategies,
133    RolePermissionData[] calldata _setRolePermissions
134  ) external onlyDelegateCall {


140  function createNewStrategiesAndNewRolesAndSetRoleHoldersAndSetRolePermissions(
141    CreateStrategies calldata _createStrategies,
142    RoleDescription[] calldata description,
143    RoleHolderData[] calldata _setRoleHolders,
144    RolePermissionData[] calldata _setRolePermissions
145  ) external onlyDelegateCall {


153  function revokePoliciesAndUpdateRoleDescriptions(
154    address[] calldata _revokePolicies,
155    UpdateRoleDescription[] calldata _updateRoleDescriptions
156  ) external onlyDelegateCall {


161  function revokePoliciesAndUpdateRoleDescriptionsAndSetRoleHolders(
162    address[] calldata _revokePolicies,
163    UpdateRoleDescription[] calldata _updateRoleDescriptions,
164    RoleHolderData[] calldata _setRoleHolders
165  ) external onlyDelegateCall {


```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L143

```solidity
File: src/LlamaPolicy.sol

143  function initialize(
144    string calldata _name,
145    RoleDescription[] calldata roleDescriptions,
146    RoleHolderData[] calldata roleHolders,
147    RolePermissionData[] calldata rolePermissions
148  ) external initializer {
```

## [L-02] `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead. If all arguments are strings and or bytes, bytes.concat() should be used instead

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

```solidity
File: src/LlamaFactory.sol

/// @audit name
250      LlamaPolicy(Clones.cloneDeterministic(address(LLAMA_POLICY_LOGIC), keccak256(abi.encodePacked(name))));

/// @audit name
253    llamaCore = LlamaCore(Clones.cloneDeterministic(address(LLAMA_CORE_LOGIC), keccak256(abi.encodePacked(name))));

```

## [L‑03] Contracts are not using their `OZ Upgradeable` counterparts

The non-upgradeable standard version of OpenZeppelin's library are inherited / used by the contracts. It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L4

```solidity
File: strategies/LlamaAbsoluteQuorum.sol

4       import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L4

```solidity
File: /src/strategies/LlamaAbsolutePeerReview.sol


4   import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L4

```solidity
File: /src/LlamaPolicyMetadata.sol

4   import {Base64} from "@openzeppelin/utils/Base64.sol";

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L4

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

4    import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L4

````solidity
File: src/LlamaFactory.sol

4   import {Clones} from "@openzeppelin/proxy/Clones.sol";

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
File: src/accounts/LlamaAccount.sol

4   import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

5   import {IERC20} from "@openzeppelin/token/ERC20/IERC20.sol";

6   import {SafeERC20} from "@openzeppelin/token/ERC20/utils/SafeERC20.sol";

7   import {IERC721} from "@openzeppelin/token/ERC721/IERC721.sol";

8   import {ERC721Holder} from "@openzeppelin/token/ERC721/utils/ERC721Holder.sol";

9   import {IERC1155} from "@openzeppelin/token/ERC1155/IERC1155.sol";

10  import {ERC1155Holder} from "@openzeppelin/token/ERC1155/utils/ERC1155Holder.sol";

11  import {Address} from "@openzeppelin/utils/Address.sol";
````

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L5

```solidity
File: src/LlamaCore.sol

4   import {Clones} from "@openzeppelin/proxy/Clones.sol";

5   import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L5

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

5   import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L4

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

4   import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```
