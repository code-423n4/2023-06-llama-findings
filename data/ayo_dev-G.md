1. In the LlamaCore contract use uint256 as value type for precompuation of hash instead of storing the hash as bytes32, this potential saves you 14 gas during deployment time
link to an example : https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaCore.sol#L142 

Instead of this 
```
bytes32 internal constant ACTION_INFO_TYPEHASH = keccak256("
      ActionInfo(uint256 id,addresscreator,uint8 creatorRole,address strategy,address target,uint256 
      value,bytes data)"
 );

do this

uint256 internal constant ACTION_INFO_TYPEHASH = 0x6e3204e4b1976f3422f302de4911fa1d568e4c18612bdb60954f18d9775df54d;
```

2.Initialization of unused Variable: this was a very popular error from the developer across the codebase, by default variable in solidity are already zero by default, no need to initialise to zero again, this was very common in loops where the developer started the the counting from zero, this cost 13 more gas in runtime and 100+ in deployment time;
Link to an example :https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/LlamaCore.sol#L656

```
    for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
      bytes32 salt = bytes32(keccak256(accountConfigs[i]));
      ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), salt));
      account.initialize(accountConfigs[i]);
      emit AccountCreated(account, llamaAccountLogic, accountConfigs[i]);
    }

correct way to this 

    for (uint256 i; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
      bytes32 salt = bytes32(keccak256(accountConfigs[i]));
      ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), salt));
      account.initialize(accountConfigs[i]);
      emit AccountCreated(account, llamaAccountLogic, accountConfigs[i]);
    }
```
again initializing default value to zero was very common in the codebase especially in loops

3. PostIncrements and PostIncrements, the author was fond of using post increments, instead of preIncrements, this was popular across the codebase again
link to example: https://github.com/code-423n4/2023-06-llama/blob/9d422c264b57657098c2784aa951852cad32e01c/src/lib/ERC721NonTransferableMinimalProxy.sol#L97
```
/// @dev Increments a `uint256` without checking for overflow.
  function uncheckedIncrement(uint256 i) internal pure returns (uint256) {
    unchecked {
      return i + 1;
    }
  }
}

instead of 

/// @dev Increments a `uint256` without checking for overflow.
  function uncheckedIncrement(uint256 i) internal pure returns (uint256) {
    unchecked {
      return  ++i;
    }
  }
}
```