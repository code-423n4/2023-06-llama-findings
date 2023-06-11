Redundant casting of keccak256 output to bytes32.

The keccak256 function returns a bytes32 value which is again unnecessarily casted to bytes32 value.

[Instance 1](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L637)

[Instance 2](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L657)
