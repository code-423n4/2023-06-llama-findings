## Non-Criticals

| |Issue|Instances|
|-|:-|:-:|
| [NC-01](#NC-01) | `_lowerBinaryLookup` function never used in the codebase | 1 |
| [NC-02](#NC-02) | `if` conditioned differs from mentioned NatSpec | 1 |


## [NC-01] `_lowerBinaryLookup` function never used in the codebase
Functions not used can either be removed or commented out.

```solidity
File: main/src/lib/Checkpoints.sol

183:      function _lowerBinaryLookup(
            Checkpoint[] storage self,
            uint64 timestamp,
            uint256 low,
            uint256 high
           ) private view returns (uint256) {
               while (low < high) {
               uint256 mid = average(low, high);
               if (_unsafeAccess(self, mid).timestamp < timestamp) {
                   low = mid + 1;
                } else {
                high = mid;
               }
        }
        return high;
    }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L183

## [NC-02] `if` conditioned differs from mentioned NatSpec 

The comment above the modifier says either of the three (Llama instance, the root Llama instance, or the Llama factory) can update the instance's attributes but the `onlyAuthorized` modifier used **&&**, meaning all the three conditions need to fulfilled. 
So, the NatSpec is contradicting the check conditions.

```solidity
File: main/src/LlamaPolicyMetadataParamRegistry.sol

17:    /// @dev Only the Llama instance, the root Llama instance, or the Llama factory can update an instance's 
       color and
       /// logo.
       modifier onlyAuthorized(LlamaExecutor llamaExecutor) {
       if (
         msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != 
        LLAMA_FACTORY
         ) revert UnauthorizedCaller();
         _;
       }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L17-L24
