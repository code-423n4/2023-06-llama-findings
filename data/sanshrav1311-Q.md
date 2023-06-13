# QA Report
## [01] nonce is not incremented with `createAction()`, `castApproval()` and `castDisapproval()` functions

according to the @dev natspec comment in
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L200-L203

	File: /src/LlamaCore.sol
	200:   /// @notice Mapping of policyholders to function selectors to current nonces for EIP-712 signatures.
	201:   /// @dev This is used to prevent replay attacks by incrementing the nonce for each operation (`createAction`,
	202:   /// `castApproval` and `castDisapproval`) signed by the policyholder.
	203:   mapping(address => mapping(bytes4 => uint256)) public nonces;


nonce should be incremented after `createAction()`, `castApproval()` and `castDisapproval()` functions but are not done in these. 
	
`createAction()` funtion - 
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L259C1-L268

`_createAction()` function - 
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L516
	

`castApproval()` function - 
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L365-L367C4

`_castApproval()` function -
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L565

`castDisapproval()` function -
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L398-L400

`_castDisapproval()` function - 
https://github.com/code-423n4/2023-06-llama/blob/aac904d31639c1b4b4e97f1c76b9c0f40b8e5cee/src/LlamaCore.sol#L576

### Recommended Mitigation Steps

use these lines in the `_createAction()`, `_castApproval()`, `_castDisapproval()` functions

	nonce = nonces[policyholder][selector];
	nonces[policyholder][selector] = LlamaUtils.uncheckedIncrement(nonce);

## [02] Names of internal and private variables should start with "_"

There are 8 instances where internal or private variables do not start with "_"

in Checkpoints.sol

	File: /src/lib/Checkpoints.sol
	215:     function average(uint256 a, uint256 b) private pure returns (uint256) {
	216:         return (a & b) + (a ^ b) / 2; // (a + b) / 2 can overflow.
	217:     }

in LlamaCore.sol

	File: /src/LlamaCore.sol
	142:   bytes32 internal constant EIP712_DOMAIN_TYPEHASH =
	143:     keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

	146:   bytes32 internal constant CREATE_ACTION_TYPEHASH = keccak256(
	147:     "CreateAction(address policyholder,uint8 role,address strategy,address target,uint256 value,bytes data,string description,uint256 nonce)"
	148:   );

	151:   bytes32 internal constant CAST_APPROVAL_TYPEHASH = keccak256(
	152:     "CastApproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"
	153:   );

	156:   bytes32 internal constant CAST_DISAPPROVAL_TYPEHASH = keccak256(
	157:     "CastDisapproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"
	158:   );

	161:   bytes32 internal constant ACTION_INFO_TYPEHASH = keccak256(
	162:     "ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"
	163:   );

	169:   mapping(uint256 => Action) internal actions;
 
 in LlamaPolicy.sol
 
	File: /src/LlamaPolicy.sol
	100:   mapping(uint256 tokenId => mapping(uint8 role => Checkpoints.History)) internal roleBalanceCkpts;


 