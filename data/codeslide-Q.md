2023-06-llama QA Report
___
- [ Summary](#-summary)
  - [ Non-Critical Issues](#-non-critical-issues)
  - [ Gas Optimizations](#-gas-optimizations)
- [ Details](#-details)
  - [ Non-Critical Issues](#-non-critical-issues-1)
    - [ \[NC-01\] Un-indexed event parameters](#-nc-01-un-indexed-event-parameters)
    - [ \[NC-02\] Functions not used internally could be marked external](#-nc-02-functions-not-used-internally-could-be-marked-external)
  - [ Gas Optimizations](#-gas-optimizations-1)
    - [ \[G-01\] Use assembly to check for `address(0)`](#-g-01-use-assembly-to-check-for-address0)
    - [ \[G-02\] Use calldata instead of memory for function arguments that do not get mutated](#-g-02-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)
    - [ \[G-03\] Use `!= 0` instead of `> 0` for unsigned integer comparison](#-g-03-use--0-instead-of--0-for-unsigned-integer-comparison)
___
## <a id="summary"></a> Summary

### <a id="summary-non-critical-issues"></a> Non-Critical Issues
There are X instances over Y issues.
|       ID        | Description                                            | Count |
| :-------------: | :----------------------------------------------------- | ----: |
| [NC-01](#nc-01) | Un-indexed event parameters                            |    17 |
| [NC-02](#nc-02) | Functions not used internally could be marked external |     7 |

### <a id="summary-gas-optimizations"></a> Gas Optimizations
There are 136 instances over 11 issues.
|      ID       | Description                                                                   | Count | Min Gas Savings (Est.) |
| :-----------: | :---------------------------------------------------------------------------- | ----: | ---------------------: |
| [G-01](#g-01) | Use assembly to check for `address(0)`                                        |    10 |                     60 |
| [G-02](#g-02) | Use calldata instead of memory for function arguments that do not get mutated |    11 |                   2321 |
| [G-03](#g-03) | Use `!= 0` instead of `> 0` for unsigned integer comparison                   |    14 |                     84 |
## <a id="details"></a> Details

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


### <a id="details-gas-optimizations"></a> Gas Optimizations
#### <a id="g-01"></a> \[G-01\] Use assembly to check for `address(0)`
##### Description:
Example code is at [Solidity Assembly: Checking if an Address is 0 (Efficiently)](https://medium.com/@kalexotsu/solidity-assembly-checking-if-an-address-is-0-efficiently-d2bfe071331).

*Saves 6 gas per instance.*

##### Instances:
There are 10 instances of this issue.
```solidity
File: src/LlamaCore.sol

298:     if (signer == address(0) || signer != policyholder) revert InvalidSignature();

389:     if (signer == address(0) || signer != policyholder) revert InvalidSignature();

422:     if (signer == address(0) || signer != policyholder) revert InvalidSignature();

```
[LlamaCore.sol - Line 298](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298)

```solidity
File: src/LlamaPolicy.sol

181:     if (llamaExecutor != address(0)) revert AlreadyInitialized();

405:     if (llamaExecutor == address(0)) return; // Skip check during initialization.

```
[LlamaPolicy.sol - Line 181](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L181)

```solidity
File: src/accounts/LlamaAccount.sol

148:     if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();

166:     if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();

199:     if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();

247:     if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();

256:     if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();

```
[accounts/LlamaAccount.sol - Line 148](https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L148)


#### <a id="g-02"></a> \[G-02\] Use calldata instead of memory for function arguments that do not get mutated
##### Description:
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

[Example with gas report](https://gist.github.com/ezcodeslide/d95c73d677bf93f4da21331e7e2c7040).

*Saves ~211 gas per parameter.*

##### Instances:
There are 11 instances of this issue.
<details><summary>View 11 Instances</summary>

```solidity
File: src/LlamaCore.sol

225:     string memory _name,
```
[LlamaCore.sol - Line 225](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L225)


```solidity
File: src/LlamaPolicyMetadata.sol

17:   function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)

111:   function contractURI(string memory name) public pure returns (string memory) {

```
[LlamaPolicyMetadata.sol - Line 17](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L17)

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

82:   function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {

90:   function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {

```
[LlamaPolicyMetadataParamRegistry.sol - Line 82](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L82)

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

```
[accounts/LlamaAccount.sol - Line 130](https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130)

```solidity
File: src/interfaces/ILlamaAccount.sol

18:   function initialize(bytes memory config) external;

```
[interfaces/ILlamaAccount.sol - Line 18](https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaAccount.sol#L18)

```solidity
File: src/interfaces/ILlamaStrategy.sol

29:   function initialize(bytes memory config) external;

```
[interfaces/ILlamaStrategy.sol - Line 29](https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaStrategy.sol#L29)

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

153:   function initialize(bytes memory config) external virtual initializer {

```
[strategies/LlamaAbsoluteStrategyBase.sol - Line 153](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L153)

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

```
[strategies/LlamaRelativeQuorum.sol - Line 156](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156)

</details>

#### <a id="g-03"></a> \[G-03\] Use `!= 0` instead of `> 0` for unsigned integer comparison
##### Description:
Since an unsigned integer can never be less than 0 and using `> 0` uses more gas then `!= 0`, always use `!= 0`. The details about the op codes used are [here](https://twitter.com/gzeon/status/1485428085885640706).

*Saves 6 gas per instance.*

##### Instances:
There are 14 instances of this issue.
<details><summary>View 14 Instances</summary>

```solidity
File: src/LlamaCore.sol

629:     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

649:     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {

```
[LlamaCore.sol - Line 629](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629)

```solidity
File: src/LlamaPolicy.sol

307:     return quantity > 0;

313:     return quantity > 0;

320:     return quantity > 0 && canCreateAction[role][permissionId];

326:     return quantity > 0 && block.timestamp > expiration;

425:     bool case1 = quantity > 0 && expiration > block.timestamp;

444:     bool hadRole = initialQuantity > 0;

445:     bool willHaveRole = quantity > 0;

```
[LlamaPolicy.sol - Line 307](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L307)

```solidity
File: src/lib/Checkpoints.sol

130:         if (pos > 0) {

```
[lib/Checkpoints.sol - Line 130](https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L130)

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

215:     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

232:     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```
[strategies/LlamaAbsoluteStrategyBase.sol - Line 215](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215)

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

223:     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242:     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```
[strategies/LlamaRelativeQuorum.sol - Line 223](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223)

</details>

