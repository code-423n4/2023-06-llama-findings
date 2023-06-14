# L01 : No check in The `initializeRole` function

The  `initializeRole` function allows the Llama executor to initialize a new role. But, there is no check to ensure that the role being initialized is not already initialized. 
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L190

Consider adding a check to prevent re-initialization of roles.


# L02 : No check in The `setRoleHolder` function
The `setRoleHolder` function assigns a role to a policyholder. It does not validate if the policyholder already holds the role or if the role has been initialized. 
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#LL199C1-L199C1
Consider adding appropriate checks before assigning the role.


 