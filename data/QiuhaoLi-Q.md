## [low] _deployStrategies() can't handle two strategies with the same config

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L638

In _deployStrategies(), we directly use config as salt:

```solidity
    uint256 strategyLength = strategyConfigs.length;
    for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
      bytes32 salt = bytes32(keccak256(strategyConfigs[i]));
      ILlamaStrategy strategy = ILlamaStrategy(Clones.cloneDeterministic(address(llamaStrategyLogic), salt));
      strategy.initialize(strategyConfigs[i]);
      strategies[strategy] = true;
      emit StrategyCreated(strategy, llamaStrategyLogic, strategyConfigs[i]);
      if (i == 0) firstStrategy = strategy;
    }
```
So when policyholders want to deploy a strategy with the same config, or there exists a strategy with the same config (e.g., for compatibility of merging two llama instances). The cloneDeterministic will fail (create2 on the same address).

Adding random salt can solve this problem.

## [low] _deployAccounts() can't handle two strategies with the same config

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L658

In _deployAccounts(), we directly use config as salt:

```solidity
    uint256 strategyLength = strategyConfigs.length;
    for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
      bytes32 salt = bytes32(keccak256(strategyConfigs[i]));
      ILlamaStrategy strategy = ILlamaStrategy(Clones.cloneDeterministic(address(llamaStrategyLogic), salt));
      strategy.initialize(strategyConfigs[i]);
      strategies[strategy] = true;
      emit StrategyCreated(strategy, llamaStrategyLogic, strategyConfigs[i]);
      if (i == 0) firstStrategy = strategy;
    }
```
So when policyholders want to deploy an account with the same config, or there exists an account with the same config (e.g., for compatibility of merging two llama instances). The cloneDeterministic will fail (create2 on the same address).

Adding random salt can solve this problem.

## [non-critical] _getDomainHash use semantic version

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L708

```solidity
  /// @dev Returns the EIP-712 domain separator.
  function _getDomainHash() internal view returns (bytes32) {
    return keccak256(
      abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))
    );
  }
```

We can use [Semantic Versioning 2.0.0](https://semver.org/) for EIP712_DOMAIN_TYPEHASH's version instead of "1".

## [non-critical] typo in isActionDisapproved @notice

https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaStrategy.sol#L99

```
vetoed --> voted
```

## [non-critical] Checkpoints _lowerBinaryLookup is unused

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol#L183

_lowerBinaryLookup is unused.

## [non-critical] useless ERC721NonTransferableMinimalProxy _burn delete getApproved[id]

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L104

Since approve is not used in our modified ERC721 policy contract. `delete getApproved[id];` is useless.

## [non-critical] use LlamaExecutor(address(this)) instead of saving EXECUTOR in LlamaSingleUseScript.sol

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaSingleUseScript.sol#L22

Since this is a delegate call, we can use the address(this) directly.
