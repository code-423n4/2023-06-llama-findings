[G-] Using calldata instead of memory for read-only arguments in external functions saves gas

[G-] The result of function calls should be cached rather than re-calling the function 3
L

[G-] State variables only set in the constructor should be declared immutable

[G-] State variables can be packed into fewer storage slots

[G-] Using storage instead of memory for structs/arrays saves gas

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