## Checkpoints with past expiration can be set

### Summary

The library Checkpoints is product of a modification to the OpenZeppelin Contracts Checkpoints library but now including an expiration expressed in `uint64`.

The problem with the addition of the expiration field is that its validity is not checked at insertion. In fact, the [`_insert` function](https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L122) only checks that:

```solidity
function _insert(
    Checkpoint[] storage self,
    uint64 timestamp,
    uint64 expiration,
    uint128 quantity
) private returns (uint128, uint128) {
    uint256 pos = self.length;

    if (pos > 0) {
        // Copying to memory is important here.
        Checkpoint memory last = _unsafeAccess(self, pos - 1);

        // Checkpoints timestamps must be increasing.
        require(last.timestamp <= timestamp, "Checkpoint: invalid timestamp");
```

But no check of `expiration > block.timestamp` is done.

### Impact

Currently, no component of the Llama system is exposed to an invalid insertion of this kind, but leaving it in the code without fixing may imply the risk of breaking the assumption of non-expired checkpoints.

### Mitigation steps

Place a `require(expiration < timestamp);` or an equivalent custom error in Line 129 of Checkpoints.sol

---

## No check for color validity in LlamaFactory `deploy`

### Summary

[In Line 150](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L150) of LlamaFactory.sol, the NatSpec for the `deploy` function indicate the color parameter is provided as _any valid SVG_ color. However, there's no such check

### Impact

An error in color settings may cause UI issues or frontend support issues.

### Mitigation steps

- Add such SVG color validity check or remove the _valid_ mention in the NatSpec

---

## NatSpec for `InvalidDeployConfiguration` error is not exhaustive

### Summary

The current declaration of the `InvalidDeployConfiguration` is:

```solidity
/// @dev The initial set of role holders has to have at least one role holder with role ID 1.
error InvalidDeployConfiguration();
```

However, a check later is performed to verify that the `expiration` of a deployed Llama instance require a Bootstrap role that never expires:

```solidity
if (initialRoleHolders.length == 0) revert InvalidDeployConfiguration();
if (initialRoleHolders[0].role != BOOTSTRAP_ROLE) revert InvalidDeployConfiguration();
if (initialRoleHolders[0].expiration != type(uint64).max) revert InvalidDeployConfiguration();
```

The NatSpec of the error doesn't mention the expiration requirement.

### Mitigation Steps

- Extend the NatSpec for `InvalidDeployConfiguration` to also include the expiration requirement