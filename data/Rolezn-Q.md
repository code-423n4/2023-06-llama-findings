## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Add to `blacklist` function | 1 |
| [LOW&#x2011;2](#LOW&#x2011;2) | External calls in an un-bounded `for-`loop may result in a DOS | 19 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Init functions are susceptible to front-running | 1 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Missing Contract-existence Checks Before Low-level Calls | 4 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Contracts are not using their OZ Upgradeable counterparts | 17 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Protect `LlamaPolicy.sol` NFT from copying in POW forks | 4 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Unbounded loop | 7 |
| [LOW&#x2011;8](#LOW&#x2011;8) | Condition will not revert when `block.timestamp` is `==` to the compared variable | 1 | 
| [LOW&#x2011;9](#LOW&#x2011;9) | Inconsistent documentation to actual function logic | 3 |
| [LOW&#x2011;10](#LOW&#x2011;10) | Missing Reentrancy-Guard when using `sendValue` from OZ's `Address.sol` | 1 |
| [LOW&#x2011;11](#LOW&#x2011;11) | `updateRoleDescription` doesn't do anything | 1 |

Total: 59 contexts over 11 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Critical Changes Should Use Two-step Procedure | 9 |
| [NC&#x2011;2](#NC&#x2011;2) | Large or complicated code bases should implement fuzzing tests | 1 |
| [NC&#x2011;3](#NC&#x2011;3) | Initial value check is missing in Set Functions | 9 |
| [NC&#x2011;4](#NC&#x2011;4) | Use @inheritdoc rather than using a non-standard annotation | 55 |
| [NC&#x2011;5](#NC&#x2011;5) | Function name should contain `InitializeRoles` instead of `NewRoles` | 1 |

Total: 75 contexts over 5 issues

## Low Risk Issues

### <a href="#QA Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function
It is noted that in this project: `LlamaPolicy.sol is an NFT`.

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.





### <a href="#QA Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> External calls in an un-bounded `for-`loop may result in a DOS

Consider limiting the number of iterations in `for-`loops that make external calls

#### <ins>Proof Of Concept</ins>

<details>

```solidity
151: for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
155: for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {
161: for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L151

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L155

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L161



```solidity
227: for (uint256 i = 1; i <= numRoles; i = LlamaUtils.uncheckedIncrement(i)) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L227

```solidity
291: for (uint256 i = start; i < end; i = LlamaUtils.uncheckedIncrement(i)) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L291

```solidity
156: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
174: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
189: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
207: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
222: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
237: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
270: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
285: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L156

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L174

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L189

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L207

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L222

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L237

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L270

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L285



```solidity
71: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
178: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
186: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
199: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L71

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L178

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L186

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L199



```solidity
177: for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
185: for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L177

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L185






</details>




### <a href="#QA Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Initialization functions are susceptible to front-running

Most contracts use an init pattern (instead of a constructor) to initialize contract parameters. Unless these are enforced to be atomic with contact deployment via deployment script or factory contracts, they are susceptible to front-running race conditions where an attacker/griefer can front-run (cannot access control because admin roles are not initialized) to initially with their own (malicious) parameters upon detecting (if an event is emitted) which the contract deployer has to redeploy wasting gas and risking other transactions from interacting with the attacker-initialized contract.

Many init functions do not have an explicit event emission which makes monitoring such scenarios harder. All of them have re-init checks; while many are explicit some (those in auction contracts) have implicit reinit checks in initAccessControls() which is better if converted to an explicit check in the main init function itself.
(details credit to: code-423n4/2021-09-sushimiso-findings#64)

#### <ins>Proof Of Concept</ins>


<details>

```solidity
function finalizeInitialization(address _llamaExecutor, bytes32 bootstrapPermissionId) external {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L180



</details>

#### <ins>Recommended Mitigation Steps</ins>

Ensure atomic calls to init functions along with deployment via robust deployment scripts or factory contracts. Emit explicit events for initializations of contracts. Enforce prevention of re-initializations via explicit setting/checking of boolean initialized variables in the main init function instead of downstream function checks.







### <a href="#QA Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Missing Contract-existence Checks Before Low-level Calls

Low-level calls return success if there is no code present at the specified address. 

#### <ins>Proof Of Concept</ins>


<details>

```solidity
34: (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaExecutor.sol#L34

```solidity
323: (success, result) = target.delegatecall(callData);
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L323

```solidity
326: (success, result) = target.call{value: msg.value}(callData);
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L326

```solidity
75: (bool success, bytes memory response) = targets[i].call(data[i]);
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L75



</details>


#### <ins>Recommended Mitigation Steps</ins>

In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`




### <a href="#QA Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

<details>

```solidity
4: import {Clones} from "@openzeppelin/proxy/Clones.sol";
5: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L4-L5

```solidity
4: import {Clones} from "@openzeppelin/proxy/Clones.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L4

```solidity
4: import {Base64} from "@openzeppelin/utils/Base64.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadata.sol#L4

```solidity
4: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";
5: import {IERC20} from "@openzeppelin/token/ERC20/IERC20.sol";
6: import {SafeERC20} from "@openzeppelin/token/ERC20/utils/SafeERC20.sol";
7: import {IERC721} from "@openzeppelin/token/ERC721/IERC721.sol";
8: import {ERC721Holder} from "@openzeppelin/token/ERC721/utils/ERC721Holder.sol";
9: import {IERC1155} from "@openzeppelin/token/ERC1155/IERC1155.sol";
10: import {ERC1155Holder} from "@openzeppelin/token/ERC1155/utils/ERC1155Holder.sol";
11: import {Address} from "@openzeppelin/utils/Address.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L4-L11

```solidity
5: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L5

```solidity
4: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsolutePeerReview.sol#L4

```solidity
4: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteQuorum.sol#L4

```solidity
4: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L4

```solidity
4: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L4



</details>

#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts



### <a href="#QA Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Protect `LlamaPolicy.sol`  NFT from copying in POW forks
Ethereum has performed the long-awaited "merge" that will dramatically reduce the environmental impact of the network

There may be forked versions of Ethereum, which could cause confusion and lead to scams as duplicated NFT assets enter the market.

If the Ethereum Merge, which took place in September 2022, results in the Blockchain splitting into two Blockchains due to the 'THE DAO' attack in 2016, this could result in duplication of immutable tokens (NFTs).

In any case, duplicate NFTs will exist due to the ETH proof-of-work chain and other potential forks, and there’s likely to be some level of confusion around which assets are 'official' or 'authentic.'

Even so, there could be a frenzy for these copies, as NFT owners attempt to flip the proof-of-work versions of their valuable tokens.

As ETHPOW and any other forks spin off of the Ethereum mainnet, they will yield duplicate versions of Ethereum’s NFTs. An NFT is simply a blockchain token, and it can work as a deed of ownership to digital items like artwork and collectibles. A forked Ethereum chain will thus have duplicated deeds that point to the same tokenURI

About Merge Replay Attack: https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

#### <ins>Proof Of Concept</ins>


<details>

```solidity
206: function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)
    external
    view
    returns (string memory)
  {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L206

```solidity
346: function tokenURI(uint256 tokenId) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L346

```solidity
17: function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)
    external
    pure
    returns (string memory)
  {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadata.sol#L17

```solidity
30: function tokenURI(uint256 id) public view virtual returns (string memory);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L30



</details>

#### <ins>Recommended Mitigation Steps</ins>

Add the following check:
```solidity
if(block.chainid != 1) { 
    revert(); 
}
```



### <a href="#QA Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Unbounded loop

New items are pushed into the following arrays but there is no option to `pop` them out. Currently, the array can grow indefinitely. E.g. there's no maximum limit and there's no functionality to remove array values.
If the array grows too large, calling relevant functions might run out of gas and revert. Calling these functions could result in a DOS condition.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
448: roleBalanceCkpts[tokenId][role].push(willHaveRole ? quantity : 0, expiration);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L448

```solidity
515: roleBalanceCkpts[tokenId][ALL_HOLDERS_ROLE].push(1);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L515

```solidity
529: roleBalanceCkpts[tokenId][ALL_HOLDERS_ROLE].push(0);

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L529

```solidity
143: self.push(Checkpoint({timestamp: timestamp, expiration: expiration, quantity: quantity}));
147: self.push(Checkpoint({timestamp: timestamp, expiration: expiration, quantity: quantity}));
143: self.push(Checkpoint({timestamp: timestamp, expiration: expiration, quantity: quantity}));
147: self.push(Checkpoint({timestamp: timestamp, expiration: expiration, quantity: quantity}));

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L143

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L147

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L143

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L147





</details>

#### <ins>Recommended Mitigation Steps</ins>
Add a functionality to delete array values or add a maximum size limit for arrays.



### <a href="#QA Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Condition will not revert when `block.timestamp` is `==` to the compared variable

The condition does not revert when block.timestamp is equal to the compared `>` or `<` variable. For example, if there is a check for `minExecutionTime < block.timestamp` then there should be a check for cases where `block.timestamp` is `==` to `minExecutionTime`
 
#### <ins>Proof Of Concept</ins>

<details>

```solidity
310: if (minExecutionTime < block.timestamp) revert MinExecutionTimeCannotBeInThePast();
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L310



</details>


### <a href="#QA Summary">[LOW&#x2011;9]</a><a name="LOW&#x2011;9"> Inconsistent documentation to actual function logic

It is mentioned in documentation of the function `validateActionCreation` that the param `actionInfo` is used. 

```solidity
  /// @notice Reverts if action creation is not allowed.
  /// @param actionInfo Data required to create an action.
  function validateActionCreation(ActionInfo calldata actionInfo) external;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaStrategy.sol#L33-L35

However, in `LlamaAbsoluteQuorum.sol` the param is commented out and is not used in the function.

```solidity
  function validateActionCreation(ActionInfo calldata /* actionInfo */ ) external view override {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L27

The same applies to `isApprovalEnabled` and `isDisapprovalEnabled`



### <a href="#QA Summary">[LOW&#x2011;10]</a><a name="LOW&#x2011;10"> Missing Reentrancy-Guard when using `sendValue` from OZ's `Address.sol`

OZ’s Address.sol library is used. Transfer of value is done with a `sendValue` call in the following functions.

There is this warning in OZ’s Address.sol library. Accordingly, he used the Check-Effect-Interaction pattern in the project:
```solidity
* IMPORTANT: because control is transferred to `recipient`, care must be
      * taken to not create reentrancy vulnerabilities. Consider using
      * {ReentrancyGuard} or the
      * https://solidity.readthedocs.io/en/v0.8.0/security-considerations.html#use-the-checks-effects-interactions-pattern[checks-effects-interactions pattern].
      */
```

It would be best practice to use re-entrancy Guard for reasons such as complicated dangers such as view Re-Entrancy that emerged in the last period and the possibility of expanding the project and its integration with other contracts.

#### <ins>Proof Of Concept</ins>


```solidity
  function transferNativeToken(NativeTokenData calldata nativeTokenData) public onlyLlama {
    if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();
    nativeTokenData.recipient.sendValue(nativeTokenData.amount);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L147-L150

### <a href="#QA Summary">[LOW&#x2011;11]</a><a name="LOW&#x2011;11"> `updateRoleDescription` doesn't do anything

This `updateRoleDescription` function doesn't do anything besides reverting when `role > numRoles` and emitting that a `RoleInitialized`, even though there is no logic in function at all

#### <ins>Proof Of Concept</ins>

```solidity
  function updateRoleDescription(uint8 role, RoleDescription description) external onlyLlama {
    if (role > numRoles) revert RoleNotInitialized(role);
    emit RoleInitialized(role, description);
  }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L236-L239


## Non Critical Issues






### <a href="#QA Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

<details>

```solidity
444: function setGuard(address target, bytes4 selector, ILlamaActionGuard guard) external onlyLlama {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L444

```solidity
197: function setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) external onlyRootLlama {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L197

```solidity
199: function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L199

```solidity
207: function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L207

```solidity
82: function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadataParamRegistry.sol#L82

```solidity
90: function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadataParamRegistry.sol#L90

```solidity
81: function setApprovalForAll(address operator, bool approved) public virtual {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L81

```solidity
183: function setRoleHolders(RoleHolderData[] calldata _setRoleHolders) public onlyDelegateCall {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L183

```solidity
196: function setRolePermissions(RolePermissionData[] calldata _setRolePermissions) public onlyDelegateCall {
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L196



</details>

#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.






### <a href="#QA Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Large or complicated code bases should implement fuzzing tests

Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement <a href="https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05">fuzzing tests</a>. Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

#### <ins>Proof Of Concept</ins>

Various in-scope contract files.



### <a href="#QA Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Initial value check is missing in Set Functions

A check regarding whether the current value and the new value are the same should be added

#### <ins>Proof Of Concept</ins>

<details>

```solidity
444: function setGuard(address target, bytes4 selector, ILlamaActionGuard guard) external onlyLlama {
    if (target == address(this) || target == address(policy)) revert RestrictedAddress();
    actionGuard[target][selector] = guard;
    emit ActionGuardSet(target, selector, guard);
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L444

```solidity
197: function setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) external onlyRootLlama {
    _setPolicyMetadata(_llamaPolicyMetadata);
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L197

```solidity
199: function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {
    _setRoleHolder(role, policyholder, quantity, expiration);
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L199

```solidity
207: function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {
    _setRolePermission(role, permissionId, hasPermission);
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L207

```solidity
386: function setApprovalForAll(address,  bool  ) public pure override nonTransferableToken {}
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L386

```solidity
82: function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {
    color[llamaExecutor] = _color;
    emit ColorSet(llamaExecutor, _color);
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadataParamRegistry.sol#L82

```solidity
90: function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {
    logo[llamaExecutor] = _logo;
    emit LogoSet(llamaExecutor, _logo);
  }
}
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicyMetadataParamRegistry.sol#L90

```solidity
183: function setRoleHolders(RoleHolderData[] calldata _setRoleHolders) public onlyDelegateCall {
    (, LlamaPolicy policy) = _context();
    uint256 length = _setRoleHolders.length;
    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
      policy.setRoleHolder(
        _setRoleHolders[i].role,
        _setRoleHolders[i].policyholder,
        _setRoleHolders[i].quantity,
        _setRoleHolders[i].expiration
      );
    }
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L183

```solidity
196: function setRolePermissions(RolePermissionData[] calldata _setRolePermissions) public onlyDelegateCall {
    (, LlamaPolicy policy) = _context();
    uint256 length = _setRolePermissions.length;
    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
      policy.setRolePermission(
        _setRolePermissions[i].role, _setRolePermissions[i].permissionId, _setRolePermissions[i].hasPermission
      );
    }
  }
```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaGovernanceScript.sol#L196



</details>





### <a href="#QA Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Use @inheritdoc rather than using a non-standard annotation

#### <ins>Proof Of Concept</ins>

<details>

```solidity
514: /// @dev Creates an action. The creator needs to hold a policy with the permission ID of the provided
  /// `(target, selector, strategy)`.
  function _createAction(
    address policyholder,
    uint8 role,
    ILlamaStrategy strategy,
    address target,
    uint256 value,
    bytes calldata data,
    string memory description
  ) internal returns (uint256 actionId) {
564: /// @dev How policyholders that have the right role contribute towards the approval of an action with a reason.
  function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)
    internal
  {
575: /// @dev How policyholders that have the right role contribute towards the disapproval of an action with a reason.
  function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)
    internal
  {
586: /// @dev The only `expectedState` values allowed to be passed into this method are Active or Queued.
  function _preCastAssertions(
    ActionInfo calldata actionInfo,
    address policyholder,
    uint8 role,
    ActionState expectedState
  ) internal returns (Action storage action, uint128 quantity) {
615: /// @dev Returns the new total count of approvals or disapprovals.
  function _newCastCount(uint128 currentCount, uint128 quantity) internal pure returns (uint128) {
621: /// @dev Deploys new strategies. Takes in the strategy logic contract to be used and an array of configurations to
  /// initialize the new strategies with. Returns the address of the first strategy, which is only used during the
  /// `LlamaCore` initialization so that we can ensure someone (specifically, policyholders with role ID 1) has the
  /// permission to assign role permissions.
  function _deployStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs)
    internal
    returns (ILlamaStrategy firstStrategy)
  {
646: /// @dev Deploys new accounts. Takes in the account logic contract to be used and an array of configurations to
  /// initialize the new accounts with.
  function _deployAccounts(ILlamaAccount llamaAccountLogic, bytes[] calldata accountConfigs) internal {
664: /// @dev Returns the hash of the `createAction` parameters using the `actionInfo` struct.
  function _infoHash(ActionInfo calldata actionInfo) internal pure returns (bytes32) {
677: /// @dev Returns the hash of the `createAction` parameters.
  function _infoHash(
    uint256 id,
    address creator,
    uint8 creatorRole,
    ILlamaStrategy strategy,
    address target,
    uint256 value,
    bytes calldata data
  ) internal pure returns (bytes32) {
690: /// @dev Validates that the hash of the `actionInfo` struct matches the provided hash.
  function _validateActionInfoHash(bytes32 actualHash, ActionInfo calldata actionInfo) internal pure {
696: /// @dev Returns the current nonce for a given policyholder and selector, and increments it. Used to prevent
  /// replay attacks.
  function _useNonce(address policyholder, bytes4 selector) internal returns (uint256 nonce) {
705: /// @dev Returns the EIP-712 domain separator.
  function _getDomainHash() internal view returns (bytes32) {
712: /// @dev Returns the hash of the ABI-encoded EIP-712 message for the `CreateAction` domain, which can be used to
  /// recover the signer.
  function _getCreateActionTypedDataHash(
    address policyholder,
    uint8 role,
    ILlamaStrategy strategy,
    address target,
    uint256 value,
    bytes calldata data,
    string memory description
  ) internal returns (bytes32) {
744: /// @dev Returns the hash of the ABI-encoded EIP-712 message for the `CastApproval` domain, which can be used to
  /// recover the signer.
  function _getCastApprovalTypedDataHash(
    address policyholder,
    uint8 role,
    ActionInfo calldata actionInfo,
    string calldata reason
  ) internal returns (bytes32) {
766: /// @dev Returns the hash of the ABI-encoded EIP-712 message for the `CastDisapproval` domain, which can be used to
  /// recover the signer.
  function _getCastDisapprovalTypedDataHash(
    address policyholder,
    uint8 role,
    ActionInfo calldata actionInfo,
    string calldata reason
  ) internal returns (bytes32) {
788: /// @dev Returns the hash of `actionInfo`.
  function _getActionInfoHash(ActionInfo calldata actionInfo) internal pure returns (bytes32) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L514

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L564

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L575

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L586

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L615

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L621

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L646

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L664

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L677

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L690

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L696

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L705

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L712

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L744

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L766

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaCore.sol#L788



```solidity
226: /// @dev Deploys a new Llama instance.
  function _deploy(
    string memory name,
    ILlamaStrategy strategyLogic,
    ILlamaAccount accountLogic,
    bytes[] memory initialStrategies,
    bytes[] memory initialAccounts,
    RoleDescription[] memory initialRoleDescriptions,
    RoleHolderData[] memory initialRoleHolders,
    RolePermissionData[] memory initialRolePermissions
  ) internal returns (LlamaExecutor llamaExecutor, LlamaCore llamaCore) {
266: /// @dev Authorizes a strategy implementation (logic) contract.
  function _authorizeStrategyLogic(ILlamaStrategy strategyLogic) internal {
272: /// @dev Authorizes an account implementation (logic) contract.
  function _authorizeAccountLogic(ILlamaAccount accountLogic) internal {
278: /// @dev Sets the Llama policy metadata contract.
  function _setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) internal {
284: /// @dev Sets the `color` and `logo` of a Llama instance.
  function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L226

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L266

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L272

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L278

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaFactory.sol#L284



```solidity
73: /// @dev Ensures that none of the ERC721 `transfer` and `approval` functions can be called, so that the policies are
  /// soulbound.
  modifier nonTransferableToken() {
358: /// @dev overriding `transferFrom` to disable transfers
  function transferFrom(address, /* from */ address, /* to */ uint256 /* policyId */ )
    public
    pure
    override
    nonTransferableToken
  {
366: /// @dev overriding `safeTransferFrom` to disable transfers
  function safeTransferFrom(address, /* from */ address, /* to */ uint256 /* id */ )
    public
    pure
    override
    nonTransferableToken
  {
374: /// @dev overriding `safeTransferFrom` to disable transfers
  function safeTransferFrom(address, /* from */ address, /* to */ uint256, /* policyId */ bytes calldata /* data */ )
    public
    pure
    override
    nonTransferableToken
  {
382: /// @dev overriding `approve` to disable approvals
  function approve(address, /* spender */ uint256 /* id */ ) public pure override nonTransferableToken {
385: /// @dev overriding `approve` to disable approvals
  function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {
392: /// @dev Initializes the next unassigned role with the given `description`.
  function _initializeRole(RoleDescription description) internal {
398: /// @dev Because role supplies are not checkpointed for simplicity, the following issue can occur
  /// if each of the below is executed within the same timestamp:
  //    1. An action is created that saves off the current role supply.
  //    2. A policyholder is given a new role.
  //    3. Now the total supply in that block is different than what it was at action creation.
  // As a result, we disallow changes to roles if an action was created in the same block.
  function _assertNoActionCreationsAtCurrentTimestamp() internal view {
411: /// @dev Checks if the conditions are met for a `role` to be updated.
  function _assertValidRoleHolderUpdate(uint8 role, uint128 quantity, uint64 expiration) internal view {
430: /// @dev Sets the `role` for the given `policyholder` to the given `quantity` and `expiration`.
  function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal {
489: /// @dev Sets a role's permission along with whether that permission is valid or not.
  function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {
496: /// @dev Revokes a policyholder's expired `role`.
  function _revokeExpiredRole(uint8 role, address policyholder) internal {
503: /// @dev Mints a policyholder's policy.
  function _mint(address policyholder) internal {
518: /// @dev Burns a policyholder's policy.
  function _burn(uint256 tokenId) internal override {
532: /// @dev Returns the token ID for a `policyholder`.
  function _tokenId(address policyholder) internal pure returns (uint256) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L73

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L358

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L366

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L374

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L382

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L385

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L392

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L398

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L411

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L430

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L489

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L496

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L503

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L518

https://github.com/code-423n4/2023-06-llama/tree/main/src/LlamaPolicy.sol#L532



```solidity
337: /// @dev Reads slot 0 from storage, used to check that storage hasn't changed after delegatecall.
  function _readSlot0() internal view returns (bytes32 slot0) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/accounts/LlamaAccount.sol#L337

```solidity
7: /**
 * @dev This library defines the `History` struct, for checkpointing values as they change at different points in
 * time, and later looking up past values by block timestamp.
 *
 * To create a history of checkpoints define a variable type `Checkpoints.History` in your contract, and store a new
 * checkpoint for the current transaction timestamp using the {push} function.
 *
 * @dev This was created by modifying then running the OpenZeppelin `Checkpoints.js` script, which generated a version
 * of this library that uses a 64 bit `timestamp` and 128 bit `quantity` field in the `Checkpoint` struct. The struct
 * was then modified to add a 64 bit `expiration` field. For simplicity, safe cast and math methods were inlined from
 * the OpenZeppelin versions at the same commit. We disable forge-fmt for this file to simplify diffing against the
 * original OpenZeppelin version: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d00acef4059807535af0bd0dd0ddf619747a044b/contracts/utils/Checkpoints.sol
 */
library Checkpoints {
31: /**
     * @dev Returns the quantity at a given block timestamp. If a checkpoint is not available at that time, the closest
     * one before it is returned, or zero otherwise. Similar to {upperLookup} but optimized for the case when the
     * searched checkpoint is probably "recent", defined as being among the last sqrt(N) checkpoints where N is the
     * timestamp of checkpoints.
     */
    function getAtProbablyRecentTimestamp(History storage self, uint256 timestamp) internal view returns (uint128) {
60: /**
     * @dev Pushes a `quantity` and `expiration` onto a History so that it is stored as the checkpoint for the current
     * `timestamp`.
     *
     * Returns previous quantity and new quantity.
     */
    function push(History storage self, uint256 quantity, uint256 expiration) internal returns (uint128, uint128) {
70: /**
     * @dev Pushes a `quantity` with no expiration onto a History so that it is stored as the checkpoint for the current
     * `timestamp`.
     *
     * Returns previous quantity and new quantity.
     */
    function push(History storage self, uint256 quantity) internal returns (uint128, uint128) {
80: /**
     * @dev Returns the quantity in the most recent checkpoint, or zero if there are no checkpoints.
     */
    function latest(History storage self) internal view returns (uint128) {
88: /**
     * @dev Returns whether there is a checkpoint in the structure (i.e. it is not empty), and if so the timestamp and
     * quantity in the most recent checkpoint.
     */
    function latestCheckpoint(History storage self)
        internal
        view
        returns (
            bool exists,
            uint64 timestamp,
            uint64 expiration,
            uint128 quantity
        )
    {
111: /**
     * @dev Returns the number of checkpoints.
     */
    function length(History storage self) internal view returns (uint256) {
118: /**
     * @dev Pushes a (`timestamp`, `expiration`, `quantity`) pair into an ordered list of checkpoints, either by inserting a new
     * checkpoint, or by updating the last one.
     */
    function _insert(
        Checkpoint[] storage self,
        uint64 timestamp,
        uint64 expiration,
        uint128 quantity
    ) private returns (uint128, uint128) {
152: /**
     * @dev Return the index of the oldest checkpoint whose timestamp is greater than the search timestamp, or `high`
     * if there is none. `low` and `high` define a section where to do the search, with inclusive `low` and exclusive
     * `high`.
     *
     * WARNING: `high` should not be greater than the array's length.
     */
    function _upperBinaryLookup(
        Checkpoint[] storage self,
        uint64 timestamp,
        uint256 low,
        uint256 high
    ) private view returns (uint256) {
176: /**
     * @dev Return the index of the oldest checkpoint whose timestamp is greater or equal than the search timestamp, or
     * `high` if there is none. `low` and `high` define a section where to do the search, with inclusive `low` and
     * exclusive `high`.
     *
     * WARNING: `high` should not be greater than the array's length.
     */
    function _lowerBinaryLookup(
        Checkpoint[] storage self,
        uint64 timestamp,
        uint256 low,
        uint256 high
    ) private view returns (uint256) {
211: /**
     * @dev Returns the average of two numbers. The result is rounded towards
     * zero.
     */
    function average(uint256 a, uint256 b) private pure returns (uint256) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L7

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L31

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L60

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L70

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L80

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L88

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L111

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L118

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L152

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L176

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Checkpoints.sol#L211



```solidity
4: /// @dev Shared helper methods for Llama's contracts.
library LlamaUtils {
  /// @dev Thrown when a value cannot be safely casted to a smaller type.
  error UnsafeCast(uint256 n);

  /// @dev Reverts if `n` does not fit in a `uint64`.
  function toUint64(uint256 n) internal pure returns (uint64) {
15: /// @dev Reverts if `n` does not fit in a `uint128`.
  function toUint128(uint256 n) internal pure returns (uint128) {
21: /// @dev Increments a `uint256` without checking for overflow.
  function uncheckedIncrement(uint256 i) internal pure returns (uint256) {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/LlamaUtils.sol#L4

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/LlamaUtils.sol#L15

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/LlamaUtils.sol#L21



```solidity
6: /// @dev Data required to create an action.
struct ActionInfo {
  uint256 id; // ID of the action.
  address creator; // Address that created the action.
  uint8 creatorRole; // The role that created the action.
  ILlamaStrategy strategy; // Strategy used to govern the action.
  address target; // Contract being called by an action.
  uint256 value; // Value in wei to be sent when the action is executed.
  bytes data; // Data to be called on the target when the action is executed.
}

/// @dev Data that represents an action.
struct Action {
  // Instead of storing all data required to execute an action in storage, we only save the hash to
  // make action creation cheaper. The hash is computed by taking the keccak256 hash of the concatenation of each
  // field in the `ActionInfo` struct.
  bytes32 infoHash;
  bool executed; // Has action executed.
  bool canceled; // Is action canceled.
  bool isScript; // Is the action's target a script.
  uint64 creationTime; // The timestamp when action was created (used for policy snapshots).
  uint64 minExecutionTime; // Only set when an action is queued. The timestamp when action execution can begin.
  uint128 totalApprovals; // The total quantity of policyholder approvals.
  uint128 totalDisapprovals; // The total quantity of policyholder disapprovals.
}

/// @dev Data that represents a permission.
struct PermissionData {
  address target; // Contract being called by an action.
  bytes4 selector; // Selector of the function being called by an action.
  ILlamaStrategy strategy; // Strategy used to govern the action.
}

/// @dev Data required to assign/revoke a role to/from a policyholder.
struct RoleHolderData {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/lib/Structs.sol#L6

```solidity
4: /// @dev This script is a template for creating new scripts, and should not be used directly.
abstract contract LlamaBaseScript {
  /// @dev Thrown if you try to CALL a function that has the `onlyDelegatecall` modifier.
  error OnlyDelegateCall();

  /// @dev Add this to your script's methods to ensure the script can only be used via delegatecall, and not a regular
  /// call.
  modifier onlyDelegateCall() {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/llama-scripts/LlamaBaseScript.sol#L4

```solidity
303: /// @dev Reverts if the given `role` is greater than `numRoles`.
  function _assertValidRole(uint8 role, uint8 numRoles) internal pure {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L303

```solidity
323: /// @dev Reverts if the given `role` is greater than `numRoles`.
  function _assertValidRole(uint8 role, uint8 numRoles) internal pure {

```

https://github.com/code-423n4/2023-06-llama/tree/main/src/strategies/LlamaRelativeQuorum.sol#L323



</details>


### <a href="#QA Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Function name should contain `InitializeRoles` instead of `NewRoles`

The function `createNewStrategiesAndNewRolesAndSetRoleHoldersAndSetRolePermissions` should be `createNewStrategiesAndInitializeRolesAndSetRoleHoldersAndSetRolePermissions` as it calls `initializeRoles(description);`.
Similar to the function `createNewStrategiesAndInitializeRolesAndSetRoleHolders`

#### <ins>Proof Of Concept</ins>

```solidity
  function createNewStrategiesAndInitializeRolesAndSetRoleHolders(
    CreateStrategies calldata _createStrategies,
    RoleDescription[] calldata description,
    RoleHolderData[] calldata _setRoleHolders
  ) external onlyDelegateCall {
    (LlamaCore core,) = _context();
    core.createStrategies(_createStrategies.llamaStrategyLogic, _createStrategies.strategies);
    initializeRoles(description);
    setRoleHolders(_setRoleHolders);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L120-L129

```solidity
  function createNewStrategiesAndNewRolesAndSetRoleHoldersAndSetRolePermissions(
    CreateStrategies calldata _createStrategies,
    RoleDescription[] calldata description,
    RoleHolderData[] calldata _setRoleHolders,
    RolePermissionData[] calldata _setRolePermissions
  ) external onlyDelegateCall {
    (LlamaCore core,) = _context();
    core.createStrategies(_createStrategies.llamaStrategyLogic, _createStrategies.strategies);
    initializeRoles(description);
    setRoleHolders(_setRoleHolders);
    setRolePermissions(_setRolePermissions);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L140-L151