[L-01] 1. ecrecover from solidity has risk of signature malleability.

Code Snippet:
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L297
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L388
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L421

Previous Submission: https://github.com/code-423n4/2021-08-realitycards-findings/issues/63

Recommendation: using ECDSA library from Openzeppelin that would check if the signature is above n /2, only allow the signature in lower half order and revert otherwise

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L141-L158
