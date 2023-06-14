
## Overview

Llama system can be decomposed into two sets of contracts:
1. Factory contract and its peripherals. Factory is deployed once and managed by Llama itself.
2. Instance contracts. These are managed by the DAOs that use Llama product and deployed once for each DAO. Instance contracts are deployed by the factory.

These two types need different security considerations in regards to access and ownership. This is further expanded upon in other sections.

The codebase is clean and has good separation of concerns between contracts. Relationships are mostly one-way, many components are self-contained or have little external dependencies. This helps with understanding the codebase and reduces chance for introducing bugs. Comments are also helpful and appropriate.

Documentation is good, however it doesn't cover all aspects of the protocol. I would suggest adding documentation for the following:
- Strategies. How strategies can be created, how much flexibility can be achieved. Examples for most commonly used strategies.
- Bootstraping new Llama instances. Factory logic and precautions taken to make sure new deployments are valid. The importance of BOOTSTRAP_ROLE.
- Action Execution. Some examples of different types actions. Constraints and risks.

## Audit approach

All core components were audited. Two areas that weren't checked in-depth are SVG rendering and the code copied from OpenZeppelin libraries (diff was reviewed).

Audit was performed by manually reviewing the code. For simplicity the code was divided into the following components: 
- Policy
- Execution (LlamaExecutor, LlamaAccount)
- Strategies
- Core (Links all previous components together)
- Factory

Components were audited in isolation where applicable and also in integration with each other. Tests were used to dive into leads and findings.

## Code Commentary

#### Executor
I really like the idea of separating executor in a separate class as an additional security measure. Same for the way LlamaAccount checks that slot 0 hasn't been changed after executing `delegatecall`. These measures make execution logic more robust and secure.

#### Policy

The idea of making LlamaPolicy an NFT can be challenged. While it allows to display the policy in NFT visualization tools, it also complicates the implementation. A single mapping would be a much simpler way to assign policies to addresses. Instead, the code relies on internal NFT logic and has to perform minting/burning as well as removing transferability. Also, it feels that non-transferability logic belongs to `ERC721NonTransferableMinimalProxy`  and not LlamaPolicy itself.

In regards to dependencies, it should be possible to fully decouple LlamaPolicy from executor as a way to simplify operation flow. Executor is currently used to prevent changing roles in the block that already contains created action. This check can be done differently; for example, with an action guard in LlamaCore.

#### Strategies

Strategies are some of the more convoluted parts of the codebase. There's plenty of code duplication between absolute and relative strategies. Perhaps relative strategy can be treated as an absolute but with voting threshold calculated dynamically each time when creating an action.

In order to reduce coupling between components it is possible to avoid referencing LlamaCore in LlamaAbsoluteStrategyBase and LlamaRelativeQuorum. I believe keeping strategies dumb and unaware of other contracts will bring more simplicity to the code. Strategy functions are called by LlamaCore, so, instead of making a call back, pass `Action` struct directly in `isActionApproved`, `isActionDisapproved`, `isActionExpired`, `approvalEndTime`.

## Non-functional aspects

Code compilation takes a long time for some reason (several minutes with optimizations disabled). This affects test runs and hinders developer experience. The issue should be investigated as a part of tech debt if further development is planned. 

Code coverage should be improved for the following contracts:
- LlamaGovernanceScript - 53.45%
- LlamaAbsoluteStrategyBase - 86%
- LlamaAbsoluteQuorum - 90.91%

It's great to see fuzzing tests being used.

## Risks

1. The factory contract of the protocol is governed by Llama DAO. It is intended to use Llama itself for managing the factory. While it is a great goal in general, it might be worth considering more battle tested multisig solutions for the initial launch. 

2. Instance contracts rely on factory for adding accounts and strategies. This dependency has several disadvantages. It makes Llama instances less flexible by only allowing a restricted set of accounts/strategies to be used. And most importantly, it brings risk in case something happens to the Llama DAO. If adding accounts and strategies was fully permissionless, then Llama instances would have been completely self-contained and that would have increased security properties of the protocol.

3. The instance contracts may be at risk in case of user errors from DAOs using the protocol. This is why it's vital to have in-depth documentation for instance management. Another approach for preventing user errors is creating a set of default action guards to be deployed by default for each Llama instance.

4. In metadata, certain properties are hardcoded to llama.xyz domain. Might be better to make them upgradeable in case the domain is lost. With upgradeability enabled, it is further possible to move images to IPFS.



### Time spent:
24 hours