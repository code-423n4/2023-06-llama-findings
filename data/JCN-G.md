# Summary
A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.17`, `optimizer on`, and `1300 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes.

**Note**
- Only optimizations for state-mutating functions (i.e. non `view`/`pure`) and `view`/`pure` functions called within state-mutating functions have been highlighted below.
- Some code snippets may be truncated to save space. Code snippets may also be accompanied by @audit tags in comments to aid in explaining the issue.

## Gas Optimizations
| Number |Issue|Instances| Gas Saved |
|-|:-|:-:|:-:|
| [G-01](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 5 | 500 | 
| [G-02](#cache-state-variables-outside-of-loop-to-avoid-readingwriting-storage-on-every-iteration) | Cache state variables outside of loop to avoid reading/writing storage on every iteration | 3 | 5869 | 
| [G-03](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate) | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate | 1 | 21838 |
| [G-04](#cache-calldatamemory-pointers-for-complex-types-to-avoid-offset-calculations) | Cache calldata/memory pointers for complex types to avoid offset calculations | 2 | 1192 |
| [G-05](#forgo-internal-function-to-save-1-staticcall) | Forgo internal function to save 1 `STATICCALL`  | 4 | 400 |
| [G-06](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache) | Multiple accesses of a mapping/array should use a local variable cache | 1 | 116 |
| [G-07](#refactor-ifrequire-statements-to-save-sloads-in-case-of-early-revert) | Refactor `If`/`require` statements to save SLOADs in case of early revert | 4 | - |

*Total Estimated Gas Saved: 29915*

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

Total Instances: `5`

Estimated Gas Saved: `5 * 100 = 500`

**Note: These are instances missed by the Automated Report**.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L542-L559

### Use already cached `actionId` to save 1 SLOAD
```solidity
File: src/LlamaCore.sol
542:    actionId = actionsCount; // @audit: 1st sload
...
559:    actionsCount = LlamaUtils.uncheckedIncrement(actionsCount); // Safety: Can never overflow a uint256 by incrementing. // @audit: 2nd sload
```
```diff
diff --git a/src/LlamaCore.sol b/src/LlamaCore.sol
index 89d60de..049f66d 100644
--- a/src/LlamaCore.sol
+++ b/src/LlamaCore.sol
@@ -556,7 +556,7 @@ contract LlamaCore is Initializable {
       newAction.isScript = authorizedScripts[target];
     }

-    actionsCount = LlamaUtils.uncheckedIncrement(actionsCount); // Safety: Can never overflow a uint256 by incrementing.
+    actionsCount = LlamaUtils.uncheckedIncrement(actionId); // Safety: Can never overflow a uint256 by incrementing.

     emit ActionCreated(actionId, policyholder, role, strategy, target, value, data, description);
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L56-L58

### Cache `minDisapprovals` to save 1 SLOAD
```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol
56:      if (
57:        minDisapprovals != type(uint128).max // @audit: 1st sload
58:          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty // @audit: 2nd sload
```
```diff
diff --git a/src/strategies/LlamaAbsolutePeerReview.sol b/src/strategies/LlamaAbsolutePeerReview.sol
index 85feb92..c8426aa 100644
--- a/src/strategies/LlamaAbsolutePeerReview.sol
+++ b/src/strategies/LlamaAbsolutePeerReview.sol
@@ -53,9 +53,10 @@ contract LlamaAbsolutePeerReview is LlamaAbsoluteStrategyBase {
       if (minApprovals > approvalPolicySupply - actionCreatorApprovalRoleQty) revert InsufficientApprovalQuantity();

       uint256 actionCreatorDisapprovalRoleQty = llamaPolicy.getQuantity(actionInfo.creator, disapprovalRole);
+      uint128 _minDisapprovals = minDisapprovals;
       if (
-        minDisapprovals != type(uint128).max
-          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
+        _minDisapprovals != type(uint128).max
+          && _minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
       ) revert InsufficientDisapprovalQuantity();
     }
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L220-L223

### Cache `forceApprovalRole[role]` to save 1 SLOAD
```solidity
File: src/strategies/LlamaRelativeQuorum.sol
220:  function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {
221:    if (role != approvalRole && !forceApprovalRole[role]) return 0; // @audit: 1st sload
222:    uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
223:    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity; // @audit: 2nd sload
```
```diff
diff --git a/src/strategies/LlamaRelativeQuorum.sol b/src/strategies/LlamaRelativeQuorum.sol
index d796ae9..8d74c92 100644
--- a/src/strategies/LlamaRelativeQuorum.sol
+++ b/src/strategies/LlamaRelativeQuorum.sol
@@ -218,9 +218,10 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {

   /// @inheritdoc ILlamaStrategy
   function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {
-    if (role != approvalRole && !forceApprovalRole[role]) return 0;
+    bool _forceApprovalRole = forceApprovalRole[role];
+    if (role != approvalRole && !_forceApprovalRole) return 0;
     uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
-    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
+    return quantity > 0 && _forceApprovalRole ? type(uint128).max : quantity;
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L240-L242

### Cache `forceDisapprovalRole[role]` to save 1 SLOAD
```solidity
File: src/strategies/LlamaRelativeQuorum.sol
240:    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0; // @audit: 1st sload
241:    uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
242:    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity; // @audit: 2nd sload
```
```diff
diff --git a/src/strategies/LlamaRelativeQuorum.sol b/src/strategies/LlamaRelativeQuorum.sol
index d796ae9..e1c3927 100644
--- a/src/strategies/LlamaRelativeQuorum.sol
+++ b/src/strategies/LlamaRelativeQuorum.sol
@@ -237,9 +237,10 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {
     view
     returns (uint128)
   {
-    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
+    bool _forceDisapprovalRole = forceDisapprovalRole[role];
+    if (role != disapprovalRole && !_forceDisapprovalRole) return 0;
     uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
-    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
+    return quantity > 0 && _forceDisapprovalRole ? type(uint128).max : quantity;
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L260-L263

### Cache `llamaCount` to save 1 SLOAD
```solidity
File: src/LlamaFactory.sol
260:    emit LlamaInstanceCreated(
261:      llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy), block.chainid // @audit: 1st sload
262:    );
263:    llamaCount = LlamaUtils.uncheckedIncrement(llamaCount); // @audit: 2nd sload
```
```diff
diff --git a/src/LlamaFactory.sol b/src/LlamaFactory.sol
index 0cc4cfd..269e4cb 100644
--- a/src/LlamaFactory.sol
+++ b/src/LlamaFactory.sol
@@ -256,11 +256,12 @@ contract LlamaFactory {
     llamaExecutor = llamaCore.executor();

     policy.finalizeInitialization(address(llamaExecutor), bootstrapPermissionId);
-
+
+    uint256 _llamaCount = llamaCount;
     emit LlamaInstanceCreated(
-      llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy), block.chainid
+      _llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy), block.chainid
     );
-    llamaCount = LlamaUtils.uncheckedIncrement(llamaCount);
+    llamaCount = LlamaUtils.uncheckedIncrement(_llamaCount);
   }
```

## Cache state variables outside of loop to avoid reading/writing storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration. In addition, for some instances we are also able to increment the cached variable in the loop and update the storage variable outside the loop to save 1 `SSTORE` per loop iteration.

Total Instances: `3`

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L227

*Gas Savings for `LlamaPolicy.revokePolicy`, obtained via protocol's tests: Avg 724 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  69040   |  110067  |  59837 |    11     |
| Before |  68180   |  108992  |  59113 |    11     |

### Cache `numRoles` outside of loop to save 1 SLOAD per iteration
```solidity
File: src/LlamaPolicy.sol
227    for (uint256 i = 1; i <= numRoles; i = LlamaUtils.uncheckedIncrement(i)) { // @audit: numRoles read on every iteration
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..7e47189 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -224,7 +224,8 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
     // We start from i = 1 here because a value of zero is reserved for the "all holders" role, and
     // that will get removed automatically when the token is burned. Similarly, use we `<=` to make sure
     // the last role is also revoked.
-    for (uint256 i = 1; i <= numRoles; i = LlamaUtils.uncheckedIncrement(i)) {
+    uint8 _numRoles = numRoles;
+    for (uint256 i = 1; i <= _numRoles; i = LlamaUtils.uncheckedIncrement(i)) {
       if (hasRole(policyholder, uint8(i))) _setRoleHolder(uint8(i), policyholder, 0, 0);
     }
     _burn(_tokenId(policyholder));
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L151-L168

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L393-L396

*Gas Savings for `LlamaFactory.deploy`, obtained via protocol's tests: Avg 4468 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  5101157 |  5406425 | 5015882 |    412   |
| After  |  5096893 |  5281811 | 5011414 |    412   |

*To benchmark this instance we will bring the logic from `_initializeRole` into the construtor in order to refactor the logic. Note that another way of achieving this is by refactoring the logic of the `_initializeRole` directly and every other function that calls `_initializeRole`.*

### Cache `numRoles` outside loop, increment cached variable in loop, and update storage outside loop to save 2 SLOADs + 1 SSTORE per iteration
```solidity
File: src/LlamaPolicy.sol
151:    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
152:      _initializeRole(roleDescriptions[i]); // @audit: sload & sstore for `numRoles` on every iteration
153:    }
...
168:    if (numRoles == 0 || getRoleSupplyAsNumberOfHolders(ALL_HOLDERS_ROLE) == 0) revert InvalidRoleHolderInput(); // @audit: sload for `numRoles`

393:  function _initializeRole(RoleDescription description) internal {
394:    numRoles += 1; // @audit: sload + sstore for `numRoles`
395:    emit RoleInitialized(numRoles, description); // @audit: 2nd sload 
  }
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..af2129b 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -148,9 +148,12 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
   ) external initializer {
     __initializeERC721MinimalProxy(_name, string.concat("LL-", LibString.replace(LibString.upper(_name), " ", "-")));
     factory = LlamaFactory(msg.sender);
+    uint8 _numRoles = numRoles;
     for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
-      _initializeRole(roleDescriptions[i]);
+        _numRoles += 1;
+        emit RoleInitialized(_numRoles, roleDescriptions[i]);
     }
+    numRoles = _numRoles;

     for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {
       _setRoleHolder(
@@ -165,7 +168,7 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
     // Must have assigned roles during initialization, otherwise the system cannot be used. However,
     // we do not check that roles were assigned "properly" as there is no single correct way, so
     // this is more of a sanity check, not a guarantee that the system will work after initialization.
-    if (numRoles == 0 || getRoleSupplyAsNumberOfHolders(ALL_HOLDERS_ROLE) == 0) revert InvalidRoleHolderInput();
+    if (_numRoles == 0 || getRoleSupplyAsNumberOfHolders(ALL_HOLDERS_ROLE) == 0) revert InvalidRoleHolderInput();
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L161-L163

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L490-L491

*Gas Savings for `LlamaFactory.deploy`, obtained via protocol's tests: Avg 677 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  5101157 |  5406425 | 5015882 |    412   |
| After  |  5101175 |  5120119 | 5015205 |    412   |

*To benchmark this instance we will refactor the logic of the `_setRolePermission` internal function directly and also refactor every other function that calls `_setRolePermission`. Another way of achieving this would be to move the logic of `_setRolePermission` into the construtor and refactoring it there.*

### Cache `numRoles` outside loop to save 1 SLOAD per iteration
```solidity
File: src/LlamaPolicy.sol
161:    for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {
162:      _setRolePermission(rolePermissions[i].role, rolePermissions[i].permissionId, rolePermissions[i].hasPermission); // @audit: sload for `numRoles` on every iteration
163:    }

490:  function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {
491:    if (role > numRoles) revert RoleNotInitialized(role); // @audit: sload for `numRoles`
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..8a3273a 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -157,15 +157,16 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
         roleHolders[i].role, roleHolders[i].policyholder, roleHolders[i].quantity, roleHolders[i].expiration
       );
     }
-
+
+    uint8 _numRoles = numRoles;
     for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {
-      _setRolePermission(rolePermissions[i].role, rolePermissions[i].permissionId, rolePermissions[i].hasPermission);
+      _setRolePermission(_numRoles, rolePermissions[i].role, rolePermissions[i].permissionId, rolePermissions[i].hasPermission);
     }

     // Must have assigned roles during initialization, otherwise the system cannot be used. However,
     // we do not check that roles were assigned "properly" as there is no single correct way, so
     // this is more of a sanity check, not a guarantee that the system will work after initialization.
-    if (numRoles == 0 || getRoleSupplyAsNumberOfHolders(ALL_HOLDERS_ROLE) == 0) revert InvalidRoleHolderInput();
+    if (_numRoles == 0 || getRoleSupplyAsNumberOfHolders(ALL_HOLDERS_ROLE) == 0) revert InvalidRoleHolderInput();
   }

   // ===========================================
@@ -181,7 +182,7 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
     if (llamaExecutor != address(0)) revert AlreadyInitialized();

     llamaExecutor = _llamaExecutor;
-    _setRolePermission(BOOTSTRAP_ROLE, bootstrapPermissionId, true);
+    _setRolePermission(numRoles, BOOTSTRAP_ROLE, bootstrapPermissionId, true);
   }

   // -------- Role and Permission Management --------
@@ -205,7 +206,7 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
   /// @param permissionId Permission ID to assign to the role.
   /// @param hasPermission Whether to assign the permission or remove the permission.
   function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {
-    _setRolePermission(role, permissionId, hasPermission);
+    _setRolePermission(numRoles, role, permissionId, hasPermission);
   }

   /// @notice Revokes a policyholder's expired role.
@@ -487,8 +488,8 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
   }

   /// @dev Sets a role's permission along with whether that permission is valid or not.
-  function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {
-    if (role > numRoles) revert RoleNotInitialized(role);
+  function _setRolePermission(uint8 _numRoles, uint8 role, bytes32 permissionId, bool hasPermission) internal {
+    if (role > _numRoles) revert RoleNotInitialized(role);
     canCreateAction[role][permissionId] = hasPermission;
     emit RolePermissionAssigned(role, permissionId, hasPermission);
   }
```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
We can combine multiple mappings below into structs. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading from the struct values we will avoid a `Gsset (20000 gas)` and `Gcoldsload (2100 gas)` since multiple struct values are now occupying the same slot.

**Note: This instance was missed by the automated report.**

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L130-L133

*Gas Savings for `LlamaCore.executeAction`, obtained via protocol's tests: Avg 21838 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  5186172 | 23819570 | 4807541 |    430   |
| After  |  5164334 | 32081589 | 4803180 |    430   |

```solidity
File: src/strategies/LlamaRelativeQuorum.sol
130:  mapping(uint8 => bool) public forceApprovalRole;
131:
132:  /// @notice Mapping of roles that can force an action to be disapproved.
133:  mapping(uint8 => bool) public forceDisapprovalRole;
```
```diff
diff --git a/src/strategies/LlamaRelativeQuorum.sol b/src/strategies/LlamaRelativeQuorum.sol
index d796ae9..2cbeb0c 100644
--- a/src/strategies/LlamaRelativeQuorum.sol
+++ b/src/strategies/LlamaRelativeQuorum.sol
@@ -125,12 +125,13 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {

   /// @notice The role that can disapprove an action.
   uint8 public disapprovalRole;
+
+  struct ForceRoles {
+    bool forceApprovalRole;
+    bool forceDisapprovalRole;
+  }

-  /// @notice Mapping of roles that can force an action to be approved.
-  mapping(uint8 => bool) public forceApprovalRole;
-
-  /// @notice Mapping of roles that can force an action to be disapproved.
-  mapping(uint8 => bool) public forceDisapprovalRole;
+  mapping(uint8 => ForceRoles) forceRoles;

   /// @notice Mapping of action ID to the supply of the approval role at the time the action was created.
   mapping(uint256 => uint256) public actionApprovalSupply;
@@ -146,6 +147,15 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {
     _disableInitializers();
   }

+  // @audit: Getters used for benchmarking purposes
+  function forceApprovalRole(uint8 role) external view returns (bool) {
+    return forceRoles[role].forceApprovalRole;
+  }
+
+  function forceDisapprovalRole(uint8 role) external view returns (bool) {
+    return forceRoles[role].forceDisapprovalRole;
+  }
+
   // ==========================================
   // ======== Interface Implementation ========
   // ==========================================
@@ -178,7 +188,7 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {
       uint8 role = strategyConfig.forceApprovalRoles[i];
       if (role == 0) revert InvalidRole(0);
       _assertValidRole(role, numRoles);
-      forceApprovalRole[role] = true;
+      forceRoles[role].forceApprovalRole = true;
       emit ForceApprovalRoleAdded(role);
     }

@@ -186,7 +196,7 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {
       uint8 role = strategyConfig.forceDisapprovalRoles[i];
       if (role == 0) revert InvalidRole(0);
       _assertValidRole(role, numRoles);
-      forceDisapprovalRole[role] = true;
+      forceRoles[role].forceDisapprovalRole = true;
       emit ForceDisapprovalRoleAdded(role);
     }

@@ -213,14 +223,14 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {

   /// @inheritdoc ILlamaStrategy
   function isApprovalEnabled(ActionInfo calldata, address, uint8 role) external view {
-    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
+    if (role != approvalRole && !forceRoles[role].forceApprovalRole) revert InvalidRole(approvalRole);
   }

   /// @inheritdoc ILlamaStrategy
   function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {
-    if (role != approvalRole && !forceApprovalRole[role]) return 0;
+    if (role != approvalRole && !forceRoles[role].forceApprovalRole) return 0;
     uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
-    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
+    return quantity > 0 && forceRoles[role].forceApprovalRole ? type(uint128).max : quantity;
   }

   // -------- When Casting Disapproval --------
@@ -228,7 +238,7 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {
   /// @inheritdoc ILlamaStrategy
   function isDisapprovalEnabled(ActionInfo calldata, address, uint8 role) external view {
     if (minDisapprovalPct > ONE_HUNDRED_IN_BPS) revert DisapprovalDisabled();
-    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
+    if (role != disapprovalRole && !forceRoles[role].forceDisapprovalRole) revert InvalidRole(disapprovalRole);
   }

   /// @inheritdoc ILlamaStrategy
@@ -237,9 +247,9 @@ contract LlamaRelativeQuorum is ILlamaStrategy, Initializable {
     view
     returns (uint128)
   {
-    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
+    if (role != disapprovalRole && !forceRoles[role].forceDisapprovalRole) return 0;
     uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
-    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
+    return quantity > 0 && forceRoles[role].forceDisapprovalRole ? type(uint128).max : quantity;
   }

   // -------- When Queueing --------
```

## Cache calldata/memory pointers for complex types to avoid offset calculations
The function parameters in the following instances are complex types (i.e. arrays which contain structs) and thus will result in more complex offset calculations to retrieve specific data from calldata/memory. We can avoid peforming some of these offset calculations by instantiating calldata/memory pointers.

Total Instances: `2`

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L155-L159

*Gas Savings for `LlamaPolicy.deploy`, obtained via protocol's tests: Avg 484 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  5101157 |  5406425 | 5015882 |    412   |
| After  |  5101034 |  5256589 | 5015398 |    412   |

```solidity
File: src/LlamaPolicy.sol
155:    for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {
156:      _setRoleHolder(
157:        roleHolders[i].role, roleHolders[i].policyholder, roleHolders[i].quantity, roleHolders[i].expiration
158:      );
159:    }
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..b46c68e 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -153,8 +153,9 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
     }

     for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {
+      RoleHolderData calldata roleHolder = roleHolders[i];
       _setRoleHolder(
-        roleHolders[i].role, roleHolders[i].policyholder, roleHolders[i].quantity, roleHolders[i].expiration
+        roleHolder.role, roleHolder.policyholder, roleHolder.quantity, roleHolder.expiration
       );
     }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L161-L163

*Gas Savings for `LlamaPolicy.deploy`, obtained via protocol's tests: Avg 708 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  5101157 |  5406425 | 5015882 |    412   |
| After  |  5101157 |  5116924 | 5015174 |    412   |

```solidity
File: src/LlamaPolicy.sol
161:    for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {
162:      _setRolePermission(rolePermissions[i].role, rolePermissions[i].permissionId, rolePermissions[i].hasPermission);
163:    }
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..c6df227 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -159,7 +159,8 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
     }

     for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {
-      _setRolePermission(rolePermissions[i].role, rolePermissions[i].permissionId, rolePermissions[i].hasPermission);
+      RolePermissionData calldata rolePermission = rolePermissions[i];
+      _setRolePermission(rolePermission.role, rolePermission.permissionId, rolePermission.hasPermission);
     }
```

## Forgo internal function to save 1 `STATICCALL` 
The `_context` internal function performs two external calls and returns both of the return values from those calls. Certain functions invoke `_context` but only use the return value from the first external call, thus performing an unnecessary extra external call. We can forgo using the internal function and instead only perform our desired external call to save 1 `STATICCALL (100 gas)`.

Total Instances: `4`

Estimated Gas Saved: `4 * 100 = 400`

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L111-L115

### Only perform `address(this).LLAMA_CORE()` to save 1 `STATICCALL`
```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol
111:  function createNewStrategiesAndSetRoleHolders(
112:    CreateStrategies calldata _createStrategies,
113:    RoleHolderData[] calldata _setRoleHolders
114:  ) external onlyDelegateCall {
115:    (LlamaCore core,) = _context(); // @audit: return value from `core.policy()` is not being used
```
```diff
diff --git a/src/llama-scripts/LlamaGovernanceScript.sol b/src/llama-scripts/LlamaGovernanceScript.sol
index 820872e..f886bf7 100644
--- a/src/llama-scripts/LlamaGovernanceScript.sol
+++ b/src/llama-scripts/LlamaGovernanceScript.sol
@@ -112,7 +112,7 @@ contract LlamaGovernanceScript is LlamaBaseScript {
     CreateStrategies calldata _createStrategies,
     RoleHolderData[] calldata _setRoleHolders
   ) external onlyDelegateCall {
-    (LlamaCore core,) = _context();
+    LlamaCore core = LlamaCore(LlamaExecutor(address(this)).LLAMA_CORE());
     core.createStrategies(_createStrategies.llamaStrategyLogic, _createStrategies.strategies);
     setRoleHolders(_setRoleHolders);
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L120-L125

### Only perform `address(this).LLAMA_CORE()` to save 1 `STATICCALL` 
```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol
120:  function createNewStrategiesAndInitializeRolesAndSetRoleHolders(
121:    CreateStrategies calldata _createStrategies,
122:    RoleDescription[] calldata description,
123:    RoleHolderData[] calldata _setRoleHolders
124:  ) external onlyDelegateCall {
125:    (LlamaCore core,) = _context(); // @audit: return value from `core.policy()` is not being used
```
```diff
diff --git a/src/llama-scripts/LlamaGovernanceScript.sol b/src/llama-scripts/LlamaGovernanceScript.sol
index 820872e..f886bf7 100644
--- a/src/llama-scripts/LlamaGovernanceScript.sol
+++ b/src/llama-scripts/LlamaGovernanceScript.sol
@@ -122,7 +122,7 @@ contract LlamaGovernanceScript is LlamaBaseScript {
     RoleDescription[] calldata description,
     RoleHolderData[] calldata _setRoleHolders
   ) external onlyDelegateCall {
-    (LlamaCore core,) = _context();
+    LlamaCore core = LlamaCore(LlamaExecutor(address(this)).LLAMA_CORE());
     core.createStrategies(_createStrategies.llamaStrategyLogic, _createStrategies.strategies);
     initializeRoles(description);
     setRoleHolders(_setRoleHolders);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L131-L135

### Only perform `address(this).LLAMA_CORE()` to save 1 `STATICCALL`
```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol
131:  function createNewStrategiesAndSetRolePermissions(
132:    CreateStrategies calldata _createStrategies,
133:    RolePermissionData[] calldata _setRolePermissions
134:  ) external onlyDelegateCall {
135:    (LlamaCore core,) = _context(); // @audit: return value from `core.policy()` is not being used
```
```diff
diff --git a/src/llama-scripts/LlamaGovernanceScript.sol b/src/llama-scripts/LlamaGovernanceScript.sol
index 820872e..f886bf7 100644
--- a/src/llama-scripts/LlamaGovernanceScript.sol
+++ b/src/llama-scripts/LlamaGovernanceScript.sol
@@ -132,7 +132,7 @@ contract LlamaGovernanceScript is LlamaBaseScript {
     CreateStrategies calldata _createStrategies,
     RolePermissionData[] calldata _setRolePermissions
   ) external onlyDelegateCall {
-    (LlamaCore core,) = _context();
+    LlamaCore core = LlamaCore(LlamaExecutor(address(this)).LLAMA_CORE());
     core.createStrategies(_createStrategies.llamaStrategyLogic, _createStrategies.strategies);
     setRolePermissions(_setRolePermissions);
   }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L140-L146

### Only perform `address(this).LLAMA_CORE()` to save 1 `STATICCALL` 
```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol
140:  function createNewStrategiesAndNewRolesAndSetRoleHoldersAndSetRolePermissions(
141:    CreateStrategies calldata _createStrategies,
142:    RoleDescription[] calldata description,
143:    RoleHolderData[] calldata _setRoleHolders,
144:    RolePermissionData[] calldata _setRolePermissions
145:  ) external onlyDelegateCall {
146:    (LlamaCore core,) = _context(); // @audit: return value from `core.policy()` is not being used
```
```diff
diff --git a/src/llama-scripts/LlamaGovernanceScript.sol b/src/llama-scripts/LlamaGovernanceScript.sol
index 820872e..f886bf7 100644
--- a/src/llama-scripts/LlamaGovernanceScript.sol
+++ b/src/llama-scripts/LlamaGovernanceScript.sol
@@ -143,7 +143,7 @@ contract LlamaGovernanceScript is LlamaBaseScript {
     RoleHolderData[] calldata _setRoleHolders,
     RolePermissionData[] calldata _setRolePermissions
   ) external onlyDelegateCall {
-    (LlamaCore core,) = _context();
+    LlamaCore core = LlamaCore(LlamaExecutor(address(this)).LLAMA_CORE());
     core.createStrategies(_createStrategies.llamaStrategyLogic, _createStrategies.strategies);
     initializeRoles(description);
     setRoleHolders(_setRoleHolders);
```

## Multiple accesses of a mapping/array should use a local variable cache
Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `stakes[tokenId_]`, save its reference via a storage pointer: `StakeInfo storage stakeInfo = stakes[tokenId_]` and use the pointer instead.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L443-L448

*Gas Savings for `LlamaPolicy.revokePolicy`, obtained via protocol's tests: Avg 116 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |   69040  |  110067  |  59837  |    11    |
| After  |   68916  |  109912  |  59721  |    11    |

```solidity
File: src/LlamaPolicy.sol
443:    uint128 initialQuantity = roleBalanceCkpts[tokenId][role].latest();
444:    bool hadRole = initialQuantity > 0;
445:    bool willHaveRole = quantity > 0;
446:
447:    // Now we update the policyholder's role balance checkpoint.
448:    roleBalanceCkpts[tokenId][role].push(willHaveRole ? quantity : 0, expiration);
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..7674061 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -440,12 +440,13 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {
     // checking if the quantity is nonzero, and we don't need to check the expiration when setting
     // the `hadRole` and `willHaveRole` variables.
     uint256 tokenId = _tokenId(policyholder);
-    uint128 initialQuantity = roleBalanceCkpts[tokenId][role].latest();
+    Checkpoints.History storage _roleBalanceCkpts = roleBalanceCkpts[tokenId][role];
+    uint128 initialQuantity = _roleBalanceCkpts.latest();
     bool hadRole = initialQuantity > 0;
     bool willHaveRole = quantity > 0;

     // Now we update the policyholder's role balance checkpoint.
-    roleBalanceCkpts[tokenId][role].push(willHaveRole ? quantity : 0, expiration);
+    _roleBalanceCkpts.push(willHaveRole ? quantity : 0, expiration);

     // If they don't hold a policy, we mint one for them. This means that even if you use 0 quantity
     // and 0 expiration, a policy is still minted even though they hold no roles. This is because
```

## Refactor `If`/`require` statements to save SLOADs in case of early revert
Checks that involve calldata should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before using excessive gas in a call that may ultimately revert in an unhappy case.

Total Instances: `4`

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L27-L35

The check in [line 35](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L35) performs an SLOAD, while the check in [lines 32-33](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L32-L33) perform an external call and two SLOADs. We can move the check in [line 35](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L35) above [lines 32-33](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L32-L33) to potentially save an SLOAD & External call in the unhappy path.

*Note: This view function is called in the state mutating `_createAction` function in `LlamaCore.sol`*
```solidity
File: src/strategies/LlamaAbsoluteQuorum.sol
27:  function validateActionCreation(ActionInfo calldata /* actionInfo */ ) external view override {
28:    LlamaPolicy llamaPolicy = policy; // Reduce SLOADs.
29:    uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(approvalRole);
30:    if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);
31:
32:    uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(disapprovalRole); // @audit: 1 SLOAD + External call
33:    if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole); // @audit: 1 SLOAD
34:
35:    if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity(); // @audit: 1 SLOAD
```
```diff
diff --git a/src/strategies/LlamaAbsoluteQuorum.sol b/src/strategies/LlamaAbsoluteQuorum.sol
index 66130c0..aee2ce3 100644
--- a/src/strategies/LlamaAbsoluteQuorum.sol
+++ b/src/strategies/LlamaAbsoluteQuorum.sol
@@ -29,10 +29,11 @@ contract LlamaAbsoluteQuorum is LlamaAbsoluteStrategyBase {
     uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(approvalRole);
     if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);

+    if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();
+
     uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(disapprovalRole);
     if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);

-    if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();
     if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {
       revert InsufficientDisapprovalQuantity();
     }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L74-L82

The check in [line 79](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L79) accesses storage, while the check in [line 80](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L80) only accesses calldata. Move the check in [line 80](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L80) above [line 79](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L79) to potentially save an SLOAD in the unhappy path.

*Note: This view function is called in the state mutating `_preCastAssertions` function in `LlamaCore.sol`*
```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol
74:  function isDisapprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role)
75:    external
76:    view
77:    override
78:  {
79:    if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled(); // @audit: accesses storage
80:    if (actionInfo.creator == policyholder) revert ActionCreatorCannotCast(); // @audit: accesses calldata
81:    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
82:  }
```
```diff
diff --git a/src/strategies/LlamaAbsolutePeerReview.sol b/src/strategies/LlamaAbsolutePeerReview.sol
index 85feb92..2df24ec 100644
--- a/src/strategies/LlamaAbsolutePeerReview.sol
+++ b/src/strategies/LlamaAbsolutePeerReview.sol
@@ -76,8 +76,9 @@ contract LlamaAbsolutePeerReview is LlamaAbsoluteStrategyBase {
     view
     override
   {
-    if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();
     if (actionInfo.creator == policyholder) revert ActionCreatorCannotCast();
+    if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();
     if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
   }
 }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L412-L418

The check in [line 414](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L414) accesses storage, while the check in [line 418](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L418) only accesses a stack variable. Move the check in [line 418](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L418) above [line 414](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L414) to potentially save 1 SLOAD on the unhappy path.

*Note: This view function is called within state mutating functions in `LlamaPolicy.sol`.*
```solidity
File: src/LlamaPolicy.sol
412:  function _assertValidRoleHolderUpdate(uint8 role, uint128 quantity, uint64 expiration) internal view {
413:    // Ensure role is initialized.
414:    if (role > numRoles) revert RoleNotInitialized(role); // @audit: accesses storage
415:
416:    // Cannot set the ALL_HOLDERS_ROLE because this is handled in the _mint / _burn methods and can
417:    // create duplicate entries if set here.
418:    if (role == ALL_HOLDERS_ROLE) revert AllHoldersRole(); // @audit: accesses stack variable
```
```diff
diff --git a/src/LlamaPolicy.sol b/src/LlamaPolicy.sol
index 3fca63e..443e74c 100644
--- a/src/LlamaPolicy.sol
+++ b/src/LlamaPolicy.sol
@@ -410,13 +410,13 @@ contract LlamaPolicy is ERC721NonTransferableMinimalProxy {

   /// @dev Checks if the conditions are met for a `role` to be updated.
   function _assertValidRoleHolderUpdate(uint8 role, uint128 quantity, uint64 expiration) internal view {
-    // Ensure role is initialized.
-    if (role > numRoles) revert RoleNotInitialized(role);
-
     // Cannot set the ALL_HOLDERS_ROLE because this is handled in the _mint / _burn methods and can
     // create duplicate entries if set here.
     if (role == ALL_HOLDERS_ROLE) revert AllHoldersRole();

+    // Ensure role is initialized.
+    if (role > numRoles) revert RoleNotInitialized(role);
+
     // An expiration of zero is only allowed if the role is being removed. Roles are removed when
     // the quantity is zero. In other words, the relationships that are required between the role
     // quantity and expiration fields are:
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L317-L324

The check in [line 324](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L324) accesses calldata, the check in [line 323](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L323) accesses storage, and the check in [lines 320-322](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L320-L322) accesses storage at least once and potentially multiple times. To save at least one SLOAD in unhappy path, place the checks in the following order:

1. Check in [line 324](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L324)
2. Check in [line 323](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L323)
3. Check in [lines 320-322](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L320-L322)

```solidity
File: src/LlamaCore.sol
317:  function executeAction(ActionInfo calldata actionInfo) external payable {
318:    // Initial checks that action is ready to execute.
319:    Action storage action = actions[actionInfo.id];
320:    ActionState currentState = getActionState(actionInfo); // @audit: accesses storage (at least 1 SLOAD, potentially more)
321:
322:    if (currentState != ActionState.Queued) revert InvalidActionState(currentState); // @audit: depends on line 320
323:    if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached(); // @audit: accesses storage (1 SLOAD)
324:    if (msg.value != actionInfo.value) revert IncorrectMsgValue(); // @audit: accesses calldata
```
```diff
diff --git a/src/LlamaCore.sol b/src/LlamaCore.sol
index 89d60de..594e9f4 100644
--- a/src/LlamaCore.sol
+++ b/src/LlamaCore.sol
@@ -316,12 +316,13 @@ contract LlamaCore is Initializable {
   /// @param actionInfo Data required to create an action.
   function executeAction(ActionInfo calldata actionInfo) external payable {
     // Initial checks that action is ready to execute.
+    if (msg.value != actionInfo.value) revert IncorrectMsgValue();
+
     Action storage action = actions[actionInfo.id];
-    ActionState currentState = getActionState(actionInfo);
+    if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();

+    ActionState currentState = getActionState(actionInfo);
     if (currentState != ActionState.Queued) revert InvalidActionState(currentState);
-    if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();
-    if (msg.value != actionInfo.value) revert IncorrectMsgValue();

     action.executed = true;
```