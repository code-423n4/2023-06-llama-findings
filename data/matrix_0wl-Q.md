## Non Critical Issues

|       | Issue                                                                   |
| ----- | :---------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                    |
| NC-2  | ADD TO BLACKLIST FUNCTION                                               |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                |
| NC-4  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                         |
| NC-5  | INCONSISTENT SOLIDITY VERSIONS                                          |
| NC-6  | SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`                           |
| NC-7  | IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED                          |
| NC-8  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS            |
| NC-9  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION           |
| NC-10 | NO SAME VALUE INPUT CONTROL                                             |
| NC-11 | USE A MORE RECENT VERSION OF SOLIDITY                                   |
| NC-12 | Return values of `approve()` not checked                                |
| NC-13 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: src/LlamaPolicy.sol

199:   function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {

207:   function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {

386:   function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}

431:   function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal {

490:   function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function to src/LlamaPolicy.sol.

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

#### **Proof Of Concept**

```solidity
File: src/LlamaFactory.sol

68:   uint8 public constant BOOTSTRAP_ROLE = 1;

```

```solidity
File: src/LlamaPolicy.sol

111:   uint8 public constant BOOTSTRAP_ROLE = 1;

```

### [NC-5] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaExecutor.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaFactory.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadata.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/accounts/LlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaActionGuard.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Checkpoints.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

3: pragma solidity >=0.8.0;

```

```solidity
File: src/lib/Enums.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/LlamaUtils.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Structs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/UDVTs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/llama-scripts/LlamaBaseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteQuorum.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

2: pragma solidity 0.8.19;

```

### [NC-6] SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`

#### Description:

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

297:     address signer = ecrecover(digest, v, r, s);

388:     address signer = ecrecover(digest, v, r, s);

421:     address signer = ecrecover(digest, v, r, s);

```

#### Recommended Mitigation Steps:

Use the `ecrecover` function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

### [NC-7] IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED

#### Description:

OpenZeppelin recommends that the initializer modifier be applied to constructors.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

212:   constructor() {
    _disableInitializers();
  }

```

```solidity
File: src/accounts/LlamaAccount.sol

124:   constructor() {
    _disableInitializers();
  }

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

62:   function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {
    name = _name;
    symbol = _symbol;
  }

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

142:   constructor() {
    _disableInitializers();
  }

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

145:   constructor() {
    _disableInitializers();
  }

```

### [NC-8] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions:

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

224:   function initialize(

```

```solidity
File: src/LlamaPolicy.sol

143:   function initialize(

```

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

```

```solidity
File: src/interfaces/ILlamaAccount.sol

18:   function initialize(bytes memory config) external;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

29:   function initialize(bytes memory config) external;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

153:   function initialize(bytes memory config) external virtual initializer {

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

```

### [NC-9] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaExecutor.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaFactory.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadata.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/accounts/LlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaActionGuard.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Checkpoints.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

3: pragma solidity >=0.8.0;

```

```solidity
File: src/lib/Enums.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/LlamaUtils.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Structs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/UDVTs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/llama-scripts/LlamaBaseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteQuorum.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

2: pragma solidity 0.8.19;

```

### [NC-10] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

233:     name = _name;

235:     policy = _policy;

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

63:     name = _name;

64:     symbol = _symbol;

```

### [NC-11] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaExecutor.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaFactory.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadata.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/accounts/LlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaActionGuard.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Checkpoints.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

3: pragma solidity >=0.8.0;

```

```solidity
File: src/lib/Enums.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/LlamaUtils.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Structs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/UDVTs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/llama-scripts/LlamaBaseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteQuorum.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

2: pragma solidity 0.8.19;

```

### [NC-12] Return values of `approve()` not checked

#### Description:

Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

503:     if (actionInfo.strategy.isActionDisapproved(actionInfo)) return ActionState.Failed;

```

```solidity
File: src/accounts/LlamaAccount.sol

215:     erc721Data.token.approve(erc721Data.recipient, erc721Data.tokenId);

```

### [NC-13] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

142:   bytes32 internal constant EIP712_DOMAIN_TYPEHASH =

146:   bytes32 internal constant CREATE_ACTION_TYPEHASH = keccak256(

151:   bytes32 internal constant CAST_APPROVAL_TYPEHASH = keccak256(

156:   bytes32 internal constant CAST_DISAPPROVAL_TYPEHASH = keccak256(

161:   bytes32 internal constant ACTION_INFO_TYPEHASH = keccak256(

```

```solidity
File: src/lib/Checkpoints.sol

37:     function getAtProbablyRecentTimestamp(History storage self, uint256 timestamp) internal view returns (uint128) {

66:     function push(History storage self, uint256 quantity, uint256 expiration) internal returns (uint128, uint128) {

76:     function push(History storage self, uint256 quantity) internal returns (uint128, uint128) {

83:     function latest(History storage self) internal view returns (uint128) {

114:     function length(History storage self) internal view returns (uint256) {

215:     function average(uint256 a, uint256 b) private pure returns (uint256) {

223:     function sqrt(uint256 x) internal pure returns (uint256 z) {

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

62:   function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {

```

```solidity
File: src/lib/LlamaUtils.sol

10:   function toUint64(uint256 n) internal pure returns (uint64) {

16:   function toUint128(uint256 n) internal pure returns (uint128) {

22:   function uncheckedIncrement(uint256 i) internal pure returns (uint256) {

```

```solidity
File: src/llama-scripts/LlamaBaseScript.sol

18:   address internal immutable SELF;

```

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

22:   LlamaExecutor internal immutable EXECUTOR;

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

99:   uint256 internal constant ONE_HUNDRED_IN_BPS = 10_000;

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | Arbitrary Jump with Function Type Variable                                     |
| L-2  | Initializing state-variables in proxy-based upgradeable contracts              |
| L-3  | Avoid using low call function ecrecover                                        |
| L-4  | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                                 |
| L-5  | The critical parameters in initialize(...) are not set safely                  |
| L-6  | DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH       |
| L-7  | Do not use deprecated library functions                                        |
| L-8  | DON’T USE OWNER AND TIMELOCK                                                   |
| L-9  | `ECRECOVER` MAY RETURN EMPTY ADDRESS                                           |
| L-10 | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY                                 |
| L-11 | MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS     |
| L-12 | FUNCTION CREATES DIRTY BITS                                                    |
| L-13 | Unify return criteria                                                          |
| L-14 | Incorrect return values for ERC721 `ownerOf()`                                 |
| L-15 | Revert if ecrecover is address(0)                                              |
| L-16 | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-17 | Timestamp Dependence                                                           |
| L-18 | Account existence check for low-level calls                                    |
| L-19 | Unsafe ERC20 operation(s)                                                      |
| L-20 | UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT                        |
| L-21 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                             |

### [L-1] Arbitrary Jump with Function Type Variable

#### Description:

Function types are supported in Solidity. This means that a variable of type function can be assigned to a function with a matching signature. The function can then be called from the variable just like any other function. Users should not be able to change the function variable, but in some cases this is possible.

If the smart contract uses certain assembly instructions, mstore for example, an attacker may be able to point the function variable to any other function. This may give the attacker the ability to break the functionality of the contract, and perhaps even drain the contract funds.

Since inline assembly is a way to access the EVM at a low level, it bypasses many important safety features. So it's important to only use assembly if it is necessary and properly understood.

[Source](https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/arbitrary-jump-function-type.md)

[SWC Registry](https://swcregistry.io/docs/SWC-127)

#### **Proof Of Concept**

```solidity
File: src/accounts/LlamaAccount.sol

339:     assembly {
       slot0 := sload(0)
    }

```

```solidity
File: src/lib/Checkpoints.sol

205:         assembly {
              mstore(0, self.slot)
            result.slot := add(keccak256(0, 0x20), pos)
        }


```

### [L-2] Initializing state-variables in proxy-based upgradeable contracts

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

224:   function initialize(

```

```solidity
File: src/LlamaPolicy.sol

143:   function initialize(

```

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

```

```solidity
File: src/interfaces/ILlamaAccount.sol

18:   function initialize(bytes memory config) external;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

29:   function initialize(bytes memory config) external;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

153:   function initialize(bytes memory config) external virtual initializer {

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

```

### [L-3] Avoid using low call function ecrecover

#### Description:

Use OZ library ECDSA that its battle tested to avoid classic errors.

https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

297:     address signer = ecrecover(digest, v, r, s);

388:     address signer = ecrecover(digest, v, r, s);

421:     address signer = ecrecover(digest, v, r, s);

```

### [L-4] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

444:   function setGuard(address target, bytes4 selector, ILlamaActionGuard guard) external onlyLlama {

```

```solidity
File: src/LlamaFactory.sol

197:   function setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) external onlyRootLlama {

279:   function _setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) internal {

285:   function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {

```

```solidity
File: src/LlamaPolicy.sol

199:   function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {

207:   function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {

386:   function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}

431:   function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal {

490:   function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

82:   function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {

90:   function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

81:   function setApprovalForAll(address operator, bool approved) public virtual {

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

183:   function setRoleHolders(RoleHolderData[] calldata _setRoleHolders) public onlyDelegateCall {

196:   function setRolePermissions(RolePermissionData[] calldata _setRolePermissions) public onlyDelegateCall {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-5] The critical parameters in initialize(...) are not set safely

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

224:   function initialize(

```

```solidity
File: src/LlamaPolicy.sol

143:   function initialize(

```

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

```

```solidity
File: src/interfaces/ILlamaAccount.sol

18:   function initialize(bytes memory config) external;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

29:   function initialize(bytes memory config) external;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

153:   function initialize(bytes memory config) external virtual initializer {

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

```

### [L-6] DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH

#### Description:

Although SHA256 is 32 bytes and is currently the most common IPFS hash function, other content could use a hash function that is larger than 32 bytes. The current implementation limits the usage and a hash length of 32 bytes.

#### **Proof Of Concept**

```solidity
File: src/lib/Checkpoints.sol

207:             result.slot := add(keccak256(0, 0x20), pos)

234:             if iszero(lt(y, 0x10000000000000000000000000000000000)) {

238:             if iszero(lt(y, 0x1000000000000000000)) {

242:             if iszero(lt(y, 0x10000000000)) {

246:             if iszero(lt(y, 0x1000000)) {

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

118:     return interfaceId == 0x01ffc9a7 // ERC165 Interface ID for ERC165

119:       || interfaceId == 0x80ac58cd // ERC165 Interface ID for ERC721

120:       || interfaceId == 0x5b5e139f; // ERC165 Interface ID for ERC721Metadata

```

#### Recommended Mitigation Steps:

Consider using a more generic implementation that can handle different hash functions and lengths and allow the user to choose.

### [L-7] Do not use deprecated library functions

#### Description:

You use safeApprove of openZeppelin although it’s deprecated. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38%3E)).

You should change it to increase/decrease Allowance as OpenZeppilin says.

#### **Proof Of Concept**

```solidity
File: src/accounts/LlamaAccount.sol

182:     erc20Data.token.safeApprove(erc20Data.recipient, erc20Data.amount);

```

### [L-8] DON’T USE OWNER AND TIMELOCK

#### Description:

The owner manipulates the contract without a lock time period.

#### **Proof Of Concept**

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

74:     require(msg.sender == owner || isApprovedForAll[owner][msg.sender], "NOT_AUTHORIZED");

```

### [L-9] `ECRECOVER` MAY RETURN EMPTY ADDRESS

#### Description:

There is a common issue that ecrecover returns empty (0x0) address when the signature is invalid. Function `signer` should check that before returning the result of ecrecover.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

297:     address signer = ecrecover(digest, v, r, s);

388:     address signer = ecrecover(digest, v, r, s);

421:     address signer = ecrecover(digest, v, r, s);

```

#### Recommended Mitigation Steps:

See the solution here: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0/contracts/cryptography/ECDSA.sol#L68

### [L-10] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

231:   ) external initializer returns (bytes32 bootstrapPermissionId) {

```

```solidity
File: src/LlamaPolicy.sol

148:   ) external initializer {

```

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

```

### [L-11] MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS

#### Description:

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

Missing call to `__ReentrancyGuard_init`

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

224:   function initialize(

```

```solidity
File: src/LlamaFactory.sol

251:     policy.initialize(name, initialRoleDescriptions, initialRoleHolders, initialRolePermissions);

```

```solidity
File: src/LlamaPolicy.sol

143:   function initialize(

```

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

```

```solidity
File: src/interfaces/ILlamaAccount.sol

18:   function initialize(bytes memory config) external;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

29:   function initialize(bytes memory config) external;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

153:   function initialize(bytes memory config) external virtual initializer {

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

```

#### Recommended Mitigation Steps:

Add missing calls to init functions of inherited contracts.

### [L-12] FUNCTION CREATES DIRTY BITS

#### Description:

This explanation should be added in the NatSpec comments of this function that sends ether with call;

Note that this code probably isn’t secure or a good use case for assembly because a lot of memory management and security checks are bypassed. Use with caution! Some functions in this contract knowingly create dirty bits at the destination of the free memory pointer.

#### **Proof Of Concept**

```solidity
File: src/lib/Checkpoints.sol

224:         assembly {
   let y := x // We start y at x, which will help us make our initial estimate.

            z := 181 // The "correct" value is 1, but this saves a multiplication later.

            // This segment is to get a reasonable initial estimate for the Babylonian method. With a bad
            // start, the correct # of bits increases ~linearly each iteration instead of ~quadratically.

            // We check y >= 2^(k + 8) but shift right by k bits
            // each branch to ensure that if x >= 256, then y >= 256.
            if iszero(lt(y, 0x10000000000000000000000000000000000)) {
                y := shr(128, y)
                z := shl(64, z)
            }
            if iszero(lt(y, 0x1000000000000000000)) {
                y := shr(64, y)
                z := shl(32, z)
            }
            if iszero(lt(y, 0x10000000000)) {
                y := shr(32, y)
                z := shl(16, z)
            }
            if iszero(lt(y, 0x1000000)) {
                y := shr(16, y)
                z := shl(8, z)
            }

            // Goal was to get z*z*y within a small factor of x. More iterations could
            // get y in a tighter range. Currently, we will have y in [256, 256*2^16).
            // We ensured y >= 256 so that the relative difference between y and y+1 is small.
            // That's not possible if x < 256 but we can just verify those cases exhaustively.

            // Now, z*z*y <= x < z*z*(y+1), and y <= 2^(16+8), and either y >= 256, or x < 256.
            // Correctness can be checked exhaustively for x < 256, so we assume y >= 256.
            // Then z*sqrt(y) is within sqrt(257)/sqrt(256) of sqrt(x), or about 20bps.

            // For s in the range [1/256, 256], the estimate f(s) = (181/1024) * (s+1) is in the range
            // (1/2.84 * sqrt(s), 2.84 * sqrt(s)), with largest error when s = 1 and when s = 256 or 1/256.

            // Since y is in [256, 256*2^16), let a = y/65536, so that a is in [1/256, 256). Then we can estimate
            // sqrt(y) using sqrt(65536) * 181/1024 * (a + 1) = 181/4 * (y + 65536)/65536 = 181 * (y + 65536)/2^18.

            // There is no overflow risk here since y < 2^136 after the first branch above.
            z := shr(18, mul(z, add(y, 65536))) // A mul() is saved from starting z at 181.

            // Given the worst case multiplicative error of 2.84 above, 7 iterations should be enough.
            z := shr(1, add(z, div(x, z)))
            z := shr(1, add(z, div(x, z)))
            z := shr(1, add(z, div(x, z)))
            z := shr(1, add(z, div(x, z)))
            z := shr(1, add(z, div(x, z)))
            z := shr(1, add(z, div(x, z)))
            z := shr(1, add(z, div(x, z)))

            // If x+1 is a perfect square, the Babylonian method cycles between
            // floor(sqrt(x)) and ceil(sqrt(x)). This statement ensures we return floor.
            // See: https://en.wikipedia.org/wiki/Integer_square_root#Using_only_integer_division
            // Since the ceil is rare, we save gas on the assignment and repeat division in the rare case.
            // If you don't care whether the floor or ceil square root is returned, you can remove this statement.
            z := sub(z, lt(div(x, z), z))
        }

```

#### Recommended Mitigation Steps:

Add this comment to \_returnDust function:

`/// @dev Use with caution! Some functions in this contract knowingly create dirty bits at the destination of the free memory pointer. Note that this code probably isn’t secure or a good use case for assembly because a lot of memory management and security checks are bypassed.`

### [L-13] Unify return criteria

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

231:   ) external initializer returns (bytes32 bootstrapPermissionId) {

266:   ) external returns (uint256 actionId) {

295:   ) external returns (uint256 actionId) {

471:   function getAction(uint256 actionId) external view returns (Action memory) {

478:   function getLastActionTimestamp() external view returns (uint256 timestamp) {

485:   function getActionState(ActionInfo calldata actionInfo) public view returns (ActionState) {

524:   ) internal returns (uint256 actionId) {

592:   ) internal returns (Action storage action, uint128 quantity) {

616:   function _newCastCount(uint128 currentCount, uint128 quantity) internal pure returns (uint128) {

627:     returns (ILlamaStrategy firstStrategy)

665:   function _infoHash(ActionInfo calldata actionInfo) internal pure returns (bytes32) {

686:   ) internal pure returns (bytes32) {

698:   function _useNonce(address policyholder, bytes4 selector) internal returns (uint256 nonce) {

706:   function _getDomainHash() internal view returns (bytes32) {

722:   ) internal returns (bytes32) {

751:   ) internal returns (bytes32) {

773:   ) internal returns (bytes32) {

789:   function _getActionInfoHash(ActionInfo calldata actionInfo) internal pure returns (bytes32) {

```

```solidity
File: src/LlamaExecutor.sol

31:     returns (bool success, bytes memory result)

```

```solidity
File: src/LlamaPolicy.sol

245:   function getQuantity(address policyholder, uint8 role) external view returns (uint128) {

252:   function getPastQuantity(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {

258:   function getRoleSupplyAsNumberOfHolders(uint8 role) public view returns (uint128) {

263:   function getRoleSupplyAsQuantitySum(uint8 role) public view returns (uint128) {

268:   function roleBalanceCheckpoints(address policyholder, uint8 role) external view returns (Checkpoints.History memory) {

282:     returns (Checkpoints.History memory)

299:   function roleBalanceCheckpointsLength(address policyholder, uint8 role) external view returns (uint256) {

305:   function hasRole(address policyholder, uint8 role) public view returns (bool) {

311:   function hasRole(address policyholder, uint8 role, uint256 timestamp) external view returns (bool) {

318:   function hasPermissionId(address policyholder, uint8 role, bytes32 permissionId) external view returns (bool) {

324:   function isRoleExpired(address policyholder, uint8 role) public view returns (bool) {

330:   function roleExpiration(address policyholder, uint8 role) external view returns (uint64) {

337:   function totalSupply() public view returns (uint256) {

346:   function tokenURI(uint256 tokenId) public view override returns (string memory) {

352:   function contractURI() public view returns (string memory) {

533:   function _tokenId(address policyholder) internal pure returns (uint256) {

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

19:   function llamaCore() external view returns (LlamaCore);

22:   function policy() external view returns (LlamaPolicy);

51:   function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128);

70:     returns (uint128);

77:   function minExecutionTime(ActionInfo calldata actionInfo) external view returns (uint64);

92:   function isActive(ActionInfo calldata actionInfo) external view returns (bool);

97:   function isActionApproved(ActionInfo calldata actionInfo) external view returns (bool);

102:   function isActionDisapproved(ActionInfo calldata actionInfo) external view returns (bool);

107:   function isActionExpired(ActionInfo calldata actionInfo) external view returns (bool);

```

```solidity
File: src/lib/Checkpoints.sol

37:     function getAtProbablyRecentTimestamp(History storage self, uint256 timestamp) internal view returns (uint128) {

66:     function push(History storage self, uint256 quantity, uint256 expiration) internal returns (uint128, uint128) {

76:     function push(History storage self, uint256 quantity) internal returns (uint128, uint128) {

83:     function latest(History storage self) internal view returns (uint128) {

114:     function length(History storage self) internal view returns (uint256) {

127:     ) private returns (uint128, uint128) {

164:     ) private view returns (uint256) {

188:     ) private view returns (uint256) {

203:         returns (Checkpoint storage result)

215:     function average(uint256 a, uint256 b) private pure returns (uint256) {

223:     function sqrt(uint256 x) internal pure returns (uint256 z) {

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

30:   function tokenURI(uint256 id) public view virtual returns (string memory);

40:   function ownerOf(uint256 id) public view virtual returns (address owner) {

44:   function balanceOf(address owner) public view virtual returns (uint256) {

117:   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {

189:   function onERC721Received(address, address, uint256, bytes calldata) external virtual returns (bytes4) {

```

### [L-14] Incorrect return values for ERC721 `ownerOf()`

#### Description:

Incorrect return values for ERC721 ownerOf(): Contracts compiled with solc >= 0.4.22 interacting with ERC721 ownerOf() that returns a bool instead of address type will revert. Use OpenZeppelin’s ERC721 contracts.

https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-erc721-interface

#### **Proof Of Concept**

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

40:   function ownerOf(uint256 id) public view virtual returns (address owner) {

```

### [L-15] Revert if ecrecover is address(0)

#### Description:

Add a revert that triggers if the response is address(0), this means that signature its not valid.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

297:     address signer = ecrecover(digest, v, r, s);

388:     address signer = ecrecover(digest, v, r, s);

421:     address signer = ecrecover(digest, v, r, s);

```

### [L-16] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: src/accounts/LlamaAccount.sol

167:     erc20Data.token.safeTransfer(erc20Data.recipient, erc20Data.amount);

```

### [L-17] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

310:     if (minExecutionTime < block.timestamp) revert MinExecutionTimeCannotBeInThePast();

323:     if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();

555:       newAction.creationTime = LlamaUtils.toUint64(block.timestamp);

```

```solidity
File: src/LlamaPolicy.sol

326:     return quantity > 0 && block.timestamp > expiration;

408:     if (lastActionCreation == block.timestamp) revert ActionCreationAtSameTimestamp();

425:     bool case1 = quantity > 0 && expiration > block.timestamp;

```

```solidity
File: src/lib/Checkpoints.sol

38:         require(timestamp < block.timestamp, "Checkpoints: timestamp is not in the past");

67:         return _insert(self._checkpoints, LlamaUtils.toUint64(block.timestamp), LlamaUtils.toUint64(expiration), LlamaUtils.toUint128(quantity));

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

239:     return LlamaUtils.toUint64(block.timestamp + queuingPeriod);

268:       block.timestamp <= approvalEndTime(actionInfo) && (isFixedLengthApprovalPeriod || !isActionApproved(actionInfo));

286:     return block.timestamp > action.minExecutionTime + expirationPeriod;

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

249:     return LlamaUtils.toUint64(block.timestamp + queuingPeriod);

278:       block.timestamp <= approvalEndTime(actionInfo) && (isFixedLengthApprovalPeriod || !isActionApproved(actionInfo));

297:     return block.timestamp > action.minExecutionTime + expirationPeriod;

```

### [L-18] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: src/LlamaExecutor.sol

34:     (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);

```

```solidity
File: src/accounts/LlamaAccount.sol

323:       (success, result) = target.delegatecall(callData);

326:       (success, result) = target.call{value: msg.value}(callData);

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

75:       (bool success, bytes memory response) = targets[i].call(data[i]);

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-19] Unsafe ERC20 operation(s)

#### Description:

There are ERC20 tokens that charge fee for every transfer() / transferFrom().

Transaction order dependence: Race conditions can be forced on specific Ethereum transactions by monitoring the mempool. For example, the classic ERC20 approve() change can be front-run using this method. Do not make assumptions about transaction order dependence.

ERC20 approve() race condition: Use safeIncreaseAllowance() and safeDecreaseAllowance() from OpenZeppelin’s SafeERC20 implementation to prevent race conditions from manipulating the allowance amounts.

https://swcregistry.io/docs/SWC-114

#### **Proof Of Concept**

```solidity
File: src/accounts/LlamaAccount.sol

200:     erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);

215:     erc721Data.token.approve(erc721Data.recipient, erc721Data.tokenId);

```

### [L-20] UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT

#### Description:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

#### **Proof Of Concept**

```solidity
File: src/accounts/LlamaAccount.sol

143:   receive() external payable {}

```

#### Recommended Mitigation Steps:

The function should call another function, otherwise it should revert

### [L-21] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: src/LlamaPolicy.sol

453:     if (balanceOf(policyholder) == 0) _mint(policyholder);

504:   function _mint(address policyholder) internal {

506:     _mint(policyholder, tokenId);

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

127:   function _mint(address to, uint256 id) internal virtual {

164:     _mint(to, id);

175:     _mint(to, id);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`
