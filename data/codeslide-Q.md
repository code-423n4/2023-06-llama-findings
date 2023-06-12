2023-06-llama QA Report
___
- [ Summary](#-summary)
  - [ Low Risk Issues](#-low-risk-issues)
  - [ Non-Critical Issues](#-non-critical-issues)
- [ Details](#-details)
  - [ Non-Critical Issues](#-non-critical-issues-1)
    - [ \[L-01\] Hash collision risk with `abi.encodePacked()`](#-l-01-hash-collision-risk-with-abiencodepacked)
  - [ Non-Critical Issues](#-non-critical-issues-2)
    - [ \[NC-01\] Un-indexed event parameters](#-nc-01-un-indexed-event-parameters)
    - [ \[NC-02\] Functions not used internally could be marked external](#-nc-02-functions-not-used-internally-could-be-marked-external)
___
## <a id="summary"></a> Summary

### <a id="summary-low-risk-issues"></a> Low Risk Issues
There is 1 instance over 1 issue.
|      ID       | Description                                   | Count |
| :-----------: | :-------------------------------------------- | ----: |
| [L-01](#l-01) | Hash collision risk with `abi.encodePacked()` |     1 |

### <a id="summary-non-critical-issues"></a> Non-Critical Issues
There are 24 instances over 2 issues.
|       ID        | Description                                            | Count |
| :-------------: | :----------------------------------------------------- | ----: |
| [NC-01](#nc-01) | Un-indexed event parameters                            |    17 |
| [NC-02](#nc-02) | Functions not used internally could be marked external |     7 |

## <a id="details"></a> Details

### <a id="details-low-risk-issues"></a> Non-Critical Issues

#### <a id="l-01"></a> \[L-01\] Hash collision risk with `abi.encodePacked()`
##### Description:
`abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` To [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode), use `abi.encode()` instead since it will pad the arguments to 32 bytes. For example, `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). Unless there is a compelling reason, `abi.encode` should be preferred. If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and/or bytes, `bytes.concat()` should be used instead.

##### Instances:
There is 1 instance of this issue.
```solidity
File: src/LlamaCore.sol

687:     return keccak256(abi.encodePacked(id, creator, creatorRole, strategy, target, value, data));
```
[LlamaCore.sol - Line 687](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L687)

### <a id="details-non-critical-issues"></a> Non-Critical Issues

#### <a id="nc-01"></a> \[NC-01\] Un-indexed event parameters
##### Description:
Indexing an event parameter makes the parameter more quickly accessible to off-chain tools that parse events. Indexed parameters appear in the structure of topics, not the data portion of the EVM log. Parameters that do not have the `indexed` attribute are ABI-encoded into the data portion of the log.

Since there is a limit of three `indexed` parameters per event, a best practice is:
* If there are <= 3 parameters, all of the parameters should have the `indexed` attribute.
* If there are > 3 parameters, add the `indexed` attribute to the three most important/useful parameters.

##### Instances:
There are 17 instances of this issue.
<details><summary>View 17 Instances</summary>

```solidity
File: src/LlamaCore.sol

103:   event ActionCanceled(uint256 id);

106:   event ActionGuardSet(address indexed target, bytes4 indexed selector, ILlamaActionGuard actionGuard);

123:   event ApprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);

126:   event DisapprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);

129:   event StrategyCreated(ILlamaStrategy strategy, ILlamaStrategy indexed strategyLogic, bytes initializationData);

132:   event AccountCreated(ILlamaAccount account, ILlamaAccount indexed accountLogic, bytes initializationData);
```

[LlamaCore.sol - Line 103](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L103)

```solidity
File: src/LlamaPolicy.sol

85:   event RoleAssigned(address indexed policyholder, uint8 indexed role, uint64 expiration, uint128 quantity);

88:   event RoleInitialized(uint8 indexed role, RoleDescription description);

91:   event RolePermissionAssigned(uint8 indexed role, bytes32 indexed permissionId, bool hasPermission);
```
[LlamaPolicy.sol - Line 85](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L85)

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

31:   event ColorSet(LlamaExecutor indexed llamaExecutor, string color);

34:   event LogoSet(LlamaExecutor indexed llamaExecutor, string logo);
```
[LlamaPolicyMetadataParamRegistry.sol - Line 31](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L31)

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

83:   event ForceApprovalRoleAdded(uint8 role);

87:   event ForceDisapprovalRoleAdded(uint8 role);

90:   event StrategyCreated(LlamaCore llamaCore, LlamaPolicy policy);
```
[strategies/LlamaAbsoluteStrategyBase.sol - Line 83](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L83)

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

75:   event ForceApprovalRoleAdded(uint8 role);

79:   event ForceDisapprovalRoleAdded(uint8 role);

82:   event StrategyCreated(LlamaCore llamaCore, LlamaPolicy policy);
```
[strategies/LlamaRelativeQuorum.sol - Line 75](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L75)

</details>

#### <a id="nc-02"></a> \[NC-02\] Functions not used internally could be marked external
##### Description:
Note that contracts are allowed to override their parents' functions and change the visibility from `external` to `public`. For example, if the parent contract/interface has a `public` function, it can be overridden and be changed to `external` visibility.

##### Instances:
There are 7 instances of this issue.

```solidity
File: src/LlamaPolicy.sol

346:   function tokenURI(uint256 tokenId) public view override returns (string memory) {

359:   function transferFrom(address, /* from */ address, /* to */ uint256 /* policyId */ )

367:   function safeTransferFrom(address, /* from */ address, /* to */ uint256 /* id */ )

375:   function safeTransferFrom(address, /* from */ address, /* to */ uint256, /* policyId */ bytes calldata /* data */ )

383:   function approve(address, /* spender */ uint256 /* id */ ) public pure override nonTransferableToken {}

386:   function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}

```
[LlamaPolicy.sol - Line 263](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L263)

```solidity
File: src/LlamaPolicyMetadata.sol

111:   function contractURI(string memory name) public pure returns (string memory) {

```
[LlamaPolicyMetadata.sol - Line 111](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L111)
