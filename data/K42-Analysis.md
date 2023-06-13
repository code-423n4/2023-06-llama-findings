## Llama Audit Analysis Report by K42

## Overview
Llama is a governance system for on-chain organizations. It uses non-transferable NFTs to encode access control, features programmatic control of funds, and includes a modular framework to define action execution rules. The Llama system consists of several contracts, each serving a specific purpose within the system.

## Codebase Analysis
The Llama codebase is well-structured and modular, with each contract serving a specific purpose. The contracts are written in Solidity and make use of libraries and interfaces to reduce code duplication and increase code reusability. The codebase also makes use of the OpenZeppelin library, which is a well-tested and community-reviewed library for secure smart contract development.

The contracts in the Llama system include:

- LlamaExecutor.sol: The exit point of a Llama instance. It calls the target contract during action execution.
- LlamaPolicyMetadataParamRegistry.sol: Parameter Registry contract for on-chain SVG colors and logos.
- LlamaAbsoluteQuorum.sol: A Llama strategy that has an absolute threshold for approvals/disapprovals and the action creator can approve or disapprove their own actions.
- LlamaPolicyMetadata.sol: Utility contract to compute llama policy metadata.
- LlamaRelativeQuorum.sol: A Llama strategy in which approval/disapproval thresholds are specified as percentages of total supply and action creators are allowed to cast approvals or disapprovals on their own actions.
- LlamaFactory.sol: Factory for deploying new Llama instances.
- LlamaGovernanceScript.sol: A script that allows users to aggregate common calls on the core and policy contracts.
- LlamaAccount.sol: This contract can be used to hold assets for a Llama instance.
- LlamaPolicy.sol: An ERC721 contract where each token is non-transferable and has roles assigned to create, approve and disapprove actions.
- LlamaCore.sol: Manages the action process from creation to execution.

## Potential Areas of Concern
The following areas were identified as potential areas of concern:

- Unauthorized action state transitions.
- Unauthorized approval and disapproval manipulation.
- Vulnerabilities in the roles and permissions system in LlamaPolicy.sol.
- Additional safety checks to help prevent Llama instances entering a state where new actions cannot be executed.
- Risks that stem from LlamaExecutor.sol delegate calling scripts.
- Vulnerabilities in LlamaAccount.sol that could lead to unauthorized access to funds and assets held in it (Especially risks that stem from the LlamaAccount.sol being able to delegate call arbitrary contracts).
## Recommendations
Based on the analysis, the following recommendations are made:

- Review the roles and permissions system in LlamaPolicy.sol to ensure that it is robust and secure.
- Implement additional safety checks to prevent Llama instances from entering a state where new actions cannot be executed.
- Review the delegate calling scripts in LlamaExecutor.sol to ensure that they do not introduce any security risks.
- Review the LlamaAccount.sol contract to ensure that it does not allow unauthorized access to funds and assets.

## Conclusion
The Llama system is a well-structured and modular system for on-chain governance. While the codebase is generally well-written and secure, there are potential areas of concern that should be addressed to ensure the security and robustness of the system.

### Time spent:
16 hours