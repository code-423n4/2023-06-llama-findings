[ L ] Missing zero address check on authoriseScript function

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L454

Change from this:

```
 function authorizeScript(address script, bool authorized) external onlyLlama {
    if (script == address(this) || script == address(policy)) revert RestrictedAddress();
    authorizedScripts[script] = authorized;
    emit ScriptAuthorized(script, authorized);
  }
```

To this:

```
  function authoriseScript(address script, bool authorized) external onlyLlama {
    if (script == address(0)) || if (script == address(this) || script == address(policy)) revert RestrictedAddress();
    authorizedScripts[script] = authorized;
    emit ScriptAuthorized(script, authorized);
  }
```