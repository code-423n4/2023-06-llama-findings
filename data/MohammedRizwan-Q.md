## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Array lengths not checked | 1 |
| [L&#x2011;02] | Missing signature parameter validations could lead to invalid signatures | 1 |
| [L&#x2011;03] | execute() function does not prevent target address from setting address(0) | 1 |


### Non-Critical Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [N&#x2011;01] | Missing important address check validations in initialize() function | 1 |
| [N&#x2011;02] | Use the latest version of openzeppelin library(v4.9.1) | 1 |


### Low Risk Issues
### [L&#x2011;01]  Array lengths not checked
If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

There is one instance of this issue:

```solidity
File: src/LlamaCore.sol

224  function initialize(
225    string memory _name,
226    LlamaPolicy _policy,
227    ILlamaStrategy _llamaStrategyLogic,
228    ILlamaAccount _llamaAccountLogic,
229    bytes[] calldata initialStrategies,
230    bytes[] calldata initialAccounts
231  ) external initializer returns (bytes32 bootstrapPermissionId) {
232    factory = LlamaFactory(msg.sender);
233    name = _name;
234    executor = new LlamaExecutor();
235    policy = _policy;
236
237    ILlamaStrategy bootstrapStrategy = _deployStrategies(_llamaStrategyLogic, initialStrategies);
238    _deployAccounts(_llamaAccountLogic, initialAccounts);
239
240    // Now we compute the permission ID used to set role permissions and return it.
241    bytes4 selector = LlamaPolicy.setRolePermission.selector;
242    return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));
243  }
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L224-L243)

### Recommended Mitigation steps
Ensure both the arrays have same length.

```solidity
File: src/LlamaCore.sol

  function initialize(
    string memory _name,
    LlamaPolicy _policy,
    ILlamaStrategy _llamaStrategyLogic,
    ILlamaAccount _llamaAccountLogic,
    bytes[] calldata initialStrategies,
    bytes[] calldata initialAccounts
  ) external initializer returns (bytes32 bootstrapPermissionId) {
+   require(initialStrategies.length == initialAccounts.length, "Arrays must have the same length");
    factory = LlamaFactory(msg.sender);
    name = _name;
    executor = new LlamaExecutor();
    policy = _policy;

    ILlamaStrategy bootstrapStrategy = _deployStrategies(_llamaStrategyLogic, initialStrategies);
    _deployAccounts(_llamaAccountLogic, initialAccounts);

    // Now we compute the permission ID used to set role permissions and return it.
    bytes4 selector = LlamaPolicy.setRolePermission.selector;
    return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));
  }
```

### [L&#x2011;02]  Missing signature parameter validations could lead to invalid signatures
The code utilizes the ecrecover function to verify the signature of the provided parameters. However, the signatures parametrs are not properly validated. To ensure that the v, r, and s parameters are valid and within the expected ranges. Verify that v is either 27 or 28, and that r and s are non-zero and within the curve order range.

There is 1 instance of this issue:

```solidity
File: src/LlamaCore.sol

  function createActionBySig(
    address policyholder,
    uint8 role,
    ILlamaStrategy strategy,
    address target,
    uint256 value,
    bytes calldata data,
    string memory description,
    uint8 v,
    bytes32 r,
    bytes32 s
  ) external returns (uint256 actionId) {
    bytes32 digest = _getCreateActionTypedDataHash(policyholder, role, strategy, target, value, data, description);
    address signer = ecrecover(digest, v, r, s);
    if (signer == address(0) || signer != policyholder) revert InvalidSignature();
    actionId = _createAction(signer, role, strategy, target, value, data, description);
  }
```
[Instance 1: Link to code](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L284-L300)

[Instance 2: Link to code](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L378-L391)

[Instance 3: Link to code](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L411-L424)

### Recommended Mitigation steps

```solidity
File: src/LlamaCore.sol

  function createActionBySig(
    address policyholder,
    uint8 role,
    ILlamaStrategy strategy,
    address target,
    uint256 value,
    bytes calldata data,
    string memory description,
    uint8 v,
    bytes32 r,
    bytes32 s
  ) external returns (uint256 actionId) {
+   require(v == 27 || v == 28, "Invalid v value");
+   require(r != 0, "Invalid r value");
+   require(s != 0, "Invalid s value");
    bytes32 digest = _getCreateActionTypedDataHash(policyholder, role, strategy, target, value, data, description);
    address signer = ecrecover(digest, v, r, s);
    if (signer == address(0) || signer != policyholder) revert InvalidSignature();
    actionId = _createAction(signer, role, strategy, target, value, data, description);
  }
```

### [L&#x2011;03]  execute() function does not prevent target address from setting address(0)
In LlamaExecutor.execute() function has a target address parameter but it does not prevent it from setting zero address. If the address(0) is set accidentally then there can be a loss of funds as the functions is capable to receive wei.

There is 1 instance of this issue:

```solidity
File: src/LlamaExecutor.sol

29  function execute(address target, uint256 value, bool isScript, bytes calldata data)
30    external
31    returns (bool success, bytes memory result)
32  {
33    if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
34    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
35  }
```

### Recommended Mitigation steps
Add address(0) check validation


```solidity
File: src/LlamaExecutor.sol

  function execute(address target, uint256 value, bool isScript, bytes calldata data)
    external
    returns (bool success, bytes memory result)
  {
+   require(target != address(0), "invalid address");
    if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
  }
```

### Non-Critical Issues
### [N&#x2011;01]  Missing important address check validations in initialize() function 
Since initialize() function can only be called once. It is important to prevent address from setting to zero address.

There are 3 instances of this issue:

```solidity
File: src/LlamaCore.sol

224  function initialize(
225    string memory _name,
226    LlamaPolicy _policy,
227    ILlamaStrategy _llamaStrategyLogic,
228    ILlamaAccount _llamaAccountLogic,
229    bytes[] calldata initialStrategies,
230    bytes[] calldata initialAccounts
231  ) external initializer returns (bytes32 bootstrapPermissionId) {
232    factory = LlamaFactory(msg.sender);
233    name = _name;
234    executor = new LlamaExecutor();
235    policy = _policy;
236
237    ILlamaStrategy bootstrapStrategy = _deployStrategies(_llamaStrategyLogic, initialStrategies);
238    _deployAccounts(_llamaAccountLogic, initialAccounts);
239
240    // Now we compute the permission ID used to set role permissions and return it.
241    bytes4 selector = LlamaPolicy.setRolePermission.selector;
242    return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));
243  }
```
[Link to code](https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L224-L243)

### Recommended Mitigation steps
Add zero address check validation. 

```solidity
File: src/LlamaCore.sol

  function initialize(
    string memory _name,
    LlamaPolicy _policy,
    ILlamaStrategy _llamaStrategyLogic,
    ILlamaAccount _llamaAccountLogic,
    bytes[] calldata initialStrategies,
    bytes[] calldata initialAccounts
  ) external initializer returns (bytes32 bootstrapPermissionId) {
+   require(address(_policy) != address(0), "Policy address cannot be zero");
+   require(address(_llamaStrategyLogic) != address(0), "Strategy logic address cannot be zero");
+   require(address(_llamaAccountLogic) != address(0), "Account logic address cannot be zero");
    factory = LlamaFactory(msg.sender);
    name = _name;
    executor = new LlamaExecutor();
    policy = _policy;

    ILlamaStrategy bootstrapStrategy = _deployStrategies(_llamaStrategyLogic, initialStrategies);
    _deployAccounts(_llamaAccountLogic, initialAccounts);

    // Now we compute the permission ID used to set role permissions and return it.
    bytes4 selector = LlamaPolicy.setRolePermission.selector;
    return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));
  }
```


### [N&#x2011;02]  Use the latest version of openzeppelin library(v4.9.1) 
The contracts has used v4.8.0 of openzeppelin library. There were securituy issues in old versions whihc are fixed in latest version. Recently openzeppelin has released version 4.9.1 which comes with most of the security patches and lots of gas optimizations. Recommend to use the latest version of openzeppelin library.

There is 1 instance of this issue.