## Foreword

This is the first time we participate in a contest on a governance system. While this does pose certain difficulties in understanding the system - it is a great learning experience, and we are also lucky to have the contest 2 days longer than usual contests with the same prize pool (perhaps because the contest goes through a weekend).

## Approach taken

The first step is to understand what we're auditing. A governance is basically an entity, that consists of many people, who generally takes the following actions:
- Create a proposal (e.g. incentives for certain pools, transfer something out of treasury, pausing certain actions etc.).
- Vote on/approve the proposal.
- Perform the action in the proposal, if it is sufficiently approved.

A good example to contextualize a governance is https://snapshot.org/, a lightweight governance platform, although it does not enforce permissions as strictly as Llama does due to its offchain nature.

With the picture of the governance clear, an oversimplified way to explain a Llama instance is "a very customizable multisig". This gives a good hint on what we're supposed to look for in the audit: Focus on permissions.
- Is there anything that a governance should be able to do, but can't?
- Can an outsider influence a governance, whereas they shouldn't be able to?

Thus we focus quite rigorously on permission checks, whether they're sufficient, and whether there are entrypoints that can bypass those.

For the purposes of an audit, we take the following approach:
- Focus on LlamaCore, assumes everything else correct, since it's the main user entrypoint.
- Try to break the previous assumption by auditing Policy/Executor/Scripts after getting the full picture of LlamaCore.
- Audit Strategies separately, since they have very separated logic from the rest of the protocol, it's meant to be an abstract, and its interaction with LlamaCore are rather limited. It is also distinctive from the rest of the codebase that it deals with the calculation behind voting power, and thus is more mathematical rather than procedural.
- Audit Factory - aside from creation and initialization, this does not interact with the rest of the system in any way. Furthermore it is not often that there's a problem with the factory contract, and it has already been audited by Spearbit, so we reckon it's better use of time to focus on the more critical parts.

## Centralization risks

There should not be centralization risks here due to its nature as a governance system.

A centralization risk **would** be a H/M issue, given that this system is all about permissions, and the purpose of the system itself is to improve decentralization within a governance. A centralization risk in this context can be defined as a single entity holding more power than it should in the governance, unless the governance itself by design has a single controlling person/entity.

## Codebase feedback

The highlight of the codebase lies in the documentation and naming:
- The contest provides wardens with clear and concise documentations, along with easy-to-understand flow diagrams on the protocol use-case
- Each functions come with clear and comprehensible documentation. Reasonings in the form of code comments are also given where necessary.
 
The lowlight lies in the inconsistency:
- There are three interfaces provided in the repository. However, while the `LlamaCore` is the only entrypoint for any user-initiated transactions (aside from Factory and Lens), it doesn't have an interface.
    - Normally we would expect interfaces to not just be inherited, but also to provide contexts on what is possible (e.g. interfaces are a great way to determine which functions are external - they are the main entrypoints of the system).
    - All of the docs are provided in the interfaces where applicable. Because LlamaCore doesn't have a separate interface file, the docs lie in the main contract. This causes a certain level of confusion, as *we didn't go straight to the Core contract first in our audit*, thus it's a bit difficult to find where the missing puzzle pieces are.
- Strategies `LlamaAbsoluteQuorum` and `LlamaAbsolutePeerReview` inherits `LlamaAbsoluteStrategyBase`. However, `LlamaRelativeQuorum` does not. This creates an inconsistency, especially when considering the Strategy Base has all functions marked as virtual, which are meant for overriding.

We also find it helpful that the sponsors provide previous audit's report with all the issues fixed/acknowledged, as well as highlighting special area of focuses in the contest README. They are very much help in contextualizing what was likely the main the area of focus in the previous audit, as well as what the sponsors are looking for in a contest-based auditing model as C4.

## Recommendations

- For the sake of consistency, we suggest providing an interface for `LlamaCore` with documentations, as with any existing interfaces.
- One thing we found would've been helpful on hindsight for the auditing is a sequence diagram, describing user interactions with the system.
    - While there was a diagram describing the architecture with how contracts interact with each other, there is no diagram describing how external actors interact with the system. The action state diagram is close, but more mentions on what contract are executed at which step would be helpful.
- Since the governance system is heavily focused on permissions, we speculate that the [Certora formal verification tool](https://github.com/Certora/Examples) will greatly benefit this project. The Certora FV is a tool to formalize any assumptions into code form, and will stress test the code to (attempt to) break any assumptions made. 


### Time spent:
18 hours