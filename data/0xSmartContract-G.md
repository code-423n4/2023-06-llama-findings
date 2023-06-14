###  Gas Optimizations List
 --------------------------------------------------
- [x] **Gas 01** → Gas Optimization for the execute Function
- [x] **Gas 02** → Remove unnecessary ``code.length`` check
- [x] **Gas 03** → Gas overflow during iteration (DoS)
- [x] **Gas 04** → Reduce Gas Costs by Emitting Events Outside of For Loops
- [x] **Gas 05** → Use ``assembly`` to write _address storage values_ 
- [x] **Gas 06** → Use nested if and, avoid multiple check combinations
- [x] **Gas 07** → ``>=`` costs less gas than ``>``


### [G-01] Gas Optimization for the execute Function

The execute() function can be optimized by splitting it into two functions: executeWithScript() and executeWithNotScript()

Such a use provides gas optimization as a result of the elimination of unnecessary controls. In addition, code readability will be ensured.

```solidity
src\LlamaExecutor.sol:

  29:   function execute(address target, uint256 value, bool isScript, bytes calldata data)
  30:     external
  31:     returns (bool success, bytes memory result)
  32:   {
  33:     if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
  34:     (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
  35:   }
  
  ```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol#L29-L35

 
```diff
src\LlamaExecutor.sol:

- 29:   function execute(address target, uint256 value, bool isScript, bytes calldata data)
- 30:     external
- 31:     returns (bool success, bytes memory result)
- 32:   {
- 33:     if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
- 34:     (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
- 35:   }


+       function executeWithScript(address _target, bytes calldata _data)
+         external
+         returns (bool success, bytes memory result)
+       {
+         if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
+         (success, result) = _target.delegatecall(_data);
+       }
 

+       function executeWithNotScript(address target, uint256 value, bytes calldata data)
+         external
+         returns (bool success, bytes memory result)
+       {
+         if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
+         (success, result) = target.call{value: value}(data);
+       }

```

### [G-02] Remove unnecessary ``code.length`` check

```solidity
src\LlamaFactory.sol:

  86:   mapping(ILlamaStrategy => bool) public authorizedStrategyLogics;

  183:   function authorizeStrategyLogic(ILlamaStrategy strategyLogic) external onlyRootLlama {
  184:     _authorizeStrategyLogic(strategyLogic);
  185:   }

  267:   function _authorizeStrategyLogic(ILlamaStrategy strategyLogic) internal {
  268:     authorizedStrategyLogics[strategyLogic] = true;
  269:     emit StrategyLogicAuthorized(strategyLogic);
  270:   }

```


In case of achievable mapping, ``code.length`` is greater than zero.

In this case, removing ``address(factory).code.length > 0`` in the _deployStrategies() function will provide gas optimization.


```solidity
src\LlamaCore.sol:

  625:   function _deployStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs)
  626:     internal
  627:     returns (ILlamaStrategy firstStrategy)
  628:   {
  629:     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {
  630:       // The only edge case where this check is skipped is if `_deployStrategies()` is called by root llama instance
  631:       // during Llama Factory construction. This is because there is no code at the Llama Factory address yet.
  632:       revert UnauthorizedStrategyLogic();
  633:     }

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629


**Recommendation Code:**
```diff
src\LlamaCore.sol:

  625:   function _deployStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs)
  626:     internal
  627:     returns (ILlamaStrategy firstStrategy)
  628:   {
- 629:     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {
+ 629:     if (!factory.authorizedStrategyLogics(llamaStrategyLogic)) {
  630:       // The only edge case where this check is skipped is if `_deployStrategies()` is called by root llama instance
  631:       // during Llama Factory construction. This is because there is no code at the Llama Factory address yet.
  632:       revert UnauthorizedStrategyLogic();
  633:     }

```


### [G-03] Gas overflow during iteration (DoS)

Each iteration of the cycle requires a gas flow. There may come a time when more gas than allocated is required to save a block. In this case, all iterations of the loop will fail.

There are 14 examples of the topic. The suggestion block is made for only one function.

8 results - 1 file:

```solidity
src\accounts\LlamaAccount.sol:

  154:   function batchTransferNativeToken(NativeTokenData[] calldata nativeTokenData) external onlyLlama {
  155:     uint256 length = nativeTokenData.length;
  156:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  157:       transferNativeToken(nativeTokenData[i]);
  158:     }
  159:   }
 

  172:   function batchTransferERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
  173:     uint256 length = erc20Data.length;
  174:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  175:       transferERC20(erc20Data[i]);
  176:     }
  177:   }


  187:   function batchApproveERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
  188:     uint256 length = erc20Data.length;
  189:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  190:       approveERC20(erc20Data[i]);
  191:     }
  192:   }


  205:   function batchTransferERC721(ERC721Data[] calldata erc721Data) external onlyLlama {
  206:     uint256 length = erc721Data.length;
  207:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  208:       transferERC721(erc721Data[i]);
  209:     }
  210:   }


  220:   function batchApproveERC721(ERC721Data[] calldata erc721Data) external onlyLlama {
  221:     uint256 length = erc721Data.length;
  222:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  223:       approveERC721(erc721Data[i]);
  224:     }
  225:   }



  235:   function batchApproveOperatorERC721(ERC721OperatorData[] calldata erc721OperatorData) external onlyLlama {
  236:     uint256 length = erc721OperatorData.length;
  237:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  238:       approveOperatorERC721(erc721OperatorData[i]);
  239:     }
  240:   }



  268:   function batchTransferMultipleERC1155(ERC1155BatchData[] calldata erc1155BatchData) external onlyLlama {
  269:     uint256 length = erc1155BatchData.length;
  270:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  271:       batchTransferSingleERC1155(erc1155BatchData[i]);
  272:     }
  273:   }



  283:   function batchApproveOperatorERC1155(ERC1155OperatorData[] calldata erc1155OperatorData) external onlyLlama {
  284:     uint256 length = erc1155OperatorData.length;
  285:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  286:       approveOperatorERC1155(erc1155OperatorData[i]);
  287:     }
  288:   }

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L154-L159
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L172-L177
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L187-L192
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L205-L210
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L220-L225
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L235-L240
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L268-L273
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L283-L288



**Recommendation Code:**

```diff
src\accounts\LlamaAccount.sol:

+  uint256 private constant ERC1155_OPERATOR_DATA_LENGTH = 333; 


  283:   function batchApproveOperatorERC1155(ERC1155OperatorData[] calldata erc1155OperatorData) external onlyLlama {
  284:     uint256 length = erc1155OperatorData.length;
+          if (erc1155OperatorData > ERC1155_OPERATOR_DATA_LENGTH) revert maxLength();  
  285:     for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
  286:       approveOperatorERC1155(erc1155OperatorData[i]);
  287:     }
  288:   }

```


### [G-04] Reduce Gas Costs by Emitting Events Outside of For Loops

Placing events inside loops results in the event being emitted at each iteration, which can significantly increase gas consumption. By moving events outside the loop and emitting them globally, unnecessary gas expenses can be minimized, leading to more efficient and cost-effective contract execution.


2 results - 1 file:

```solidity
src\LlamaCore.sol:

  636:     for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
  637:       bytes32 salt = bytes32(keccak256(strategyConfigs[i]));
  638:       ILlamaStrategy strategy = ILlamaStrategy(Clones.cloneDeterministic(address(llamaStrategyLogic), salt));
  639:       strategy.initialize(strategyConfigs[i]);
  640:       strategies[strategy] = true;
  641:       emit StrategyCreated(strategy, llamaStrategyLogic, strategyConfigs[i]);
  642:       if (i == 0) firstStrategy = strategy;
  643:     }


  656:     for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
  657:       bytes32 salt = bytes32(keccak256(accountConfigs[i]));
  658:       ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), salt));
  659:       account.initialize(accountConfigs[i]);
  660:       emit AccountCreated(account, llamaAccountLogic, accountConfigs[i]);
  661:     }

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L641
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L660


### [G-05] Use ``assembly`` to write _address storage values_ 

5 results - 4 files:

```solidity
src\LlamaCore.sol:

  232:     factory = LlamaFactory(msg.sender);

  235:     policy = _policy;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L232
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L235



```solidity
src\strategies\LlamaAbsoluteStrategyBase.sol:

  155:     llamaCore = LlamaCore(msg.sender);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L155



```solidity
src\strategies\LlamaRelativeQuorum.sol:

  158:     llamaCore = LlamaCore(msg.sender);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L158



```solidity
src\LlamaPolicy.sol:

  150:     factory = LlamaFactory(msg.sender);

 183:     llamaExecutor = _llamaExecutor; 

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L150
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L183


### [G-06] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

17 results - 7 files:

```solidity
src\LlamaCore.sol:

  629:     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

  649:     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629


```solidity
src\LlamaPolicyMetadataParamRegistry.sol:

  20:     if (
  21:       msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
  22:     ) revert UnauthorizedCaller();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L20-LL22


```solidity  
src\llama-scripts\LlamaGovernanceScript.sol:

  74:       if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L74



```solidity
src\strategies\LlamaAbsolutePeerReview.sol:

  56:       if (
  57:         minDisapprovals != type(uint128).max
  58:           && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
  59:       ) revert InsufficientDisapprovalQuantity();


  68:     if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

  81:     if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L56-L59
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L68
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L81



```solidity
src\strategies\LlamaAbsoluteQuorum.sol:

  36:     if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

  49:     if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

  61:     if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L36
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L49
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L61



```solidity
src\strategies\LlamaAbsoluteStrategyBase.sol:

  213:     if (role != approvalRole && !forceApprovalRole[role]) return 0;

  230:     if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L213
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L230



```solidity
src\strategies\LlamaRelativeQuorum.sol:

  216:     if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

  221:     if (role != approvalRole && !forceApprovalRole[role]) return 0;

  231:     if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

  240:     if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L216
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L221
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L231
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L240


**Recomendation Code:**
```diff
src\strategies\LlamaRelativeQuorum.sol:

- 216:     if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
+ 216:     if (role != approvalRole) {
+              if (!forceApprovalRole[role]) {
+                 revert InvalidRole(approvalRole);
+              }
+          }

```


### [G-07] ``>=`` costs less gas than ``>``

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

2 results - 2 files:

```solidity
src\strategies\LlamaAbsoluteStrategyBase.sol:

  215:     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

 232:     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L232



```solidity
src\strategies\LlamaRelativeQuorum.sol:

  223:     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

  242:     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```  
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L242