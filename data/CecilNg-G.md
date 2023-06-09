//This is a gas report that revolves around optimizing storage of this contracy.

Opt 1 is these two variables

<https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L113-L116> 

Can be moved down here

<https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L127-L130>

That will help in saving 2 slots for setting, 17100 gas each.

Secondly
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L133-L136
These two mappings use uint8
Considering that its value's are up to 0-225
The technique is to use bitmasks. Use the bits of one uint256 to be able to stand in place of 256 different booleans
Helping you save 17100 from the second role onwards, for a maximum of 4360500 if 255 different roles are present.