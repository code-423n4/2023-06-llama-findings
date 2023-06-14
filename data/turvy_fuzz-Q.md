## abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L741
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L763
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L687
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L785

Use abi.encode() instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).

## strict equality for amount sent could result to DOS, use < and send remaining excess value back at the end of function execution instead
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L324

## multiple instance of same revert checks should be refactored to a modifier or function
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L445
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L455

## Initialization to 0 is unnecessary
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L43

## no access control or limit to incrementing nonce
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L463

## Avoid unnecessary repeated data hashing operation
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L741
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L763
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L785

I recommend removing the unnecessary double encoding and hashing

## use nested check rather than && to improve readbility and save gas
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L21
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L68
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L213
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L230
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L473
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L476
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L36
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L49
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L61
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L470
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L81

## this function interact with storage data on the blockchain so should not be set to pure
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L202