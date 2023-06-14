### Llama Project - QA Report Issues List

- [x] **Low 01** → It is stated in the documents that the project uses `timelock`, but there is no timelock function
- [x] **Low 02** → A single point of failure is not acceptable for this project.
- [x] **Low 03** → There may be problems with the use of Layer2
- [x] **Low 04** → Call and Delegatecall return values are not checked in `execute` function
- [x] **Low 05** → The description argument of the `createAction()` function has no input control
- [x] **Low 06** → Unnecessary use of "&&" in if condition
- [x] **Low 07**→ Project Upgrade and Stop Scenario should be
- [x] **Low 08**→ Missing Event for  initialize
- [x] **Non-Critical  09**→ Unused Import
- [x] **Non-Critical  10**→ Assembly Codes Specific – Should Have Comments



### [Low-01] It is stated in the documents that the project uses `timelock`, but there is no timelock function

It is stated in the documents that the project uses `timelock`, but there is no timelock function


```solidity
README.md:
  146: - Does it use a timelock function?: Yes
```

### [Low-02] A single point of failure is not acceptable for this project.


## Impact

Centrality risk is high in the project, the role of 'onlyLlama' detailed below has very critical and important powers
 
Project and funds may be compromised by a malicious or stolen private key `onlyLlama` msg.sender

```solidity

32 results - 3 files



src/LlamaCore.sol:

   68:   modifier onlyLlama() {
   69:     if (msg.sender != llamaExecutor) revert OnlyLlama();


  429:   function createStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs) external onlyLlama {
  436:   function createAccounts(ILlamaAccount llamaAccountLogic, bytes[] calldata accountConfigs) external onlyLlama {

  444:   function setGuard(address target, bytes4 selector, ILlamaActionGuard guard) external onlyLlama {

  454:   function authorizeScript(address script, bool authorized) external onlyLlama {



src/LlamaPolicy.sol:

   68:   modifier onlyLlama() {

   69:     if (msg.sender != llamaExecutor) revert OnlyLlama();



  190:   function initializeRole(RoleDescription description) external onlyLlama {

  199:   function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {

  207:   function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {

  222:   function revokePolicy(address policyholder) external onlyLlama {

  236:   function updateRoleDescription(uint8 role, RoleDescription description) external onlyLlama {



src/accounts/LlamaAccount.sol:

  103:   modifier onlyLlama() {

  104:     if (msg.sender != llamaExecutor) revert OnlyLlama();



  147:   function transferNativeToken(NativeTokenData calldata nativeTokenData) public onlyLlama {

  154:   function batchTransferNativeToken(NativeTokenData[] calldata nativeTokenData) external onlyLlama {

  165:   function transferERC20(ERC20Data calldata erc20Data) public onlyLlama {

  172:   function batchTransferERC20(ERC20Data[] calldata erc20Data) external onlyLlama {

  181:   function approveERC20(ERC20Data calldata erc20Data) public onlyLlama {

  187:   function batchApproveERC20(ERC20Data[] calldata erc20Data) external onlyLlama {

  198:   function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {

  205:   function batchTransferERC721(ERC721Data[] calldata erc721Data) external onlyLlama {

  214:   function approveERC721(ERC721Data calldata erc721Data) public onlyLlama {

  220:   function batchApproveERC721(ERC721Data[] calldata erc721Data) external onlyLlama {

  229:   function approveOperatorERC721(ERC721OperatorData calldata erc721OperatorData) public onlyLlama {

  235:   function batchApproveOperatorERC721(ERC721OperatorData[] calldata erc721OperatorData) external onlyLlama {

  246:   function transferERC1155(ERC1155Data calldata erc1155Data) external onlyLlama {

  255:   function batchTransferSingleERC1155(ERC1155BatchData calldata erc1155BatchData) public onlyLlama {

  268:   function batchTransferMultipleERC1155(ERC1155BatchData[] calldata erc1155BatchData) external onlyLlama {

  277:   function approveOperatorERC1155(ERC1155OperatorData calldata erc1155OperatorData) public onlyLlama {

  283:   function batchApproveOperatorERC1155(ERC1155OperatorData[] calldata erc1155OperatorData) external onlyLlama {

```



### [Low-03] There may be problems with the use of Layer2

According to the scope information of the project, it is stated that it can be used in rollup chains and popular EVM chains.

```js
README.md:
  150: - Does it use rollups?: It should be able to be deployed on rollup chains
  151: - Is it multi-chain?: It should be able to be deployed an any popular EVM chain

```

Some conventions in the project are set to Pragma ^0.8.19, allowing the conventions to be compiled with any 0.8.x compiler. The problem with this is that Arbitrum is [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected. The default behavior of the compiler will be to use the latest version which means it will compile with version 0.8.20 which will produce broken code by default.


### [Low-04] Call and Delegatecall return values are not checked in `execute` function

**Description:**
execute() function does not check the return value of the delegatecall or call functions.

```solidity

src/LlamaExecutor.sol:
  28    /// @return result The data returned by the function being called.
  29:   function execute(address target, uint256 value, bool isScript, bytes calldata data)
  30:     external
  31:     returns (bool success, bytes memory result)
  32:   {
  33:     if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
  34:     (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
  35:   }
  36: }

```


The `success` control is made in the LlmamaCore.sol contract, but it is best practice to have this control directly in the function itself, not in the calling function;

```
src/LlamaCore.sol:
  331  
  332:     // Execute action.
  333:     (bool success, bytes memory result) =
  334:       executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);
  335: 
  336:     if (!success) revert FailedActionExecution(result);
```


**Recommendation:**
The return value will be checked if the code is corrected as below;

```solidity
function execute(address target, uint256 value, bool isScript, bytes calldata data) external
  returns (bool success, bytes memory result)
{
  if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
  if (isScript) {
    (success, result) = target.delegatecall(data);
  } else {
    (success, result) = target.call{value: value}(data);
  }
  require(success, "Call failed");
}

```


### [Low-05] The description argument of the `createAction()` function has no input control



```diff

src/LlamaCore.sol:
  248  
  249:   /// @notice Creates an action. The creator needs to hold a policy with the permission ID of the provided
  250:   /// `(target, selector, strategy)`.
  251:   /// @dev Use `""` for `description` if there is no description.
  252:   /// @param role The role that will be used to determine the permission ID of the policyholder.
  253:   /// @param strategy The strategy contract that will determine how the action is executed.
  254:   /// @param target The contract called when the action is executed.
  255:   /// @param value The value in wei to be sent when the action is executed.
  256:   /// @param data Data to be called on the target when the action is executed.
  257:   /// @param description A human readable description of the action and the changes it will enact.
  258:   /// @return actionId Action ID of the newly created action.
  259:   function createAction(
  260:     uint8 role,
  261:     ILlamaStrategy strategy,
  262:     address target,
  263:     uint256 value,
  264:     bytes calldata data,
  265:     string memory description
  266:   ) external returns (uint256 actionId) {
+	if (bytes(description).length == 0) {
+	        description = ""; // Set description to empty string if it is not provided
+	}
  267:     actionId = _createAction(msg.sender, role, strategy, target, value, data, description);
  268:   }

```


### [Low-06] Unnecessary use of "&&" in if condition

The `&&` condition searches for two conditions;
1- address(factory).code.length > 0
2- !factory.authorizedStrategyLogics(llamaStrategyLogic))

If the !factory.authorizedStrategyLogics(llamaStrategyLogic) condition is met, the 'address(factory).code.length > 0' condition will already be fulfilled, so it is unnecessary

```diff

src/LlamaCore.sol:
  628    {
- 629:     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {
+             if (!factory.authorizedStrategyLogics(llamaStrategyLogic)) {
  630:       // The only edge case where this check is skipped is if `_deployStrategies()` is called by root llama instance
  631:       // during Llama Factory construction. This is because there is no code at the Llama Factory address yet.
  632:       revert UnauthorizedStrategyLogic();
  633:     }

```


### [Low-07] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

### [Low-08] Missing Event for  initialize

**Context:**
```solidity

12 results - 8 files

src/LlamaCore.sol:
  223    /// @return bootstrapPermissionId The permission ID that's used to set role permissions.
  224:   function initialize(
  225      string memory _name,

src/LlamaPolicy.sol:
  142    /// @param rolePermissions The `role`, `permissionId` and whether the role has the permission of the role permissions.
  143:   function initialize(
  144      string calldata _name,

  189    /// @notice Initializes a new role with the given role ID and description
  190:   function initializeRole(RoleDescription description) external onlyLlama {
  191      _initializeRole(description);

src/accounts/LlamaAccount.sol:
  129    /// @param config Llama account initialization configuration.
  130:   function initialize(bytes memory config) external initializer {
  131      llamaExecutor = address(LlamaCore(msg.sender).executor());

src/interfaces/ILlamaAccount.sol:
  17    /// different account logic contracts.
  18:   function initialize(bytes memory config) external;
  19  }

src/interfaces/ILlamaStrategy.sol:
  28    /// different strategies.
  29:   function initialize(bytes memory config) external;
  30  

src/llama-scripts/LlamaGovernanceScript.sol:
   84  
   85:   function initializeRolesAndSetRoleHolders(
   86      RoleDescription[] calldata description,

   92  
   93:   function initializeRolesAndSetRolePermissions(
   94      RoleDescription[] calldata description,

  100  
  101:   function initializeRolesAndSetRoleHoldersAndSetRolePermissions(
  102      RoleDescription[] calldata description,

  174  
  175:   function initializeRoles(RoleDescription[] calldata description) public onlyDelegateCall {
  176      (, LlamaPolicy policy) = _context();

src/strategies/LlamaAbsoluteStrategyBase.sol:
  152    /// @inheritdoc ILlamaStrategy
  153:   function initialize(bytes memory config) external virtual initializer {
  154      Config memory strategyConfig = abi.decode(config, (Config));

src/strategies/LlamaRelativeQuorum.sol:
  155    /// @inheritdoc ILlamaStrategy
  156:   function initialize(bytes memory config) external initializer {
  157      Config memory strategyConfig = abi.decode(config, (Config));

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [Non Critical-09] Unused Import

Some imports aren’t used inside the project

```solidity
src/strategies/LlamaAbsolutePeerReview.sol:
   4: import {Initializable} from "@openzeppelin/proxy/utils/Initializable.sol";
   6: import {FixedPointMathLib} from "@solmate/utils/FixedPointMathLib.sol";
  10: import {ActionState} from "src/lib/Enums.sol";
  11: import {LlamaUtils} from "src/lib/LlamaUtils.sol";

```


### [Non Critical-10] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does


This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.


```solidity
3 results - 2 files

src/accounts/LlamaAccount.sol:
  338    function _readSlot0() internal view returns (bytes32 slot0) {
  339:     assembly {
  340        slot0 := sload(0)

src/lib/Checkpoints.sol:
  204      {
  205:         assembly {
  206              mstore(0, self.slot)

  223      function sqrt(uint256 x) internal pure returns (uint256 z) {
  224:         assembly {
  225              let y := x // We start y at x, which will help us make our initial estimate.
```

