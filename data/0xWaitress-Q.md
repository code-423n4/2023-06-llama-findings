[L-01] 1. ecrecover from solidity has risk of signature malleability.

Code Snippet:
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L297
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L388
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L421

Previous Submission: https://github.com/code-423n4/2021-08-realitycards-findings/issues/63

Recommendation: using ECDSA library from Openzeppelin that would check if the signature is above n /2, only allow the signature in lower half order and revert otherwise

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L141-L158

[I-01] 1. isDisapprovalEnabled&isApprovalEnabled in LlamaRelativeQuorem should return a bool if possible.
The function name is indicative of a bool return as a a standard practice, and this is most of the case in the rest of the codebase. A simple revert of pass is quite unusual.

```solidity
  function isDisapprovalEnabled(ActionInfo calldata, address, uint8 role) external view {
    if (minDisapprovalPct > ONE_HUNDRED_IN_BPS) revert DisapprovalDisabled();
    if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);
  }

  function isApprovalEnabled(ActionInfo calldata, address, uint8 role) external view {
    if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
  }
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L229-L232
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L215-L217

## Recommendation 
Consider returning a bool.

