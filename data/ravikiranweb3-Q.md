1) Contract Name: Checkpoints
   _lowerBinaryLookup function is not used in the project.  Also, the logic is same as 
   _upperBinaryLookup() and hence redundant.


2) Contract Name: Checkpoints
  In the getAtProbablyRecentTimestamp function, the span to look for checkpoint seems incorrect. If the array is zero indexed, then high should be limited to array.length - 1.

  But, length is set as high per below code for reference.a But the logic was adjust to handle this.

  ```solidity
      uint256 len = self._checkpoints.length;

      uint256 low = 0;
      uint256 high = len; @audit, high should be len -1.

      
  ```

3) Modifiers should ideally be user to modify the behaviour of the function they are attached to.
   They should not change the state variables of the contract. The state variables should be updated in 
   functions.

   In the below contract the modifier is updating the state variable as below.
   
   Contract: LlamaSingleUseScript

   modifier code:
   ```
 modifier unauthorizeAfterRun() {
    _;
    LlamaCore core = LlamaCore(EXECUTOR.LLAMA_CORE());
    core.authorizeScript(SELF, false);
  }

   ```

  

  