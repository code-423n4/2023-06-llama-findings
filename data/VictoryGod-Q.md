## Summary
### Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|01|Security vulnerability in setColor and setLogo| LlamaPolicyMetadataParamRegistry.sol |
|02|Reentrancy vulnerability in validateActionCancelation| strategies/LlamaRelativeQuorum.sol |
|03|Input validation in initialize function| strategies/LlamaRelativeQuorum.sol |
|04|Use IERC20 interface| LlamaFactory.sol |
|05|Check strategy and account authorization| LlamaFactory.sol |
|06|Use IERC721Metadata for URIs| LlamaFactory.sol |
|07|Validate input parameters in initialize| LlamaCore.sol |

Total 7 issues

## File -> LlamaPolicyMetadataParamRegistry.sol
### [01] There is a potential security vulnerability in the setColor and setLogo functions. 
These functions use the onlyAuthorized modifier to restrict access, but they are marked as public, which means they can still be called by any address. To fix this vulnerability, you should change the visibility of these functions to internal. This change will ensure that they can only be called from within the contract or derived contracts.

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L82
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicyMetadataParamRegistry.sol#L90


## File -> strategies/LlamaRelativeQuorum.sol

### [02] Potential reentrancy vulnerability in validateActionCancelation function:
The function validateActionCancelation performs external calls to llamaCore.getActionState and llamaCore.getAction, which may introduce reentrancy vulnerabilities if the contract's state is not properly protected. To mitigate this risk, consider using the Checks-Effects-Interactions pattern.
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#L255

### [03] Lack of input validation in initialize function:
The initialize function does not validate the input config before decoding it. This may cause unexpected behavior if the input is malformed or incorrect. To improve safety, consider adding input validation checks before decoding the config.
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/strategies/LlamaRelativeQuorum.sol#LL157C27-L157C27

## File -> LlamaFactory.sol

### [04] Use IERC20 interface instead of custom interfaces: 
The contract uses custom interfaces ILlamaStrategy and ILlamaAccount. If these interfaces are meant to represent ERC20 tokens, it would be better to use the standard IERC20 interface provided by OpenZeppelin. This would make the contract more secure and easier to integrate with other contracts.

### [05] Check if strategy and account logic contracts are authorized: 
In the _deploy() function, you should check if the provided strategyLogic and accountLogic contracts are authorized before deploying a new Llama instance. This would prevent unauthorized contracts from being used in the Llama instances.
```
if (!authorizedStrategyLogics[strategyLogic]) revert UnauthorizedStrategyLogic();
if (!authorizedAccountLogics[accountLogic]) revert UnauthorizedAccountLogic();
```
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaFactory.sol#L227

 ### [06] Use IERC721Metadata for tokenURI and contractURI: 
 The tokenURI and contractURI functions should follow the IERC721Metadata standard interface, making the contract more compatible with other dApps and wallets.


## File -> LlamaCore.sol
### [07] The initialize function does not perform any validation on the input parameters,
such as checking for non-zero addresses for _policy, _llamaStrategyLogic, and _llamaAccountLogic. This can lead to unintended behavior or potential security issues. To mitigate this, you can add checks to ensure the input parameters are valid. 
```
   require(address(_policy) != address(0), "Invalid policy address");
   require(address(_llamaStrategyLogic) != address(0), "Invalid strategy logic address");
   require(address(_llamaAccountLogic) != address(0), "Invalid account logic address");
```
