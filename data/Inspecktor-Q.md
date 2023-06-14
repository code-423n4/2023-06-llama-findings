Add inverse actions to the _authorizeAccountLogic() and _authorizeStrategyLogic() functions.

The _authorizeAccountLogic() and _authorizeStrategyLogic() functions can only set the parameter to true. To protect against future use, you must provide a choice of the parameter and false.

Links:
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol#L267-L276

Correction:
Add bool to function input parameter. And set bool in the function.