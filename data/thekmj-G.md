## [G-01] Pack storage to optimize gas

In `LlamaAbsoluteStrategyBase`, the current storage layout is as follow (each empty line denotes a new storage slot):

```solidity
LlamaCore public llamaCore;

LlamaPolicy public policy;
bool public isFixedLengthApprovalPeriod;
uint64 public approvalPeriod;

uint64 public queuingPeriod;
uint64 public expirationPeriod;
uint128 public minApprovals;

uint128 public minDisapprovals;
uint8 public approvalRole;
uint8 public disapprovalRole;
```

We suggest the following layout to save one storage slot:

```solidity
LlamaCore public llamaCore;
uint48 public queuingPeriod;
uint48 public expirationPeriod;

LlamaPolicy public policy;
uint48 public approvalPeriod;
bool public isFixedLengthApprovalPeriod;
uint8 public approvalRole;
uint8 public disapprovalRole;

uint128 public minApprovals;
uint128 public minDisapprovals;
```

Note that this reduces each time variable data type to `uint48`, but the range should still be sufficient for any reasonable time frame.

## [G-02] `mapping(uint8 => bool)` can be replaced with a bitmask of `uint256` to save storage writes

The `uint8` data type's valid range is from 0 to 255, inclusive. For a mapping of `(uint8 => bool)`, we can use a single `uint256` as a bitmask to represent the truth values of all possible integer keys from 0 to 255. In this technique, the $i$-th bit of the `uint256` will be set as 1 if the value $i$ in the mapping is `true` and vice-versa.

Such a technique can be implemented as following, with each mapping being replaced by a single `uint256`:

```solidity
// @dev given a "mask" mapping, sets the "b"-th bit to 1 if "value" is true, 0 otherwise
function setValue(uint256 mask, uint8 b, bool value) internal pure returns (uint256 newMask) {
    if (value) {
        // turn the b-th bit on
        newMask = mask | (1 << b);
    } else {
        // turn the b-th bit off
        newMask = mask & (~(1 << b));
    }
}

// @dev given a "mask" mapping, return whether the "b"-th bit is set 
function getValue(uint256 mask, uint8 b) internal pure returns (bool value) {
    return ((mask >> key) & 1) == 1;
}
```

This will convert one **GSSET (21000 gas)** to a **GSRESET (2900 gas)** for every key that is set to true, from the second key onwards, for a maximum of **4360500 gas** saved per mapping, if all of the 256 keys are used (17100 gas per key). 

The realistic saving amount is 17100 gas per approval/disapproval role except the first one, thus is dependent on the expected number of roles per deployment.

This is applicable to four instances found in the strategy contracts:

- https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L133
- https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L136
- https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L130
- https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L133

