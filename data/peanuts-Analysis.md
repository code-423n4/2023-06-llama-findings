### 1. Any comments for the judge to contextualize your findings

Findings largely focuses on Action creation instead of the policy side.

### 2. Approach taken in evaluating the codebase

Due to limited time, I focused only on the "Action" part of the code, such as actions are created, approved, queued, cancelled, executed, failed, and expired. Scrutinized the flow of actions and any logical errors that may be present in the code. 

### 3. Architecture recommendations

Some functions that are not used should not be in the code, like safeMint(). Guard check is done well and adds an additional layer of security.

Probably good to have more docs, a good workflow and explanation of how llama core and llama policy is created as instances from llama factory, how accounts are created, who can be a policy maker, what are accounts, what is the executor. 


### 4. Codebase quality analysis

The encoding and hashes are done well from a glance, and the extension to incorporate svgs is also done well. Cloning and proxies are written well from a glance as well, with clonedeterministic using two parameters instead of one. 

Adding additional hooks like Guard is a nice touch in terms of security. Interesting use of scripts to batch multiple calls together into one action, despite the generally unsafe use of delegatecall.

The codebase is quite confusing for newer developers/auditors because of several high level designs and usage of higher level architecture like minimal proxies and clones. It is also a little hard to navigate between files, and one must have a good knowledge of how solidity works to understand what the code is trying to do. For example, the `validateActionCreation` function in the strategies contract has a line that states `LlamaPolicy llamaPolicy = policy;`. At a glance this is extremely confusing because the policy declared is not found in the strategies contract. In fact, it is pointing towards the instance of the policy contract which is initialized in the llamacore contract which is initialized in the factory contract. 

Another instance is the confusion between the word strategies and the actual strategy contract, as call to interfaces usually means call to the contract with the same name. For example, when creating an action, the policyholder passes in `ILlamaStrategy`, but it actually refers to either of the four strategy contracts that will determine how the action is executed, and not the actual `LlamaStrategy.sol` contract (which doesn't exist).

Cancellation state should not be in the strategy contracts but in the llamaCore contract, together with `createAction`, `executeAction`, and `queueAction` for better readability.

Overall, quality of code is good with proper design structure and testing coverage, but readability may be difficult to newer developers/auditors. 

### 5. Centralization risks

Appointing anyone as a policy holder, giving important policies to an unknown entity and manipulating the approve-disapprove mechanism.

### 6. Mechanism review

Approval and disapproval is a unique, such as only being able to approve during approval phase and only being able to disapprove during queued phase. Conventionally, the concept approval and disapproval should be used in tandem, for example, some people approve the approval phase, whereas some people disapprove the approval phase. Code interweaves both approve and disapprove under `_preCastAssertions`, which is extremely interesting. Good to know about this unique mechanism of approving and disapproving. 

Action State checks are pretty refreshing as well, order is extremely important - return canceled state first, followed by executed, active, failed, approved, failed, expired and finally queued. There are 7 states in an Action flow: Active, Canceled, Failed, Approved, Queued, Expired, Executed.

Generally, the sequence goes like this if no problem arises:

```
Active -> Approved -> Queued -> Executed
```

In most sequences of the event, there is a time buffer. Here is the full sequence and the interactions with the time buffer.

```
Acitve -> Approved
Time buffer: approvalPeriod.
Explanation: Action must gain a minimum amount of approval to be set from Active to Approved. Else, the Action will end in a failed state.

Approved -> Queued
Time buffer: -
Explanation: Once the threshold of Approvals is reached within the approvalPeriod, isActive() becomes false and the state automatically shifts to Queued

Queued -> Execution
Time buffer: queuingPeriod
Explanation: Once the Action reaches the Queued stage, a period of time is given (queuingPeriod) for policyholders to vote on whether to disapprove the action. If the disapproval threshold is breached, then the action will end in a failed state. Otherwise, if there is little disapprovals, the action will go to the execution stage.

Execution -> Expiration
Time buffer: expirationPeriod
Explanation: When the Action reaches the execution stage, the creator will have a period of time (expirationPeriod) to execute the action via LlamaCore.executeAction. If the action is not executed within the expirationPeriod, then the Action will end in an expired state.
``` 
 
After a thorough look through, sequence seems to be executed correctly, although the time buffer should have more security in case the executor accidently sets any time buffer to zero since there is no deletion of improperly written strategy contracts.

### 7. Systemic risks

-NIL-


### Time spent:
8 hours