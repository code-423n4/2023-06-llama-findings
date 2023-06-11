** Limited time spent on this one as I was working on other contests that was active at same time.**

- What did you learn from reviewing this codebase?

It was first time for me to review a code base that is access control based, I learned how protocols can apply specific access to a specific function at certain time, the technic used to generate the `PermissionData` and compare it to the user permissionID give me an example for how to be very strict on access controls.


- What approach did you take in evaluating this codebase in order to help you grow your skills and code review resources?

1. I read the full codebase and take notes without going into the details or the logic flow
2. After I get a high level overview about each contract I start to identify the entrypoints
3. After doing so, I split the entrypoints into different flow, for example, the creation of an action is Flow1, Approving action if Flow2.
4. I then start to analyze each flow and build the flowchart for it.
5. For each flow, I ask my self some questions such as, a) what is the intended behavior of this function? b) what is the cases that could cause a revert? c) is there any funds involved in this flow? d) what this function do to the state?
6. While answering this question I take notes and start linking different notes with each other to identify any broken or possible attack vector.

How much time did you spend?

3 hours on day1 and 5 hours on day2 with total of 8 hours.

### Time spent:
8 hours