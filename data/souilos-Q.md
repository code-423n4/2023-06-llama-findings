# VULN 1 

## [LOW] Empty receive()/payable fallback() function does not authenticate requests
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 143 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  receive() external payable {}

------------------------------------------------------------------------ 

### Mitigation 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.










# VULN 2 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 453 at 2023-06-llama/src/LlamaPolicy.sol:

    if (balanceOf(policyholder) == 0) _mint(policyholder);


Found in line 504 at 2023-06-llama/src/LlamaPolicy.sol:

  function _mint(address policyholder) internal {


Found in line 506 at 2023-06-llama/src/LlamaPolicy.sol:

    _mint(policyholder, tokenId);


Found in line 127 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  function _mint(address to, uint256 id) internal virtual {


Found in line 164 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    _mint(to, id);


Found in line 175 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    _mint(to, id);

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 3 

## [LOW] safeApprove of openZeppelin is deprecated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 182 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    erc20Data.token.safeApprove(erc20Data.recipient, erc20Data.amount);

------------------------------------------------------------------------ 

### Mitigation 

Deprecated, its usage is discouraged. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20. Whenever possible, use safeIncreaseAllowance and safeDecreaseAllowance instead. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeIncreaseAllowance-contract-IERC20-address-uint256- & https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeDecreaseAllowance-contract-IERC20-address-uint256-










# VULN 4 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 71 at 2023-06-llama/src/LlamaFactory.sol:

  LlamaCore public immutable LLAMA_CORE_LOGIC;


Found in line 74 at 2023-06-llama/src/LlamaFactory.sol:

  LlamaPolicy public immutable LLAMA_POLICY_LOGIC;


Found in line 77 at 2023-06-llama/src/LlamaFactory.sol:

  LlamaPolicyMetadataParamRegistry public immutable LLAMA_POLICY_METADATA_PARAM_REGISTRY;


Found in line 80 at 2023-06-llama/src/LlamaFactory.sol:

  LlamaExecutor public immutable ROOT_LLAMA_EXECUTOR;


Found in line 83 at 2023-06-llama/src/LlamaFactory.sol:

  LlamaCore public immutable ROOT_LLAMA_CORE;


Found in line 12 at 2023-06-llama/src/LlamaExecutor.sol:

  address public immutable LLAMA_CORE;


Found in line 41 at 2023-06-llama/src/LlamaPolicyMetadataParamRegistry.sol:

  LlamaExecutor public immutable ROOT_LLAMA_EXECUTOR;


Found in line 44 at 2023-06-llama/src/LlamaPolicyMetadataParamRegistry.sol:

  address public immutable LLAMA_FACTORY;


Found in line 22 at 2023-06-llama/src/LlamaSingleUseScript.sol:

  LlamaExecutor internal immutable EXECUTOR;


Found in line 18 at 2023-06-llama/src/llama-scripts/LlamaBaseScript.sol:

  address internal immutable SELF;


Found in line 22 at 2023-06-llama/src/llama-scripts/LlamaSingleUseScript.sol:

  LlamaExecutor internal immutable EXECUTOR;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
