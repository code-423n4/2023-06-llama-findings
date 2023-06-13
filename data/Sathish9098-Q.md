












[M‑01]	Some tokens may revert when zero value transfers are made	1
[M‑02]	_safeMint() should be used rather than _mint() wherever possible	2
[L‑01]	approve()/safeApprove() may revert if the current approval is not zero	1
[L‑02]	Missing checks for ecrecover() signature malleability	3
[L‑03]	Missing checks for address(0x0) when assigning values to address state variables	6
[L‑04]	File allows a version of solidity that is susceptible to an assembly optimizer bug	1
[L‑05]	Solidity version 0.8.20 may not work on other chains due to PUSH0	5
[L‑06]	image_data should be used for raw svg	1
[L‑07]	Unsafe downcast	1
[L‑08]	tokenURI() does not follow EIP-721	2
[L‑09]	Empty receive()/payable fallback() function does not authorize requests	1
[L‑10]	safeApprove() is deprecated	1

[N‑01]	Contracts containing only utility functions should be made into libraries	1
[N‑02]	Consider implementing EIP-5267 to securely describe EIP-712 domains being used	1
[N‑03]	Events are missing sender information	9
[N‑04]	Variables need not be initialized to zero	25
[N‑05]	Consider using named mappings	26
[N‑06]	Non-external/public variable and function names should begin with an underscore	21
[N‑07]	Constants in comparisons should appear on the left side	30
[N‑08]	else-block not required	1
[N‑09]	Cast to bytes or bytes32 for clearer semantic meaning	2
[N‑10]	Use bytes.concat() on bytes instead of abi.encodePacked() for clearer semantic meaning	3
[N‑11]	Custom error has no error details	37
[N‑12]	Imports could be organized more systematically	2
[N‑13]	Long functions should be refactored into multiple, smaller, functions	2
[N‑14]	public functions not called by the contract should be declared external instead	5
[N‑15]	constants should be defined rather than using magic numbers	70
[N‑16]	Event is not properly indexed	2
[N‑17]	Duplicated require()/revert() checks should be refactored to a modifier or function	1
[N‑18]	Events that mark critical parameter changes should contain both the old and the new value	5
[N‑19]	Constant redefined elsewhere	1
[N‑20]	Lines are too long	22
[N‑21]	Inconsistent method of specifying a floating pragma	2
[N‑22]	Non-library/interface files should use fixed compiler versions, not floating ones	1
[N‑23]	Using >/>= without specifying an upper bound is unsafe	1
[N‑24]	Typos	1
[N‑25]	File is missing NatSpec	1
[N‑26]	NatSpec @param is missing	181
[N‑27]	NatSpec @return argument is missing	53
[N‑28]	Function ordering does not follow the Solidity style guide	17
[N‑29]	Contract does not follow the Solidity style guide's suggested layout ordering	16
[N‑30]	Control structures do not follow the Solidity Style Guide	4
[N‑31]	Strings should use double quotes rather than single quotes	19
[N‑32]	Expressions for constant values such as a call to keccak256(), should use immutable rather than constant	5
[N‑33]	Contracts should have full test coverage	1