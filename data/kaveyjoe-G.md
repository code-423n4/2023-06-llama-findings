1. Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol

Minimize storage reads: in the validateActionCreation function, the llamaPolicy variable can be accessed directly instead of assigning it to a new variable.

 in the validateActionCreation function, you can combine the two revert statements by using an && operator.

 if the roles approvalRole and disapprovalRole have a limited range, consider using smaller data types such as uint8 instead of uint256.

2 . Target :https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol

   -    Minimize the number of times you read from storage, as storage reads are expensive in terms of gas. In the validateActionCreation function, you can store llamaPolicy in a local variable instead of accessing it multiple times.

function validateActionCreation(ActionInfo calldata actionInfo) external view override {
  LlamaPolicy llamaPolicy = policy; // Reduce SLOADs.
  // ...
}


-  Simplify validation conditions: Simplify the validation conditions to reduce gas usage, instead of checking if the approvalPolicySupply and disapprovalPolicySupply are equal to zero separately, you can combine them into a single condition.


-Use require instead of revert: In the error handling code, use the require statement instead of revert when possible. The require statement is cheaper in terms of gas compared to revert.

if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);

can be changed to:

require(approvalPolicySupply != 0, "Role has zero supply");


3 . Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol

-Use bytes32 instead of string for fixed-size data: The color and logo parameters in the tokenURI function are expected to be fixed-size strings. Instead of using string,  can use bytes32 to represent these parameters. It will save gas costs compared to string types.

-Avoid unnecessary memory allocation: The parts array in the tokenURI function is declared with a fixed size, even though not all elements are used. You can optimize the array size to match the actual number of elements used in the string construction.

-Optimize loop iterations: The LibString.slice function is called twice in the tokenURI function. Each function call uses a loop to copy characters from one string to another.  can optimize this by avoiding the second call to LibString.slice and reusing the initial result.

4 .  Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

Avoid Redundant Storage Access: Minimize unnecessary storage reads and writes.  in the initialize function, strategyConfig.approvalRole and strategyConfig.disapprovalRole are assigned to approvalRole and disapprovalRole, respectively, but they are also checked later with _assertValidRole.  can eliminate redundant assignments to optimize gas usage.

5 . Target :https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

Minimize External Function Calls: External function calls are expensive in terms of gas. Try to minimize the number of external calls within functions.  in the _setDeploymentMetadata function, you can consider combining the setColor and setLogo calls into a single function to reduce gas costs.

6 . Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

- In the initializeRolesAndSetRoleHoldersAndSetRolePermissions and other similar functions, there are multiple function calls to initializeRoles, setRoleHolders, and setRolePermissions. Instead of making separate calls,  can combine these operations into a single function to reduce gas costs.

- In the setRoleHolders and setRolePermissions functions, there are loops iterating over arrays to perform individual operations. If possible, you can batch these operations by modifying the contract to accept arrays of multiple values and process them in a single iteration, reducing the gas cost per operation.

-Reduce unnecessary data copying: In the aggregate function,  can pass the data array as a memory reference (bytes[] memory data) instead of calldata. This avoids unnecessary data copying and reduces gas consumption.

-Optimize loop variables: In several for loops, the loop variables are incremented using LlamaUtils.uncheckedIncrement. If the loop doesn't have any special requirements, we can use the standard i++ increment operator instead.

7 . Target :https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

- Check if any storage writes can be eliminated or reduced.  in functions like transferNativeToken,  can remove the line nativeTokenData.recipient.sendValue(nativeTokenData.amount); and directly use nativeTokenData.recipient.call{value: nativeTokenData.amount}(""); as it consumes less gas.

-Where applicable, use batch operations like safeBatchTransferFrom instead of individual transfers. This reduces the number of external function calls and saves gas.

8 . Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

- Instead of storing the contract name and symbol in storage variables, you can directly define them as constant variables. This change will save gas by avoiding unnecessary storage read operations.

string public constant name = "YourContractName";
string public constant symbol = "SYM";

-Optimize Safe Transfer Functions: The safeTransferFrom and safeTransferFrom functions call the onERC721Received function of the recipient. To reduce gas consumption,  can combine the common logic of these functions into a single internal function and call it within both functions.

function _safeTransferFrom(address from, address to, uint256 id, bytes memory data) internal virtual {
    _transferFrom(from, to, id);

    if (to.isContract()) {
        bytes4 onERC721ReceivedSelector = ERC721TokenReceiver(to).onERC721Received(msg.sender, from, id, data);
        require(onERC721ReceivedSelector == ERC721TokenReceiver.onERC721Received.selector, "UNSAFE_RECIPIENT");
    }
}

function safeTransferFrom(address from, address to, uint256 id) public virtual {
    _safeTransferFrom(from, to, id, "");
}

function safeTransferFrom(address from, address to, uint256 id, bytes calldata data) public virtual {
    _safeTransferFrom(from, to, id, data);
}


9 . Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

-Inline Libraries:  the imported libraries (Initializable.sol and FixedPointMathLib.sol) are small and used only in this contract, you can consider copying their code directly into this contract. This eliminates the need for separate library calls and reduces gas costs.

- Optimize data storage: Review the storage variables and data structures used in the contract. Consider if any mappings or arrays can be optimized or removed to reduce gas costs,can check if the mappings forceApprovalRole and forceDisapprovalRole can be replaced with a more efficient data structure.

10 . Target : https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol

-Simplify calculations: Look for opportunities to simplify calculations or remove redundant computations. For example, in the _lowerBinaryLookup and _upperBinaryLookup functions, the average of two numbers is calculated using bitwise operations. can replace this with a simpler calculation.

- Split the getAtProbablyRecentTimestamp function into smaller, more focused functions if possible. This helps to reduce the complexity and gas consumption of individual functions.

- In the _insert function,  create a copy of the last checkpoint using Checkpoint memory last = _unsafeAccess(self, pos - 1);. Instead of copying the struct,  can directly access and modify the last checkpoint in the array, which can save gas by avoiding unnecessary memory operations.






