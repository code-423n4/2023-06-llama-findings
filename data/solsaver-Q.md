## Consider creating the image off-chain for tokenURI

Instead of constructing the complete Base64 image, consider only placing the attributes in the JSON, and then constructing the actual image off chain.

```
function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)
  ...
  ...
  ' to view their profile page.", "external_url": "https://app.llama.xyz", "image": "data:image/svg+xml;base64,',
  Base64.encode(bytes(output)),
  ...
  ...
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L91-L92

## `minDisapprovals != type(uint128).max` seems unnecessary, otherwise it should be applied to `minApprovals` as well

In `LlamaAbsoluteQuorum.sol`, the `validateActionCreation` method checks if `minDisapprovals != type(uint128).max`, but the same check is not added for `minApprovals`. Either both the checks should add the `type(uint128).max` or none of them should.

```
function validateActionCreation(ActionInfo calldata /* actionInfo */ ) external view override {
    ...
    ...
    if (minApprovals > approvalPolicySupply) revert InsufficientApprovalQuantity();
    if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {
      revert InsufficientDisapprovalQuantity();
    }
    ...
    ...
}
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L35-L38

## Action Creation should be allowed, given that `DisapprovalDisabled` is possible

In `LlamaAbsoluteQuorum.sol`, the `validateActionCreation` method reverts if `disapprovalPolicySupply` is 0. But at the same time, if `minDisapprovals` is set to `type(uint128).max`, then its considered as `DisapprovalDisabled`.

If `DisapprovalDisabled` is possible, then `disapprovalPolicySupply` should be allowed to be 0 as well, otherwise it seems conflicting.

```
function validateActionCreation(ActionInfo calldata /* actionInfo */ ) external view override {
    ...
    ...
    uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(disapprovalRole);
    if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);
    ...
    ...
}
```
Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L32-L33
