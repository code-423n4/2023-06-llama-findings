## Approach
The resources provided by the Llama team such as the video explainer and docs were quite helpful in understanding how the codebase functions on the surface level. To delve deeper into the technicalities of policies, permission IDs and roles, I created a mindmap of my own to trace the step by step action and permissioning flow of the system since it is of utmost important for an onchain governance system.

Here is the miro board link: [Llama mindmap](https://miro.com/app/board/uXjVMEPd6dI=/?share_link_id=153942146123)

## Learnings from the codebase
A]
The two most notable features of the codebase were:
1. Core contracts were heavy (more nSLOC)
2. While the dependencies were made short and crisp

This design is appropriate as it allows the core files to build on the smaller chunks of code, drastically decreasing the time required to understand the codebase. Personally, I started evaluating in the following order: less nSLOC to more nSLOC. This made it convenient to review and understand most of the codebase in time.

B]
Another learning would be lack of knowledge. There are short sections of Yul code in the repo that are not quite easy to navigate through if one does not know how to read assembly code.

C] 
Although this is my second audit, this is the first time I've seen the core contract being separated from the execution contract. I think this design is safer as it adds an intermediate contract between the target contract/script and the core while still providing the extended functionality for a more flexible governance model.

## Context on my findings
Most of my findings are minute QA issues. I came across low severity issues but most were covered by the bots.

### Time spent:
15 hours