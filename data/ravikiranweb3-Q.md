1) Contract Name: Checkpoints
   _lowerBinaryLookup function is not used in the project.  Also, the logic is same as 
   _upperBinaryLookup() and hence redundant.


2) Contract Name: Checkpoints
  In the getAtProbablyRecentTimestamp function, the span to look for checkpoint seems incorrect. If the array is zero indexed, then high should be limited to array.length - 1.

  But, length is set as high per below code for reference.

  ```solidity
      uint256 len = self._checkpoints.length;

      uint256 low = 0;
      uint256 high = len; @audit, high should be len -1.
  ```
  