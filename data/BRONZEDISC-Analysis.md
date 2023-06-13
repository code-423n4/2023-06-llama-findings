#### 1 - What approach did you take in evaluating this codebase in order to help you grow your skills and code review resources? 

To evaluate the codebase I started with a high-level overview of the main contracts (Factory, Executor, Core, Policy) and then took a look at them and how they interact with each other, after giving a look at the LlamaExecutor.sol I realized the false condition from the execute() function would always revert since the function is not payable. 
  I can be mistaken and this can actually not be a high vulnerability but it made me believe that the whole repo has a high probability of having multiple high vulnerabilities, since I had spent just few hours with a limited time. Then another thing that got my attention was the ERC721NonTransferableMinimalProxy.sol contract implementing a whole new variant of the ERC721 which I think to be very dangerous, then I realized that the function _safeMint() is implementing this checking and I immediately found a way around it. 
```solidity 
require(to.code.length == 0,"UNSAFE_RECIPIENT");
```
This can be bypassed by creating an Attacker contract with the opcode CREATE2 which gives the future address without the contract having any length yet, hence passing through the `to.code.length` and potentially creating new critical vulnerability attack vectors (it worked as I expected after running some tests) what I still don't know is how to create an attack vector with it.


### 2 - What did you learn from reviewing this codebase? Be specific.
  I learned that checking if the an address is a contract by doing `to.code.length` is not a good idea because of the CREATE2 opcode that predicts the future address. 
  Also I reinforced my beliefs that rewriting protocols like ERC20/ERC721s are very dangerous and I'm sure that wardens will come up with ways to gamify the `ERC721NonTransferableMinimalProxy.sol` contract in specific.


### 3 - How much time did you spend?
  I spent approximately 20 hours reviewing the codebase. Not just reading the code but getting in depth on the concepts I explained above and understanding them better, also running some tests with Foundry.


### Time spent:
15 hours