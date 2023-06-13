1. Should check whether script address is zero address.
   which may lead zero address to a authorized script.
   add tese use in LlamaCore.t.sol:

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455#L459

```solidity
  function testFuzz_AddZeroAddresstoAuthorizeScript() public {
    vm.prank(address(mpExecutor));
    address ZEROAddress = address(0);

    mpCore.authorizeScript(ZEROAddress, true);
    assertEq(mpCore.authorizedScripts(ZEROAddress), true);
  }
```

2. Should check whether setGuard target address is zero address.
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445#L449

3. Merge duplication code
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L590#L617
```solidity
  function _preCastAssertions(
    ActionInfo calldata actionInfo,
    address policyholder,
    uint8 role,
    ActionState expectedState
  ) internal returns (Action storage action, uint128 quantity) {
    action = actions[actionInfo.id];
    ActionState currentState = getActionState(actionInfo);
    if (currentState != expectedState) revert InvalidActionState(currentState);

    bool isApproval = expectedState == ActionState.Active;
    bool alreadyCast = isApproval ? approvals[actionInfo.id][policyholder] : disapprovals[actionInfo.id][policyholder];
    if (alreadyCast) revert DuplicateCast();

    bool hasRole = policy.hasRole(policyholder, role, action.creationTime);
    if (!hasRole) revert InvalidPolicyholder();

    if (isApproval) {
      actionInfo.strategy.isApprovalEnabled(actionInfo, policyholder, role);
      quantity = actionInfo.strategy.getApprovalQuantityAt(policyholder, role, action.creationTime); //@audit not check
        // expiration ?
-     if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);
    } else {
      actionInfo.strategy.isDisapprovalEnabled(actionInfo, policyholder, role);
      quantity = actionInfo.strategy.getDisapprovalQuantityAt(policyholder, role, action.creationTime);
-     if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);
    }
+   if (quantity == 0) revert CannotCastWithZeroQuantity(policyholder, role);
  }
```