## Any comments for the judge to contextualize your findings

I am still a new auditor, and there is so much to learn. This is the second audit for me on this platform.  In the first audit, I spent most of my time trying to find Gas and QA issues to get a hang of things. In this audit, I tried to take it up a bit, and actually discovered possible high and medium issues. I come from a Web 2.0 bug bounty background, and I might have reported many findings as QA which could be a low/medium, and vice-versa, and I am still learning what can be classified as mediums/highs.

## Approach taken in evaluating the codebase

It took me around 2 days to go over the documentation and the explainer video as a first pass.
After that I started with the first pass of the code, and started to use inline bookmarks to make notes and summarize issues.
I found a possible High on the 5th day, where I spent around 6 hours to create as best proof of concept as I could, and to triple check if it was really an issue.
Thats when I realized that I might be running out of time, as only 2 more days remains. Thats why I started to go over my inline bookmarks, create this analysis, and start the submission process so that I get enough time to document everything I have so far, and then continue more after that.


## Analysis of the codebase (What's unique? What's using existing patterns?)

Given how simple the architecture seemed to be, the codebase seemed just a bit more complicated. 
I really liked the examples of strategies which really helped me in understanding what strategies really were, and what kind of strategies could be written.
The codebase was really well commented and documented. That did make the audit a bit more friendlier.
Throughout the codebase, most of the Solidity best practices were already in place.

## Architecture feedback

The architecture looks very simple, and thats how it should be. The diagrams made understanding the architecture very easy.
I referenced the architecture multiple times as I took various passes through the code to validatate my understanding, and to cross check things to find potential issues.
To take this up a notch, a sequence diagram of some end to end flows would have nailed it further.

## Centralization risks

Probably I am not experienced enough to answer this well yet, but there did not seem to be any real centralization risks, as everything was consensus based using the different strategies.

## Systemic risks

As with the previous question, probably I am not experienced enough to answer this well yet, but there did not seem to be any special systemic risk. The classic risks of crypto collapse, high gas fees and vulnerabilities are always applicable.

## Other recommendations

- Please consider storing images off chain to reduce gas costs. The images just require certain attributes, and the SVG can be easily constructed on the UI based on it.
- Running tests took a couple of minutes for me everytime. I am not sure if it was because of Alchemy setup or just my lack of experience. But it will be nice to have this project to not rely on Alchemy, and rather just run everything locally. Its very easy to lose the train of thoughts while waiting for tests to finish.

## How much time did you spend?

Since the start date, I have spent around 3-4 hours every day on this codebase after my work.

### Time spent:
20 hours