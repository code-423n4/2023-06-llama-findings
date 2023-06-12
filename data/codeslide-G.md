### <a id="summary-gas-optimizations"></a> Gas Optimizations
There are 35 instances over 3 issues.
|      ID       | Description                                                                   | Count | Min Gas Savings (Est.) |
| :-----------: | :---------------------------------------------------------------------------- | ----: | ---------------------: |
| [G-01](#g-01) | Use assembly to check for `address(0)`                                        |    10 |                     60 |
| [G-02](#g-02) | Use calldata instead of memory for function arguments that do not get mutated |    11 |                   2321 |
| [G-03](#g-03) | Use `!= 0` instead of `> 0` for unsigned integer comparison                   |    14 |                     84 |

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

