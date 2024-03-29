# [G-01] Application does not immediately revert on error in LlamaExecutor

`execute()` function calls/delegatecalls to the external address:

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaExecutor.sol#L29

  function execute(address target, uint256 value, bool isScript, bytes calldata data)
    external
    returns (bool success, bytes memory result)
    (...)

 (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
```

However, when call/delegatecall fails, function does not immediately revert. Instead, the `success` variable is being returned by function and is being passed to LlamaCore.sol and checked for success there:


```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L333

 (bool success, bytes memory result) =
      executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);

    if (!success) revert FailedActionExecution(result);
```

This is a waste of gas, since `success` variable is being initialized twice and passed by the function. Instead, the `execute()` function should revert immediately when call/delegatecall fails - and only `result` should be passed to `LlamaCore`.


# [G-02] Refactor code to store color as hex-decimal instead of string

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L47

mapping(LlamaExecutor => string) public color;
(...)
string memory rootColor = "#6A45EC";

```

The `color` is defined as 6 hex-decimal digits followed by hash. This makes the mapping to be defined as `LlamaExecutor => string`.
However, since the color has always a constant length (6 digits + followed by hash) - instead of using more gas costly `string`, consider using `bytes7` instead.

```
mapping(LlamaExecutor => bytes7) public color;
(...)
bytes7 rootColor = "#6A45EC";
```

Additionally, it's worth to reconsider the whole logic of the smart contract (the part where the color is being concatenated).
The even more optimized version would require using uint256 instead of string (get rid of hash and store color as hex value, such as: `uint rootColor = 0x6A45EC;`). This modification, however, would require a lot of changes in the concatenation part - thus is not recommended without futher review.



# [G-03] Values in constructor can be hardcoded

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L57

 constructor(LlamaExecutor rootLlamaExecutor) {
    ROOT_LLAMA_EXECUTOR = rootLlamaExecutor;
    LLAMA_FACTORY = msg.sender;

    string memory rootColor = "#6A45EC";
    string memory rootLogo =
      '<g><path fill="#fff" d="M91.749 446.038H85.15v2.785h2.54v14.483h-3.272v2.785h9.746v-2.785h-2.416v-17.268ZM104.122 446.038h-6.598v2.785h2.54v14.483h-3.271v2.785h9.745v-2.785h-2.416v-17.268ZM113.237 456.162c.138-1.435 1.118-2.2 2.885-2.2 1.767 0 2.651.765 2.651 2.423v.403l-4.859.599c-2.885.362-5.149 1.63-5.149 4.484 0 2.841 2.14 4.47 5.383 4.47 2.72 0 3.921-1.044 4.487-1.935h.276v1.685h3.782v-9.135c0-3.983-2.54-5.78-6.488-5.78-3.975 0-6.404 1.797-6.694 4.568v.418h3.726Zm-.483 5.528c0-1.1.829-1.629 2.03-1.796l3.989-.529v.626c0 2.354-1.546 3.537-3.672 3.537-1.491 0-2.347-.724-2.347-1.838ZM125.765 466.091h3.727v-9.386c0-1.796.938-2.576 2.25-2.576 1.173 0 1.753.682 1.753 1.838v10.124h3.727v-9.386c0-1.796.939-2.576 2.236-2.576 1.187 0 1.753.682 1.753 1.838v10.124h3.741v-10.639c0-2.646-1.657-4.22-4.183-4.22-2.264 0-3.312.989-3.92 2.075h-.276c-.414-.947-1.436-2.075-3.534-2.075-2.056 0-2.954.864-3.45 1.741h-.277v-1.462h-3.547v14.58ZM151.545 456.162c.138-1.435 1.118-2.2 2.885-2.2 1.767 0 2.65.765 2.65 2.423v.403l-4.859.599c-2.885.362-5.149 1.63-5.149 4.484 0 2.841 2.14 4.47 5.384 4.47 2.719 0 3.92-1.044 4.486-1.935h.276v1.685H161v-9.135c0-3.983-2.54-5.78-6.488-5.78-3.975 0-6.404 1.797-6.694 4.568v.418h3.727Zm-.484 5.528c0-1.1.829-1.629 2.03-1.796l3.989-.529v.626c0 2.354-1.546 3.537-3.672 3.537-1.491 0-2.347-.724-2.347-1.838Z"/><g fill="#6A45EC"><path d="M36.736 456.934c.004-.338.137-.661.372-.901.234-.241.552-.38.886-.389h16.748a5.961 5.961 0 0 0 2.305-.458 6.036 6.036 0 0 0 3.263-3.287c.303-.737.46-1.528.46-2.326V428h-4.738v21.573c-.004.337-.137.66-.372.901-.234.24-.552.379-.886.388H38.01a5.984 5.984 0 0 0-4.248 1.781A6.108 6.108 0 0 0 32 456.934v14.891h4.736v-14.891ZM62.868 432.111h-.21l.2.204v4.448h4.36l2.043 2.084a6.008 6.008 0 0 0-3.456 2.109 6.12 6.12 0 0 0-1.358 3.841v27.034h4.717v-27.04c.005-.341.14-.666.38-.907.237-.24.56-.378.897-.383h.726c2.783 0 3.727-1.566 4.006-2.224.28-.658.711-2.453-1.257-4.448l-4.617-4.702h-1.437M50.34 469.477a7.728 7.728 0 0 1 3.013.61c.955.403 1.82.994 2.547 1.738h5.732a12.645 12.645 0 0 0-4.634-5.201 12.467 12.467 0 0 0-6.658-1.93c-2.355 0-4.662.669-6.659 1.93a12.644 12.644 0 0 0-4.634 5.201h5.733a7.799 7.799 0 0 1 2.546-1.738 7.728 7.728 0 0 1 3.014-.61Z"/></g></g>';
    setColor(ROOT_LLAMA_EXECUTOR, rootColor);
    setLogo(ROOT_LLAMA_EXECUTOR, rootLogo);
  }
```

Instead of creating two variables: `string memory rootColor`, `string memory rootLogo` - those values can be explicitly used in `setColor` and `setLogo` functions, e.g.:

```
setColor(ROOT_LLAMA_EXECUTOR, "#6A45EC");
```



# [G-04] Unnecessary if condition in LlamaAbsoluteQuorum


```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaAbsoluteQuorum.sol#L30

if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);

(...)

if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();

```
The first condition is unnessesary. Since `minApprovals` is uint128 (0 or larger), whenever `approvalPolicySupply` is 0, the 2nd condition will always revert.
Whatever `minApprovals` value is, `if (minApprovals > approvalPolicySupply)` will always revert when `approvalPolicySupply` is 0, thus checking it in `if (approvalPolicySupply == 0)` is not needed.

The same issue occurs for `if (disapprovalPolicySupply == 0)` condition in the same file.


# [G-05] Incorrect order of if conditions in LlamaRelativeQuorum

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L255

function validateActionCancelation(ActionInfo calldata actionInfo, address caller) external view {
    (...)

    // Check 1.
    ActionState state = llamaCore.getActionState(actionInfo);
    if (
      state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
        || state == ActionState.Failed
    ) revert CannotCancelInState(state);

    // Check 2.
    if (caller != actionInfo.creator) revert OnlyActionCreator();
  }
```

To save gas for users who are not action creators but who run this external function - the `check 2` should be performed before `check 1`. It will save gas for non-action creators who run this function.
Recommendation: Firstly perform `if (caller != actionInfo.creator)` check, then check action's state.


# [G-06] Unnecessary if condition in LlamaFactory

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L242

 if (initialRoleHolders.length == 0) revert InvalidDeployConfiguration();
    if (initialRoleHolders[0].role != BOOTSTRAP_ROLE) revert InvalidDeployConfiguration();
    if (initialRoleHolders[0].expiration != type(uint64).max) revert InvalidDeployConfiguration();
```

The first conditions is not needed and should be removed for gas optimization. If the `initialRoleHolders.length` is 0 (an empty array), then other two conditions would still revert. 


# [G-07] Length of the array is calculated twice in LlamaGovernanceScript

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/llama-scripts/LlamaGovernanceScript.sol#L67

 if (targets.length != data.length) revert MismatchedArrayLengths();
    (LlamaCore core, LlamaPolicy policy) = _context();
    uint256 length = data.length;
```

The length of data is being calculated and assigned in `uint256 length = data.length;`. However, the same length is being calculated in the if condition.
The recommendation is to calculate length once:

```
uint256 length = data.length;
 if (targets.length != length) revert MismatchedArrayLengths();
    (LlamaCore core, LlamaPolicy policy) = _context();
    
```


# [G-08] Unnecessary declaration of variables in loop in LlamaGovernanceScript

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/llama-scripts/LlamaGovernanceScript.sol#L71

for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
      bool addressIsCore = targets[i] == address(core);
      bool addressIsPolicy = targets[i] == address(policy);
      if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);
```

For-loop declares `addressIsCore` and `addressIsPolicy` on every loop interation, only to compare those values in the next line. To save gas usage, this can be rewritten as:

```
for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
      if (!(targets[i] == address(core)) && !(targets[i] == address(policy)))) revert UnauthorizedTarget(targets[i]);
```

# [G-09] Use calldata instead of memory for function parameters

If you are not modifying the function parameters, use calldata instead of memory.

Multiple of instances were detected with similar issue:
```
../accounts/LlamaAccount.sol:  function initialize(bytes memory config) external initializer {
../interfaces/ILlamaAccount.sol:  function initialize(bytes memory config) external;
../interfaces/ILlamaStrategy.sol:  function initialize(bytes memory config) external;
../LlamaFactory.sol:  function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)
../LlamaFactory.sol:  function contractURI(string memory name) external view returns (string memory) {
../LlamaFactory.sol:  function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {
../LlamaPolicyMetadata.sol:  function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)
../LlamaPolicyMetadata.sol:  function contractURI(string memory name) public pure returns (string memory) {
../LlamaPolicyMetadataParamRegistry.sol:  function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {
../LlamaPolicyMetadataParamRegistry.sol:  function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {
../strategies/LlamaAbsoluteStrategyBase.sol:  function initialize(bytes memory config) external virtual initializer {
../strategies/LlamaRelativeQuorum.sol:  function initialize(bytes memory config) external initializer {
```



# [G-10] Use unchecked keyword for loop counters

Using unchecked blocks saves just a little of gas since the contract does not perform additional actions for checking overflow.
The contract uses unchecked, however, it imports external library to do so:

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol

function uncheckedIncrement(uint256 i) internal pure returns (uint256) {
    unchecked {
      return i + 1;
    }
  }
```

This is very gas-cost ineffective, since every time there needs to be call to that function. 
Every loop in a form of:
```
for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {}
```

should be rewritten to:

```
for (uint256 i = 0; i < rolePermissions.length; ) {
  
  unchecked {++i;}
  
}
```

Multiple of loops (which were not detected by automated tools) are affected:

```
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaCore.sol:    for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaCore.sol:    for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 1; i <= numRoles; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = start; i < end; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaAbsoluteStrategyBase.sol:    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaAbsoluteStrategyBase.sol:    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaRelativeQuorum.sol:    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaRelativeQuorum.sol:    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

It's important to notice, that not only loops are affacted. Everywhere where overflow is not possible, and smart contract uses `LlamaUtils.unheckedIncrement()`, this piece of code should be rewritten to an explicit unchecked block:

```
../LlamaCore.sol:    nonces[msg.sender][selector] = LlamaUtils.uncheckedIncrement(nonces[msg.sender][selector]);
../LlamaCore.sol:    actionsCount = LlamaUtils.uncheckedIncrement(actionsCount); // Safety: Can never overflow a uint256 by incrementing.
../LlamaCore.sol:    nonces[policyholder][selector] = LlamaUtils.uncheckedIncrement(nonce);
../LlamaFactory.sol:    llamaCount = LlamaUtils.uncheckedIncrement(llamaCount);
```

# [G-11] Variables with default initialization values in loop are re-initialized to their default values.
In multiple of loops, the code re-initializes the counter to 0. To save some gas, the re-initialization should not occur, since the default value for `uint256` is already 0.

Rewrite every loop from `for (uint256 i = 0;` to `for (uint256 i;`

Multiple of loops are affected:

```
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../accounts/LlamaAccount.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {
../llama-scripts/LlamaGovernanceScript.sol:    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaCore.sol:    for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaCore.sol:    for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {
../LlamaPolicy.sol:    for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaAbsoluteStrategyBase.sol:    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaAbsoluteStrategyBase.sol:    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaRelativeQuorum.sol:    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
../strategies/LlamaRelativeQuorum.sol:    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
```


# [G-12] Unnecessary initialization of variables in LlamaPolicy

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L285

 uint256 checkpointsLength = roleBalanceCkpts[_tokenId(policyholder)][role]._checkpoints.length;
    if (end > checkpointsLength) revert InvalidIndices();
```

`checkpointsLength` is only intialized to be used in if-condition. This variable is used only once. The code can be rewritten more straighforwardly without initializing this variable:

```
if (end > roleBalanceCkpts[_tokenId(policyholder)][role]._checkpoints.length) revert InvalidIndices();
```


The same issue occurs for multiple of functions for `tokenId`:

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

function roleBalanceCheckpointsLength(address policyholder, uint8 role) external view returns (uint256) {
    uint256 tokenId = _tokenId(policyholder);
    return roleBalanceCkpts[tokenId][role]._checkpoints.length;
  }

   function roleBalanceCheckpoints(address policyholder, uint8 role) external view returns (Checkpoints.History memory) {
    uint256 tokenId = _tokenId(policyholder);
    return roleBalanceCkpts[tokenId][role];

    function getPastQuantity(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {
    uint256 tokenId = _tokenId(policyholder);
    return roleBalanceCkpts[tokenId][role].getAtProbablyRecentTimestamp(timestamp);
  }

    function getQuantity(address policyholder, uint8 role) external view returns (uint128) {
    uint256 tokenId = _tokenId(policyholder);
    return roleBalanceCkpts[tokenId][role].latest();
```

Those function can be rewritten without initialization of `uint256 tokenId`, e.g.:

```
  function getQuantity(address policyholder, uint8 role) external view returns (uint128) {
    return roleBalanceCkpts[_tokenId(policyholder)][role].latest();
```

The same issue occurs for `quantity ` variable:

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

 function hasRole(address policyholder, uint8 role) public view returns (bool) {
    uint128 quantity = roleBalanceCkpts[_tokenId(policyholder)][role].latest();
    return quantity > 0;
  }

 
  function hasRole(address policyholder, uint8 role, uint256 timestamp) external view returns (bool) {
    uint256 quantity = roleBalanceCkpts[_tokenId(policyholder)][role].getAtProbablyRecentTimestamp(timestamp);
    return quantity > 0;
  }


  function hasPermissionId(address policyholder, uint8 role, bytes32 permissionId) external view returns (bool) {
    uint128 quantity = roleBalanceCkpts[_tokenId(policyholder)][role].latest();
    return quantity > 0 && canCreateAction[role][permissionId];
  }


```

Those functions can be rewritten, for example like this:

```
 function hasPermissionId(address policyholder, uint8 role, bytes32 permissionId) external view returns (bool) {
    return roleBalanceCkpts[_tokenId(policyholder)][role].latest() > 0 && canCreateAction[role][permissionId];
  }
```


# [G-13] Unnecessary initialization of variables in LlamaCore
On every loop iteration, the `salt` variable is being intialized. This variable is only used once. To save gas, it shoudn't be initialized on every loop iteration.

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L636

 bytes32 salt = bytes32(keccak256(strategyConfigs[i]));
      ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), salt));
```

To avoid initialization of this variable, change this line of code to:

```
ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), bytes32(keccak256(accountConfigs[i]))));
```

# [G-14] If statements that use && can be refactored into nested if statements

To save gas usage if conditaions in a form of:

```
if (A && B)
```
can be refactored to:

```
if(A) {
  if(B)
}
```

Multiple of instances have been detected:

```
./llama-scripts/LlamaGovernanceScript.sol:      if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);
./LlamaCore.sol:    if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {
./LlamaCore.sol:    if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {
./LlamaPolicy.sol:    if (hadRole && !willHaveRole) {
./LlamaPolicyMetadataParamRegistry.sol:      msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
./strategies/LlamaAbsolutePeerReview.sol:    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
./strategies/LlamaAbsolutePeerReview.sol:    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
./strategies/LlamaAbsoluteQuorum.sol:    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {
./strategies/LlamaAbsoluteQuorum.sol:    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
./strategies/LlamaAbsoluteQuorum.sol:    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
./strategies/LlamaAbsoluteStrategyBase.sol:      block.timestamp <= approvalEndTime(actionInfo) && (isFixedLengthApprovalPeriod || !isActionApproved(actionInfo));
./strategies/LlamaRelativeQuorum.sol:    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
./strategies/LlamaRelativeQuorum.sol:    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
./strategies/LlamaRelativeQuorum.sol:      block.timestamp <= approvalEndTime(actionInfo) && (isFixedLengthApprovalPeriod || !isActionApproved(actionInfo));
```


# [G-15] Using ++var is more gas-cost effective than var += 1
The same issue occurs for --var and var -= 1

There multiple of instances had been detected, which were not detected by automated tests.

```
./LlamaPolicy.sol:    numRoles += 1;
./LlamaPolicy.sol:      currentRoleSupply.numberOfHolders += 1;
./LlamaPolicy.sol:      allHoldersRoleSupply.numberOfHolders += 1;
./LlamaPolicy.sol:      allHoldersRoleSupply.totalQuantity += 1;

./LlamaPolicy.sol:      currentRoleSupply.numberOfHolders -= 1;
./LlamaPolicy.sol:      allHoldersRoleSupply.numberOfHolders -= 1;
./LlamaPolicy.sol:      allHoldersRoleSupply.totalQuantity -= 1;
```


