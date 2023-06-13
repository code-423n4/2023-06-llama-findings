# Gas
## summary
No | Issue |Instances||
|-|:-|:-:|:-:|
|[G-1] |If statements that use && can be refactored into nested if statements|25|
|[G-2] |Use assembly for loops|10|
|[G-3] |Emit can be rearranged|12|
|[G-4] |bytes constants are more eficient than string constans|2|
|[G-5] |>= costs less gas than >|12|
|[G-6] |Using calldata instead of memory for read-only arguments in external functions saves gas|2|
|[G-7] | Use assembly to check for address(0)|22|
|[G-8] |Use hardcode address instead address(this)|4|
|[G-9] |Use unchecked when it is safe|4| 
|[G-10]|Unnecessary look up in if condition|12|
|[G-11]|Using > 0 costs more gas than != 0 when used on a uint in a require() statement|3|
|[G-12]|Using storage instead of memory for structs/arrays saves gas|1| 
|[G-13]|Don’t apply the same value to state variables|2| 
|[G-14]|Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate|3| 
|[G-15]|abi.encode() is less efficient than abi.encodePacked()|11| 
|[G-16]|Can Make The Variable Outside The Loop To Save Gas |8| 
|[G-17]|State variables can be packed to use fewer storage slots|1| 
|[G-18]|Use constants instead of type(uintx).max|12|
|[G-19]|Duplicated require()/if() checks should be refactored to a modifier or function|3| 
|[G-20]| Use assembly to hash instead of Solidity|3|
|[G-21]|With assembly, .call (bool success) transfer can be done gas-optimized|3|
|[G-22]|Should use arguments instead of state variable|4|
|[G-23]| With assembly, .call (bool success)  transfer can be done gas-optimized|2|
|[G-24]|Use ERC721A instead ERC721|11|
|[G-25]|Should Use Unchecked Block where Over/Underflow is not Possible|4|
|[G-26]|Use assembly for math (add, sub, mul, div)|4|
|[G-27]|Amounts should be checked for 0 before calling a transfer|1|
## details

## [G-1] If statements that use && can be refactored into nested if statements

```solidity
 20   if (
      msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
    ) revert UnauthorizedCaller();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L20-L22

```solidity
 36   if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {
      revert InsufficientDisapprovalQuantity();
 38   }

 49     if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

 61  if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);  
```solidity 

 56     if (
        minDisapprovals != type(uint128).max
          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
59      ) revert InsufficientDisapprovalQuantity();

68   if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);


81  if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L56-L59
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L68
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L81

```solidity
216  if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
221   if (role != approvalRole && !forceApprovalRole[role]) return 0;
       uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
223   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

231  if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
240  if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
     uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
243  return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L216
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L221--l223
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L231
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L240--l243

```solidity
74  if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L74
```solidity
467  if (hadRole && !willHaveRole) {
      currentRoleSupply.numberOfHolders -= 1;
      currentRoleSupply.totalQuantity -= quantityDiff;
    } else if (!hadRole && willHaveRole) {
      currentRoleSupply.numberOfHolders += 1;
      currentRoleSupply.totalQuantity += quantityDiff;
    } else if (hadRole && willHaveRole && initialQuantity > quantity) {
      // currentRoleSupply.numberOfHolders is unchanged
      currentRoleSupply.totalQuantity -= quantityDiff;
476    } else if (hadRole && willHaveRole && initialQuantity < quantity) 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L467-L476

```solidity
629  if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic))
649   if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L649
```solidity
213   if (role != approvalRole && !forceApprovalRole[role]) return 0;
     uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
215   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L213-L215
```solidity
425  bool case1 = quantity > 0 && expiration > block.timestamp;
426    bool case2 = quantity == 0 && expiration == 0;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L425-L426

## [G-2] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops. 

```solidity
178  for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i))
186  for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i))
199   for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i))
210   for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) 
217   for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L178
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L186
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L199
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L210
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L217

```solidity
151   for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L151

```solidity
636     for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i))
656   for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L636
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L656

```solidity
177 for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i))
185   for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L177
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L185

## [G-3] Emit can be rearranged

```solidity
82   function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {
    color[llamaExecutor] = _color;
    emit ColorSet(llamaExecutor, _color);
85  }
90   function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {
    logo[llamaExecutor] = _logo;
    emit LogoSet(llamaExecutor, _logo);
 93 }
 ```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L82-L85
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L90-L93

```solidity
267   function _authorizeStrategyLogic(ILlamaStrategy strategyLogic) internal {
    authorizedStrategyLogics[strategyLogic] = true;
    emit StrategyLogicAuthorized(strategyLogic);
270  }
273   function _authorizeAccountLogic(ILlamaAccount accountLogic) internal {
    authorizedAccountLogics[accountLogic] = true;
    emit AccountLogicAuthorized(accountLogic);
276  }
279   function _setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) internal {
    llamaPolicyMetadata = _llamaPolicyMetadata;
    emit PolicyMetadataSet(_llamaPolicyMetadata);
282  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L267-L270
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L273-L276
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L279-L282

```solidity
236   function updateRoleDescription(uint8 role, RoleDescription description) external onlyLlama {
    if (role > numRoles) revert RoleNotInitialized(role);
    emit RoleInitialized(role, description);
239  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L236-L239

## [G-4] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
183    string public name;

225    string memory _name,
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L183

## [G-14] >= costs less gas than >
 
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas


```solidity

223        return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242        return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223


```solidity

320       return quantity > 0 && canCreateAction[role][permissionId];

326       return quantity > 0 && block.timestamp > expiration;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L320

```solidity

649       if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L649



https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolut

## [G-6] Use calldata instead of memory for function arguments that do not get mutated


When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

Note: We are able to change these instances from memory to calldata without causing a stack too deep error
```solidity
file:    src/LlamaFactory.sol
154      function deploy(
    string memory name,
    ILlamaStrategy strategyLogic,
    ILlamaAccount accountLogic,
    bytes[] memory initialStrategies,
    bytes[] memory initialAccounts,
    RoleDescription[] memory initialRoleDescriptions,
    RoleHolderData[] memory initialRoleHolders,
    RolePermissionData[] memory initialRolePermissions,
    string memory color,
    string memory logo
  )
227       function _deploy(
    string memory name,
    ILlamaStrategy strategyLogic,
    ILlamaAccount accountLogic,
    bytes[] memory initialStrategies,
    bytes[] memory initialAccounts,
    RoleDescription[] memory initialRoleDescriptions,
    RoleHolderData[] memory initialRoleHolders,
    RolePermissionData[] memory initialRolePermissions
  )

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#LL154C1-L165

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#LL227C1-L236
```solidity
  function deploy(
    string memory name,
    ILlamaStrategy strategyLogic,
    ILlamaAccount accountLogic,
-   bytes[] memory initialStrategies,
+   bytes[] calldata initialStrategies,    
-   bytes[] memory initialAccounts,
+   bytes[] calldata initialAccounts,

    RoleDescription[] memory initialRoleDescriptions,
    RoleHolderData[] memory initialRoleHolders,
    RolePermissionData[] memory initialRolePermissions,
    string memory color,
    string memory logo
  )

```

## [G-7] Use assembly to check for address(0)
Saves 6 gas per instance
```solidity
48  if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();
166   if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();
199   if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();
247 if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();
256   if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L148
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L166
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L199
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L247
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L256

```solidity
181   if (llamaExecutor != address(0)) revert AlreadyInitialized();
405  if (llamaExecutor == address(0)) return; 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L181
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L405

```solidity
298    if (signer == address(0) || signer != policyholder) revert InvalidSignature();
330 if (guard != ILlamaActionGuard(address(0))) guard.validatePreActionExecution(actionInfo);
339   if (guard != ILlamaActionGuard(address(0))) guard.validatePostActionExecution(actionInfo);
389    if (signer == address(0) || signer != policyholder) revert InvalidSignature();
422   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
550 if (guard != ILlamaActionGuard(address(0))) guard.validateActionCreation(actionInfo);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L330
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L339
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L389
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L422
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L550

```solidity
41    require((owner = _ownerOf[id]) != address(0), "NOT_MINTED");
45     require(owner != address(0), "ZERO_ADDRESS");
90   require(to != address(0), "INVALID_RECIPIENT");
128   require(to != address(0), "INVALID_RECIPIENT");
145  require(owner != address(0), "NOT_MINTED");
168    || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, "")
179    || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, data)
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L41
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L45
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L90
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L128
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L145
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L168
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L179

```solidity
181   if (llamaExecutor != address(0)) revert AlreadyInitialized();
405   if (llamaExecutor == address(0)) return; 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L181
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L405

## [G-8] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. 
```solidity
223  core = LlamaCore(LlamaExecutor(address(this)).LLAMA_CORE());
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L223

```solidity
200   erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);
249    address(this), erc1155Data.recipient, erc1155Data.tokenId, erc1155Data.amount, erc1155Data.data
258   address(this),
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L200
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L249
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L258

```solidity
445   if (target == address(this) || target == address(policy)) revert RestrictedAddress();
455   if (script == address(this) || script == address(policy)) revert RestrictedAddress();
708  abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708

```solidity
12   if (address(this) == SELF) revert OnlyDelegateCall();
21    SELF = address(this);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaBaseScript.sol#L12
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaBaseScript.sol#L21

## [G-9] Use unchecked when it is safe
For example, those dealing with eth value will basically never overflow:
```solidity
469    currentRoleSupply.totalQuantity -= quantityDiff;
472   currentRoleSupply.totalQuantity += quantityDiff;
475   currentRoleSupply.totalQuantity -= quantityDiff;
478    currentRoleSupply.totalQuantity += quantityDiff;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L469
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L472
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L475
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L478

## [G-10] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.
```solidity
264    if (
      state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
       || state == ActionState.Failed
267         )
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L264-L267

```solidity
168   if (numRoles == 0 || getRoleSupplyAsNumberOfHolders(ALL_HOLDERS_ROLE) == 0) revert InvalidRoleHolderInput();   
427  if (!(case1 || case2)) revert InvalidRoleHolderInput(); 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L168
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L427

```solidity
298   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
389   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
422  if (signer == address(0) || signer != policyholder) revert InvalidSignature();
445  if (target == address(this) || target == address(policy)) revert RestrictedAddress();
455    if (script == address(this) || script == address(policy)) revert RestrictedAddress();
617   if (currentCount == type(uint128).max || quantity == type(uint128).max) return type(uint128).max;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L389
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L422
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L617

```solidity
254    if (
      state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
        || state == ActionState.Failed
257   )
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L254-L257

## [G-11] Using > 0 costs more gas than != 0 when used on a uint in a require() statement
This change saves 6 gas per instance.

```solidity
130      if (pos > 0)
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L130

```solidity
629      if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic))

649      if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) 

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL629C4-L629
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL649C5-L649

## [G-12] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity
290      Checkpoints.Checkpoint[] memory checkpoints = new Checkpoints.Checkpoint[](sliceLength);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L290

## [G-13] Don’t apply the same value to state variables
Assigning the same value to a state variable repeatedly can cause unnecessary gas costs, because each assignment operation incurs a gas cost. Additionally, if the value being assigned is the same as the current value of the state variable, the assignment operation does not actually change anything, which can waste gas.

```solidity
115      LLAMA_CORE_LOGIC = llamaCoreLogic;
116      LLAMA_POLICY_LOGIC = llamaPolicyLogic;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L115-L116

## [G-14]  Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.


```solidity
133      mapping(uint8 => bool) public forceApprovalRole;

  /// @notice Mapping of roles that can force an action to be disapproved.
  mapping(uint8 => bool) public forceDisapprovalRole;
  ```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L133-L137

```solidity
86      mapping(ILlamaStrategy => bool) public authorizedStrategyLogics;

  /// @notice Mapping of all authorized Llama account implementation (logic) contracts.
  mapping(ILlamaAccount => bool) public authorizedAccountLogics;
  ```
  https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L86-L90

  ```solidity
47        mapping(LlamaExecutor => string) public color;

  /// @notice Mapping of Llama instance to logo for SVG.
  mapping(LlamaExecutor => string) public log

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#LL47C1-L50

## [G-15] abi.encode() is less efficient than abi.encodePacked()

Consider changing it if possible.


```solidity
242      return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy))); 
529      bytes32 permissionId = keccak256(abi.encode(permission));
687      return keccak256(abi.encodePacked(id, creator, creatorRole, strategy, target, value, data));
708     abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
728     abi.encode(
        CREATE_ACTION_TYPEHASH,
        policyholder,
        role,
        address(strategy),
        target,
        value,
        keccak256(data),
        keccak256(bytes(description)),
        nonce
      )
741   return keccak256(abi.encodePacked("\x19\x01", _getDomainHash(), createActionHash));
753     abi.encode(
        CAST_APPROVAL_TYPEHASH,
        policyholder,
        role,
        _getActionInfoHash(actionInfo),
        keccak256(bytes(reason)),
        _useNonce(policyholder, msg.sig)
      )
763    return keccak256(abi.encodePacked("\x19\x01", _getDomainHash(), castApprovalHash));
775           abi.encode(
        CAST_DISAPPROVAL_TYPEHASH,
        policyholder,
        role,
        _getActionInfoHash(actionInfo),
        keccak256(bytes(reason)),
        _useNonce(policyholder, msg.sig)
      )
785    return keccak256(abi.encodePacked("\x19\x01", _getDomainHash(), castDisapprovalHash));
791         abi.encode(
        ACTION_INFO_TYPEHASH,
        actionInfo.id,
        actionInfo.creator,
        actionInfo.creatorRole,
        address(actionInfo.strategy),
        actionInfo.target,
        actionInfo.value,
        keccak256(actionInfo.data)
      )
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L242-L242
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L529-L529
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L687-L687
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708-L708
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L728-L738
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L741-L741
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L753-L760
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L763-L763
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L775-L782
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L785-L785
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L791-L800

## [G-16] Can Make The Variable Outside The Loop To Save Gas 

Consider making the stack variables before the loop which gonna save gas


```solidity

/// @audit the  bytes32 salt  should be declare outside the loop.

636         for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
637            bytes32 salt = bytes32(keccak256(strategyConfigs[i]));


/// @audit the  bytes32 salt  should be declare outside the loop.

656       for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
657          bytes32 salt = bytes32(keccak256(accountConfigs[i]));

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L636-L637

```solidity


/// @audit the  uint8 role   should be declare outside the loop.

177      for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
178         uint8 role = strategyConfig.forceApprovalRoles[i];


185        for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
186            uint8 role = strategyConfig.forceDisapprovalRoles[i];

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L177-L178


```solidity


/// @audit the   bool addressIsCore  &  bool addressIsPolicy  should be declare outside the loop.

71     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
72        bool addressIsCore = targets[i] == address(core);
73        bool addressIsPolicy = targets[i] == address(policy);


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L71

```solidity

177      for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
178        uint8 role = strategyConfig.forceApprovalRoles[i];


185      for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
186           uint8 role = strategyConfig.forceDisapprovalRoles[i];

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L177-L178

## [G-17] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

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
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L106-L130

## Recommend code
```solidity
bool public isFixedLengthApprovalPeriod;
uint128 public minApprovals;
uint128 public minDisapprovals;
uint64 public approvalPeriod;
uint64 public queuingPeriod;
int64 public expirationPeriod;
uint8 public approvalRole;
uint8 public disapprovalRole;
```

## [G-18] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity

36       if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

60       if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L36

```solidity
56     if (
57        minDisapprovals != type(uint128).max
58          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty


79        if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L57


```solidity

244       if (initialRoleHolders[0].expiration != type(uint64).max) revert InvalidDeployConfiguration();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L244


```solidity

617       if (currentCount == type(uint128).max || quantity == type(uint128).max) return type(uint128).max;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L617

## [G-19] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```

```solidity
298   if (signer == address(0) || signer != policyholder) revert InvalidSignature();

389   if (signer == address(0) || signer != policyholder) revert InvalidSignature();

422   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298

## [G-20] Use assembly to hash instead of Solidity

```
contract GasTest is DSTest {
  Contract0 c0;
  Contract1 c1;
  function setUp() public {
      c0 = new Contract0();
      c1 = new Contract1();
  }
  function testGas() public view {
      c0.solidityHash(2309349, 2304923409);
      c1.assemblyHash(2309349, 2304923409);
  }
}
contract Contract0 {
  function solidityHash(uint256 a, uint256 b) public view {
      //unoptimized
      keccak256(abi.encodePacked(a, b));
  }
}
contract Contract1 {
  function assemblyHash(uint256 a, uint256 b) public view {
      //optimized
      assembly {
          mstore(0x00, a)
          mstore(0x20, b)
          let hashedVal := keccak256(0x00, 0x40)
      }
  }
}

```

```solidity
529    bytes32 permissionId = keccak256(abi.encode(permission));

707    return keccak256(
      abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
    );

727    bytes32 createActionHash = keccak256(
      abi.encode(
        CREATE_ACTION_TYPEHASH,
        policyholder,
        role,
        address(strategy),
        target,
        value,
        keccak256(data),
        keccak256(bytes(description)),
        nonce
      )
    );
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L529
---
## [G-21] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
34    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol#L34
```solidity
326    (success, result) = target.call{value: msg.value}(callData);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L326
```
75   (bool success, bytes memory response) = targets[i].call(data[i]);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L72

## [G-22] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   


```solidity

/// @audit  llamaCore, policy   are state varaibles.

193         emit StrategyCreated(llamaCore, policy);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L193

```solidity
/// @audit  llamaCount is state variable.

260     emit LlamaInstanceCreated(
261      llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy), block.chainid
262    );
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L260-L262

```solidity
/// @audit  numRoles is state variable.

395       emit RoleInitialized(numRoles, description);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L395

## [G-23] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.

```solidity
file: /src/LlamaCore.sol

333    (bool success, bytes memory result) =
334      executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L333-L334

## [G-24] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

Reference: https://nextrope.com/erc721-vs-erc721a-2/

```solidity
7 import {ERC721NonTransferableMinimalProxy} from "src/lib/ERC721NonTransferableMinimalProxy.sol";
20 contract LlamaPolicy is ERC721NonTransferableMinimalProxy
149  __initializeERC721MinimalProxy(_name, string.concat("LL-", LibString.replace(LibString.upper(_name), " ", "-")));
520  ERC721NonTransferableMinimalProxy._burn(tokenId);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L7
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L20
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L149
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L520

```solidity
11 abstract contract ERC721NonTransferableMinimalProxy is Initializable
62   function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal 
168 1Received(msg.sender, address(0), id, "")
          == ERC721TokenReceiver.onERC721Received.selector,

179 
        || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, data)
180          == ERC721TokenReceiver.onERC721Received.selector,   

188 abstract contract ERC721TokenReceiver {
  function onERC721Received(address, address, uint256, bytes calldata) external virtual returns (bytes4) {
    return ERC721TokenReceiver.onERC721Received.selector;
  }
192 }                 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L11
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L62
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L168-L169
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L179-L180
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L188-L192


## [G-25] Should Use Unchecked Block where Over/Underflow is not Possible

Issue
Since Solidity 0.8.0, all arithmetic operations revert on overflow and underflow by default.
However in places where overflow and underflow is not possible, it is better to use unchecked block to reduce the gas usage.
Reference: https://docs.soliditylang.org/en/v0.8.15/control-structures.html#checked-or-unchecked-arithmetic

```solidity
286   return block.timestamp > action.minExecutionTime + expirationPeriod;

296   return action.creationTime + approvalPeriod;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L286

```solidity
297    return block.timestamp > action.minExecutionTime + expirationPeriod;

307    return action.creationTime + approvalPeriod;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L297

## [G-26] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Subtraction
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Multiplication
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265
```solidity
47    uint256 mid = len - sqrt(len);

51    low = mid + 1;

170   low = mid + 1;

192   low = mid + 1;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L47

## [G-27] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
167   erc20Data.token.safeTransfer(erc20Data.recipient, erc20Data.amount);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L167