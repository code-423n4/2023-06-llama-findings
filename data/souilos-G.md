# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 307 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0;


Found in line 313 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0;


Found in line 320 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0 && canCreateAction[role][permissionId];


Found in line 326 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0 && block.timestamp > expiration;


Found in line 425 at 2023-06-llama/src/LlamaPolicy.sol:

    bool case1 = quantity > 0 && expiration > block.timestamp;


Found in line 444 at 2023-06-llama/src/LlamaPolicy.sol:

    bool hadRole = initialQuantity > 0;


Found in line 445 at 2023-06-llama/src/LlamaPolicy.sol:

    bool willHaveRole = quantity > 0;


Found in line 130 at 2023-06-llama/src/lib/Checkpoints.sol:

        if (pos > 0) {


Found in line 223 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;


Found in line 242 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;


Found in line 215 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;


Found in line 232 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 216 at 2023-06-llama/src/lib/Checkpoints.sol:

        return (a & b) + (a ^ b) / 2; // (a + b) / 2 can overflow.

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 99 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

      _balanceOf[to]++;


Found in line 134 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

      _balanceOf[to]++;

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 636 at 2023-06-llama/src/LlamaCore.sol:

    for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 656 at 2023-06-llama/src/LlamaCore.sol:

    for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 151 at 2023-06-llama/src/LlamaPolicy.sol:

    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 155 at 2023-06-llama/src/LlamaPolicy.sol:

    for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 161 at 2023-06-llama/src/LlamaPolicy.sol:

    for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 156 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 174 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 189 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 207 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 222 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 237 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 270 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 285 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 43 at 2023-06-llama/src/lib/Checkpoints.sol:

        uint256 low = 0;


Found in line 71 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 178 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 186 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 199 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 210 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 217 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 177 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 185 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 177 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 185 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 5 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 38 at 2023-06-llama/src/lib/Checkpoints.sol:

        require(timestamp < block.timestamp, "Checkpoints: timestamp is not in the past");


Found in line 135 at 2023-06-llama/src/lib/Checkpoints.sol:

            require(last.timestamp <= timestamp, "Checkpoint: invalid timestamp");


Found in line 41 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require((owner = _ownerOf[id]) != address(0), "NOT_MINTED");


Found in line 45 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(owner != address(0), "ZERO_ADDRESS");


Found in line 74 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(msg.sender == owner || isApprovedForAll[owner][msg.sender], "NOT_AUTHORIZED");


Found in line 88 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(from == _ownerOf[id], "WRONG_FROM");


Found in line 90 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(to != address(0), "INVALID_RECIPIENT");


Found in line 92 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(msg.sender == from || isApprovedForAll[from][msg.sender] || msg.sender == getApproved[id], "NOT_AUTHORIZED");


Found in line 128 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(to != address(0), "INVALID_RECIPIENT");


Found in line 130 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(_ownerOf[id] == address(0), "ALREADY_MINTED");


Found in line 145 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(owner != address(0), "NOT_MINTED");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 6 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 206 at 2023-06-llama/src/LlamaFactory.sol:

  function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)


Found in line 218 at 2023-06-llama/src/LlamaFactory.sol:

  function contractURI(string memory name) external view returns (string memory) {


Found in line 285 at 2023-06-llama/src/LlamaFactory.sol:

  function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {


Found in line 17 at 2023-06-llama/src/LlamaPolicyMetadata.sol:

  function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)


Found in line 104 at 2023-06-llama/src/LlamaPolicyMetadata.sol:

  function contractURI(string memory name) public pure returns (string memory) {


Found in line 82 at 2023-06-llama/src/LlamaPolicyMetadataParamRegistry.sol:

  function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {


Found in line 90 at 2023-06-llama/src/LlamaPolicyMetadataParamRegistry.sol:

  function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {


Found in line 565 at 2023-06-llama/src/LlamaCore.sol:

  function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)


Found in line 576 at 2023-06-llama/src/LlamaCore.sol:

  function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)


Found in line 130 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function initialize(bytes memory config) external initializer {


Found in line 62 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {


Found in line 174 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  function _safeMint(address to, uint256 id, bytes memory data) internal virtual {


Found in line 29 at 2023-06-llama/src/interfaces/ILlamaStrategy.sol:

  function initialize(bytes memory config) external;


Found in line 18 at 2023-06-llama/src/interfaces/ILlamaAccount.sol:

  function initialize(bytes memory config) external;


Found in line 156 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  function initialize(bytes memory config) external initializer {


Found in line 153 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  function initialize(bytes memory config) external virtual initializer {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 7 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 151 at 2023-06-llama/src/LlamaPolicy.sol:

    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 155 at 2023-06-llama/src/LlamaPolicy.sol:

    for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 161 at 2023-06-llama/src/LlamaPolicy.sol:

    for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 210 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 217 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 177 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 185 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 177 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {


Found in line 185 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 8 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 298 at 2023-06-llama/src/LlamaCore.sol:

    if (signer == address(0) || signer != policyholder) revert InvalidSignature();


Found in line 389 at 2023-06-llama/src/LlamaCore.sol:

    if (signer == address(0) || signer != policyholder) revert InvalidSignature();


Found in line 422 at 2023-06-llama/src/LlamaCore.sol:

    if (signer == address(0) || signer != policyholder) revert InvalidSignature();


Found in line 405 at 2023-06-llama/src/LlamaPolicy.sol:

    if (llamaExecutor == address(0)) return; // Skip check during initialization.


Found in line 148 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();


Found in line 166 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();


Found in line 199 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();


Found in line 247 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();


Found in line 256 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();


Found in line 130 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    require(_ownerOf[id] == address(0), "ALREADY_MINTED");

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 9 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 68 at 2023-06-llama/src/LlamaFactory.sol:

  uint8 public constant BOOTSTRAP_ROLE = 1;


Found in line 28 at 2023-06-llama/src/LlamaCore.sol:

  error CannotCastWithZeroQuantity(address policyholder, uint8 role);


Found in line 94 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 123 at 2023-06-llama/src/LlamaCore.sol:

  event ApprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);


Found in line 126 at 2023-06-llama/src/LlamaCore.sol:

  event DisapprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);


Found in line 147 at 2023-06-llama/src/LlamaCore.sol:

    "CreateAction(address policyholder,uint8 role,address strategy,address target,uint256 value,bytes data,string description,uint256 nonce)"


Found in line 152 at 2023-06-llama/src/LlamaCore.sol:

    "CastApproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"


Found in line 157 at 2023-06-llama/src/LlamaCore.sol:

    "CastDisapproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"


Found in line 162 at 2023-06-llama/src/LlamaCore.sol:

    "ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"


Found in line 260 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 286 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 292 at 2023-06-llama/src/LlamaCore.sol:

    uint8 v,


Found in line 365 at 2023-06-llama/src/LlamaCore.sol:

  function castApproval(uint8 role, ActionInfo calldata actionInfo, string calldata reason) external {


Found in line 380 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 383 at 2023-06-llama/src/LlamaCore.sol:

    uint8 v,


Found in line 398 at 2023-06-llama/src/LlamaCore.sol:

  function castDisapproval(uint8 role, ActionInfo calldata actionInfo, string calldata reason) external {


Found in line 413 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 416 at 2023-06-llama/src/LlamaCore.sol:

    uint8 v,


Found in line 518 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 565 at 2023-06-llama/src/LlamaCore.sol:

  function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)


Found in line 576 at 2023-06-llama/src/LlamaCore.sol:

  function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)


Found in line 590 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 681 at 2023-06-llama/src/LlamaCore.sol:

    uint8 creatorRole,


Found in line 716 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 748 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 770 at 2023-06-llama/src/LlamaCore.sol:

    uint8 role,


Found in line 65 at 2023-06-llama/src/LlamaPolicy.sol:

  error RoleNotInitialized(uint8 role);


Found in line 85 at 2023-06-llama/src/LlamaPolicy.sol:

  event RoleAssigned(address indexed policyholder, uint8 indexed role, uint64 expiration, uint128 quantity);


Found in line 88 at 2023-06-llama/src/LlamaPolicy.sol:

  event RoleInitialized(uint8 indexed role, RoleDescription description);


Found in line 91 at 2023-06-llama/src/LlamaPolicy.sol:

  event RolePermissionAssigned(uint8 indexed role, bytes32 indexed permissionId, bool hasPermission);


Found in line 100 at 2023-06-llama/src/LlamaPolicy.sol:

  mapping(uint256 tokenId => mapping(uint8 role => Checkpoints.History)) internal roleBalanceCkpts;


Found in line 105 at 2023-06-llama/src/LlamaPolicy.sol:

  uint8 public constant ALL_HOLDERS_ROLE = 0;


Found in line 111 at 2023-06-llama/src/LlamaPolicy.sol:

  uint8 public constant BOOTSTRAP_ROLE = 1;


Found in line 114 at 2023-06-llama/src/LlamaPolicy.sol:

  mapping(uint8 role => mapping(bytes32 permissionId => bool)) public canCreateAction;


Found in line 119 at 2023-06-llama/src/LlamaPolicy.sol:

  mapping(uint8 role => RoleSupply) public roleSupply;


Found in line 122 at 2023-06-llama/src/LlamaPolicy.sol:

  uint8 public numRoles;


Found in line 199 at 2023-06-llama/src/LlamaPolicy.sol:

  function setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) external onlyLlama {


Found in line 207 at 2023-06-llama/src/LlamaPolicy.sol:

  function setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) external onlyLlama {


Found in line 217 at 2023-06-llama/src/LlamaPolicy.sol:

  function revokeExpiredRole(uint8 role, address policyholder) external {


Found in line 228 at 2023-06-llama/src/LlamaPolicy.sol:

      if (hasRole(policyholder, uint8(i))) _setRoleHolder(uint8(i), policyholder, 0, 0);


Found in line 236 at 2023-06-llama/src/LlamaPolicy.sol:

  function updateRoleDescription(uint8 role, RoleDescription description) external onlyLlama {


Found in line 245 at 2023-06-llama/src/LlamaPolicy.sol:

  function getQuantity(address policyholder, uint8 role) external view returns (uint128) {


Found in line 252 at 2023-06-llama/src/LlamaPolicy.sol:

  function getPastQuantity(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {


Found in line 258 at 2023-06-llama/src/LlamaPolicy.sol:

  function getRoleSupplyAsNumberOfHolders(uint8 role) public view returns (uint128) {


Found in line 263 at 2023-06-llama/src/LlamaPolicy.sol:

  function getRoleSupplyAsQuantitySum(uint8 role) public view returns (uint128) {


Found in line 268 at 2023-06-llama/src/LlamaPolicy.sol:

  function roleBalanceCheckpoints(address policyholder, uint8 role) external view returns (Checkpoints.History memory) {


Found in line 279 at 2023-06-llama/src/LlamaPolicy.sol:

  function roleBalanceCheckpoints(address policyholder, uint8 role, uint256 start, uint256 end)


Found in line 299 at 2023-06-llama/src/LlamaPolicy.sol:

  function roleBalanceCheckpointsLength(address policyholder, uint8 role) external view returns (uint256) {


Found in line 305 at 2023-06-llama/src/LlamaPolicy.sol:

  function hasRole(address policyholder, uint8 role) public view returns (bool) {


Found in line 311 at 2023-06-llama/src/LlamaPolicy.sol:

  function hasRole(address policyholder, uint8 role, uint256 timestamp) external view returns (bool) {


Found in line 318 at 2023-06-llama/src/LlamaPolicy.sol:

  function hasPermissionId(address policyholder, uint8 role, bytes32 permissionId) external view returns (bool) {


Found in line 324 at 2023-06-llama/src/LlamaPolicy.sol:

  function isRoleExpired(address policyholder, uint8 role) public view returns (bool) {


Found in line 330 at 2023-06-llama/src/LlamaPolicy.sol:

  function roleExpiration(address policyholder, uint8 role) external view returns (uint64) {


Found in line 412 at 2023-06-llama/src/LlamaPolicy.sol:

  function _assertValidRoleHolderUpdate(uint8 role, uint128 quantity, uint64 expiration) internal view {


Found in line 431 at 2023-06-llama/src/LlamaPolicy.sol:

  function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal {


Found in line 490 at 2023-06-llama/src/LlamaPolicy.sol:

  function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {


Found in line 497 at 2023-06-llama/src/LlamaPolicy.sol:

  function _revokeExpiredRole(uint8 role, address policyholder) internal {


Found in line 10 at 2023-06-llama/src/lib/Structs.sol:

  uint8 creatorRole; // The role that created the action.


Found in line 41 at 2023-06-llama/src/lib/Structs.sol:

  uint8 role; // ID of the role to set (uint8 ensures on-chain enumerability when burning policies).


Found in line 49 at 2023-06-llama/src/lib/Structs.sol:

  uint8 role; // ID of the role to set (uint8 ensures on-chain enumerability when burning policies).


Found in line 29 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    uint8 role; // Role to update.


Found in line 44 at 2023-06-llama/src/interfaces/ILlamaStrategy.sol:

  function isApprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role) external;


Found in line 51 at 2023-06-llama/src/interfaces/ILlamaStrategy.sol:

  function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128);


Found in line 60 at 2023-06-llama/src/interfaces/ILlamaStrategy.sol:

  function isDisapprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role) external;


Found in line 67 at 2023-06-llama/src/interfaces/ILlamaStrategy.sol:

  function getDisapprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp)


Found in line 30 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint16 minApprovalPct; // Minimum percentage of total approval quantity / total approval supply.


Found in line 31 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint16 minDisapprovalPct; // Minimum percentage of total disapproval quantity / total disapproval supply.


Found in line 33 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint8 approvalRole; // Anyone with this role can cast approval of an action.


Found in line 34 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint8 disapprovalRole; // Anyone with this role can cast disapproval of an action.


Found in line 35 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint8[] forceApprovalRoles; // Anyone with this role can single-handedly approve an action.


Found in line 36 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint8[] forceDisapprovalRoles; // Anyone with this role can single-handedly disapprove an action.


Found in line 56 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  error InvalidRole(uint8 role);


Found in line 63 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  error RoleHasZeroSupply(uint8 role);


Found in line 67 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  error RoleNotInitialized(uint8 role);


Found in line 75 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  event ForceApprovalRoleAdded(uint8 role);


Found in line 79 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  event ForceDisapprovalRoleAdded(uint8 role);


Found in line 116 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  uint16 public minApprovalPct;


Found in line 121 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  uint16 public minDisapprovalPct;


Found in line 124 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  uint8 public approvalRole;


Found in line 127 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  uint8 public disapprovalRole;


Found in line 130 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  mapping(uint8 => bool) public forceApprovalRole;


Found in line 133 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  mapping(uint8 => bool) public forceDisapprovalRole;


Found in line 169 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    uint8 numRoles = policy.numRoles();


Found in line 178 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

      uint8 role = strategyConfig.forceApprovalRoles[i];


Found in line 186 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

      uint8 role = strategyConfig.forceDisapprovalRoles[i];


Found in line 215 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  function isApprovalEnabled(ActionInfo calldata, address, uint8 role) external view {


Found in line 220 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {


Found in line 229 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  function isDisapprovalEnabled(ActionInfo calldata, address, uint8 role) external view {


Found in line 235 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  function getDisapprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp)


Found in line 324 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

  function _assertValidRole(uint8 role, uint8 numRoles) internal pure {


Found in line 66 at 2023-06-llama/src/strategies/LlamaAbsolutePeerReview.sol:

  function isApprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role) external view override {


Found in line 74 at 2023-06-llama/src/strategies/LlamaAbsolutePeerReview.sol:

  function isDisapprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role)


Found in line 35 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    uint8 approvalRole; // Anyone with this role can cast approval of an action.


Found in line 36 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    uint8 disapprovalRole; // Anyone with this role can cast disapproval of an action.


Found in line 37 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    uint8[] forceApprovalRoles; // Anyone with this role can single-handedly approve an action.


Found in line 38 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    uint8[] forceDisapprovalRoles; // Anyone with this role can single-handedly disapprove an action.


Found in line 64 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  error InvalidRole(uint8 role);


Found in line 71 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  error RoleHasZeroSupply(uint8 role);


Found in line 75 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  error RoleNotInitialized(uint8 role);


Found in line 83 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  event ForceApprovalRoleAdded(uint8 role);


Found in line 87 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  event ForceDisapprovalRoleAdded(uint8 role);


Found in line 127 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  uint8 public approvalRole;


Found in line 130 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  uint8 public disapprovalRole;


Found in line 133 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  mapping(uint8 => bool) public forceApprovalRole;


Found in line 136 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  mapping(uint8 => bool) public forceDisapprovalRole;


Found in line 169 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    uint8 numRoles = policy.numRoles();


Found in line 178 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

      uint8 role = strategyConfig.forceApprovalRoles[i];


Found in line 186 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

      uint8 role = strategyConfig.forceDisapprovalRoles[i];


Found in line 204 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  function isApprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role) external view virtual;


Found in line 207 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp)


Found in line 221 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  function isDisapprovalEnabled(ActionInfo calldata actionInfo, address policyholder, uint8 role) external view virtual;


Found in line 224 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  function getDisapprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp)


Found in line 304 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

  function _assertValidRole(uint8 role, uint8 numRoles) internal pure {


Found in line 44 at 2023-06-llama/src/strategies/LlamaAbsoluteQuorum.sol:

  function isApprovalEnabled(ActionInfo calldata, /* actionInfo */ address, /* policyholder */ uint8 role)


Found in line 55 at 2023-06-llama/src/strategies/LlamaAbsoluteQuorum.sol:

  function isDisapprovalEnabled(ActionInfo calldata, /* actionInfo */ address, /* policyholder */ uint8 role)

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 10 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 346 at 2023-06-llama/src/LlamaPolicy.sol:

  function tokenURI(uint256 tokenId) public view override returns (string memory) {


Found in line 383 at 2023-06-llama/src/LlamaPolicy.sol:

  function approve(address, /* spender */ uint256 /* id */ ) public pure override nonTransferableToken {}


Found in line 386 at 2023-06-llama/src/LlamaPolicy.sol:

  function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 11 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 394 at 2023-06-llama/src/LlamaPolicy.sol:

    numRoles += 1;


Found in line 471 at 2023-06-llama/src/LlamaPolicy.sol:

      currentRoleSupply.numberOfHolders += 1;


Found in line 472 at 2023-06-llama/src/LlamaPolicy.sol:

      currentRoleSupply.totalQuantity += quantityDiff;


Found in line 478 at 2023-06-llama/src/LlamaPolicy.sol:

      currentRoleSupply.totalQuantity += quantityDiff;


Found in line 511 at 2023-06-llama/src/LlamaPolicy.sol:

      allHoldersRoleSupply.numberOfHolders += 1;


Found in line 512 at 2023-06-llama/src/LlamaPolicy.sol:

      allHoldersRoleSupply.totalQuantity += 1;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 12 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 445 at 2023-06-llama/src/LlamaCore.sol:

    if (target == address(this) || target == address(policy)) revert RestrictedAddress();


Found in line 455 at 2023-06-llama/src/LlamaCore.sol:

    if (script == address(this) || script == address(policy)) revert RestrictedAddress();


Found in line 708 at 2023-06-llama/src/LlamaCore.sol:

      abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))


Found in line 200 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    erc721Data.token.transferFrom(address(this), erc721Data.recipient, erc721Data.tokenId);


Found in line 249 at 2023-06-llama/src/accounts/LlamaAccount.sol:

      address(this), erc1155Data.recipient, erc1155Data.tokenId, erc1155Data.amount, erc1155Data.data


Found in line 258 at 2023-06-llama/src/accounts/LlamaAccount.sol:

      address(this),


Found in line 223 at 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol:

    core = LlamaCore(LlamaExecutor(address(this)).LLAMA_CORE());


Found in line 12 at 2023-06-llama/src/llama-scripts/LlamaBaseScript.sol:

    if (address(this) == SELF) revert OnlyDelegateCall();


Found in line 21 at 2023-06-llama/src/llama-scripts/LlamaBaseScript.sol:

    SELF = address(this);

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 13 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 307 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0;


Found in line 313 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0;


Found in line 320 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0 && canCreateAction[role][permissionId];


Found in line 326 at 2023-06-llama/src/LlamaPolicy.sol:

    return quantity > 0 && block.timestamp > expiration;


Found in line 223 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;


Found in line 242 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;


Found in line 297 at 2023-06-llama/src/strategies/LlamaRelativeQuorum.sol:

    return block.timestamp > action.minExecutionTime + expirationPeriod;


Found in line 215 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;


Found in line 232 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;


Found in line 286 at 2023-06-llama/src/strategies/LlamaAbsoluteStrategyBase.sol:

    return block.timestamp > action.minExecutionTime + expirationPeriod;

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 14 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-06-llama/src/LlamaExecutor.sol (function execute(address target, uint256 value, bool isScript, bytes calldata data)):

  /// @param data Data to be called on the `target` when the action is executed.

  /// @return success A boolean that indicates if the call succeeded.

  /// @return result The data returned by the function being called.

  function execute(address target, uint256 value, bool isScript, bytes calldata data)

    external

    returns (bool success, bytes memory result)

  {

    if (msg.sender != LLAMA_CORE) revert OnlyLlamaCore();

    (success, result) = isScript ? target.delegatecall(data) : target.call{value: value}(data);

  }

}



Found in 2023-06-llama/src/LlamaCore.sol (function executeAction(ActionInfo calldata actionInfo) external payable {):

    // Check pre-execution action guard.

    ILlamaActionGuard guard = actionGuard[actionInfo.target][bytes4(actionInfo.data)];

    if (guard != ILlamaActionGuard(address(0))) guard.validatePreActionExecution(actionInfo);



    // Execute action.

    (bool success, bytes memory result) =

      executor.execute(actionInfo.target, actionInfo.value, action.isScript, actionInfo.data);



    if (!success) revert FailedActionExecution(result);



    // Check post-execution action guard.



Found in 2023-06-llama/src/llama-scripts/LlamaGovernanceScript.sol (function aggregate(address[] calldata targets, bytes[] calldata data)):

    returnData = new bytes[](length);

    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

      bool addressIsCore = targets[i] == address(core);

      bool addressIsPolicy = targets[i] == address(policy);

      if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);

      (bool success, bytes memory response) = targets[i].call(data[i]);

      if (!success) revert CallReverted(i, response);

      returnData[i] = response;

    }

  }




------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 15 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 364 at 2023-06-llama/src/LlamaPolicy.sol:

  {}


Found in line 372 at 2023-06-llama/src/LlamaPolicy.sol:

  {}


Found in line 380 at 2023-06-llama/src/LlamaPolicy.sol:

  {}


Found in line 383 at 2023-06-llama/src/LlamaPolicy.sol:

  function approve(address, /* spender */ uint256 /* id */ ) public pure override nonTransferableToken {}


Found in line 386 at 2023-06-llama/src/LlamaPolicy.sol:

  function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}


Found in line 143 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  receive() external payable {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 16 

## [GAS] bytes constants are more eficient than string constans
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 183 at 2023-06-llama/src/LlamaCore.sol:

  string public name;


Found in line 118 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  string public name;


Found in line 26 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  string public name;


Found in line 28 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  string public symbol;

------------------------------------------------------------------------ 

### Mitigation 

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.










# VULN 17 

## [GAS] Use a more recent version of solidity
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol at line 3: pragma solidity >=0.8.0;
------------------------------------------------------------------------ 

### Mitigation 

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath * Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining * Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads * Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings * Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value * In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller. * In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.










# VULN 18 

## [GAS] Use ERC721A instead ERC721
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 20 at 2023-06-llama/src/LlamaPolicy.sol:

contract LlamaPolicy is ERC721NonTransferableMinimalProxy {


Found in line 149 at 2023-06-llama/src/LlamaPolicy.sol:

    __initializeERC721MinimalProxy(_name, string.concat("LL-", LibString.replace(LibString.upper(_name), " ", "-")));


Found in line 520 at 2023-06-llama/src/LlamaPolicy.sol:

    ERC721NonTransferableMinimalProxy._burn(tokenId);


Found in line 20 at 2023-06-llama/src/accounts/LlamaAccount.sol:

contract LlamaAccount is ILlamaAccount, ERC721Holder, ERC1155Holder, Initializable {


Found in line 47 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  struct ERC721Data {


Found in line 48 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    IERC721 token; // The ERC721 token to transfer.


Found in line 54 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  struct ERC721OperatorData {


Found in line 55 at 2023-06-llama/src/accounts/LlamaAccount.sol:

    IERC721 token; // The ERC721 token to transfer.


Found in line 198 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {


Found in line 205 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function batchTransferERC721(ERC721Data[] calldata erc721Data) external onlyLlama {


Found in line 208 at 2023-06-llama/src/accounts/LlamaAccount.sol:

      transferERC721(erc721Data[i]);


Found in line 214 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function approveERC721(ERC721Data calldata erc721Data) public onlyLlama {


Found in line 220 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function batchApproveERC721(ERC721Data[] calldata erc721Data) external onlyLlama {


Found in line 223 at 2023-06-llama/src/accounts/LlamaAccount.sol:

      approveERC721(erc721Data[i]);


Found in line 229 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function approveOperatorERC721(ERC721OperatorData calldata erc721OperatorData) public onlyLlama {


Found in line 235 at 2023-06-llama/src/accounts/LlamaAccount.sol:

  function batchApproveOperatorERC721(ERC721OperatorData[] calldata erc721OperatorData) external onlyLlama {


Found in line 238 at 2023-06-llama/src/accounts/LlamaAccount.sol:

      approveOperatorERC721(erc721OperatorData[i]);


Found in line 11 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

abstract contract ERC721NonTransferableMinimalProxy is Initializable {


Found in line 33 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

                      ERC721 BALANCE/OWNER STORAGE


Found in line 51 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

                         ERC721 APPROVAL STORAGE


Found in line 62 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {


Found in line 68 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

                              ERC721 LOGIC


Found in line 119 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

      || interfaceId == 0x80ac58cd // ERC165 Interface ID for ERC721


Found in line 120 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

      || interfaceId == 0x5b5e139f; // ERC165 Interface ID for ERC721Metadata


Found in line 168 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

        || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, "")


Found in line 169 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

          == ERC721TokenReceiver.onERC721Received.selector,


Found in line 179 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

        || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, data)


Found in line 180 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

          == ERC721TokenReceiver.onERC721Received.selector,


Found in line 188 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

abstract contract ERC721TokenReceiver {


Found in line 189 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

  function onERC721Received(address, address, uint256, bytes calldata) external virtual returns (bytes4) {


Found in line 190 at 2023-06-llama/src/lib/ERC721NonTransferableMinimalProxy.sol:

    return ERC721TokenReceiver.onERC721Received.selector;

------------------------------------------------------------------------ 

### Mitigation 

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: https://nextrope.com/erc721-vs-erc721a-2/










# VULN 19 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 135 at 2023-06-llama/src/lib/Checkpoints.sol:

            require(last.timestamp <= timestamp, "Checkpoint: invalid timestamp");

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
