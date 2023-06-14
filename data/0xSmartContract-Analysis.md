# 🛠️ Analysis - Llama 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Architectural | Architecture feedback |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |Systemic risks | Potential systemic risks in the project |
|h) |Competition analysis| What are similar projects? |
|i) |Security Approach of the Project | Audit approach of the Project |
|j) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|k) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|l) |Project team | Details about the project team and approach |
|m) |Other recommendations | What is unique? How are the existing patterns used? |
|n) |New insights and learning from this audit | Things learned from the project |
|o) |Resources - Frameworks | Details of the sources, documents and frameworks used in the above analysis |
|p) |Time spent on analysis | Time detail of individual stages in my code review and analysis |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-06-llama/tree/main#scoping-details

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-06-llama/tree/main#installation)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [video](https://www.loom.com/share/2cd1513d4ee34aa8ace6df51e402c259) explainer provides a high-level overview of the Llama system and the [docs](https://github.com/code-423n4/2023-06-llama/tree/main/docs) describe the core components.|Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-06-llama/blob/main/.slither-report)|The Slither output did not indicate a direct security risk to us, but at some points it did give some insight into the project's careful scrutiny. These ; 1) Re-entrancy risks                                    2) Pragma version0.8.19 necessitates a version too recent to be trusted.|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-06-llama/tree/main/test)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-06-llama/tree/main#scope)|According to the architectural design, the codes were examined starting from the factory contract, which is the main starting point.|
|6|Project Spearbit Audit |[Spearbit](https://github.com/code-423n4/2023-06-llama/blob/main/audits/Llama-Spearbit-Audit.pdf)|According to the project team on the Discord channel, all the problems related to this audit have been fixed, this report gives us a clue about the topics that should be considered in the current audit (Ex: Access Control)|



## b) Analysis of the code base

Llama is a governance system for onchain organizations. It uses non-transferable NFTs to encode access control, features programmatic control of funds, and includes a modular framework to define action execution rules.

Code architecture ; It starts in a structure that produces instances with the Factory and clone structure that we see in many projects.

##### The first stage of the audit; 
Distribution of Llama instances has been examined by Factory. [LlamaFactory](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol)
Deploys a new Llama instance .Factory contract using OpenZeppelin's "Clones.sol" library. Llama examples are in Minimal Proxy structure.
The `deploy()` function is the most important review function and coordinates the deployment, especially the access control privileges are checked

##### The second stage of the audit; 
It is the [LlamaCore.sol](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol) contract used by Factory for distribution of Llama instances. This contract is central to all projects, therefore its interaction with all contracts is critical and the usual suspect from a security point of view.
Since the LlamaCore contract is a proxy contract, security queries were made in the "Initializable" structure (For example; Openzeppelin "Initializable.sol" was imported, imported with the `is Initializable` keyword, double execution was prevented with `_disableInitializers()` in the constructor.

The function [executeAction()](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L317) interacts with the `LlamaExecutor.sol` contract by external call and It runs the `execute()` function there, the reason for this is explained in the project as follows;

``` diff
Using a separate executor contract ensures `target` being delegatecalled cannot write to 
`LlamaCore`'s storage. By using a sole executor for calls and delegatecalls, a Llama instance 
is represented by one contract address.
```

The `LlamaCore` contract uses the EIP712 standard, we confirm whether it uses this standard correctly [eip-712](https://eips.ethereum.org/EIPS/eip-712)
The alternative in this section is the use of eip-5267; [eip-5267](https://gist.github.com/itsmetechjay/610f1b31f419156f06898ee10c89402d#n02-consider-implementing-eip-5267-to-securely-describe-eip-712-domains-being-used)

##### The third stage of the audit;
[LlamaPolicy](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol)
An ERC721 contract where each token is non-transferable, functions as the respective policy for a given policyholder and has roles assigned to `create`, `approve` and `disapprove` actions.

What makes this contract different; It is a non-transferable token management. The focus should be on security in this section.

##### The fourth stage of the audit;
Other contracts were examined in the final stage.

## c) Test analysis
There is a comprehensive test kit with many tests including fuzzing tests using Foundry
Test coverage is 90% and content quality is above average


## d) Architectural 

The design of the project is based on a structure that produces minimal proxy clone with Factory, this structure is frequently used since Uniswapv2.
Llama Core Instances cloned by the Factory contract are the base contract and provide project authorizations and flow

The script structure is the original part of the project, it is not a structure that is seen mostly, so the focus should be on the code control here.

Innovative proposal to the project: Instead of the Factory + Instance structure, the only contract structure used in Uniswapv4 can be used, this is a radical solution that will reduce gas costs by 90% at the same time: “singleton” contracts from Uniswapv4

https://blog.uniswap.org/uniswap-v4#what-is-uniswap-v4

## e) Documents 
It is recommended to increase the documents in terms of auditability of the project, only the general structure is explained.
https://github.com/code-423n4/2023-06-llama/tree/main/docs

It is stated in the documents that the project uses `timelock`, but there is no timelock function.
```
README.md:
  146: - Does it use a timelock function?: Yes

```

The philosophy of the project is not stated in the documents, the main philosophy stated in the figure below can help this;

Philosophy of the Project:
In order to understand the Philosophy of the Project, it is necessary to start by looking at the differences between traditional governance and on-chain governance.

**Centralization vs. decentralization:** Traditional governance is typically centralized, meaning that there is a single entity or group of entities that holds power and makes decisions. On-chain governance, on the other hand, is decentralized, meaning that power is distributed among all participants in the network.
**Transparency vs. opacity:** Traditional governance is often opaque, meaning that it is difficult or impossible for stakeholders to know how decisions are being made. On-chain governance, on the other hand, is transparent, meaning that all decisions are made on the blockchain and can be viewed by anyone.
**Speed vs. slowness:** Traditional governance can be slow, as it often requires a lot of deliberation and consensus-building. On-chain governance, on the other hand, can be much faster, as decisions can be made quickly and easily through voting.
**Cost vs. free:** Traditional governance can be expensive, as it often requires the hiring of lawyers, consultants, and other experts. On-chain governance, on the other hand, is free, as it is all done through the blockchain.

Overall, on-chain governance is a more transparent, decentralized, and efficient way to make decisions. However, it is important to note that on-chain governance is still a relatively new concept, and there are still some challenges that need to be addressed. For example, it can be difficult to get a quorum of participants to vote on proposals, and there is always the risk of malicious actors manipulating the voting process.

Despite these challenges, on-chain governance has the potential to revolutionize the way that organizations are governed. It can help to make organizations more transparent, accountable, and efficient. As the technology continues to develop, on-chain governance is likely to become more widely adopted.

##  f) Centralization risks 
A single point of failure is not acceptable for this project
Centrality risk is high in the project, the role of 'onlyLlama' detailed below has very critical and important powers
 
Project and funds may be compromised by a malicious or stolen private key `onlyLlama` msg.sender

```solidity
32 results - 3 files

src/LlamaCore.sol:
   68:   modifier onlyLlama() {
   69:     if (msg.sender != llamaExecutor) revert OnlyLlama();

  429:   function createStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs) external onlyLlama {
  436:   function createAccounts(ILlamaAccount llamaAccountLogic, bytes[] calldata accountConfigs) external onlyLlama {
  444:   function setGuard(address target, bytes4 selector, ILlamaActionGuard guard) external onlyLlama {
  454:   function authorizeScript(address script, bool authorized) external onlyLlama {

src/LlamaPolicy.sol:
   68:   modifier onlyLlama() {
   69:     if (msg.sender != llamaExecutor) revert OnlyLlama();

  190:   function initializeRole(RoleDescription description) external onlyLlama {
  199:   function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {
  207:   function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {
  222:   function revokePolicy(address policyholder) external onlyLlama {
  236:   function updateRoleDescription(uint8 role, RoleDescription description) external onlyLlama {

src/accounts/LlamaAccount.sol:
  103:   modifier onlyLlama() {
  104:     if (msg.sender != llamaExecutor) revert OnlyLlama();

  147:   function transferNativeToken(NativeTokenData calldata nativeTokenData) public onlyLlama {
  154:   function batchTransferNativeToken(NativeTokenData[] calldata nativeTokenData) external onlyLlama {
  165:   function transferERC20(ERC20Data calldata erc20Data) public onlyLlama {
  172:   function batchTransferERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
  181:   function approveERC20(ERC20Data calldata erc20Data) public onlyLlama {
  187:   function batchApproveERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
  198:   function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {
  205:   function batchTransferERC721(ERC721Data[] calldata erc721Data) external onlyLlama {
  214:   function approveERC721(ERC721Data calldata erc721Data) public onlyLlama {
  220:   function batchApproveERC721(ERC721Data[] calldata erc721Data) external onlyLlama {
  229:   function approveOperatorERC721(ERC721OperatorData calldata erc721OperatorData) public onlyLlama {
  235:   function batchApproveOperatorERC721(ERC721OperatorData[] calldata erc721OperatorData) external onlyLlama {
  246:   function transferERC1155(ERC1155Data calldata erc1155Data) external onlyLlama {
  255:   function batchTransferSingleERC1155(ERC1155BatchData calldata erc1155BatchData) public onlyLlama {
  268:   function batchTransferMultipleERC1155(ERC1155BatchData[] calldata erc1155BatchData) external onlyLlama {
  277:   function approveOperatorERC1155(ERC1155OperatorData calldata erc1155OperatorData) public onlyLlama {
  283:   function batchApproveOperatorERC1155(ERC1155OperatorData[] calldata erc1155OperatorData) external onlyLlama {
```

##  g) Systemic risks 
According to the scope information of the project, it is stated that it can be used in rollup chains and popular EVM chains.

We can talk about a systemic risk here because there are some Layer2 special cases.
Some conventions in the project are set to Pragma ^0.8.19, allowing the conventions to be compiled with any 0.8.x compiler. The problem with this is that Arbitrum is [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected. The default behavior of the compiler will be to use the latest version which means it will compile with version 0.8.20 which will produce broken code by default.

## h) Competition analysis
There is no such onchain governance organization project represented by a non-transferable token.


## i) Security Approach of the Project

Successful current security understanding of the project;
1 - First they did the main audit from a reputable auditing organization like SpearBit and resolved all the security concerns in the report
2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

## j) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://gist.github.com/itsmetechjay/610f1b31f419156f06898ee10c89402d

2 Medium and 10 Low risks should be examined in detail and recommendations such as `_safeMint() should be used rather than _mint() wherever possible` should be considered.


**Other Audit Report (SpearBit):**
https://github.com/code-423n4/2023-06-llama/blob/main/audits/Llama-Spearbit-Audit.pdf

While investigating the potential risks in the project, the past audit reports can give us serious insights, so they should be taken into account and analyzed in detail. When we look at the detail of Spearbit's report, we see that a significant number of security vulnerabilities were detected in the `Access Control` category, so in this audit, `Access Controls` are also included. should be prioritized;

## Spearbit Audit Report
| No |Risk Level |Description |Category|
|:--|:----------------|:------|:------|
| 5.1.1|Critical Risk | The castApprovalBySig and castDisapprovalBySig functions can revert | Access Control |
| 5.1.2|Critical Risk| The castApproval/castDisapproval functions don't check if the role parameter is the approvalRole. |Access Control |
| 5.2.1|High Risk| Reducing the quantity of a policyholder results in an increase instead of a decrease in totalQuantity | Logic |
| 5.3.1|Medium Risk| LlamaPolicy.revokePolicy cannot be called repeatedly and may result in burned tokens retaining active roles. |Improper Input Validation |
| 5.3.2|Medium Risk|Role, permission, strategy, and guard management or config errors may prevent creating/approving/queuing/executing actions. | Improper Input Validation|
| 5.3.3|Medium Risk|LlamaPolicy.hasRole doesn't check if a policyholder holds a token |Access Control |
| 5.3.4|Medium Risk| Incorrect isActionApproved behavior if new policyholders get added after the createAction in the same block.timestamp |Logic |
| 5.3.5|Medium Risk|LlamaCore delegate calls can bring Llama into an unusable state | Access Control |
| 5.3.6|Medium Risk| The execution opcode of an action can be changed from call to delegate_call after approval | Access Control|
| 5.4.1| Low Risk | LlamaFactory is governed by Llama itself. |Access Control |
| 5.4.2| Low Risk | The permissionId doesn't include call or delegate-call for LlamaAccount.execute.| Access Control|
| 5.4.3| Low Risk | Nonconforming EIP-712 typehash.| Access Control|
| 5.4.4| Low Risk | Various events do not add the role as a parameter.| Access Control|
| 5.4.5| Low Risk | LlamaCore doesn't check if minExecutionTime returned by the strategy is in the past.| Logic|
| 5.4.6| Low Risk | Address parsing from tokenId to address string does not account for leading 0s.| Data Handling|
| 5.4.7| Low Risk | The ALL_HOLDERS_ROLE can be set as a force role by mistake.| Access Control|
| 5.4.8| Low Risk | LlamaPolicy.setRolePermission allows setting permissions for non-existing roles.| Access Control|
| 5.4.9| Low Risk |The RoleAssigned event in LlamaPolicy emits the currentRoleSupply instead of the quantity. |Data Handling|
| 5.4.10| Low Risk | ETH can remain in the contract if msg.value is greater than expected.|Ether Handling |
| 5.4.11| Low Risk |Cannot re-authorize an unauthorized strategy config. |Access Control |
| 5.4.12| Low Risk | Signed messages may not be canceled.| Access Control|
| 5.4.13|Low Risk  |LlamaCore name open to squatting or impersonation. |Access Control |
| 5.4.14| Low Risk |Expired policyholders are active until they are explicitly revoked. | Access Control|
| 5.5.1| Gas Opt. | A few Gas optimizations.| Gas optimization|
| 5.5.2| Gas Opt. | Unused code.| Gas optimization|
| 5.5.3| Gas Opt.| Duplicate storage reads and external calls.|Gas optimization |
| 5.5.4| Gas Opt. | Consider clones-with-immutable-args. |Gas optimization |
| 5.5.5| Gas Opt. | The domainSeparator may be cached.|Gas optimization |
| 5.6.1|  Informational| Prefer on-chain SVGs or IPFS links over server links for contractURI.| Informational|
| 5.6.2|  Informational| Consider making the delegate-call scripts functions only callable by delegate-call.| Informational|
| 5.6.3|  Informational| Missing tests for SingleUseScript.sol.| Test|
| 5.6.4|  Informational| Role not available to Guards.| Informational|
| 5.6.5|  Informational| Global guards are not supported.|Informational |
| 5.6.6| Informational | Consider using _disableInitializers in the constructor.| Informational|
| 5.6.7|  Informational| Consider renaming initialAccounts to initialAccountNames for clarity.| Informational|
| 5.6.8|  Informational| Revoking and setting a role edge cases.| Informational|
| 5.6.9| Informational | Use built-in string.concat.| Informational|
| 5.6.10| Informational | Inconsistencies.| Informational|
| 5.6.11|  Informational| Policyholders with large quantities may not both create and exercise their large quantity for the same action.| Informational|
| 5.6.12|  Informational| The roleBalanceCheckpoints can run out of gas.| Informational|
| 5.6.13|  Informational| GovernanceScript.revokeExpiredRoles should be avoided in favor of calling LlamaPolicy.revokeExpiredRole from EOA.| Informational|
| 5.6.14| Informational| The InvalidActionState can be improved.| Informational|
| 5.6.15| Informational |_uncheckedIncrement function written in multiple contracts. | Informational|


## k) Gas Optimization
When the project is evaluated in terms of gas optimization; It is highly gas optimized, if `uint256` is used instead of `bool` in mapping structures, it will provide even higher gas gain, other gas optimizations are minimal
With the `LlamaPolicyMetadata.sol` contract, the use of onchain SVG can consume high gas, since the metadata of NFT is not important for this project, it can increase gas optimization with an offline data usage.

The highest gas expenditure in the project occurs when creating LlamaCore instances, if the architecture here is designed like the singleton architecture in UniswapV4, gas savings will be close to 90%.


##  l) Project team
https://llama.substack.com/about
https://twitter.com/austingreen
https://twitter.com/0xrajath

## m) Other recommendations

Although the project is not a direct NFT project, NFT projects are colorful projects, different side design architectures add value and interest to the project, such as a bot that tweets when NFT is minted;
https://github.com/Cyfrin/foundry-full-course-f23/issues/13
https://github.com/0xAnon0602/Foundery-Course-Twitter-Bot



## n) New insights and learning from this audit 
The project allows us to create a Governance that can be performed with a uniquely non-transferable token

In addition, Script usage is a structure that we can learn differently from this control.

Roles are of type uint8, meaning roles are specified as unsigned integers and the maximum number of roles a Llama instance can have is 255, this part can be increased, the number 255 may be insufficient in the future


## o) Resources - Frameworks
The resources and frameworks I use while doing code audit and analysis;
https://www.halborn.com/blog/post/delegatecall-vulnerabilities-in-solidity
https://book.getfoundry.sh/forge/fuzz-testing
Ide: VsCode
Ide Extension: Surya , Solidity Metrict
Framework: Foundry


## p) Time spent on analysis
A total of 16 hours was spent, 1/3 of the time is reading documentation, 1/3 is code review and the remaining 1/3 is writing reports.

### Time spent:
16 hours