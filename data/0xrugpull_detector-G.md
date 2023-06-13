https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaPolicy.sol#L448
```````
    bool willHaveRole = quantity > 0;

    // Now we update the policyholder's role balance checkpoint.
-    roleBalanceCkpts[tokenId][role].push(willHaveRole ? quantity : 0, expiration);
+    roleBalanceCkpts[tokenId][role].push(quantity, expiration);
```````