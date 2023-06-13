## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 7 | 700 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Consider activating `via-ir` for deploying | 1 | - |
| [GAS&#x2011;3](#GAS&#x2011;3) | Build the `DOMAIN_TYPEHASH` In The `initialize()` To Save Gas and Clarify/Clean Code | 1 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 2 | 56 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Using fixed bytes is cheaper than using `string` | 1 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Using `delete` statement can save gas | 1 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied | 19 | - |
| [GAS&#x2011;8](#GAS&#x2011;8) | Multiple accesses of a mapping/array should use a local variable cache | 33 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | The result of a function call should be cached rather than re-calling the function | 2 | - |
| [GAS&#x2011;10](#GAS&#x2011;10) | Shorten the array rather than copying to a new one | 1 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | State variables can be packed into fewer storage slots | 1 | 2000 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 1 | - |
| [GAS&#x2011;13](#GAS&#x2011;13) | Structs can be packed into fewer storage slots | 1 | 2000 |
| [GAS&#x2011;14](#GAS&#x2011;14) | Using `unchecked` blocks to save gas | 5 | 100 |
| [GAS&#x2011;15](#GAS&#x2011;15) | Structs can be packed into fewer storage slots by editing time variables | 1 | 2000 |


Total: 77 contexts over 15 issues

## Gas Optimizations

### <a href="#GAS Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


<details>

```solidity
242: return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L242

```solidity
529: bytes32 permissionId = keccak256(abi.encode(permission));
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L529

```solidity
708: abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L708

```solidity
728: abi.encode(
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L728

```solidity
753: abi.encode(
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L753

```solidity
775: abi.encode(
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L775

```solidity
791: abi.encode(
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L791



</details>



### <a href="#GAS Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Consider activating `via-ir` for deploying

The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add --via-ir to your deploy command

More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html





### <a href="#GAS Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Build the `DOMAIN_TYPEHASH` In The `initialize()` To Save Gas and Clarify/Clean Code

Building the DOMAIN_TYPEHASH in the initialize() can save gas and clean the code up.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
142: bytes32 internal constant EIP712_DOMAIN_TYPEHASH =
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L142



</details>

#### <ins>Recommended Mitigation Steps</ins>

1. Remove the `bytes32 internal constant EIP712_DOMAIN_TYPEHASH`
2. Assign the `EIP712_DOMAIN_TYPEHASH` in the `initialize()`




#### <a href="#GAS Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

<details>

```solidity
90: require(to != address(0), "INVALID_RECIPIENT");
128: require(to != address(0), "INVALID_RECIPIENT");

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L90

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L128





</details>





### <a href="#GAS Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using fixed bytes is cheaper than using `string`

As a rule of thumb, use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
61: string memory rootColor = "#6A45EC";
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadataParamRegistry.sol#L61



</details>




### <a href="#GAS Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

<details>

```solidity
43: uint256 low = 0;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L43



</details>




### <a href="#GAS Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied

#### <ins>Proof Of Concept</ins>


<details>

```solidity
636: for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L636

```solidity
656: for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L656

```solidity
151: for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L151

```solidity
156: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L156

```solidity
174: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L174

```solidity
189: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L189

```solidity
207: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L207

```solidity
222: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L222

```solidity
237: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L237

```solidity
270: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L270

```solidity
285: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L285

```solidity
71: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L71

```solidity
178: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L178

```solidity
186: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L186

```solidity
199: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L199

```solidity
210: for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L210

```solidity
217: for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L217

```solidity
177: for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L177

```solidity
177: for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L177



</details>





### <a href="#GAS Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
637: bytes32 salt = bytes32(keccak256(strategyConfigs[i]));
639: strategy.initialize(strategyConfigs[i]);
641: emit StrategyCreated(strategy, llamaStrategyLogic, strategyConfigs[i]);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L637

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L639

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L641



```solidity
657: bytes32 salt = bytes32(keccak256(accountConfigs[i]));
659: account.initialize(accountConfigs[i]);
660: emit AccountCreated(account, llamaAccountLogic, accountConfigs[i]);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L657

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L659

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L660



```solidity
699: nonce = nonces[policyholder][selector];
700: nonces[policyholder][selector] = LlamaUtils.uncheckedIncrement(nonce);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L699-L700

```solidity
243: if (initialRoleHolders[0].role != BOOTSTRAP_ROLE) revert InvalidDeployConfiguration();
244: if (initialRoleHolders[0].expiration != type(uint64).max) revert InvalidDeployConfiguration();

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L243-L244

```solidity
443: uint128 initialQuantity = roleBalanceCkpts[tokenId][role].latest();
448: roleBalanceCkpts[tokenId][role].push(willHaveRole ? quantity : 0, expiration);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L443

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L448



```solidity
88: require(from == _ownerOf[id], "WRONG_FROM");
102: _ownerOf[id] = to;
137: _ownerOf[id] = to;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L88

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L102

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L137



```solidity
130: require(_ownerOf[id] == address(0), "ALREADY_MINTED");
102: _ownerOf[id] = to;
137: _ownerOf[id] = to;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L130

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L102

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L137



```solidity
72: bool addressIsCore = targets[i] == address(core);
73: bool addressIsPolicy = targets[i] == address(policy);
74: if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L72-L74

```solidity
188: _setRoleHolders[i].role,
189: _setRoleHolders[i].policyholder,
190: _setRoleHolders[i].quantity,
191: _setRoleHolders[i].expiration

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L188-L191

```solidity
213: if (role != approvalRole && !forceApprovalRole[role]) return 0;
215: return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L213

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215



```solidity
230: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
232: return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L230

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L232



```solidity
221: if (role != approvalRole && !forceApprovalRole[role]) return 0;
223: return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L221

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L223



```solidity
240: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
242: return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L240

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L242





</details>




### <a href="#GAS Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


<details>

```solidity
322: bytes32 originalStorage = _readSlot0();
324: if (originalStorage != _readSlot0()) revert Slot0Changed();

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L322

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L324





</details>



### <a href="#GAS Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

#### <ins>Proof Of Concept</ins>


<details>

```solidity
290: Checkpoints.Checkpoint[] memory checkpoints = new Checkpoints.Checkpoint[](sliceLength);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L290



</details>



### <a href=3D"#GAS Summary">[GAS&#x2011;11]</a><a name=3D"GAS&#x2011;11"> State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

#### <ins>Proof Of Concept</ins>

<details>

```solidity
 /// @notice If `false`, action be queued before approvalEndTime.
 bool public isFixedLengthApprovalPeriod;

 /// @notice Length of approval period in seconds.
 uint64 public approvalPeriod;

 /// @notice Minimum time, in seconds, between queueing and execution of action.
 uint64 public queuingPeriod;

 /// @notice Time, in seconds, after `minExecutionTime` that action can be executed before permanently expiring.
 uint64 public expirationPeriod;

 /// @notice Minimum total quantity of approvals for the action to be queued.
 /// @dev We use a `uint128` here since quantities are stored as `uint128` in the policy.
 uint128 public minApprovals;

 /// @notice Minimum total quantity of disapprovals for the action to be canceled.
 /// @dev We use a `uint128` here since quantities are stored as `uint128` in the policy.
 uint128 public minDisapprovals;

 /// @notice The role that can approve an action.
 uint8 public approvalRole;

 /// @notice The role that can disapprove an action.
 uint8 public disapprovalRole;

```

Save 1 storage slot by changing to:

```solidity
 /// @notice If `false`, action be queued before approvalEndTime.
 bool public isFixedLengthApprovalPeriod;

 /// @notice Length of approval period in seconds.
 uint64 public approvalPeriod;

 /// @notice Minimum time, in seconds, between queueing and execution of action.
 uint64 public queuingPeriod;

 /// @notice Time, in seconds, after `minExecutionTime` that action can be executed before permanently expiring.
 uint64 public expirationPeriod;

 /// @notice The role that can approve an action.
 uint8 public approvalRole;

 /// @notice The role that can disapprove an action.
 uint8 public disapprovalRole;

 /// @notice Minimum total quantity of approvals for the action to be queued.
 /// @dev We use a `uint128` here since quantities are stored as `uint128` in the policy.
 uint128 public minApprovals;

 /// @notice Minimum total quantity of disapprovals for the action to be canceled.
 /// @dev We use a `uint128` here since quantities are stored as `uint128` in the policy.
 uint128 public minDisapprovals;
```
https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L22



</details>



### <a href="#GAS Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
292: checkpoints[i - start] = roleBalanceCkpts[tokenId][role]._checkpoints[i];
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L292



</details>




### <a href=3D"#GAS Summary">[GAS&#x2011;13]</a><a name=3D"GAS&#x2011;13"> Structs can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

#### <ins>Proof Of Concept</ins>

<details>





</details>





### <a href="#GAS Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

<details>

```solidity
47: uint256 mid = len - sqrt(len);
57: return pos == 0 ? 0 : _unsafeAccess(self._checkpoints, pos - 1).quantity;
85: return pos == 0 ? 0 : _unsafeAccess(self._checkpoints, pos - 1).quantity;

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L47

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L57

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L85



```solidity
132: Checkpoint memory last = _unsafeAccess(self, pos - 1);
139: Checkpoint storage ckpt = _unsafeAccess(self, pos - 1);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L132

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L139





</details>


### <a href="#GAS Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Structs can be packed into fewer storage slots by editing time variables

The following structs contain time variables that are unlikely to ever reach max `uint256` then it is possible to set them as `uint32`, saving gas slots. Max value of `uint32` is equal to Year 2016, which is more than enough.

#### <ins>Proof Of Concept</ins>

<details>
 

```solidity
25: struct Config {
    uint64 approvalPeriod; // The length of time of the approval period.
    uint64 queuingPeriod; // The length of time of the queuing period. The disapproval period is the queuing period when
      // enabled.
    uint64 expirationPeriod; // The length of time an action can be executed before it expires.
    uint16 minApprovalPct; // Minimum percentage of total approval quantity / total approval supply.
    uint16 minDisapprovalPct; // Minimum percentage of total disapproval quantity / total disapproval supply.
    bool isFixedLengthApprovalPeriod; // Determines if an action be queued before approvalEndTime.
    uint8 approvalRole; // Anyone with this role can cast approval of an action.
    uint8 disapprovalRole; // Anyone with this role can cast disapproval of an action.
    uint8[] forceApprovalRoles; // Anyone with this role can single-handedly approve an action.
    uint8[] forceDisapprovalRoles; // Anyone with this role can single-handedly disapprove an action.
  }
```

Save 1 storage slot by changing to:

```solidity
25: struct Config {
    uint32 approvalPeriod; // The length of time of the approval period.
    uint32 queuingPeriod; // The length of time of the queuing period. The disapproval period is the queuing period when
      // enabled.
    uint32 expirationPeriod; // The length of time an action can be executed before it expires.
    uint16 minApprovalPct; // Minimum percentage of total approval quantity / total approval supply.
    uint16 minDisapprovalPct; // Minimum percentage of total disapproval quantity / total disapproval supply.
    bool isFixedLengthApprovalPeriod; // Determines if an action be queued before approvalEndTime.
    uint8 approvalRole; // Anyone with this role can cast approval of an action.
    uint8 disapprovalRole; // Anyone with this role can cast disapproval of an action.
    uint8[] forceApprovalRoles; // Anyone with this role can single-handedly approve an action.
    uint8[] forceDisapprovalRoles; // Anyone with this role can single-handedly disapprove an action.
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L25



</details>