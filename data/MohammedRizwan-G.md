## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Use nested if and, avoid multiple check combinations | 14 |
| [G&#x2011;02] | Save gas, by preventing zero value transfers | 1 |
| [G&#x2011;03] | Save gas, by refactoring code | 2 |

### [G&#x2011;01]  Use nested if and, avoid multiple check combinations
Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
It save 6 gas per instance, total of 84 gas can be saved.

There are 14 instances of this issue:

```solidity
File: contracts/PriceRegistry.sol

  modifier onlyAuthorized(LlamaExecutor llamaExecutor) {
    if (
      msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY
    ) revert UnauthorizedCaller();
    _;
  }
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaPolicyMetadataParamRegistry.sol#L19-L24)

```solidity
File: src/strategies/LlamaAbsoluteQuorum.sol

49    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

61    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/strategies/LlamaAbsoluteQuorum.sol#L49)

```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol

56      if (
        minDisapprovals != type(uint128).max
          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
      ) revert InsufficientDisapprovalQuantity();


68    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);


81    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/strategies/LlamaAbsolutePeerReview.sol#LL81C1-L81C101)

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

216    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

221    if (role != approvalRole && !forceApprovalRole[role]) return 0;

231    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

240    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/strategies/LlamaRelativeQuorum.sol#L240)

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

74      if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/llama-scripts/LlamaGovernanceScript.sol#L74)

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

213    if (role != approvalRole && !forceApprovalRole[role]) return 0;

230    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/strategies/LlamaAbsoluteStrategyBase.sol#LL213C1-L213C68)

### Recommended Mitigation steps
Use nested if.

For example:

```solidity

modifier onlyAuthorized(LlamaExecutor llamaExecutor) {
  if (msg.sender != address(llamaExecutor)) {
    if (msg.sender != address(ROOT_LLAMA_EXECUTOR)) {
      if (msg.sender != LLAMA_FACTORY) {
        revert UnauthorizedCaller();
      }
    }
  }
  _;
}
```

### [G&#x2011;02]  Save gas, by preventing zero value transfers
While transferring tokens, It is recommended to have zero value check validation to prevent the wastage of gas. This will save gas of end user.

There is 1 instance of this issue:

```solidity
File: src/LlamaExecutor.sol

  function execute(address target, uint256 value, bool isScript, bytes calldata data)
    external
    returns (bool success, bytes memory result)
  {
    if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
  }
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaExecutor.sol#L29-L35)

### Recommended Mitigation steps

```solidity
File: src/LlamaExecutor.sol

  function execute(address target, uint256 value, bool isScript, bytes calldata data)
    external
    returns (bool success, bytes memory result)
  {
+   require(value != 0, "invalid value");
    if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
  }
```

### [G&#x2011;03]  Save gas, by refactoring code
Following code can be refactored which will reduce bytes which can save some gas. Less storage bytes, less deployment cost.

There are 2 istances of this issue:

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

171    approvalRole = strategyConfig.approvalRole;
172    _assertValidRole(strategyConfig.approvalRole, numRoles);
173
174    disapprovalRole = strategyConfig.disapprovalRole;
175    _assertValidRole(strategyConfig.disapprovalRole, numRoles);
```

### Recommended Mitigation steps

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

    approvalRole = strategyConfig.approvalRole;
-    _assertValidRole(strategyConfig.approvalRole, numRoles);
+    _assertValidRole(approvalRole, numRoles);

    disapprovalRole = strategyConfig.disapprovalRole;
-    _assertValidRole(strategyConfig.disapprovalRole, numRoles);
+    _assertValidRole(disapprovalRole, numRoles);
```