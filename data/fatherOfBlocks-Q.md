**src/strategies/LlamaAbsoluteQuorum.sol**
- L4/6/10/11/13 - Multiple imports are made that are never used, therefore they generate an extra gas expense and also generate less understanding in the deploy, since there will be dead code.


**src/strategies/LlamaAbsolutePeerReview.sol**
- L4/6/10/11/13 - Multiple imports are made that are never used, therefore they generate an extra gas expense and also generate less understanding in the deploy, since there will be dead code.


**src/strategies/LlamaAbsoluteStrategyBase.sol**
- L6/50/53/56/71 - The import of FixedPointMathLib that is not used is performed, therefore they generate an extra gas expense and also generate less understanding in the deploy, since there will be dead code.
The same is true for errors: DisapprovalDisabled, InsufficientApprovalQuantity, InsufficientDisapprovalQuantity, RoleHasZeroSupply.
