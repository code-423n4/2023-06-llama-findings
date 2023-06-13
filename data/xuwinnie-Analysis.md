This analysis report presents a refined permission control framework for on-chain governance and provides recommendations to enhance the flexibility of llama's permission control.

### Introduction
What is governance? Llama is the core of governance, and the governance process involves using strategy S to decide whether llama will execute action A. We denote this process as (S, A). There can be various types of strategies, for example, a significant matter may require a 2/3 majority vote, while a routine decision may only need a 1/2 majority vote. Actions also fall into different categories, some change llama's internal state while others alter external states.

In llama, we denote the action that initiates a governance process as a' = S(s,a). However, not all roles have the privilege to initiate a process (possess a'). The function setRolePermission is used to address this. We use anyAction(c,s) to refer to all possible operation on a function f of a contract c. What function setRolePermission do is to give role r the permission to S(s, allAction(c,s)), where s is a specific strategy. We name this action a''= P(r,S(s,allAction(c,f))). The initialization process involves assigning the bootstrap role the bootstrap permission can be represented as P(bootRole,S(bootStrategy,P(anyRole,S(anyStrategy,allAction(anyContract,anyFunction))))), here "any" refers to arbitary input parameter.

### Insufficient Flexibility in the Current Framework
Currently, in llama, the only available action type for permissions is setRolePermission, denoted as P(r,S(s,allAction(c,f))). This limitation prevents us from achieving the following functionalities:
1. Granting certain roles the right to perform specific actions without the need for voting, such as allowing a super administrator to directly assign the execution right of an action without requiring a vote, P(superRole,a).
2. Granting a role the proposal right to operate on a contract (without specifying a function), S(s,allAction(c)).
3. Granting a role the proposal right to specify calldata operations on a function, S(s,allAction(c,f,d)).
4. Granting a role the proposal right to any function with a conservative strategy s, S(s,allAction).
5. Meta-operations regarding permits, such as authorizing role r to grant any role the ability to perform a specific action: P(r,P(anyRole,a)).

### More Concise and Universal Permission Control
To achieve a more concise and universal permission control, we can modify setRolePermission. We still use permission[role][permissionId] to record permissions. However, permissionId no longer represents the permission for creating an action, it can represent all three types of actions: S(s, a), P(r, a), and External. Here parameter could be a set (e.g., specific roles, any strategies, or all S-type actions) ("any" is different from "all"!). By setting up encoding and decoding rules for S(s, a) and P(r, a) operations, we can directly submit the corresponding permissionId when starting an action. Then the contract decodes the corresponding action and strategy for further processing. Similarly, when setting role permissions, we can submit the possessed permissionId and the contract will decode the corresponding role and action using the P decoding process. This approach creates a more concise and universal way to express any type of permission control.

#### Support External Role Balance (independent from previous part)
Currently, the role balance is stored internally, which limits its scalability. For example, we maybe want to directly use the balance of a governance token as the role balance. So it would be better if we support fetching the role balance from an interface of external contract.


### Time spent:
10 hours