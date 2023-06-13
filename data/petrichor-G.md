# Gas Optimization

## Summary 



|      |  Issue   |  Instance  |
|------|----------|------------|
|[G-01]| Save gas by declaring the variable outside the loop to reduce unnecessary memory allocation.| 10|
|[G-02]|Use nested if statements instead of &&|14|
|[G-03]|abi.encode() is less efficient than abi.encodePacked()|3||
|[G-04]|Use != 0 instead of > 0 for unsigned integer comparison |4|
|[G-05]| State variables can be packed to use fewer storage slots|1|
|[G-06]|Use hardcode address instead address(this)|6|
|[G-07]|Use Assembly To Check For address(0)|12|
|[G-08]|Use constants instead of type(uintx).max|12|
|[G-09]|Duplicated require()/if() checks should be refactored to a modifier or function|3|
|[G-10]|bytes constants are more eficient than string constans|2|
|[G-11]|Use assembly to hash instead of Solidity|3|
|[G-12]|With assembly, .call (bool success) transfer can be done gas-optimized|3|
|[G-13]|Using calldata instead of memory for read-only arguments in external functions saves gas | 6|
|[G-14]|Using storage instead of memory for structs/arrays saves gas|1|
|[G-15]|use Mappings Instead of Arrays|1|
|[G-16]| Use calldata instead of memory|5|
|[G-17]| Amounts should be checked for 0 before calling a transfer|1|
|[G-18]|>= costs less gas than > |9|


# Detials

## [G-01]  Save gas by declaring the variable outside the loop to reduce unnecessary memory allocation.

Declaring variables outside of loops in Solidity can help reduce gas usage because it avoids unnecessary memory allocation. By initializing a variable outside of a loop, it only needs to be allocated memory once, rather than every time the loop runs. This can significantly reduce the amount of gas required to execute a smart contract, which can be important for minimizing transaction costs and ensuring the efficient operation of the Ethereum network.

example:

function sumArray(uint[] memory array) public view returns (uint) {
    uint sum = 0; // declare variable outside loop
    for (uint i = 0; i < array.length; i++) {
        sum += array[i];
    }
    return sum;
}



```solodity
104   bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey[i]);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L104

```solidity
106   uint256 _mevTheftPenalty = calculateMEVTheftPenalty(pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L106

```solidity
107   uint256 _missedAttestationPenalty = calculateMissedAttestationPenalty(pubkeyRoot);
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/Penalty.sol#L107


```solidity
92    address operator = _permissionedNOs[i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L92

```solidity
157   address withdrawVault = IVaultFactory(vaultFactory).deployWithdrawVault(
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L92

```solidity
273   uint256 validatorId = validatorIdByPubkey[_frontRunPubkey[i]];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L273

```solidity
286   uint256 validatorId = validatorIdByPubkey[_invalidSignaturePubkey[i]];

```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L286


```solidity

535   uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L535

```solidity
318   uint256 validatorId = validatorIdByPubkey[_pubkeys``[i]];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L318


```solidity
638   uint256 validatorId = validatorIdsByOperatorId[operatorId][i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol#L638


```solodity
101  uint256 validatorToDeposit = selectedOperatorCapacity[i];
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L101

```solidity
110  uint256 index = nextQueuedValidatorIndex;
```
https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedPool.sol#L110




## [G-02] Use nested if statements instead of &&

To save gas, we can use nested if statements instead of the && operator. This allows us to evaluate each condition separately, and avoid unnecessary gas usage for evaluating conditions that are already known to be false. Here's an example:

function checkValues(uint256 x, uint256 y, uint256 z) public view returns (bool) {
    if (x > y) {
        if (y > z) {
            return true;
        }
    }
    return false;
}





```solidity
629    if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L629

```solidity
649     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {    
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L649

```solidity
74   if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L72

```solidity
56   if (
        minDisapprovals != type(uint128).max
          && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty
      ) revert InsufficientDisapprovalQuantity();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L56

```solidity
68  if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L68

```solidity
81  if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L81




```solidity
36    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L36
```solidity
49    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L49

```solidity
61        if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L61

```solidity
230    if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L230

```solidity
216       if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L216

```solidity
221        if (role != approvalRole && !forceApprovalRole[role]) return 0;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L221

```solidity
231        if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L231

```solidity
240        if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L240




## [G-03] abi.encode() is less efficient than abi.encodePacked()


In terms of gas usage, abi.encodePacked() is generally more efficient than abi.encode() because it does not add any unnecessary metadata or padding. When using abi.encode(), the function adds metadata to the output byte array, which includes the function signature and the length of each argument. This metadata can add additional bytes to the output, which can result in more gas usage.

```solidity
242   return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L242

```solidity
529   bytes32 permissionId = keccak256(abi.encode(permission));
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L529


```solidity
708    abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708





## [G-04]Use != 0 instead of > 0 for unsigned integer comparison

In Solidity, unsigned integers are represented as uint, and they are commonly used to represent numeric values such as amounts of tokens or ether. When comparing unsigned integers to zero, it is more gas-efficient to use the != 0 operator instead of the > 0 operator.



```solidity
File: /src/LlamaCore.sol
229     if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {
```    
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L229


```solidity
249     if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {    
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L249

```solidity
307   return quantity > 0;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L307



```solidity
313   return quantity > 0;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L313

```solidity
320   return quantity > 0 && canCreateAction[role][permissionId];
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L320


```solidity
326   return quantity > 0 && block.timestamp > expiration;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L326

```solidity
425   bool case1 = quantity > 0 && expiration > block.timestamp;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L425

```solidity
444   bool hadRole = initialQuantity > 0;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L444

```solidity
445   bool willHaveRole = quantity > 0;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L445



```solidity
215   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215


```solidity
232   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L232

```solidity
223   return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223

```solidiy
242   return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L242






## [G-05] State variables can be packed to use fewer storage slots

One way to reduce the cost of storage is by packing multiple state variables into a single storage slot. This can be achieved by using data types that fit within a single 32-byte slot, such as uint256 or bytes32, and then packing multiple variables of these types into a single storage variable.
example:
uint256 packed;

function pack() public {
    packed = (uint256(a) << 192) | (uint256(b) << 128) | (uint256(c) << 64) | uint256(d);
}

function unpack() public view returns (uint64, uint64, uint64, uint64) {
    uint64 a = uint64(packed >> 192);
    uint64 b = uint64(packed >> 128);
    uint64 c = uint64(packed >> 64);
    uint64 d = uint64(packed);
    return (a, b, c, d);
}




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
  
  ### Note the above code uses 4 slots in evm but if we arrange them in decending it will use 3 slots (128-64-8)




  ## [G-06] Use hardcode address instead address(this)

if you use a hardcoded address, there is no need for the EVM to execute a selfdestruct operation to obtain the contract address, which can potentially save gas.



example:
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function foo() public {
        // Use hardcoded address instead of address(this)
        SomeOtherContract(myAddress).bar();
    }
}


```solidity
445       if (target == address(this) || target == address(policy)) revert RestrictedAddress();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445

```solidity
455       if (script == address(this) || script == address(policy)) revert RestrictedAddress();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455

```solidity
708       abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708

```solidity
200        erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L200


```solidity
249        address(this), erc1155Data.recipient, erc1155Data.tokenId, erc1155Data.amount, erc1155Data.data
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L249

```solidity
258      address(this),
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L258



## [G-07] Use Assembly To Check For address(0)

 you can use assembly to check if an address is equal to address(0) (also known as the zero address or the null address). This can potentially save gas compared to using a Solidity expression to perform the check.


 example:
 function isZeroAddress(address addr) internal pure returns (bool) {
    assembly {
        // Load the word from memory at the address of the input parameter
        let word := mload(addr)
        // Check if the word is equal to 0
        // If it is, return 1 (true), otherwise return 0 (false)
        switch word
        case 0 {
            // Word is equal to 0, return true
            mstore(0, 1)
            return(0, 32)
        }
        default {
            // Word is not equal to 0, return false
            mstore(0, 0)
            return(0, 32)
        }
    }
}



```solidity
181   if (llamaExecutor != address(0)) revert AlreadyInitialized();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L181

```solidity
405   if (llamaExecutor == address(0)) return; // Skip check during initialization.
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L405

```solidity
148   if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L148

```solidity
166   if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L166

```solidity
199   if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L199

```solidity
247   if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L247

```solidity
256   if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L256

```solidity
45   require(owner != address(0), "ZERO_ADDRESS");
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L45

```solidity
90   require(to != address(0), "INVALID_RECIPIENT")
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L90

```solidity
128  require(to != address(0), "INVALID_RECIPIENT");
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L128

```solidity
145  require(owner != address(0), "NOT_MINTED");
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L145





## [G-08] Use constants instead of type(uintx).max

In Solidity, type(uintX).max is a built-in constant that represents the maximum value of an unsigned integer of X bits. This constant can be useful in some cases, but it has a gas cost associated with it when used in expressions or assignments.

Instead of using type(uintX).max, you can define your own constant with the maximum value that you need. This can potentially save gas, especially if the constant is used frequently in the contract.

example:
contract MyContract {
    uint256 constant MAX_VALUE = 2**256 - 1;

    function foo() public {
        uint256 x = MAX_VALUE;
        // Do something with x
    }
}

```solidity
617       if (currentCount == type(uint128).max || quantity == type(uint128).max) return type(uint128).max;

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L617

```solidity
77    return push(self, quantity, type(uint64).max);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L77

```solidity
11   if (n > type(uint64).max) revert UnsafeCast(n);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol#11

```solidity
17   if (n > type(uint128).max) revert UnsafeCast(n);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol#17

```solidity
57    minDisapprovals != type(uint128).max
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L57

```solidity
79    if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L79

```solidity
36    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L36

```solidity
60   if (minDisapprovals == type(uint128).max) revert DisapprovalDisabled();    
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L60


```solidity
215     return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L215

```solidity
232     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L232


```solidity
223    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L223

```solidity
242     return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L242




## [G-09] Duplicated require()/if() checks should be refactored to a modifier or function


When the same require() or if() condition is repeated multiple times in a contract, it can result in redundant code and increase the gas cost of the contract. By refactoring the condition into a modifier or function, we can eliminate the redundant code and potentially reduce the gas cost.

example:

contract MyContract {
    uint256 public myValue;

    modifier isPositive(uint256 value) {
        require(value > 0, "Value must be positive");
        _;
    }

    function setValue(uint256 value1, uint256 value2) public isPositive(value1) isPositive(value2) {
        myValue = value1 + value2;
    }
}


```solidity
298   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L298

```solidity
389   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L389


```solidity
422   if (signer == address(0) || signer != policyholder) revert InvalidSignature();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L422



## [G-10] bytes constants are more eficient than string constans

In Solidity, bytes constants are generally more efficient than string constants in terms of gas cost. This is because bytes constants are stored as a contiguous block of memory, while string constants are stored as an array of pointers to the individual characters.


```solidity
183    string public name;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L183

```solidity
225    string memory _name,
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L225



## [G-11] Use assembly to hash instead of Solidity


In Solidity, you can use assembly to hash data instead of using the built-in Solidity hash functions like keccak256(). This can potentially save gas in the contract, especially if you need to hash large amounts of data

```solidity
529    bytes32 permissionId = keccak256(abi.encode(permission));
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L529

```solidity
707    return keccak256(
      abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
    );
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L707

```solidity
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
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L727







## [G-12] With assembly, .call (bool success) transfer can be done gas-optimized

In Solidity, you can use assembly to perform a gas-optimized transfer() or send() operation on an address by using the .call() method. This can potentially save gas in the contract, especially if you need to perform many transfers.

```solidity
34    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol#L34


```solidity
326    (success, result) = target.call{value: msg.value}(callData);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L326


```solidity
75   (bool success, bytes memory response) = targets[i].call(data[i]);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol#L75




## [G-13]  Using calldata instead of memory for read-only arguments in external functions saves gas 



In Solidity, using calldata instead of memory for read-only arguments in external functions can potentially save gas. This is because calldata is a cheaper and more efficient way to access function arguments that are only read and not modified.

When you define a function in Solidity with the external visibility specifier, the arguments of the function are passed in calldata instead of memory. calldata is a special area of memory that contains the function arguments and is read-only. By using calldata instead of memory to access read-only arguments, you can potentially save gas in the contract.


```solidity

/// @audit use calldata instead of memory on the name, color, and  logo   parameters.

17    function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)
18     external
19     pure
20     returns (string memory)

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L17-L20


```solidity
156      function initialize(bytes memory config) external initializer {
``` 
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156

```solidity

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
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L154-L165

```solidity
206     function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L206


```solidity
218     function contractURI(string memory name) external view returns (string memory) {

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L218


```solidity
130    function initialize(bytes memory config) external initializer {
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130



## [G-14] Using storage instead of memory for structs/arrays saves gas


In Solidity, using storage instead of memory for structs and arrays can potentially save gas. This is because storage is a more efficient way to store and access the data on the blockchain, especially for large data structures like arrays and structs.



```solidity
290        Checkpoints.Checkpoint[] memory checkpoints = new Checkpoints.Checkpoint[](sliceLength);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L290


## [G-15]use Mappings Instead of Arrays


Mappings are a data structure in Solidity that allow you to store key-value pairs. In contrast, arrays are a collection of elements of the same type. Mappings are more efficient than arrays in terms of gas usage for several reasons.




```solidity
22   string[21] memory parts;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L22



## [G-16] Use calldata instead of memory


By using calldata instead of memory, you can save gas because the function does not need to create a copy of the input data in memory. Instead, it can directly read the data from the calldata section of memory, which is a cheaper operation.



```solidity
130   function initialize(bytes memory config) external initializer {
    llamaExecutor = address(LlamaCore(msg.sender).executor());
    Config memory accountConfig = abi.decode(config, (Config));
    name = accountConfig.name;
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L130

```solidity
153  function initialize(bytes memory config) external virtual initializer {
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L153

```solidity
156   function initialize(bytes memory config) external initializer {
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L156


```solidity
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
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L154

```solidity
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
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L227





## [G-17] Amounts should be checked for 0 before calling a transfer

When sending cryptocurrency or tokens in a smart contract, it's important to ensure that the amount being transferred is greater than zero. This is because transferring zero amounts can result in unnecessary transactions being added to the blockchain, which increases gas costs.


```solidity
167   erc20Data.token.safeTransfer(erc20Data.recipient, erc20Data.amount);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L167




## [G-18] >= costs less gas than > 

Using >= instead of > can save gas because it eliminates the need for an additional comparison operation when the values are equal. This is because the Solidity compiler can optimize the >= operator to perform a single comparison operation that returns true if the values are equal or the first value is greater than the second value.

```solidity
35         if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L35

```solidity
305       if (role > numRoles) revert RoleNotInitialized(role);

```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L305

```soidity
237     if (role > numRoles) revert RoleNotInitialized(role);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL237C5-L237

```solidity
284     if (start > end) revert InvalidIndices();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL237C5-L284

```solidity
286     if (end > checkpointsLength) revert InvalidIndices();
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL237C5-L286

```solidity
414     if (role > numRoles) revert RoleNotInitialized(role);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL237C5-L414

```solidity
491     if (role > numRoles) revert RoleNotInitialized(role);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL237C5-L491

```solidity
230      if (minDisapprovalPct > ONE_HUNDRED_IN_BPS) revert DisapprovalDisabled();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L230

```solidity

325      if (role > numRoles) revert RoleNotInitialized(role);
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L325
