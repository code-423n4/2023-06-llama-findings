This report contains issues that were not included in analysis report and the previous audit by Spearbit.

### L-01 Policy can be initialized without executor
`finalizeInitialization` method doesn't check whether llamaExecutor is address(0)
```solidity
function  finalizeInitialization(address _llamaExecutor, bytes32 bootstrapPermissionId) external {
	if (llamaExecutor != address(0)) revert  AlreadyInitialized();
	llamaExecutor = _llamaExecutor;
	_setRolePermission(BOOTSTRAP_ROLE, bootstrapPermissionId, true);
}
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L180

This way, the check in `_assertNoActionCreationsAtCurrentTimestamp` will be broken. 
```solidity
function  _assertNoActionCreationsAtCurrentTimestamp() internal  view {
	if (llamaExecutor == address(0)) return; // Skip check during initialization.
	address llamaCore = LlamaExecutor(llamaExecutor).LLAMA_CORE();
	uint256 lastActionCreation = LlamaCore(llamaCore).getLastActionTimestamp();
	if (lastActionCreation == block.timestamp) revert  ActionCreationAtSameTimestamp();
}
```
This can lead to further issues that this check is supposed to prevent. A role can be modified in the same block after a new action creation. Since checkpointing is done based on `block.timestamp`, this will change approval/disapproval criteria for this action after it is created.

### L-02 use safeTransferFrom for transfering NFTs 
In LlamaAccount.sol NFTs are transferred using simple `transferFrom`. This doesn't check whether receiver is able to receive NFTs which can lead to lost tokens.
```solidity
function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {
  if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();
  erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);
}
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol#L200

### L-03 Bad strategy and account contracts cannot be unauthorized
There is no way to remove strategy and account contracts from the list of authorized addresses in LlamaFactory. In case a bug is discovered in one of these contracts, it will still be available for Llama instances.
```solidity
  function _authorizeStrategyLogic(ILlamaStrategy strategyLogic) internal {
    authorizedStrategyLogics[strategyLogic] = true;
    emit StrategyLogicAuthorized(strategyLogic);
  }

  /// @dev Authorizes an account implementation (logic) contract.
  function _authorizeAccountLogic(ILlamaAccount accountLogic) internal {
    authorizedAccountLogics[accountLogic] = true;
    emit AccountLogicAuthorized(accountLogic);
  }
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L267-L276

### NC-01  Don't revert on view methods
In LlamaAbsoluteQuorum.sol and LlamaRelativeQuorum.sol view methods can revert. This is not considered a good practice.
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L49
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L60-L61
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol#L27
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L216
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L230-L231
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L255
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L40
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L66
https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol#L74

For `validateActionCancelation` - remove view keyword. For others - return bool value and let consumers revert if necessary.

### NC-02 Errors missing helpful parameters
Throughout the codebase, very few errors have parameters. Examples from LlamaCore:
```solidity
  // Missing address parameter
  error RestrictedAddress();

  // Missing policyholder address
  error DuplicateCast();

  // Missing action id and perhaps infohash parameters
  error InfoHashMismatch();

  // Missing provided and expected message value
  error IncorrectMsgValue();

  // Missing policyholder address
  error InvalidPolicyholder();

  // Missing expected and actual signer address
  error InvalidSignature();

  // Missing provided strategy address
  error InvalidStrategy();

  // Missing timestamp and provided execution time
  error MinExecutionTimeCannotBeInThePast();

  // Missing policyholder address and permission id
  error PolicyholderDoesNotHavePermission();

  // Missing timestamp and execution time
  error MinExecutionTimeNotReached();

  // Missing provided strategy address
  error UnauthorizedStrategyLogic();

  // Missing provided account address
  error UnauthorizedAccountLogic();
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L34-L78
There are too many cases to display all in this report. It is better to provide parameters in errors for the purposes of debugging and handling on the front-end.

### NC-03 Role validation can be moved to a modifier
Role validation is repeated several times through LlamaPolicy. It can be moved to a modifier to improve code quality
```solidity
if (role > numRoles) revert RoleNotInitialized(role)
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L237
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L414
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L491

### NC-04 Remove unused code from ERC721NonTransferableMinimalProxy
Some functions in ERC721NonTransferableMinimalProxy are unused and can be removed.
```solidity
function _safeMint(address to, uint256 id) internal virtual {
    _mint(to, id);

    require(
      to.code.length == 0
        || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, "")
          == ERC721TokenReceiver.onERC721Received.selector,
      "UNSAFE_RECIPIENT"
    );
}

function _safeMint(address to, uint256 id, bytes memory data) internal virtual {
    _mint(to, id);

    require(
      to.code.length == 0
        || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, data)
          == ERC721TokenReceiver.onERC721Received.selector,
      "UNSAFE_RECIPIENT"
    );
}
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L163
https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol#L174
This code was copy-pasted from OpenZeppelin library but is not actually needed for the project.


### NC-05 Mapping variable should be named in plural
actionGuards should be named in plural according to naming convention and other variables in the same contract
```solidity
 /// @notice Mapping of target to selector to actionGuard address.
  mapping(address target => mapping(bytes4 selector => ILlamaActionGuard)) public actionGuard;
```
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L206
