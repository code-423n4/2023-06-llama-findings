# Gas Optimization

# Summary

| Number | Optimization Details                                                              | Context |
| :----: | :-------------------------------------------------------------------------------- | :-----: |
| [G-01] | Use  `calldata` instead of memory for `read-only` arguments in external functions |    5    |
| [G-02] | `abi.encode()` is less efficient than `abi.encodePacked()`                        |    3    |
| [G-03] | State variables can be packed to use fewer `storage` slots                        |    1    |
| [G-04] | Use assembly to check for `address(0)`                                            |   11    |
| [G-05] | Using `storage` instead of memory for `structs/arrays` saves gas                  |    3    |
| [G-06] | bytes `constants` are more eficient than string constans                          |    1    |
| [G-07] | `>=` costs less gas than `>`                                                      |    9    |
| [G-08] | Can make the `variable outside` the loop to save gas                              |    8    |
| [G-09] | Duplicated `require()/if()` checks should be refactored to a modifier or function |
|   3    |
| [G-10] | Use nested if statements instead of `&&`                                          |   14    |
| [G-11] | Use hardcode address instead `address(this)`                                      |    6    |
| [G-12] | Use != 0 instead of > 0 for unsigned integer comparison                           |   13    |
| [G-13] | With assembly, `.call` (bool success) transfer can be done gas-optimized          |    3    |
| [G-14] | Use constants instead of `type(uintx).max`                                        |   12    |
| [G-15] | Should Use `Unchecked` Block where `Over/Underflow` is not Possible               |    4    |
| [G-16] | Amounts should be checked for 0 before calling a `transfer`                       |    1    |
| [G-17] | Use `assembly` to hash instead of Solidity                                        |    3    |
| [G-18] | Use `Mappings` Instead of Arrays                                                  |    1    |
| [G-19] | Using `ERC721A` instead of `ERC721` for more gas-efficient                        |    1    |

## [G-01] Use  `calldata` instead of memory for `read-only` arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 \* <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.
Some methods are declared as external but the arguments are defined as memory instead of as calldata.

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130

```solidity
File: /src/accounts/LlamaAccount.sol
130   function initialize(bytes memory config) external initializer {
131    llamaExecutor = address(LlamaCore(msg.sender).executor());
132    Config memory accountConfig = abi.decode(config, (Config));
133    name = accountConfig.name;
134  }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L153

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

153  function initialize(bytes memory config) external virtual initializer {

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

156   function initialize(bytes memory config) external initializer {

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

```solidity
File:    src/LlamaFactory.sol
154      function deploy(
154    string memory name,
154    ILlamaStrategy strategyLogic,
154    ILlamaAccount accountLogic,
154    bytes[] memory initialStrategies,
154    bytes[] memory initialAccounts,
154    RoleDescription[] memory initialRoleDescriptions,
154    RoleHolderData[] memory initialRoleHolders,
154    RolePermissionData[] memory initialRolePermissions,
154    string memory color,
154    string memory logo
  )



227       function _deploy(
228    string memory name,
229    ILlamaStrategy strategyLogic,
230    ILlamaAccount accountLogic,
231    bytes[] memory initialStrategies,
232    bytes[] memory initialAccounts,
233    RoleDescription[] memory initialRoleDescriptions,
234    RoleHolderData[] memory initialRoleHolders,
235    RolePermissionData[] memory initialRolePermissions
236  )

```

## [G-02] `abi.encode()` is less efficient than `abi.encodePacked()`

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
File: /src/LlamaCore.sol

242   return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));


529   bytes32 permissionId = keccak256(abi.encode(permission));


708    abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

```

## [G-03] State variables can be packed to use fewer `storage` slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

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

### Recommend code

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

## [G-04] Use Assembly To Check For `address(0)`

it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```solidity

contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);

        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)

            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)

            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```

In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

```solidity
File: /src/LlamaPolicy.sol

181   if (llamaExecutor != address(0)) revert AlreadyInitialized();

405   if (llamaExecutor == address(0)) return; // Skip check during initialization.
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
File: /src/accounts/LlamaAccount.sol

148   if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();

166   if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();

199   if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();

247   if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();

256   if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

```solidity
File: /src/lib/ERC721NonTransferableMinimalProxy.sol


45   require(owner != address(0), "ZERO_ADDRESS");

90   require(to != address(0), "INVALID_RECIPIENT")

128  require(to != address(0), "INVALID_RECIPIENT");

145  require(owner != address(0), "NOT_MINTED");
```

## [G-05] Using `storage` instead of memory for `structs/arrays` saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
File: /src/accounts/LlamaAccount.sol

132   Config memory accountConfig = abi.decode(config, (Config));
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

154        Config memory strategyConfig = abi.decode(config, (Config));

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

157       Config memory strategyConfig = abi.decode(config, (Config));
```

## [G-06] bytes `constants` are more eficient than string constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L183

```solidity
File: /src/LlamaCore.sol

183    string public name;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L225

```solidity
File: src/LlamaCore.sol

225    string memory _name,
```

## [G-07] `>=` costs less gas than `>`

In Solidity, comparing values using the >= operator can sometimes be more gas-efficient than using the > operator. This is because the >= operator is implemented in a way that can sometimes require fewer computational steps than the > operator.

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L35

```solidity
File:     src/strategies/LlamaAbsoluteQuorum.sol

35         if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L305

```solidity
File:    src/strategies/LlamaAbsoluteStrategyBase.sol

305       if (role > numRoles) revert RoleNotInitialized(role);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

```soidity
File:   src/LlamaPolicy.sol

237     if (role > numRoles) revert RoleNotInitialized(role);

284     if (start > end) revert InvalidIndices();

286     if (end > checkpointsLength) revert InvalidIndices();

414     if (role > numRoles) revert RoleNotInitialized(role);

491     if (role > numRoles) revert RoleNotInitialized(role);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File:    src/strategies/LlamaRelativeQuorum.sol

230      if (minDisapprovalPct > ONE_HUNDRED_IN_BPS) revert DisapprovalDisabled();

325      if (role > numRoles) revert RoleNotInitialized(role);
```

## [G-08] Can make the `variable outside` the loop to save gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```solidity

contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;

        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }

        return total;
    }
}
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
File: /src/LlamaCore.sol

637    bytes32 salt = bytes32(keccak256(strategyConfigs[i]));

657    bytes32 salt = bytes32(keccak256(accountConfigs[i]));
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
File: /src/llama-scripts/LlamaGovernanceScript.sol

72    bool addressIsCore = targets[i] == address(core);

73    bool addressIsPolicy = targets[i] == address(policy);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

178   uint8 role = strategyConfig.forceApprovalRoles[i];

186   uint8 role = strategyConfig.forceDisapprovalRoles[i];
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

178    uint8 role = strategyConfig.forceApprovalRoles[i];

186    uint8 role = strategyConfig.forceDisapprovalRoles[i];
```

## [G-09] Duplicated `if()` checks should be refactored to a modifier or function

sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
File: /src/LlamaCore.sol

298   if (signer == address(0) || signer != policyholder) revert InvalidSignature();

389   if (signer == address(0) || signer != policyholder) revert InvalidSignature();

422   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
```

Recommendation
You can consider adding a modifier like below

```solidity

 modifer check (address checkToAddress) {
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");
      _;
 }
```

## [G-10] Use nested if statements instead of `&&`

If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) {
       if (input<10 && input>0 && input!=6){
           return "If condition passed";
       }

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) {
    if (input<10) {
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
File: /src/LlamaCore.sol

629    if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

649     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
File: /src/llama-scripts/LlamaGovernanceScript.sol

74   if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol

```solidity
File: /src/strategies/LlamaAbsolutePeerReview.sol

56   if (
57        minDisapprovals != type(uint128).max
58          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
59      ) revert InsufficientDisapprovalQuantity();


68  if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

81  if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol

```solidity
File: /src/strategies/LlamaAbsoluteQuorum.sol

36    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

49    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

61        if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L178

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

230    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

216       if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

221        if (role != approvalRole && !forceApprovalRole[role]) return 0;

231        if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

240        if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
```

## [G-11] Use hardcode address instead `address(this)`

it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```solidity

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

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
File: /src/LlamaCore.sol

445       if (target == address(this) || target == address(policy)) revert RestrictedAddress();

455       if (script == address(this) || script == address(policy)) revert RestrictedAddress();

708       abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
File: /src/accounts/LlamaAccount.sol

200        erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);

249        address(this), erc1155Data.recipient, erc1155Data.tokenId, erc1155Data.amount, erc1155Data.data

258      address(this),
```

## [G-12] Use `!= 0` instead of `> 0` for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
while the > 0 comparison requires an additional subtraction operation.
As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```solidity

contract MyContract {
    uint256 public myUnsignedInteger;

    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
File: /src/LlamaCore.sol

229     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

249     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

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

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

215   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

232   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

223   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```

## [G-13] With assembly, `.call` (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol

```solidity
File: /src/LlamaExecutor.sol

34    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L326

```solidity
File: /src/accounts/LlamaAccount.sol

326    (success, result) = target.call{value: msg.value}(callData);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L75

```solidity
File: /src/llama-scripts/LlamaGovernanceScript.sol

75   (bool success, bytes memory response) = targets[i].call(data[i]);
```

## [G-14] Use constants instead of `type(uintx).max`

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```solidity

contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;

    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");

        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L617

```solidity
File: /src/LlamaCore.sol

617       if (currentCount == type(uint128).max || quantity == type(uint128).max) return type(uint128).max;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L77

```solidity
File: /src/lib/Checkpoints.sol

77    return push(self, quantity, type(uint64).max);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol

```solidity
File: /src/lib/LlamaUtils.sol

11   if (n > type(uint64).max) revert UnsafeCast(n);

17   if (n > type(uint128).max) revert UnsafeCast(n);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol

```solidity
File: /src/strategies/LlamaAbsolutePeerReview.sol
57    minDisapprovals != type(uint128).max

79    if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

36    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

60   if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

215     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

232     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

223    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```

## [G-15] Should Use `Unchecked` Block where `Over/Underflow` is not Possible

Since Solidity 0.8.0, all arithmetic operations revert on overflow and underflow by default.
However in places where overflow and underflow is not possible, it is better to use unchecked block to reduce the gas usage.

Reference: https://docs.soliditylang.org/en/v0.8.15/control-structures.html#checked-or-unchecked-arithmetic

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
File: /src/strategies/LlamaAbsoluteStrategyBase.sol

286   return block.timestamp > action.minExecutionTime + expirationPeriod;

296   return action.creationTime + approvalPeriod;

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol

297    return block.timestamp > action.minExecutionTime + expirationPeriod;

307    return action.creationTime + approvalPeriod;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L47

## [G-16] Amounts should be checked for 0 before calling a `transfer`

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```solidity

function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L167

```solidity
File: /src/accounts/LlamaAccount.sol

167   erc20Data.token.safeTransfer(erc20Data.recipient, erc20Data.amount);

```

## [G-17] Use `assembly` to hash instead of Solidity

```solidity

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

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
529    bytes32 permissionId = keccak256(abi.encode(permission));


707    return keccak256(
      abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
    );

727    bytes32 createActionHash = keccak256(
728      abi.encode(
729        CREATE_ACTION_TYPEHASH,
730        policyholder,
731        role,
732        address(strategy),
733        target,
734        value,
735        keccak256(data),
736        keccak256(bytes(description)),
737        nonce
738      )
739    );
```

## [G-18] Use `Mappings` Instead of Arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L22

```solidity
File: /src/LlamaPolicyMetadata.sol

22 string[21] memory parts;
```

## [G-19] Using `ERC721A` instead of `ERC721` for more gas-efficient

ERC721A is a proposed extension to the ERC721 standard that aims to improve gas efficiency and reduce the cost of deploying and using non-fungible tokens (NFTs) on the Ethereum blockchain.

Reference : https://nextrope.com/erc721-vs-erc721a-2/

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

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
