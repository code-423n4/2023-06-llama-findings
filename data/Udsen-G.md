## 1. REDUNDANT CONDITIONAL CHECKS CAN BE OMITTED TO SAVE GAS

In the `LlamaAbsoluteStrategyBase.queuingPeriod` variable denotes the minimum time in seconds between queueing and execution of action.

The `LlamaAbsoluteStrategyBase.minExecutionTime()` returns the `block.timestamp + queuingPeriod` timestamp.
Here `queuingPeriod` is stored as a storage variable of the `LlamaAbsoluteStrategyBase` abstract contract.

The `LlamaAbsoluteStrategyBase.minExecutionTime()` function is called by the `LlamaCore.queueAction` function to check whether the `minExecutionTime` is in the past. If so it will revert as follows:

    uint64 minExecutionTime = actionInfo.strategy.minExecutionTime(actionInfo);
    if (minExecutionTime < block.timestamp) revert MinExecutionTimeCannotBeInThePast();

But the `LlamaAbsoluteStrategyBase.minExecutionTime()` calculates the `minExecutionTime` as follows:

  function minExecutionTime(ActionInfo calldata) external view virtual returns (uint64) {
    return LlamaUtils.toUint64(block.timestamp + queuingPeriod);
  }

Hence there is no condition underwhich `minExecutionTime < block.timestamp` will occur. Because the current `block.timestamp` is used in both the called function and the `if` condition check. Hence the smallest value the `minExecutionTime()` function can return is `block.timestamp` when `queuingPeriod = 0`; 

So the `LlamaCore.queueAction` function will execute the following:

    if (block.timestamp < block.timestamp) revert MinExecutionTimeCannotBeInThePast();

This is false and the transaction will not revert. 

Hence for all the other conditions this `if` statement will be false.

Hence we can remvoe this conditional check from the `LlamaCore.queueAction` functino to save gas.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L309-L310
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol#L238-L240