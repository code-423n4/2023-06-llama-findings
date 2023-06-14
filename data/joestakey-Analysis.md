Llama introduces a complex yet elegant infrastructure to give more granularity to on-chain organizations.

I evaluated the codebase following the architecture diagram given in the docs:
- LlamaFactory and PolicyMetadata handling
- Llama Instance (including the Quorum Logic)
- Governed contract (ie the LlamaAccount)

You will find below an overview of my analysis method, what risks can arise from the design decisions and whether these were tackled appropriately, and high-level conclusions of the potential problems encountered - which are detailed in the vulnerability reports sent.

## LlamaFactory and PolicyMetadata handling

I reviewed parameters used in deployments, focusing on the validation done to ensure no deployments could be made with an erroneous configuration.

The `BOOTSTRAP_ROLE` ensures a safety net is always present in a Llama instance. While it does not guarantee fully functioning configuration, it is sufficient in the context of the factory, which is governed by the protocol.

The handling of custom URI in `PolicyMetadata` requires care to be taken in the string handling, as problems can arise from malformed data strings in front end tools. More specifically, the "arbitrary" data in string concatenation could use more validation, as it can potentially lead to errors in off-chain tools querying those view functions.

## Llama Instance

The core contract was separated from the policy logic, the execution logic, and the strategy logic.

This modular approach not only makes the codebase more readable, but also adds a layer of security - especially on the executor side, preventing modifying the core instance during arbitrary action executions, ensuring safe use of delegatecalls.
On the other hand, care needs to be taken to ensure the flow of data and value through the different "blocks" (ie contracts) is correct.
In this modular instance, there is different storage in each part (policy, core, strategy). This requires care to be taken in storage updates, to ensure each storage update is in "sync" with the others.

The review was conducted to ensure:

- valid state transitions
- approval/disapproval soundness
- correct role validation

All the above points were found to be appropriately handled, having enough sanitization and safeguards to ensure a smooth and safe action flow.

More attention was then taken onto the `LlamaExecutor` - as it seems to have been amended following the latest audit.

A few flaws were found regarding the native token handling in the execution flow (more details in the dedicated reports)


## LlamaAccount

As this contract is a template for the governed contracts that will perform execution logic, the focus was on the functions handling token approvals and transfers, ensuring in particular the soundness of function arguments.

No issues were found in this area.

### Time spent:
12 hours