## Approach

I usually start by reading white papers and related documents to learn the architecture of each protocol.

Then determine the type of project to audit to determine the vulnerabilities that need attention.

Some projects involving currency often have some liquidity and flash loan problems. However this project has nothing to do with these, so I can only audit some common logic issues and some permissions issues.

## Learning

This project is very novel and has a unique architecture, such as some separated designs inside, but there are still some logical problems, such as the legitimacy of the `delegatecall` in the `executor` is not checked

### Time spent:
4 hours