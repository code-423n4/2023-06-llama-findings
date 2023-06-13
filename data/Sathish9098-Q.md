# LOW FINDINGS

| Issues Count| Issues | Instances|
|----------|----------|----------|
| [L-1]  | ``initialize()`` functions could be ``front run``   | 3  |
| [L-2]  | ``target`` contract is not checked before ``delegatecall`` and ''call''   | 1  |
| [L-3]  | Gas ``griefing/theft`` is possible on ``unsafe external call``   |  2 |
| [L-4]  |  Use ``latest version`` of OpenZeppelin instead of ``vulnerable version v4.8.0``  |  - |
| [L-5]  | Usage of ``tranferFrom`` function is ``discouraged``   | -  |
| [L-6]  |  ``Token Ownership`` and ``token existence`` is not checked  | -  |
| [L-7]  | ``abi.encodePacked()`` should not be used with ``dynamic types`` when passing the result to a hash function such as ``keccak256()``   | -  |
| [L-8]  | The ``ERC1155.sol`` contract does not implement any security features, such as ``access control`` or ``event logging``   | -  |
| [L-9]  |  Potential risks in ``ERC1155.sol``  | -  |
| [L-10]  |  A ``single`` point of ``failure``  | -  |
| [L-11]  |    |   |
| [L-12]  |    |   |
| [L-13]  |    |   |

##

## [L-1] initialize() functions could be front run 

### Impact

Front-running in this context refers to a situation where an attacker exploits their knowledge of a pending initialize() function call in an upgradeable contract to their advantage. They may, for example, deploy a new version of the contract with malicious intent just before the legitimate initialization occurs, thereby gaining control or manipulating the state of the contract

```solidity
FILE: 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol

156: function initialize(bytes memory config) external initializer {

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#LL156C2-L156C66

```solidity
FILE: 2023-06-llama/src/accounts/LlamaAccount.sol

130: function initialize(bytes memory config) external initializer {

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#LL130C2-L130C66

```solidity
File: 2023-06-llama/src/LlamaCore.sol

224: function initialize(

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#LL224C2-L224C23

### Recommended Mitigation
Implement access control mechanisms to limit who can call the initialize() function. Restricting the function's access to trusted entities or using permissions can help mitigate the risk of front-running

##

## [L-2] ``target`` contract is not checked before ``delegatecall`` and ''call'' 

### Impact
- If ``isScript`` is true, then the ``delegatecall`` function is used to invoke the method on the target contract. The ``delegatecall`` function does not check whether the target contract is valid. This means that an attacker could create a ``malicious contract`` that looks like a ``valid contract``, but does not actually have any code. If this malicious contract is invoked using ``delegatecall``, then the attacker could execute arbitrary code on the calling contract.

- The ``isScript`` variable is a security feature that helps to protect against malicious contracts. However, it is important to note that the ``isScript`` variable is not ``foolproof``. An attacker could still create a ``malicious contract`` that looks like a ``valid contract``, but has a ``different isScript`` value than expected

```solidity
FILE: Breadcrumbs2023-06-llama/src/LlamaPolicyMetadataParamRegistry.sol

34: (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);

```

### Recommended Mitigation:
This can be done by using the isContract function to check if the target variable is a reference to a contract. If the isContract function returns false, then the target variable is not a reference to a contract and should not be invoked.

##

## [L-3] Gas griefing/theft is possible on unsafe external call

### Impact
target.call{value: value}(data) method in Solidity is susceptible to gas griefing attacks

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

### POC 

```solidity
FILE: 2023-06-llama/src/LlamaExecutor.sol

34: (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);

```

```solidity
FILE: Breadcrumbs2023-06-llama/src/accounts/LlamaAccount.sol

326: (success, result) = target.call{value: msg.value}(callData);

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#LL326C7-L326C67

### Recommended Mitigation

``return`` data ``(success, result)`` has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

##

## [L-4] Use latest version of OpenZeppelin instead of vulnerable version v4.8.0

### Impact

Please check [official Link](https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.8.0)  . There are known vulnerabilities in v4.8.0 

- Missing Authorization
- Denial of Service (DoS)
- Improper Input Validation
- Incorrect Calculation

### POC 

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/49c0e4370d0cc50ea6090709e3835a3091e33ee2/contracts/proxy/utils/Initializable.sol#L2

### Recommended Mitigation

Use at least openzeppelin  4.9.0

##

## [L-5] Usage of tranferFrom function is discouraged

As per [ERC721 Docs](https://docs.openzeppelin.com/contracts/2.x/api/token/erc721) tranferFrom function is discouraged

```
transferFrom(address from, address to, uint256 tokenId) public

Transfers the ownership of a given token ID to another address. Usage of this method is discouraged, use safeTransferFrom whenever possible. Requires the msg.sender to be the owner, approved, or operator.

```
### Impact

The reason for discouraging the use of transferFrom is primarily for security purposes. The transferFrom function requires the msg.sender (the caller of the function) to be the owner of the token, approved by the owner, or an approved operator. This requirement ensures that only authorized entities can transfer the token

```solidity
FILE: 2023-06-llama/src/accounts/LlamaAccount.sol

200: erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#L200

### Recommended Mitigation

When ever possible use safeTransferFrom() function

##

## [L-6] Token Ownership and token existence is not checked

### Impact
The transferERC721 function does not check to see if the Llama is the owner of the token, or if the token even exists. This could lead to the following problems

- The Llama could attempt to transfer a token that it does not own.
- The Llama could attempt to transfer a token that does not exist.

### POC

```solidity
FILE: 2023-06-llama/src/accounts/LlamaAccount.sol

198: function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {
199:    if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();
200;    erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);
201:   }

```

### Recommended Mitigation

``transferERC721`` function should be updated to check for ``token ownership`` and ``token existence``

```solidity

require(erc721Data.token.ownerOf(erc721Data.tokenId) == address(this), "Llama is not the owner of the token");
require(erc721Data.token._exists(erc721Data.tokenId), "Token does not exist");

```

##

## [L-7] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

### Impact 

Use ``abi.encode()`` instead which will pad items to 32 bytes, which will ``prevent hash collisions`` (e.g. ``abi.encodePacked(0x123,0x456) => 0x123456`` => ``abi.encodePacked(0x1,0x23456)``, but ``abi.encode(0x123,0x456)`` => ``0x0...1230...456)``. “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.

If all arguments are strings and or bytes, bytes.concat() should be used instead.

### POC

```solidity
FILE: Breadcrumbs2023-06-llama/src/LlamaCore.sol

687: return keccak256(abi.encodePacked(id, creator, creatorRole, strategy, target, value, data));

741: return keccak256(abi.encodePacked("\x19\x01", _getDomainHash(), createActionHash));

763: return keccak256(abi.encodePacked("\x19\x01", _getDomainHash(), castApprovalHash));

785: return keccak256(abi.encodePacked("\x19\x01", _getDomainHash(), castDisapprovalHash));

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L741

### Recommended Mitigation
Consider using abi.encode over abi.encodePacked

##

## [L-8] The ``ERC1155.sol`` contract does not implement any security features, such as ``access control`` or ``event logging``

### Impact 
This means that anyone can mint tokens, transfer tokens, and burn tokens.

### POC 

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#L9

##

## [L-9] Potential risks in ``ERC1155.sol``

### Impact

- The contract is still under development, so there may be bugs or other issues
- The contract is not audited, so there is no guarantee that it is secure
- The contract is not interoperable with other ERC1155 contracts, so it cannot be used to exchange tokens with other users

##

## [L-10] A single point of failure

### Impact
The ``onlyLlama`` role has a single point of failure and ``onlyLlama`` can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

### POC 

```solidity
FILE: Breadcrumbs2023-06-llama/src/LlamaCore.sol

454:  function authorizeScript(address script, bool authorized) external onlyLlama {

444: function setGuard(address target, bytes4 selector, ILlamaActionGuard guard) external onlyLlama {

436: function createAccounts(ILlamaAccount llamaAccountLogic, bytes[] calldata accountConfigs) external onlyLlama {

429: function createStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs) external onlyLlama {


```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#LL429C1-L429C118


### Recommended Mitigation Steps

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Also detail them in documentation and NatSpec comments.



















