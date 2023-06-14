
# Summary



# Gas Optimization    
no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |  Amounts should be checked for 0 before calling a transfer | 1 |
| [G-02] |  Use assembly for math (add, sub, mul, div) | 4 |
| [G-03] |  Use assembly to check for address(0) | 18 |  
| [G-04] |  Should Use Unchecked Block where Over/Underflow is not Possible | 4 |
| [G-05] |  bytes constants are more eficient than string constans| 2 |
| [G-06] |  Using storage instead of memory for structs/arrays saves gas | 1 |
| [G-07] |  Use Mappings Instead of Arrays | 4 |
| [G-08] |  With assembly, .call (bool success) transfer can be done gas-optimized | 5 |
| [G-09] |  Use assembly to hash instead of Solidity | 3 |
| [G-10] |  Use nested if statements instead of &&- (saves 8 gas per &&) | 9 |
| [G-11] |  Use != 0 instead of > 0 for unsigned integer comparison | 12 |
| [G-12] |  State variables can be packed to use fewer storage slots | 1 |
| [G-13] |  Using calldata instead of memory for read-only arguments in external functions saves gas| 4 |
| [G-14] |  Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate | 3 |
| [G-15] |  >= costs less gas than >  | 12 |
| [G-16] |  Can Make The Variable Outside The Loop To Save Gas| 8 |
| [G-17] |  Assigning keccak operations to constant variables results in extra gas costs | 5 |
| [G-18] |  Use hardcode address instead address(this) | 6 |
| [G-19] |  Duplicated require()/if() checks should be refactored to a modifier or function| 13 |
| [G-20] |  abi.encode() is less efficient than abi.encodePacked() | 3 |
| [G-21] |  Use constants instead of type(uintx).max | 10 |


## [G-01] Amounts should be checked for 0 before calling a transfer
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
File: /src/accounts/LlamaAccount.sol
167   erc20Data.token.safeTransfer(erc20Data.recipient, erc20Data.amount);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L167


## [G-02] Use assembly for math (add, sub, mul, div)
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
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L51
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L170
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L192




## [G-03] Use assembly to check for address(0)

Using assembly code to check for the zero address can sometimes be more gas-efficient than using the address type, especially if the operation is being performed frequently or in a loop. However, it's important to note that using assembly code can introduce additional complexity into the contract code, and should only be used if the potential gas savings outweigh the added complexity.

```solidity
file:     src/lib/ERC721NonTransferableMinimalProxy.sol
41        require((owner = _ownerOf[id]) != address(0), "NOT_MINTED");
45        require(owner != address(0), "ZERO_ADDRESS");
90        require(to != address(0), "INVALID_RECIPIENT");
128       require(to != address(0), "INVALID_RECIPIENT");
          require(_ownerOf[id] == address(0), "ALREADY_MINTED");
145       require(owner != address(0), "NOT_MINTED");
166           require(
      to.code.length == 0
        || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, "")
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#LL41C4-L41
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#LL45C4-L45
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#LL90C4-L90
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#LL128C1-L130
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#LL145C5-L145
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#LL166C1-L168


```solidity
file:     src/accounts/LlamaAccount.sol
148       if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();
166       if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();
199       if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();
247       if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#LL148C4-L148
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#LL166C5-L166
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#LL199C5-L199
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#LL247C3-L247

```solidity
file:      src/LlamaCore.sol
298        if (signer == address(0) || signer != policyholder) revert InvalidSignature();
330        if (guard != ILlamaActionGuard(address(0))) guard.validatePreActionExecution(actionInfo);
339        if (guard != ILlamaActionGuard(address(0))) guard.validatePostActionExecution(actionInfo);
389        if (signer == address(0) || signer != policyholder) revert InvalidSignature();
422        if (signer == address(0) || signer != policyholder) revert InvalidSignature();
550        if (guard != ILlamaActionGuard(address(0))) guard.validateActionCreation(actionInfo);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL298C4-L298
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL330C5-L330
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL339C5-L339
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL389C5-L389C83
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL422C4-L422
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL550C7-L550


```solidity
file:    src/LlamaPolicy.sol
181      if (llamaExecutor != address(0)) revert AlreadyInitialized();
405      if (llamaExecutor == address(0)) return; 
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL181C5-L182
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL405C5-L405

## [G-04] Should Use Unchecked Block where Over/Underflow is not Possible

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
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L296


```solidity
File: /src/strategies/LlamaRelativeQuorum.sol
297    return block.timestamp > action.minExecutionTime + expirationPeriod;

307    return action.creationTime + approvalPeriod;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L297
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L307


## [G-05] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
File: /src/LlamaCore.sol
183    string public name;

225    string memory _name,
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L183
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L225



## [G-06] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity
file:    src/LlamaPolicy.sol
290      Checkpoints.Checkpoint[] memory checkpoints = new Checkpoints.Checkpoint[](sliceLength);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL290C5-L290



## [G-07]  Use Mappings Instead of Arrays
There are two data types to describe lists of data in Solidity, arrays and maps, and their syntax and structure are quite different, allowing each to serve a distinct purpose. While arrays are packable and iterable, mappings are less expensive.



```solidity
file:     src/strategies/LlamaAbsoluteStrategyBase.sol
37        uint8[] forceApprovalRoles; // Anyone with this role can single-handedly approve an action.
38        uint8[] forceDisapprovalRoles;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#LL37C1-L38

```solidity
file:     src/accounts/LlamaAccount.sol
73        uint256[] tokenIds; // The tokenId of the token to transfer.
74          uint256[] amounts

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#LL73C1-L74


## [G-08] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
file: src/LlamaExecutor.sol
34    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol#L34


```solidity
File: /src/accounts/LlamaAccount.sol
326    (success, result) = target.call{value: msg.value}(callData);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L326


```solidity
File: /src/llama-scripts/LlamaGovernanceScript.sol
75   (bool success, bytes memory response) = targets[i].call(data[i]);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L72

```solidity
file: /src/LlamaCore.sol

333    (bool success, bytes memory result) =
334      executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L333-L334
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L334



## [G-09] Use assembly to hash instead of Solidity

```solidity
file:  src/LlamaCore.sol
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
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L707
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L727


## [G-10] Use nested if statements instead of &&- (saves 8 gas per &&)
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.
Instead of using the && operator in a single if statement to check multiple conditions,using multiple if statements with 1 condition per if statement will save 8 GAS per && The gas difference would only be realized if the revert condition is realized(met). The Gas saved could be higher as evident from the first instance due to refactoring which condition is checked first.

```solidity
file:    src/strategies/LlamaAbsoluteQuorum.sol
36       if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply)
49       if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
61       if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#LL36C5-L36
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#LL49C5-L49
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#LL61C5-L61


```solidity
file:  src/strategies/LlamaAbsoluteStrategyBase.sol
230    if (role != disapprovalRole && !forceDisapprovalRole[role]) 

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#LL230C5-L230

```solidity
file:    src/LlamaCore.sol
629       if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) 
649       if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic))


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL629C5-L629
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL649C4-L649


```solidity

file:    src/strategies/LlamaRelativeQuorum.sol
221      if (role != approvalRole && !forceApprovalRole[role])
231      if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
240      if (role != disapprovalRole && !forceDisapprovalRole[role])
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL221C4-L221
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL231C5-L231
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL240C5-L240



## [G-11] Use != 0 instead of > 0 for unsigned integer comparison
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
```solidity
File: /src/LlamaCore.sol
629     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

649     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {    
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L649

```solidity
File: /src/LlamaPolicy.sol
307   return quantity > 0;

313   return quantity > 0;

320   return quantity > 0 && canCreateAction[role][permissionId];

326   return quantity > 0 && block.timestamp > expiration;

425   bool case1 = quantity > 0 && expiration > block.timestamp;

444   bool hadRole = initialQuantity > 0;



```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L307
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L313
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L320
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L326
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L425
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L444



```solidity
file: src/strategies/LlamaAbsoluteStrategyBase.sol
215   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

232   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L232

```solidity
File: /src/strategies/LlamaRelativeQuorum.sol
223   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

242   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L242

## [G-12] State variables can be packed to use fewer storage slots
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


## [G-13] Using calldata instead of memory for read-only arguments in external functions saves gas

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
```solidity
file: /src/accounts/LlamaAccount.sol

130    function initialize(bytes memory config) external initializer {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130

```solidity
file: /src/strategies/LlamaRelativeQuorum.sol

156      function initialize(bytes memory config) external initializer {

``` 
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156

## [G-14]  Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Note missed from Automate:

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.


```solidity
file:    src/strategies/LlamaAbsoluteStrategyBase.sol
133      mapping(uint8 => bool) public forceApprovalRole;

  /// @notice Mapping of roles that can force an action to be disapproved.
  mapping(uint8 => bool) public forceDisapprovalRole;


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#LL133C1-L137

```solidity
file:   src/LlamaFactory.sol
86      mapping(ILlamaStrategy => bool) public authorizedStrategyLogics;

  /// @notice Mapping of all authorized Llama account implementation (logic) contracts.
  mapping(ILlamaAccount => bool) public authorizedAccountLogics;


```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#LL86C1-L90

```solidity
file:    src/LlamaPolicyMetadataParamRegistry.sol
47        mapping(LlamaExecutor => string) public color;

  /// @notice Mapping of Llama instance to logo for SVG.
  mapping(LlamaExecutor => string) public logo;

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#LL47C1-L50


## [G-15] >= costs less gas than > 


In Solidity, comparing values using the >= operator can sometimes be more gas-efficient than using the > operator. This is because the >= operator is implemented in a way that can sometimes require fewer computational steps than the > operator.

```solidity
file:   src/strategies/LlamaAbsolutePeerReview.sol
53      if (minApprovals > approvalPolicySupply - actionCreatorApprovalRoleQty) revert InsufficientApprovalQuantity();

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#LL53C7-L53

```solidity
file:     src/strategies/LlamaAbsoluteQuorum.sol
35         if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();
    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) 

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#LL35C2-L36

```solidity
file:    src/strategies/LlamaAbsoluteStrategyBase.sol
162      if (strategyConfig.minApprovals > policy.getRoleSupplyAsQuantitySum(strategyConfig.approvalRole))
305       if (role > numRoles) revert RoleNotInitialized(role);

```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#LL162C5-L162
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#LL305C5-L305

```soidity
file:   src/LlamaPolicy.sol
237     if (role > numRoles) revert RoleNotInitialized(role);
284     if (start > end) revert InvalidIndices();
286     if (end > checkpointsLength) revert InvalidIndices();
414     if (role > numRoles) revert RoleNotInitialized(role);
491     if (role > numRoles) revert RoleNotInitialized(role);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL237C5-L237
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL284C5-L285
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL286C5-L287
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL414C4-L414
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL491C5-L491

```solidity
file:    src/strategies/LlamaRelativeQuorum.sol
165      if (strategyConfig.minApprovalPct > ONE_HUNDRED_IN_BPS) revert InvalidMinApprovalPct(minApprovalPct);
230      if (minDisapprovalPct > ONE_HUNDRED_IN_BPS) revert DisapprovalDisabled();
325      if (role > numRoles) revert RoleNotInitialized(role);
```
https://giabi.encode() is less efficient than abi.encodePacked()thub.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL165C5-L165
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL230C5-L230
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL325C5-L325



## [G-16] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

```solidity
File: /src/LlamaCore.sol
637    bytes32 salt = bytes32(keccak256(strategyConfigs[i]));

657    bytes32 salt = bytes32(keccak256(accountConfigs[i]));
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L637
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L657

```solidity
File: /src/llama-scripts/LlamaGovernanceScript.sol
72    bool addressIsCore = targets[i] == address(core);

73    bool addressIsPolicy = targets[i] == address(policy);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L72

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L73

```solidity
file:  src/strategies/LlamaAbsoluteStrategyBase.sol
178   uint8 role = strategyConfig.forceApprovalRoles[i];

186   uint8 role = strategyConfig.forceDisapprovalRoles[i];
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L178

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L186

```solidity
file:  src/strategies/LlamaRelativeQuorum.sol
178    uint8 role = strategyConfig.forceApprovalRoles[i];

186    uint8 role = strategyConfig.forceDisapprovalRoles[i];
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L178

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L186




## [G-17] Assigning keccak operations to constant variables results in extra gas costs

```solidity
file:
142       bytes32 internal constant EIP712_DOMAIN_TYPEHASH =
    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

146       bytes32 internal constant CREATE_ACTION_TYPEHASH = keccak256(
    "CreateAction(address policyholder,uint8 role,address strategy,address target,uint256 value,bytes data,string description,uint256 nonce)"
  );

151   bytes32 internal constant CAST_APPROVAL_TYPEHASH = keccak256(
    "CastApproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"
  );

156   bytes32 internal constant CAST_DISAPPROVAL_TYPEHASH = keccak256(
    "CastDisapproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"
  );
161     bytes32 internal constant ACTION_INFO_TYPEHASH = keccak256(
    "ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"
  );

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL142C1-L144
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL146C1-L148
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL151C3-L153
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL156C3-L158
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#LL161C2-L163
 
###  Recommended Mitigation Steps

Change the variable from constant to immutable

## [G-18] Use hardcode address instead address(this)
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

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)


```solidity
File: /src/LlamaCore.sol
445       if (target == address(this) || target == address(policy)) revert RestrictedAddress();

455       if (script == address(this) || script == address(policy)) revert RestrictedAddress();

708       abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708

```solidity
File: /src/accounts/LlamaAccount.sol
200        erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);

249        address(this), erc1155Data.recipient, erc1155Data.tokenId, erc1155Data.amount, erc1155Data.data

258      address(this),
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L200
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L249
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L258


## [G-19] Duplicated require()/if() checks should be refactored to a modifier or function
to reduce code duplication and improve readability.
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.



```solidity
File: /src/LlamaCore.sol
298   if (signer == address(0) || signer != policyholder) revert InvalidSignature();

389   if (signer == address(0) || signer != policyholder) revert InvalidSignature();

422   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L389
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L422

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




## [G-20] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

```solidity
File: /src/LlamaCore.sol
242   return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));

529   bytes32 permissionId = keccak256(abi.encode(permission));

708    abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L242
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L529
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708


## [G-21] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
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

```solidity
File: /src/LlamaCore.sol
617       if (currentCount == type(uint128).max || quantity == type(uint128).max) return type(uint128).max;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L637

```solidity
file:   src/lib/Checkpoints.sol
77      return push(self, quantity, type(uint64).max);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L77

```solidity
file:   src/lib/LlamaUtils.sol
11      if (n > type(uint64).max) revert UnsafeCast(n);

17      if (n > type(uint128).max) revert UnsafeCast(n);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol#11
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol#L17

```solidity
File: /src/strategies/LlamaAbsolutePeerReview.sol
57    minDisapprovals != type(uint128).max

79    if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L57
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L79

```solidity
file:  src/strategies/LlamaRelativeQuorum.sol
223     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity; 
242     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL223C5-L223
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#LL242C4-L242


```solidity
file:   src/strategies/LlamaAbsoluteStrategyBase.sol
215     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

232     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L232


