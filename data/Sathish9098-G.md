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

## [G-2] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

Subsequent reads as well as writes have smaller gas savings

The uint96 type is an unsigned integer that occupies 96 bits (12 bytes) of storage. It can represent values from 0 to 2^96 - 1. 

https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/accounts/LlamaAccount.sol#L34-L37

### Saves 1 SLOT and 2000 gas 

```diff
FILE: 023-06-llama/src/accounts/LlamaAccount.sol

61: struct ERC1155Data {
62:    IERC1155 token; // The ERC1155 token to transfer.
63:    address recipient; // The address to transfer the token to.
- 64:    uint256 tokenId; // The tokenId of the token to transfer. 
+ 64:    uint96 tokenId; // The tokenId of the token to transfer. 
65:    uint256 amount; // The amount of tokens to transfer.
66:    bytes data; // The data to pass to the ERC1155 token.
67:  }

```
##

## [G-3] Multiple accesses of a mapping/array should use a local variable cache

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

##

## [G-4]  IF’s/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaCore.sol#L323

### Cheaper to check the function parameter before making an state variable check. ``action.minExecutionTime`` involving the state variable . Should be checked last 

```diff
FILE: 2023-06-llama/src/LlamaCore.sol

322: if (currentState != ActionState.Queued) revert InvalidActionState(currentState);
- 323: if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();
324: if (msg.value != actionInfo.value) revert IncorrectMsgValue();
+ 323: if (block.timestamp < action.minExecutionTime) revert MinExecutionTimeNotReached();

```
https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/strategies/LlamaRelativeQuorum.sol#L165

### Condition check should be top of the initialize() to avoid unwanted state variables write 

```diff
FILE: Breadcrumbs2023-06-llama/src/strategies/LlamaRelativeQuorum.sol

157: Config memory strategyConfig = abi.decode(config, (Config));
+ 165:    if (strategyConfig.minApprovalPct > ONE_HUNDRED_IN_BPS) revert InvalidMinApprovalPct(minApprovalPct);
    minApprovalPct = strategyConfig.minApprovalPct;
158:    llamaCore = LlamaCore(msg.sender);
159:    policy = llamaCore.policy();
160:    queuingPeriod = strategyConfig.queuingPeriod;
161:    expirationPeriod = strategyConfig.expirationPeriod;
162:    isFixedLengthApprovalPeriod = strategyConfig.isFixedLengthApprovalPeriod;
163:    approvalPeriod = strategyConfig.approvalPeriod;
164:
- 165:    if (strategyConfig.minApprovalPct > ONE_HUNDRED_IN_BPS) revert InvalidMinApprovalPct(minApprovalPct);
    minApprovalPct = strategyConfig.minApprovalPct;

```

##

## [G-5] Repeated condition checks should be refactored to modifiers

repeated condition checks should be refactored to modifiers.

https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/strategies/LlamaRelativeQuorum.sol#L179


### 

```solidity
FILE: 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol

179:  if (role == 0) revert InvalidRole(0);
187:  if (role == 0) revert InvalidRole(0);

216:  if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
221:  if (role != approvalRole && !forceApprovalRole[role]) return 0;

231: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
240: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

```




 

















