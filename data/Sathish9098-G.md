# GAS OPTIMIZATIONS

##

## [G-1] Using storage instead of memory for structs/arrays saves gas

### Saves ``8400 GAS`` in ``2 Instances`` 

When fetching data from a storage location, assigning the data to a ``memory`` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional ``MLOAD`` rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/lib/Checkpoints.sol#L106

### _unsafeAccess function returns the ``storage Checkpoint `` . Then the implicit conversion storage to memory costs ``Gcoldsload `` extra ``2100 gas `` . ``ckpt, last `` only reading the values no write operations. So using ``storage`` is more efficient than ``memory``. Saves ``8400 GAS`` approximately 

Checkpoint struct in storage, using _unsafeAccess directly can be more gas-efficient than assigning it to a local variable. This is because accessing the struct in storage via _unsafeAccess involves reading from storage once, whereas assigning it to a local variable creates a copy of the struct in memory, which incurs additional gas costs. 

```diff
FILE: 2023-06-llama/src/lib/Checkpoints.sol

- 106: Checkpoint memory ckpt = _unsafeAccess(self._checkpoints, pos - 1);
+ 106: Checkpoint storage ckpt = _unsafeAccess(self._checkpoints, pos - 1);
107: return (true, ckpt.timestamp, ckpt.expiration, ckpt.quantity);

- 132: Checkpoint memory last = _unsafeAccess(self, pos - 1);
+ 132: Checkpoint storage last = _unsafeAccess(self, pos - 1);


```
##
## [G-2] Multiple accesses of a mapping/array should use a local variable cache

### Saves ``480 GAS''   Instances 6

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata

```solidity
FILE: 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol

219: function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {
220:    if (role != approvalRole && !forceApprovalRole[role]) return 0; ///@audit first call forceApprovalRole[role]
221:    uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
222:    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity; ///@audit second call forceApprovalRole[role]
223:  }


240: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
241:    uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
242:    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;
243:  }

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L219-L224

```solidity
FILE: Breadcrumbs2023-06-llama/src/LlamaCore.sol

699: nonce = nonces[policyholder][selector];
700: nonces[policyholder][selector] = LlamaUtils.uncheckedIncrement(nonce);

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#LL699C5-L700C75

```solidity
FILE: 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol

 require(_ownerOf[id] == address(0), "ALREADY_MINTED");

    // Counter overflow is incredibly unrealistic.
    unchecked {
      _balanceOf[to]++;
    }

    _ownerOf[id] = to;

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/lib/ERC721NonTransferableMinimalProxy.sol#LL130C4-L137C23

```solidity
FILE: Breadcrumbs2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol

213: if (role != approvalRole && !forceApprovalRole[role]) return 0;
214:    uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
215:    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

230: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
231:    uint128 quantity = policy.getPastQuantity(policyholder, role, timestamp);
232:    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaAbsoluteStrategyBase.sol#L213-L215















