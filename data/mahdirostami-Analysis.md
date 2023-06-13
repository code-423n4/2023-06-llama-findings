## 1. Analysis of the codebase (What's unique? What's using existing patterns?)
1. I like the way that calls work, it's unique that there is one single exit point.(“LlamaExecutor”)
2. Variety of roles and permissions and strategies comes to me unique which is so scalable and straightforward.
3. Documentation is perfect in both code and GitHub.
* * *
## 2. Approach taken in evaluating the codebase?
1. I fully understand how Llama works, then I jump to code and check my understanding.
2. I look at Mirror function and check state changes in them. I compare opposite variables and functions. For an 
example Granting Policies and Revoking Policies.
3. I look at 
— What should happen
— What shouldn't happen
4. Do some test.
5. check previous audit
* * *
## 2. Architecture feedback
1. this architecture in more scalable compare to common governance system.
2. Separating roles and permissions also make it more scalene and easy to Handel.
* * *
## 3. Centralization risks
1. in my idea, there is Centralization risks because of Force Approval/Disapproval Roles.
2. it's better to not have too few roles as it's dangerously powerful accounts
* * *
## 4. Systemic risks and Mechanism review
1. as it's new and because it's different from other form of governance system, I think when projects want to use this type of governance system they need best practice plan to how they create roles and how they give them to users in the way that make it more decentralize.
2. it's better to not have too many roles because it will be difficult to manage.
3. implementation in so straightforward and easy to understand, and we can have complexity in mechanism and design. So it's need to be more careful about how systemic will be work (mechanism). How we project give permission what strategy they use, the distribution of roles and their quantity and …
* * *
## 5. Other recommendations.

1. about Centralization risks that I mentioned, it will become better to decrease force power for example it's become more decentralize if you make it in the way that it requires more than 1 force role to immediately reach the respective quorum. 

2. It's better to execute actions respectively in order to prevent conflict, now there is no such thing, but it will become better to create a process which previous actions which are queued and not expired have to execute first.
* * *
## 6. Codebase quality analysis
1. code base is great, but there are some big functions which I recommend splitting them and have more with fewer lines of code.
2. Documenting is awesome.
* * *
## 7. How much time did you spend?
I spend 4 days each day, about 4 hours.

### Time spent:
16 hours