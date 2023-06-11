# [L-01] IMMUTABLE ADDRESSES LACK ZERO ADDRESS CHECK

Constructors should check the address written in an immutable address variable is not the zero address.

##### Reference
[LlamaExecutor.sol#L12](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaExecutor.sol#L12)
[LlamaExecutor.sol#L16](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaExecutor.sol#L16)
[LlamaPolicyMetadataParamRegistry.sol#l43](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaPolicyMetadataParamRegistry.sol#L44-L45)
##### Mitigation Step
Add a zero address check for `LLAMA_CORE`

# [L-02] ENSURE THAT `address target` MUST BE VALID IN FUNCTION `execute` 

In the function `execute` passing all arguments along with a null address does not throw any null address check error. Failing to do so may lead to unexpected behavior in the contract.

##### Reference [LlamaExecutor.sol#L29]

##### Mitigation Step

Consider adding a validation address check in the function `execute`


# [N-01] NOT USING A GOOD PRACTICE FOR NETSPAC COMMENTS

##### Reference

[LlamaExecutor.sol#L21-21](https://github.com/code-423n4/2023-06-llama/blob/9d641b32e3f4092cc81dbac7b1c451c695e78983/src/LlamaExecutor.sol#L21-L22)

##### Mitigation Step 

Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction)

# [N-02] ALL ERRORS SHOULD BE DEFINED IN A SEPARATE ONE CONTRACT TO AVOID WRITING ERRORS IN MULTIPLE CONTRACTS

Defining errors in a separate smart contract file provides modularity and reusability. By having a dedicated error contract, you can define all the possible errors. making it easier to manage and maintain your code. Additionally, it allows you to import the error contract in other contracts that may need to handle or propagate these errors, promoting code reuse and consistency

##### Reference

All Contracts

# [N-03] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURES

The lack of a two-step procedure for critical operations leaves them error-prone. Consider adding a two-step procedure on the critical functions.

``` solidity
function setApprovalForAll(address operator, bool approved) public virtual {
    isApprovedForAll[msg.sender][operator] = approved;

    emit ApprovalForAll(msg.sender, operator, approved);
  }
```

##### References

[ERC721NonTransferableMinimalProxy.sol#L81](https://github.com/code-423n4/2023-06-llama/blob/6b3e3e2b1023c428298d6d72d906a5e89df4b227/src/lib/ERC721NonTransferableMinimalProxy.sol#LL81C18-L81C18)
[LlamaCore.sol#L447](https://github.com/code-423n4/2023-06-llama/blob/6b3e3e2b1023c428298d6d72d906a5e89df4b227/src/LlamaCore.sol#L447)
[LlamaPolicy.sol#L238](https://github.com/code-423n4/2023-06-llama/blob/6b3e3e2b1023c428298d6d72d906a5e89df4b227/src/LlamaPolicy.sol#L238)

##### Mitigation Step

Consider adding a two-step pattern on critical changes to avoid mistakenly setting approval for all function or critical functionalities to the wrong address.

