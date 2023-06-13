## Summary

### !Note:  The last 23 and 24 number reports gas is Mised From bot.

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Using fixed bytes is cheaper than using string | 1 | - |
| [G-02] | Can make the variable outside the loop to save gas | 7 | - |
| [G-03] | Using calldata instead of memory for read-only arguments in external functions saves gas | 6 | - |
| [G-04] | Public function not called by the contract should be declared external instead | 5 | - |
| [G-05] | Should use arguments instead of state variable  | 11 | - |
| [G-06] | Use assembly to check for address(0) | 11 | - |
| [G-07] | Using storage instead of memory for structs/arrays saves gas | 1 | - |
| [G-08] | With assembly, .call (bool success)  transfer can be done gas-optimized | 2 | - |
| [G-09] | Duplicated require()/if() checks should be refactored to a modifier or function | 5 | - |
| [G-10] | Use constants instead of type(uintx).max | 6 | - |
| [G-11] | Use nested if and, avoid multiple check combinations | 12 | - |
| [G-12] | abi.encode() is less efficient than abi.encodepacked() | 3 | - |
| [G-13] | >= costs less gas than >| 5 | - |
| [G-14] | Using ERC721A instead of ERC721 for more gas-efficient | 6 | - |
| [G-15] | Don’t apply the same value to state variables | 1 | - |
| [G-16] | Use != 0 instead of > 0 for unsigned integer comparison | 13 | - |
| [G-17] | State variables can be packed to use fewer storage slots | 1 | - |
| [G-18] | Use hardcode address instead of address(this)  | 6 | - |
| [G-19] | Use assembly to hash instead of Solidity | 3 | - |
| [G-20] | Minimize external calls if possible | more files | - |
| [G-21] | Inverting the condition of an if-else-statement wastes gas | 3 | - |
| [G-22] | Emit can be rearranged | 5 | - | 
| [G-23] | Empty blocks should be removed or emit something | 6 | - |
| [G-24] | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate | 3 | - |
| [G-25] | Amounts should be checked for 0 before calling a transfer | 1 | - |
| [G-26] | Use assembly for math (add, sub, mul, div) | 4 | - |
| [G-27] | Should Use Unchecked Block where Over/Underflow is not Possible | 4 | - |

## Gas Optimizations  

## [G-1] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.


```solidity
file: /src/lib/ERC721NonTransferableMinimalProxy.sol

28    string public symbol;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#28



## [G-2] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas


```solidity
File: /src/LlamaCore.sol

/// @audit the  bytes32 salt  should be declare outside the loop.

636         for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
637            bytes32 salt = bytes32(keccak256(strategyConfigs[i]));


/// @audit the  bytes32 salt  should be declare outside the loop.

656       for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
657          bytes32 salt = bytes32(keccak256(accountConfigs[i]));

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L636-L637

```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

/// @audit the  uint8 role   should be declare outside the loop.

177      for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
178         uint8 role = strategyConfig.forceApprovalRoles[i];


185        for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
186            uint8 role = strategyConfig.forceDisapprovalRoles[i];

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L177-L178


```solidity
file: /src/llama-scripts/LlamaGovernanceScript.sol

/// @audit the   bool addressIsCore  &  bool addressIsPolicy  should be declare outside the loop.

71     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
72        bool addressIsCore = targets[i] == address(core);
73        bool addressIsPolicy = targets[i] == address(policy);


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L71

```solidity
file: /src/strategies/LlamaAbsoluteStrategyBase.sol

177      for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
178        uint8 role = strategyConfig.forceApprovalRoles[i];


185      for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
186           uint8 role = strategyConfig.forceDisapprovalRoles[i];

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L177-L178


## [G-3]  Using calldata instead of memory for read-only arguments in external functions saves gas 

calldata must be used when declaring  an external function's dynamic parameters.
When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
file: /src/LlamaPolicyMetadata.sol

/// @audit Using storage instead of memory for structs/arrays saves gas on the name, color, and  logo   parameters.

17    function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)
18     external
19     pure
20     returns (string memory)

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L17-L20


```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

156      function initialize(bytes memory config) external initializer {

``` 
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156

```solidity
file: /src/LlamaFactory.sol

154   function deploy(
155     string memory name,
156     ILlamaStrategy strategyLogic,
157     ILlamaAccount accountLogic,
158     bytes[] memory initialStrategies,
159     bytes[] memory initialAccounts,
160     RoleDescription[] memory initialRoleDescriptions,
161     RoleHolderData[] memory initialRoleHolders,
162     RolePermissionData[] memory initialRolePermissions,
163     string memory color,
164     string memory logo
165     ) external onlyRootLlama returns (LlamaExecutor executor, LlamaCore core) {


206     function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)


218     function contractURI(string memory name) external view returns (string memory) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L154-L165


```solidity
file: /src/accounts/LlamaAccount.sol

130    function initialize(bytes memory config) external initializer {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130



## [G-4] Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.


```solidity
file: /lib/ERC721NonTransferableMinimalProxy.sol

87     function transferFrom(address from, address to, uint256 id) public virtual {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L87



```solidity
file: /src/LlamaPolicyMetadata.sol

104       function contractURI(string memory name) public pure returns (string memory) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L104


```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

288       function isActionDisapproved(ActionInfo calldata actionInfo) public view returns (bool) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L288


```solidity 
file: /src/LlamaPolicy.sol

359    function transferFrom(address, /* from */ address, /* to */ uint256 /* policyId */ )
362      public
361      pure
362      override
363     nonTransferableToken


375    function safeTransferFrom(address, /* from */ address, /* to */ uint256, /* policyId */ bytes calldata /* data */ )
376     public
377     pure
378     override
379     nonTransferableToken

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L359-L363



## [G-5] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   


```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

/// @audit  llamaCore, policy   are state varaibles.

193         emit StrategyCreated(llamaCore, policy);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L193


```solidity
file: /src/LlamaFactory.sol

/// @audit  llamaCount is state variable.

260     emit LlamaInstanceCreated(
261      llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy), block.chainid
262    );


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L260-L262


```solidity
file: /src/LlamaPolicy.sol

/// @audit  numRoles is state variable.

395       emit RoleInitialized(numRoles, description);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L395


## [G-6] Use assembly to check for address(0)

  Save 6 gas per instance.

```solidity
file: /src/accounts/LlamaAccount.sol

148        if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();

166        if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();

199        if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();

247       if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();

256       if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L148

```solidity
file: /src/LlamaPolicy.sol

181       if (llamaExecutor != address(0)) revert AlreadyInitialized();

405       if (llamaExecutor == address(0)) return; // Skip check during initialization.

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L181

```solidity
file:

298       if (signer == address(0) || signer != policyholder) revert InvalidSignature();

330       if (guard != ILlamaActionGuard(address(0))) guard.validatePreActionExecution(actionInfo);

339       if (guard != ILlamaActionGuard(address(0))) guard.validatePostActionExecution(actionInfo);
 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298

```solidity
file: /src/lib/ERC721NonTransferableMinimalProxy.sol

128       require(to != address(0), "INVALID_RECIPIENT");

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L128


## [G-7] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to 
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity
file: /src/LlamaPolicy.sol

290        Checkpoints.Checkpoint[] memory checkpoints = new Checkpoints.Checkpoint[](sliceLength);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L290


## [G-8] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.


```solidity
file: /src/llama-scripts/LlamaGovernanceScript.sol

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L75

```solidity
file: /src/LlamaCore.sol

333    (bool success, bytes memory result) =
334      executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L333-L334


## [G-9] Duplicated require()/if() checks should be refactored to a modifier or function   

   to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.


```solidity
file: /src/lib/ERC721NonTransferableMinimalProxy.sol

/// @audit  the bellow require are duplicate.

90      require(to != address(0), "INVALID_RECIPIENT");
128     require(to != address(0), "INVALID_RECIPIENT");


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L90


```solidity
file: /src/strategies/LlamaAbsolutePeerReview.sol

67        if (actionInfo.creator == policyholder) revert ActionCreatorCannotCast();   
80        if (actionInfo.creator == policyholder) revert ActionCreatorCannotCast();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L67

```solidity
file: /src/strategies/LlamaRelativeQuorum.sol#

216        if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
221        if (role != approvalRole && !forceApprovalRole[role]) return 0;


231       if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
240       if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L216

```solidity
file: /src/LlamaPolicy.sol

223       if (balanceOf(policyholder) == 0) revert AddressDoesNotHoldPolicy(policyholder);
453       if (balanceOf(policyholder) == 0) _mint(policyholder);
 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L223


## [G-10]  Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /src/strategies/LlamaAbsoluteQuorum.sol

36       if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

60       if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L36

```solidity
file: /src/strategies/LlamaAbsolutePeerReview.sol
56     if (
57        minDisapprovals != type(uint128).max
58          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty


79        if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L57


```solidity
file: /src/LlamaFactory.sol

244       if (initialRoleHolders[0].expiration != type(uint64).max) revert InvalidDeployConfiguration();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L244


```solidity
file: /src/LlamaCore.sol

617       if (currentCount == type(uint128).max || quantity == type(uint128).max) return type(uint128).max;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L617


## [G-11] Use nested if and, avoid multiple check combinations (saves 8 gas per &&)

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
file: /src/LlamaPolicyMetadataParamRegistry.sol

20     if (
21        msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
22        ) revert UnauthorizedCaller();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L20-L22

```solidity
file: /src/strategies/LlamaAbsoluteQuorum.sol

36       if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) { 

49       if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);    

61       if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L36

```solidity
file: /src/strategies/LlamaAbsolutePeerReview.sol

68        if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

81        if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L68


```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

216        if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

221        if (role != approvalRole && !forceApprovalRole[role]) return 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L216


```solidity
file: /src/LlamaPolicy.sol

467       if (hadRole && !willHaveRole) {

476       } else if (hadRole && willHaveRole && initialQuantity < quantity) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L467
 

```solidity
file: /src/LlamaCore.sol

629       if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

649       if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {    

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629



## [G-12] abi.encode() is less efficient than  abi.encodepacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison


```solidity
file: /src/LlamaCore.sol

242       return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));

529       bytes32 permissionId = keccak256(abi.encode(permission));

708       abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L242


## [G-13] >= costs less gas than >
 
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas


```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

223        return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242        return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223


```solidity
file: /src/LlamaPolicy.sol

320       return quantity > 0 && canCreateAction[role][permissionId];

326       return quantity > 0 && block.timestamp > expiration;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L320

```solidity
file: /src/LlamaCore.sol

649       if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L649


## [G-14]  Using ERC721A instead of ERC721 for more gas-efficient 

ERC721A is a proposed extension to the ERC721 standard that aims to improve gas efficiency and reduce the cost of deploying and using non-fungible tokens (NFTs) on the Ethereum blockchain.

Reference : https://nextrope.com/erc721-vs-erc721a-2/

```solidity
file: /src/accounts/LlamaAccount.sol

/// @audit ERC721Data

198      function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {

205      function batchTransferERC721(ERC721Data[] calldata erc721Data) external onlyLlama {    

214      function approveERC721(ERC721Data calldata erc721Data) public onlyLlama {   

220      function batchApproveERC721(ERC721Data[] calldata erc721Data) external onlyLlama {     

229      function approveOperatorERC721(ERC721OperatorData calldata erc721OperatorData) public onlyLlama {    
    
235      function batchApproveOperatorERC721(ERC721OperatorData[] calldata erc721OperatorData) external onlyLlama {    

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L198


## [G-15] Don’t apply the same value to state variables

Assigning the same value to a state variable repeatedly can cause unnecessary gas costs, because each assignment operation incurs a gas cost. Additionally, if the value being assigned is the same as the current value of the state variable, the assignment operation does not actually change anything, which can waste gas.

```solidity
file: /src/LlamaFactory.sol

115      LLAMA_CORE_LOGIC = llamaCoreLogic;
116      LLAMA_POLICY_LOGIC = llamaPolicyLogic;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#LL115C1-L116

## [G-16] Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```
Instances:

```solidity
File: /src/LlamaCore.sol

629     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

649     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {    

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629


```solidity
File: /src/LlamaPolicy.sol

307   return quantity > 0;

313   return quantity > 0;

320   return quantity > 0 && canCreateAction[role][permissionId];

326   return quantity > 0 && block.timestamp > expiration;

425   bool case1 = quantity > 0 && expiration > block.timestamp;

444   bool hadRole = initialQuantity > 0;

445   bool willHaveRole = quantity > 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L307


```solidity
file: src/strategies/LlamaAbsoluteStrategyBase.sol

215   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

232   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215


```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

223   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223


## [G-17] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).


 Instances:
```solidity
file: /src/strategies/LlamaAbsoluteStrategyBase.sol

106    /// @notice If `false`, action be queued before approvalEndTime.
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
130    uint8 public disapprovalRole;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L106-L130


### Recommend code:
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

## [G-18] Use hardcode address instead of address(this)

it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address


```solidity
File: /src/LlamaCore.sol
445       if (target == address(this) || target == address(policy)) revert RestrictedAddress();

455       if (script == address(this) || script == address(policy)) revert RestrictedAddress();

708       abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445


```solidity
File: /src/accounts/LlamaAccount.sol

200        erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);

249        address(this), erc1155Data.recipient, erc1155Data.tokenId, erc1155Data.amount, erc1155Data.data

258      address(this),

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L200


## [G-19] Use assembly to hash instead of Solidity

Example:
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
Instances:

```solidity
file: /src/LlamaCore.sol

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


## [G-20]  Minimize external calls if possible

 External calls to other contracts are expensive in terms of gas. Try to minimize external calls by using in-line code or caching data in memory.

```solidity
file: more files

If posible minimize the the external call and use istead inline functions to use inside the contract.

```
https://github.com/code-423n4/2023-06-llama/blob/main/


## [G-21] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas

References: https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde

```solidity
file: /src/LlamaCore.sol

479       return actionsCount == 0 ? 0 : actions[actionsCount - 1].creationTime;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L479


```solidity
file: /src/lib/Checkpoints.sol

57           return pos == 0 ? 0 : _unsafeAccess(self._checkpoints, pos - 1).quantity;

85           return pos == 0 ? 0 : _unsafeAccess(self._checkpoints, pos - 1).quantity;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L57


## [G-22] Emit can be rearranged


```solidity
file: /src/LlamaFactory.sol

267   function _authorizeStrategyLogic(ILlamaStrategy strategyLogic) internal {
      authorizedStrategyLogics[strategyLogic] = true;
      emit StrategyLogicAuthorized(strategyLogic);
270    }


273   function _authorizeAccountLogic(ILlamaAccount accountLogic) internal {
     authorizedAccountLogics[accountLogic] = true;
     emit AccountLogicAuthorized(accountLogic);
276    }


279   function _setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) internal {
    llamaPolicyMetadata = _llamaPolicyMetadata;
    emit PolicyMetadataSet(_llamaPolicyMetadata);
282  }

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L267-L270

```solidity
file: /src/LlamaPolicyMetadataParamRegistry.sol

82   function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {
     color[llamaExecutor] = _color;
     emit ColorSet(llamaExecutor, _color);
85    }


90   function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {
     logo[llamaExecutor] = _logo;
     emit LogoSet(llamaExecutor, _logo);
93   }

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L82-L85


## [G-23] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be 
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when 
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) . 
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.


```solidity
file: /src/accounts/LlamaAccount.sol

143      receive() external payable {}

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L143


```solidity
file: /src/LlamaPolicy.sol

364     {}

372     {}

380     {}


383     function approve(address, /* spender */ uint256 /* id */ ) public pure override nonTransferableToken {}

386    function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L364


## [G-24]  Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.


```solidity
file: /src/strategies/LlamaAbsoluteStrategyBase.sol

133      mapping(uint8 => bool) public forceApprovalRole;
134
135     /// @notice Mapping of roles that can force an action to be disapproved.
136     mapping(uint8 => bool) public forceDisapprovalRole;


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L133-L136


```solidity
file: /src/LlamaFactory.sol

86      mapping(ILlamaStrategy => bool) public authorizedStrategyLogics;
87
88     /// @notice Mapping of all authorized Llama account implementation (logic) contracts.
89     mapping(ILlamaAccount => bool) public authorizedAccountLogics;


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L86-L89


```solidity
file: /src/LlamaPolicyMetadataParamRegistry.sol

47        mapping(LlamaExecutor => string) public color;
48
49        /// @notice Mapping of Llama instance to logo for SVG.
50        mapping(LlamaExecutor => string) public logo;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L47-L50


## [G-25] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

Example:
```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

Instances:
```solidity
File: /src/accounts/LlamaAccount.sol

167   erc20Data.token.safeTransfer(erc20Data.recipient, erc20Data.amount);
  }

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L167

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
File: /src/lib/Checkpoints.sol

47    uint256 mid = len - sqrt(len);

51    low = mid + 1;

170   low = mid + 1;

192   low = mid + 1;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L47


## [G-16] Should Use Unchecked Block where Over/Underflow is not Possible

Issue
Since Solidity 0.8.0, all arithmetic operations revert on overflow and underflow by default.
However in places where overflow and underflow is not possible, it is better to use unchecked block to reduce the gas usage.
Reference: https://docs.soliditylang.org/en/v0.8.15/control-structures.html#checked-or-unchecked-arithmetic

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

286   return block.timestamp > action.minExecutionTime + expirationPeriod;

296   return action.creationTime + approvalPeriod;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L286

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

297    return block.timestamp > action.minExecutionTime + expirationPeriod;

307    return action.creationTime + approvalPeriod;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L297