# LOW FINDINGS

| Issues Count| Issues | Instances|
|----------|----------|----------|
| [L-1]  | ``finalizeInitialization()`` function not implemented as per docs   | 1  |
| [L-2]  | Anyone can deploy the contract and become the ``LLAMA_FACTORY`` owner.   | 1  |
| [L-3]  | ``sendValue()`` function does not return a status indicating the success or failure of the transfer and Consumes more gas    |  1 |
| [L-4]  | ``initialize()`` functions could be ``front run``   | 3  |
| [L-5]  | ``target`` contract is not checked before ``delegatecall`` and ''call''   | 1  |
| [L-6]  | Gas ``griefing/theft`` is possible on ``unsafe external call``   |  2 |
| [L-7]  |  Use ``latest version`` of OpenZeppelin instead of ``vulnerable version v4.8.0``  |  - |
| [L-8]  | Usage of ``tranferFrom`` function is ``discouraged``   | -  |
| [L-9]  |  ``Token Ownership`` and ``token existence`` is not checked  | -  |
| [L-10]  | ``abi.encodePacked()`` should not be used with ``dynamic types`` when passing the result to a hash function such as ``keccak256()``   | -  |
| [L-11]  | The ``ERC1155.sol`` contract does not implement any security features, such as ``access control`` or ``event logging``   | -  |
| [L-12]  |  Potential risks in ``ERC1155.sol``  | -  |
| [L-13]  |  A ``single`` point of ``failure``  | -  |
| [L-14]  |  ``Emit events`` for critical ``initialize()`` functions   | 3  |
| [L-15]  |  In ``tokenURI()`` function if ``tokenid`` not exists should return ``null`` instead of revert|- |


##

## [L-1] ``finalizeInitialization()`` function not implemented as per docs

## Impact

```
177: /// @dev This method can only be called once.

```
Ensure that the initialization process can only be executed once, preventing any potential misuse or unauthorized modifications of critical contract parameters.

### POC

```solidity

  /// @notice Sets the address of the `LlamaExecutor` contract and gives holders of role ID 1 permission
  /// to change role permissions.
  /// @dev This method can only be called once.
  /// @param _llamaExecutor The address of the `LlamaExecutor` contract.
  /// @param bootstrapPermissionId The permission ID that allows holders to change role permissions.
  function finalizeInitialization(address _llamaExecutor, bytes32 bootstrapPermissionId) external {
    if (llamaExecutor != address(0)) revert AlreadyInitialized();

    llamaExecutor = _llamaExecutor;
    _setRolePermission(BOOTSTRAP_ROLE, bootstrapPermissionId, true);
  }

```

### Recommended Mitigation

```solidity

+ bool private initialized; // Track initialization status

function finalizeInitialization(address _llamaExecutor, bytes32 bootstrapPermissionId) external {
+   require(!initialized, "Already initialized"); // Check if already initialized
+   initialized = true; // Mark as initialized
  if (llamaExecutor != address(0)) revert AlreadyInitialized();
  llamaExecutor = _llamaExecutor;
  _setRolePermission(BOOTSTRAP_ROLE, bootstrapPermissionId, true);
}

```

##

## [L-2] Anyone can deploy the contract and become the ``LLAMA_FACTORY`` owner.

### Impact

Anyone can deploy the contract and become the ``LLAMA_FACTORY`` owner.This means that they can mint new LLAMAs, burn LLAMAs, and change the Llama contract's code. This could be used to exploit the contract or to take control of it.

### POC

```solidity
FILE: 2023-06-llama/src/LlamaPolicyMetadataParamRegistry.sol

59:  LLAMA_FACTORY = msg.sender;

```
https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaPolicyMetadataParamRegistry.sol#L59

### Recommended Mitigation
A better way to assign the value of the LLAMA_FACTORY variable is to use a decentralized governance system. This would allow the community to vote on who should be the LLAMA_FACTORY owner. This would make it more difficult for anyone to exploit the contract or to take control of it

##

## [L-3]  ``sendValue()`` function does not return a status indicating the success or failure of the transfer and Consumes more gas 

### Impact 
- using sendValue() lies in its limitations in error handling.
- function has a gas stipend of 2,300 gas units, which may not be sufficient for contracts with complex fallback functions or extensive operations. If the recipient contract consumes more than the stipulated gas limit, the transfer will fail, and the entire transaction will be reverted


### POC

```solidity
FILE: 2023-06-llama/src/accounts/LlamaAccount.sol

149: nativeTokenData.recipient.sendValue(nativeTokenData.amount);

```

### Recommended Mitigation
Use call() instead of sendValue()


## [L-4] initialize() functions could be front run 

### Impact

Front-running in this context refers to a situation where an attacker exploits their knowledge of a pending initialize() function call in an upgradeable contract to their advantage. They may, for example, deploy a new version of the contract with malicious intent just before the legitimate initialization occurs, thereby gaining control or manipulating the state of the contract

### POC

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

## [L-5] ``target`` contract is not checked before ``delegatecall`` and ''call'' 

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

## [L-6] Gas griefing/theft is possible on unsafe external call

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

## [L-7] Use latest version of OpenZeppelin instead of vulnerable version v4.8.0

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

## [L-8] Usage of tranferFrom function is discouraged

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

## [L-9] Token Ownership and token existence is not checked

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

## [L-10] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

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

## [L-11] The ``ERC1155.sol`` contract does not implement any security features, such as ``access control`` or ``event logging``

### Impact 
This means that anyone can mint tokens, transfer tokens, and burn tokens.

### POC 

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#L9

##

## [L-12] Potential risks in ``ERC1155.sol``

### Impact

- The contract is still under development, so there may be bugs or other issues
- The contract is not audited, so there is no guarantee that it is secure
- The contract is not interoperable with other ERC1155 contracts, so it cannot be used to exchange tokens with other users

##

## [L-13] A single point of failure

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

##

## [L-14] Emit events for critical initialize() functions 

### Impact

Emitting events for critical initialize() functions, it can be a good practice to provide important information about the initialization process. Events can be used to notify external entities or frontend applications about the state changes or successful completion of the initialization

### POC

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
ADD EVENT-EMIT


##

## [L-15] In ``tokenURI()`` function if ``tokenid`` not exists should return null instead of revert 

### Impact

It is important to note that reverting in the ``tokenURI()`` function will prevent any further execution of the function. This means that any other code that was called by the ``tokenURI()`` function will not be executed. By returning ``null`` ``instead`` of ``reverting``, you can allow other code to ``continue executing``

### POC 

```solidity
FILE: 2023-06-llama/src/LlamaPolicyMetadata.sol

function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)
    external
    pure
    returns (string memory)
  {
    string[21] memory parts;
    string memory policyholder = LibString.toHexString(address(uint160(tokenId)));

    parts[0] =
      '<svg xmlns="http://www.w3.org/2000/svg" width="390" height="500" fill="none"><g clip-path="url(#a)"><rect width="390" height="500" fill="#0B101A" rx="13.393" /><mask id="b" width="364" height="305" x="4" y="30" maskUnits="userSpaceOnUse" style="mask-type:alpha"><ellipse cx="186.475" cy="182.744" fill="#8000FF" rx="196.994" ry="131.329" transform="rotate(-31.49 186.475 182.744)" /></mask><g mask="url(#b)"><g filter="url(#c)"><ellipse cx="226.274" cy="247.516" fill="url(#d)" rx="140.048" ry="59.062" transform="rotate(-31.49 226.274 247.516)" /></g><g filter="url(#e)"><ellipse cx="231.368" cy="254.717" fill="url(#f)" rx="102.858" ry="43.378" transform="rotate(-31.49 231.368 254.717)" /></g></g><g filter="url(#g)"><ellipse cx="237.625" cy="248.969" fill="url(#h)" rx="140.048" ry="59.062" transform="rotate(-31.49 237.625 248.969)" /></g><circle cx="109.839" cy="147.893" r="22" fill="url(#i)" /><rect width="150" height="35.071" x="32" y="376.875" fill="';

    parts[1] = color;

    parts[2] =
      '" rx="17.536" /><text xml:space="preserve" fill="#0B101A" font-family="ui-monospace,Cascadia Mono,Menlo,Monaco,Segoe UI Mono,Roboto Mono,Oxygen Mono,Ubuntu Monospace,Source Code Pro,Droid Sans Mono,Fira Mono,Courier,monospace" font-size="16"><tspan x="45.393" y="399.851">';

    parts[3] = string.concat(LibString.slice(policyholder, 0, 6), "...", LibString.slice(policyholder, 38, 42));

    parts[4] =
      '</tspan></text><path fill="#fff" d="M341 127.067a11.433 11.433 0 0 0 8.066-8.067 11.436 11.436 0 0 0 8.067 8.067 11.433 11.433 0 0 0-8.067 8.066 11.43 11.43 0 0 0-8.066-8.066Z" /><path stroke="#fff" stroke-width="1.5" d="M349.036 248.018V140.875" /><circle cx="349.036" cy="259.178" r="4.018" fill="#fff" /><path stroke="#fff" stroke-width="1.5" d="M349.036 292.214v-21.429" /><path fill="#fff" d="M343.364 33.506a1.364 1.364 0 0 0-2.728 0V43.85l-7.314-7.314a1.364 1.364 0 0 0-1.929 1.928l7.315 7.315h-10.344a1.364 1.364 0 0 0 0 2.727h10.344l-7.315 7.315a1.365 1.365 0 0 0 1.929 1.928l7.314-7.314v10.344a1.364 1.364 0 0 0 2.728 0V50.435l7.314 7.314a1.364 1.364 0 0 0 1.929-1.928l-7.315-7.315h10.344a1.364 1.364 0 1 0 0-2.727h-10.344l7.315-7.315a1.365 1.365 0 0 0-1.929-1.928l-7.314 7.314V33.506ZM73.81 44.512h-4.616v1.932h1.777v10.045h-2.29v1.932h6.82V56.49h-1.69V44.512ZM82.469 44.512h-4.617v1.932h1.777v10.045h-2.29v1.932h6.82V56.49h-1.69V44.512ZM88.847 51.534c.097-.995.783-1.526 2.02-1.526 1.236 0 1.854.531 1.854 1.68v.28l-3.4.416c-2.02.251-3.603 1.13-3.603 3.11 0 1.971 1.497 3.101 3.767 3.101 1.903 0 2.743-.724 3.14-1.343h.192v1.17h2.647v-6.337c0-2.763-1.777-4.009-4.54-4.009-2.782 0-4.482 1.246-4.685 3.168v.29h2.608Zm-.338 3.835c0-.763.58-1.13 1.42-1.246l2.792-.367v.435c0 1.632-1.082 2.453-2.57 2.453-1.043 0-1.642-.502-1.642-1.275ZM97.614 58.42h2.608v-6.51c0-1.246.657-1.787 1.575-1.787.821 0 1.226.474 1.226 1.275v7.023h2.609v-6.51c0-1.247.656-1.788 1.564-1.788.831 0 1.227.474 1.227 1.275v7.023h2.618v-7.38c0-1.835-1.159-2.927-2.927-2.927-1.584 0-2.318.686-2.743 1.44h-.194c-.289-.657-1.004-1.44-2.472-1.44-1.44 0-2.067.6-2.415 1.208h-.193v-1.015h-2.483v10.114ZM115.654 51.534c.097-.995.782-1.526 2.019-1.526 1.236 0 1.854.531 1.854 1.68v.28l-3.4.416c-2.019.251-3.603 1.13-3.603 3.11 0 1.971 1.498 3.101 3.767 3.101 1.903 0 2.744-.724 3.14-1.343h.193v1.17h2.647v-6.337c0-2.763-1.778-4.009-4.54-4.009-2.782 0-4.482 1.246-4.685 3.168v.29h2.608Zm-.338 3.835c0-.763.58-1.13 1.42-1.246l2.791-.367v.435c0 1.632-1.081 2.453-2.569 2.453-1.043 0-1.642-.502-1.642-1.275ZM35.314 52.07a.906.906 0 0 1 .88-.895h11.72a4.205 4.205 0 0 0 3.896-2.597 4.22 4.22 0 0 0 .323-1.614V32h-3.316v14.964a.907.907 0 0 1-.88.894H36.205a4.206 4.206 0 0 0-2.972 1.235A4.219 4.219 0 0 0 32 52.07v10.329h3.314v-10.33ZM53.6 34.852h-.147l.141.14v3.086h3.05l1.43 1.446a4.21 4.21 0 0 0-2.418 1.463 4.222 4.222 0 0 0-.95 2.664v18.752h3.3V43.647a.909.909 0 0 1 .894-.895h.508c1.947 0 2.608-1.086 2.803-1.543.196-.456.498-1.7-.88-3.085l-3.23-3.261h-1.006" /><path fill="#fff" d="M44.834 60.77a5.448 5.448 0 0 1 3.89 1.629h4.012a8.8 8.8 0 0 0-3.243-3.608 8.781 8.781 0 0 0-12.562 3.608h4.012a5.459 5.459 0 0 1 3.89-1.629Z" />';

    parts[5] = logo;

    parts[6] =
      '</g><defs><radialGradient id="d" cx="0" cy="0" r="1" gradientTransform="rotate(-90.831 270.037 36.188) scale(115.966 274.979)" gradientUnits="userSpaceOnUse"><stop stop-color="';

    parts[7] = color;

    parts[8] = '" /><stop offset="1" stop-color="';

    parts[9] = color;

    parts[10] =
      '" stop-opacity="0" /></radialGradient><radialGradient id="f" cx="0" cy="0" r="1" gradientTransform="matrix(7.1866 -72.99558 127.41796 12.54463 239.305 292.746)" gradientUnits="userSpaceOnUse"><stop stop-color="';

    parts[11] = color;

    parts[12] = '" /><stop offset="1" stop-color="';

    parts[13] = color;

    parts[14] =
      '" stop-opacity="0" /></radialGradient><radialGradient id="h" cx="0" cy="0" r="1" gradientTransform="rotate(-94.142 264.008 51.235) scale(212.85 177.126)" gradientUnits="userSpaceOnUse"><stop stop-color="';

    parts[15] = color;

    parts[16] = '" /><stop offset="1" stop-color="';

    parts[17] = color;

    parts[18] =
      '" stop-opacity="0" /></radialGradient><radialGradient id="i" cx="0" cy="0" r="1" gradientTransform="matrix(23.59563 32 -33.15047 24.44394 98.506 137.893)" gradientUnits="userSpaceOnUse"><stop stop-color="#0B101A" /><stop offset=".609" stop-color="';

    parts[19] = color;

    parts[20] =
      '" /><stop offset="1" stop-color="#fff" /></radialGradient><filter id="c" width="346.748" height="277.643" x="52.9" y="108.695" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="25" /></filter><filter id="e" width="221.224" height="170.469" x="120.757" y="169.482" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="10" /></filter><filter id="g" width="446.748" height="377.643" x="14.251" y="60.147" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="50" /></filter><clipPath id="a"><rect width="390" height="500" fill="#fff" rx="13.393" /></clipPath></defs></svg>';

    // This output has been broken up into multiple outputs to avoid a stack too deep error
    string memory output1 =
      string.concat(parts[0], parts[1], parts[2], parts[3], parts[4], parts[5], parts[6], parts[7], parts[8]);
    string memory output2 =
      string.concat(parts[9], parts[10], parts[11], parts[12], parts[13], parts[14], parts[15], parts[16], parts[17]);
    string memory output = string.concat(output1, output2, parts[18], parts[19], parts[20]);

    string memory json = Base64.encode(
      bytes(
        string.concat(
          '{"name": "',
          name,
          ' Member", "description": "This NFT represents membership in the Llama organization: ',
          name,
          ". The owner of this NFT can participate in governance according to their roles and permissions. Visit https://app.llama.xyz/profiles/",
          policyholder,
          ' to view their profile page.", "external_url": "https://app.llama.xyz", "image": "data:image/svg+xml;base64,',
          Base64.encode(bytes(output)),
          '"}'
        )
      )
    );
    output = string.concat("data:application/json;base64,", json);

    return output;
  }

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadata.sol#L17-L100

### Recommended Mitigation

```solidity
if (!_exists(tokenId)) {
    // Return null if the tokenId does not exist
    return null;
  }

```






















