### My method to approach the codebase

This contest is 2000 sLOCs within the duration of only 7 days. The duration is quite short for the large codebase and the topic (Governing mechanism) is not so familiar to me, so at first I tried to focus on reviewing different types of strategies. Then after following the flow of action (Creation, cast approval, disapproval,..) and analyze the interactions between 3 core components: `LlamaCore`, `LlamaPolicy` and `LlamaStrategy`, I found some bugs that are the results of these interactions.

My approach mainly involves following the process in which actions get through, from creation to different states like Active, Canceled, Expired,...reading the involved functions of core components (`LlamaCore`, `LlamaPolicy`,`LlamaStrategy`) and analyze the impact of different scenarios.

### What I think the protocol did well

- The protocol did a good job in designing the architecture, in which the core, the policy and strategies are separated, the separation makes it easier to implement security on each component and also extending the protocol in the future.
- The fact that `guard` is only an interface allow users to implement their custom security policies.
- The pattern `if...revert` is used extensively, even functions that sound like they should return boolean value e.g `isDisapprovalEnabled`, this may cause some trouble to users who insist on getting the results instead of revert but is great in terms of security because it terminate the code right when detecting if something wrong.
- I think the action's state is managed quite well, I found no way to bypass states or do anything not according to the flow.

### What I think the protocol could improve

- Maintain the consistency of state actions functions (like `isActionExpired`,`isActionDisapproved`,...) and `LlamaCore.getActionState` function.
- Introduce some functions allowing users to undo their mistaken acts, for example cancel their approval/disapproval cast.
- Parameters passed to creating strategies should be checked more carefully, for example the values of `approvalPeriod`, `queuingPeriod`, `expirationPeriod` is not checked at all, with special values (0 or very large), there could be some unpredictable consequences.
- Users should reserve the right to revoke their roles if they want to, the `LlamaExecutor` currently has absolute power over assigning/revoking roles to everyone.

### Time spent:
20 hours