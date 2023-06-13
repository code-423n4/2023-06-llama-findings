I spent the first 1h30 reading the docs and the publicly known issues, including the Spearbit audit report and the automated finding output.
I then opened the code and started the review by the executor and the core contracts to understand the core concept of the protocol because it represent the more "outside" layer of the governance system. After 10 minutes I noticed the severe bug that I reported concerning the executor.
I spent the next hour writing the report for this bug and trying to get my hands on foundry for the first time to write a Proof Of Concept.
During the next 5 hours I read the all others contracts and tried to find any other flaw, without any success.

Overall, I think the code is very complex for what it does, and could be a bit simplified. For instance, I don't really see the point of having Checkpoints to store roles history, and having 2 judgment phases (one for approval and one for disapproval) when both could be done at the same time.

Finally, I liked the modular approach of the protocol to create a new highly customizable governance system. It was the first time I audited such a comprehensive governance system. I also learnt about the benefits of non-transferable ERC721 tokens.

### Time spent:
7.5 hours