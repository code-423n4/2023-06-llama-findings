## Lines of code

https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaCore.sol#LL587C1-L599C45


## Vulnerability details

The `expectedState` parameter maybe not `Active` or `Queued` status. When `expectedState` is other status such as approved or failed, the parameter should checked is `Active` or `Queued` status.


## Recommended Mitigation Steps

Should add `expectedState` check when `expectedState` is `Active` or `Queued` status:

```solidity
    function _preCastAssertions(
        ActionInfo calldata actionInfo,
        address policyholder,
        uint8 role,
        ActionState expectedState
    ) internal returns (Action storage action, uint128 quantity) {
        if (expectedState != ActionState.Queued || expectedState != ActionState.Active) {
        revert InvalidActionState(expectedState);
    }
```


