### Auditing approach
I followed below steps while analyzing and auditing the code base.
1) Read the contest Readme.md and took the required notes in excel.
2) After that i have taken a high level view of smart contracts.
3) Study of documentation to understand each contract purpose, its functionality, how it is connected with other contracts, etc.
4) I have gone through with slither report and automated finding report. I study both of them to improve my knowledge and skills. In parallel i checked and found few interesting things like using safeMint() introduces reentrancy opportunity if there is violation of CEI pattern during my research.
5) Run the tests to checks all test passed.
6) Finally, I started with the auditing the code base. I started understanding line by line code and took the necessary notes to ask some questions to sponsors. Similarly i throughly checked the code and discussed few issues with sponsors. Some M/H issues are agreed by sponsors which i submitted as my final report.
7) At last, I rechecked the code base with the basic mindset that the bugs are still present there in code and finally concluded the audit from my end.

### Learnings
1) As usual, i learnt a lot from this audit about Llama governance system, their smart contracts, modular framework, non-transferrable NFTs use cases. In addition Action creation with key concepts like core, policy and executor are also understood. 
2) This audit contest has developed my understanding on governance system. In addition, did lots of research on various ethereum concepts like ecrecover, signatures, ECDSA, nonce usage in signature, block.chainId to prevent cross chain reply attacks, etc. 
3) This was one of the best audit. I highly interacted with Sponsors which developed my asking questions quality. While discussing few findings got confirmed and i confidently reported in submissions. This contest has boosted my confidence to do more better in future contest.

### Comments
1) This is most interacted contest with sponsors. Some findings got confirmed during discussion which i reported in my submission, i even shared the private chat discussion link by sponsor permission. 
2) I have learnt lots of things from this contest. I have given my best to apply my smart contract security knowledge and concepts while drafting reports. I submitted 1H,4M,5QA in this contest, This made me more confident on future audits.


### Time spent:
24 hours