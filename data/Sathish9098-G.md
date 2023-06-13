# GAS OPTIMIZATIONS

##

## [G-] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a ``memory`` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional ``MLOAD`` rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/lib/Checkpoints.sol#L106

### _unsafeAccess function returns the ``storage Checkpoint `` . Then the implicit conversion storage to memory costs ``Gcoldsload `` extra ``2100 gas `` . ``ckpt, last `` only reading the values no write operations. So using ``storage`` is more efficient than ``memory``. Saves ``8400 GAS`` approximately 

Checkpoint struct in storage, using _unsafeAccess directly can be more gas-efficient than assigning it to a local variable. This is because accessing the struct in storage via _unsafeAccess involves reading from storage once, whereas assigning it to a local variable creates a copy of the struct in memory, which incurs additional gas costs. 

```diff
FILE: 2023-06-llama/src/lib/Checkpoints.sol

- 106: Checkpoint memory ckpt = _unsafeAccess(self._checkpoints, pos - 1);
+ 106: Checkpoint storage ckpt = _unsafeAccess(self._checkpoints, pos - 1);
107: return (true, ckpt.timestamp, ckpt.expiration, ckpt.quantity);

- 132: Checkpoint memory last = _unsafeAccess(self, pos - 1);
+ 132: Checkpoint storage last = _unsafeAccess(self, pos - 1);


```
[G-] The result of function calls should be cached rather than re-calling the function 3
L

[G-] State variables can be packed into fewer storage slots

[G-] Multiple accesses of a mapping/array should use a local variable cache

[G-] Using bools for storage incurs overhead

[G-] Save gas by checking against default WETH address

[G-] Avoid emitting constants










[G‑01]	Reduce gas usage by moving to Solidity 0.8.19 or later	2	-
[G‑02]	Avoid updating storage when the value hasn't changed	1	800
[G‑03]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	2	-
[G‑04]	Structs can be packed into fewer storage slots	2	-
[G‑05]	Avoid contract existence checks by using low level calls	2	200
[G‑06]	State variables should be cached in stack variables rather than re-reading them from storage	31	3007
[G‑07]	<x> += <y> costs more gas than <x> = <x> + <y> for state variables	1	113
[G‑08]	internal functions only called once can be inlined to save gas	8	160
[G‑09]	<array>.length should not be looked up in every loop of a for-loop	9	27
[G‑10]	require()/revert() strings longer than 32 bytes cost extra gas	1	-
[G‑11]	Optimize names to save gas	12	264
[G‑12]	Using bools for storage incurs overhead	14	239400
[G‑13]	Use a more recent version of solidity	2	-
[G‑14]	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	4	20
[G‑15]	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	18	-
[G‑16]	Using private rather than public for constants, saves gas	3	-
[G‑17]	Division by two should use bit shifting	1	20
[G‑18]	Empty blocks should be removed or emit something	1	-
[G‑19]	Use custom errors rather than revert()/require() strings to save gas	13	-
[G‑20]	Functions guaranteed to revert when called by normal users can be marked payable	47	987
[G‑21]	Constructors can be marked payable	10	210
[G‑22]	Not using the named return variables anywhere in the function is confusing	2	-