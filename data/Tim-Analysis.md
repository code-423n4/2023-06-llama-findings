Overall, I mostly found the code to be well written and the structure and tests were thorough and sufficient. Great job Llama team. 

I liked the idea of a separate executor contract however, there is a risk of someone somehow managing to deploy a strategy with favorable conditions to control the executor and therefore, the whole contract, providing a single point of failure.

I would also like to suggest a couple things for consideration:

1. Action state could be stored in the object itself. There is a clear pathway and checkpoints for Actions to take and the state of an Action could be updated within the object and not have the code bounce around retrieving the state from getActionState(). 

2. Proper natspec comments/documentation.  

### Time spent:
10 hours