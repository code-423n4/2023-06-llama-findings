# [L-01] Lack of potential reentrancy guards in LlamaAccount.

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#L147

  function transferNativeToken(NativeTokenData calldata nativeTokenData) public onlyLlama {
    if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();
    nativeTokenData.recipient.sendValue(nativeTokenData.amount);
  }

 (...)

  function batchTransferNativeToken(NativeTokenData[] calldata nativeTokenData) external onlyLlama {
    uint256 length = nativeTokenData.length;
    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
      transferNativeToken(nativeTokenData[i]);
    }
  }
```

`transferNativeToken` and `batchTransferNativeToken` use `sendValue` function.
According to OpenZeppelin documentation, there should be reentrancy guards implemented while using this function:

```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/b425a722409fdf4f3e0ec8f42d686f3c0522af19/contracts/utils/Address.sol#L36

* IMPORTANT: because control is transferred to `recipient`, care must be
     * taken to not create reentrancy vulnerabilities. Consider using
     * {ReentrancyGuard} or the
     * https://solidity.readthedocs.io/en/v0.8.0/security-considerations.html#use-the-checks-effects-interactions-pattern[checks-effects-interactions pattern].
```



# [L-02] Usage of transferFor instead of safeTransferFor in ECR721

According to OpenZeppelin documentation:

```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/b425a722409fdf4f3e0ec8f42d686f3c0522af19/contracts/token/ERC721/IERC721.sol#L75

 * WARNING: Note that the caller is responsible to confirm that the recipient is capable of receiving ERC721
     * or else they may be permanently lost. Usage of {safeTransferFrom} prevents loss, though the caller must
     * understand this adds an external call which potentially creates a reentrancy vulnerability.
```

it's recommended to use `safeTransferFor` instead

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/accounts/LlamaAccount.sol#L200

 function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {
    if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();
    erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);
  }
```

# [L-03] Function execute() is a single point of failure
Having a single EOA as the only owner of contracts poses a huge centralization risk. The private key may stolen in case of a hack, or the holder of the key may not be able to retrieve the key when necessary. Consider changing to a multi-signature setup, or having a role-based authorization model.


The execute() function performs only a single check:  `if (msg.sender != LLAMA_CORE)` before executing a function which calls/delegatecalls to the provided address.


```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaExecutor.sol#L29

  function execute(address target, uint256 value, bool isScript, bytes calldata data)
    external
    returns (bool success, bytes memory result)
  {
    if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();
    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);
  }
}
```

According to the documentation:

```
https://github.com/code-423n4/2023-06-llama/blob/main/docs/action-creation.md

Executor: The single exit point of a Llama instance. All actions that are executed will be sent from the Llama executor. This is the address that should be the owner or other privileged role in a system controlled by the llama instance
```

This means, that a single exit point of a Llama istance is being controlled by a single EOA only. Moreover, even thought, according to documentation: "This contract is deployed in the core's `initialize` function.", the `execute` function is declare as `external`, so it's possible to call (thus execute `target.delegatecall(data)` or `target.call{value: value}(data)`) directly from `LlamaExecutor`. This is, in my opinion, important to reduce a centralization risk of this piece of code.



# [N-01] Variable name length is confusing in LlamaGovernanceScript

```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/llama-scripts/LlamaGovernanceScript.sol#L69

uint256 length = data.length;
```
Function `aggregate` has two parameters: `targets` and `data`. Function defines the length of `data` as `uint256 length`. This naming convention might be confusing, since this name of the variable does not explicitly states that this is a length of `data` (and not e.g. `targets`'s').
The recommendation is to change naming from `length` to `dataLength` to straightforwardly point out that this variable is a length of `data`.