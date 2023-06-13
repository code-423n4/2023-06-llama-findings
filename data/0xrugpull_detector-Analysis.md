## Analysis of codebase
What's unique about Llama is that it took very modular approach for every aspect of DAO.
Especially Core, Strategy, Executor, Account have very distinct role.

### Core and Strategy
I fear that majority role holder can gamify the voting mechanism on his/her favor by choosing relative/absolute strategy according to his interest.

### Executor
authorizeScript feature is there to check if target is script, but script by definition should not have access to storage but immutable and calldata.
If there might be a way to check if any contract has access to storage like checking SSTORE or SLOAD OPCODE, it would be clear if contract is strictly script or not.
Executor might create perpetual action by executing a function that recursively createAction, right now there's no way to stop it.

### Account
In the future, Account can have asset management or yield aggregation feature of some sort without idling asset in it.

### Time spent:
40 hours