## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, errors, modifiers, constructor and functions.

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

```solidity
/// state variables coming after functions. place them together
36:  mapping(uint256 => address) internal _ownerOf;
38:  mapping(address => uint256) internal _balanceOf;
54:  mapping(uint256 => address) public getApproved;
56:  mapping(address => mapping(address => bool)) public isApprovedForAll;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaSingleUseScript.sol

```solidity
/// place this before the modifier
22:  LlamaExecutor internal immutable EXECUTOR;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaBaseScript.sol

```solidity
/// place this before all the errors and modifiers
18:  address internal immutable SELF;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
/// place these state variables before errors the modifiers
115:  address public llamaExecutor;
118:  string public name;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#LL57C49-L57C49

```solidity
// modifier coming before state variables
19:  modifier onlyAuthorized(LlamaExecutor llamaExecutor) {

// events coming before state variables
31:  event ColorSet(LlamaExecutor indexed llamaExecutor, string color);
34:  event LogoSet(LlamaExecutor indexed llamaExecutor, string logo);
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaStrategy.sol

```solidity
/// external view functions should come after all the non-view externals
19:  function llamaCore() external view returns (LlamaCore);
22:  function policy() external view returns (LlamaPolicy);
51:  function getApprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128);
67:  function getDisapprovalQuantityAt(address policyholder, uint8 role, uint256 timestamp)
77:  function minExecutionTime(ActionInfo calldata actionInfo) external view returns (uint64);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol

```solidity
/// internal function coming after private ones.
223:    function sqrt(uint256 x) internal pure returns (uint256 z) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
/// public function coming after external ones.
272:  function isActionApproved(ActionInfo calldata actionInfo) public view virtual returns (bool) {
278:  function isActionDisapproved(ActionInfo calldata actionInfo) public view virtual returns (bool) {
294:  function approvalEndTime(ActionInfo calldata actionInfo) public view virtual returns (uint256) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

```solidity
// internal coming before public functions
62:  function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
// public function coming after external functions
485:  function getActionState(ActionInfo calldata actionInfo) public view returns (ActionState) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

```solidity
258:  function getRoleSupplyAsNumberOfHolders(uint8 role) public view returns (uint128) {
263:  function getRoleSupplyAsQuantitySum(uint8 role) public view returns (uint128) {
305:  function hasRole(address policyholder, uint8 role) public view returns (bool) {
324:  function isRoleExpired(address policyholder, uint8 role) public view returns (bool) {
337:  function totalSupply() public view returns (uint256) {
346:  function tokenURI(uint256 tokenId) public view override returns (string memory) {
352:  function contractURI() public view returns (string memory) {
359:  function transferFrom(address, /* from */ address, /* to */ uint256 /* policyId */ )
367:  function safeTransferFrom(address, /* from */ address, /* to */ uint256 /* id */ )
375:  function safeTransferFrom(address, /* from */ address, /* to */ uint256, /* policyId */ bytes calldata /* data */ )
383:  function approve(address, /* spender */ uint256 /* id */ ) public pure override nonTransferableToken {}
386:  function setApprovalForAll(address, /* operator */ bool /* approved */ ) public pure override nonTransferableToken {}
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
/// place receive function right after constructor
143:  receive() external payable {}

/// public functions coming after external ones
165:  function transferERC20(ERC20Data calldata erc20Data) public onlyLlama {
181:  function approveERC20(ERC20Data calldata erc20Data) public onlyLlama {
198:  function transferERC721(ERC721Data calldata erc721Data) public onlyLlama {
214:  function approveERC721(ERC721Data calldata erc721Data) public onlyLlama {
229:  function approveOperatorERC721(ERC721OperatorData calldata erc721OperatorData) public onlyLlama {
255:  function batchTransferSingleERC1155(ERC1155BatchData calldata erc1155BatchData) public onlyLlama {
277:  function approveOperatorERC1155(ERC1155OperatorData calldata erc1155OperatorData) public onlyLlama {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
/// public functions coming after external ones.
175:  function initializeRoles(RoleDescription[] calldata description) public onlyDelegateCall {
183:  function setRoleHolders(RoleHolderData[] calldata _setRoleHolders) public onlyDelegateCall {
196:  function setRolePermissions(RolePermissionData[] calldata _setRolePermissions) public onlyDelegateCall {
208:  function revokePolicies(address[] calldata _revokePolicies) public onlyDelegateCall {
215:  function updateRoleDescriptions(UpdateRoleDescription[] calldata roleDescriptions) public onlyDelegateCall {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
// public functions coming after external ones
282:  function isActionApproved(ActionInfo calldata actionInfo) public view returns (bool) {
288:  function isActionDisapproved(ActionInfo calldata actionInfo) public view returns (bool) {
305:  function approvalEndTime(ActionInfo calldata actionInfo) public view returns (uint256) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol

```solidity
// public function coming after external ones
104:  function contractURI(string memory name) public pure returns (string memory) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol

```solidity
// external functions coming before public functions
74  function getMetadata(LlamaExecutor llamaExecutor) external view returns (string memory _color, string memory _logo) {
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaStrategy.sol

```solidity
// @return missing
19:  function llamaCore() external view returns (LlamaCore);
22:  function policy() external view returns (LlamaPolicy);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/interfaces/ILlamaAccount.sol

```solidity
// @return missing
11:  function llamaExecutor() external view returns (address);
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol

```solidity
21:    struct History {
25:    struct Checkpoint {

37:    function getAtProbablyRecentTimestamp(History storage self, uint256 timestamp) internal view returns (uint128) {
66:    function push(History storage self, uint256 quantity, uint256 expiration) internal returns (uint128, uint128) {
76:    function push(History storage self, uint256 quantity) internal returns (uint128, uint128) {
83:    function latest(History storage self) internal view returns (uint128) {
92:    function latestCheckpoint(History storage self)
114:    function length(History storage self) internal view returns (uint256) {
122:    function _insert(
159:    function _upperBinaryLookup(
183:    function _lowerBinaryLookup(
200:    function _unsafeAccess(Checkpoint[] storage self, uint256 pos)
215:    function average(uint256 a, uint256 b) private pure returns (uint256) {
223:    function sqrt(uint256 x) internal pure returns (uint256 z) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol

```solidity
5:    library LlamaUtils {

/// @param and @return missing
10:  function toUint64(uint256 n) internal pure returns (uint64) {
16:  function toUint128(uint256 n) internal pure returns (uint128) {
22:  function uncheckedIncrement(uint256 i) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
142:  constructor() {
294:  function approvalEndTime(ActionInfo calldata actionInfo) public view virtual returns (uint256) {
304:  function _assertValidRole(uint8 role, uint8 numRoles) internal pure {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

```solidity
16:  event Transfer(address indexed from, address indexed to, uint256 indexed id);
18:  event Approval(address indexed owner, address indexed spender, uint256 indexed id);
20:  event ApprovalForAll(address indexed owner, address indexed operator, bool approved);
30:  function tokenURI(uint256 id) public view virtual returns (string memory);
40:  function ownerOf(uint256 id) public view virtual returns (address owner) {
44:  function balanceOf(address owner) public view virtual returns (uint256) {
62:  function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {
71:  function approve(address spender, uint256 id) public virtual {
81:  function setApprovalForAll(address operator, bool approved) public virtual {
87:  function transferFrom(address from, address to, uint256 id) public virtual {
109:  function safeTransferFrom(address from, address to, uint256 id) public virtual;
111:  function safeTransferFrom(address from, address to, uint256 id, bytes calldata data) public virtual;
117:  function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {
127:  function _mint(address to, uint256 id) internal virtual {
142:  function _burn(uint256 id) internal virtual {
163:  function _safeMint(address to, uint256 id) internal virtual {
174:  function _safeMint(address to, uint256 id, bytes memory data) internal virtual {
189:  function onERC721Received(address, address, uint256, bytes calldata) external virtual returns (bytes4) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaSingleUseScript.sol

```solidity
24:  constructor(LlamaExecutor executor) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaBaseScript.sol

```solidity
20:  constructor() {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
212:  constructor() {
565:  function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)
691:  function _validateActionInfoHash(bytes32 actualHash, ActionInfo calldata actionInfo) internal pure {

// @return missing
478:  function getLastActionTimestamp() external view returns (uint256 timestamp) {
708:  function _getDomainHash() internal view returns (bytes32) {

// @param and @return missing
516:  function _createAction(
576:  function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)
587:  function _preCastAssertions(
616:  function _newCastCount(uint128 currentCount, uint128 quantity) internal pure returns (uint128) {
625:  function _deployStrategies(ILlamaStrategy llamaStrategyLogic, bytes[] calldata strategyConfigs)
648:  function _deployAccounts(ILlamaAccount llamaAccountLogic, bytes[] calldata accountConfigs) internal {
665:  function _infoHash(ActionInfo calldata actionInfo) internal pure returns (bytes32) {
678:  function _infoHash(
698:  function _useNonce(address policyholder, bytes4 selector) internal returns (uint256 nonce) {
714:  function _getCreateActionTypedDataHash(
746:  function _getCastApprovalTypedDataHash(
768:  function _getCastDisapprovalTypedDataHash(
789:  function _getActionInfoHash(ActionInfo calldata actionInfo) internal pure returns (bytes32) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

```solidity
134:  constructor() {

// @params missing
222:  function revokePolicy(address policyholder) external onlyLlama {
431:  function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal {
490:  function _setRolePermission(uint8 role, bytes32 permissionId, bool hasPermission) internal {
497:  function _revokeExpiredRole(uint8 role, address policyholder) internal {
504:  function _mint(address policyholder) internal {
519:  function _burn(uint256 tokenId) internal override {

// @params and @return missing
252:  function getPastQuantity(address policyholder, uint8 role, uint256 timestamp) external view returns (uint128) {
258:  function getRoleSupplyAsNumberOfHolders(uint8 role) public view returns (uint128) {
263:  function getRoleSupplyAsQuantitySum(uint8 role) public view returns (uint128) {
268:  function roleBalanceCheckpoints(address policyholder, uint8 role) external view returns (Checkpoints.History memory) {
299:  function roleBalanceCheckpointsLength(address policyholder, uint8 role) external view returns (uint256) {
305:  function hasRole(address policyholder, uint8 role) public view returns (bool) {
311:  function hasRole(address policyholder, uint8 role, uint256 timestamp) external view returns (bool) {
318:  function hasPermissionId(address policyholder, uint8 role, bytes32 permissionId) external view returns (bool) {
324:  function isRoleExpired(address policyholder, uint8 role) public view returns (bool) {
330:  function roleExpiration(address policyholder, uint8 role) external view returns (uint64) {
533:  function _tokenId(address policyholder) internal pure returns (uint256) {

// @return missing
279:  function roleBalanceCheckpoints(address policyholder, uint8 role, uint256 start, uint256 end)
337:  function totalSupply() public view returns (uint256) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
// @returns missing
338:  function _readSlot0() internal view returns (bytes32 slot0) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
62:  function aggregate(address[] calldata targets, bytes[] calldata data)
85:  function initializeRolesAndSetRoleHolders(
93:  function initializeRolesAndSetRolePermissions(
101:  function initializeRolesAndSetRoleHoldersAndSetRolePermissions(
111:  function createNewStrategiesAndSetRoleHolders(
120:  function createNewStrategiesAndInitializeRolesAndSetRoleHolders(
131:  function createNewStrategiesAndSetRolePermissions(
140:  function createNewStrategiesAndNewRolesAndSetRoleHoldersAndSetRolePermissions(
153:  function revokePoliciesAndUpdateRoleDescriptions(
161:  function revokePoliciesAndUpdateRoleDescriptionsAndSetRoleHolders(
175:  function initializeRoles(RoleDescription[] calldata description) public onlyDelegateCall {
183:  function setRoleHolders(RoleHolderData[] calldata _setRoleHolders) public onlyDelegateCall {
196:  function setRolePermissions(RolePermissionData[] calldata _setRolePermissions) public onlyDelegateCall {
208:  function revokePolicies(address[] calldata _revokePolicies) public onlyDelegateCall {
215:  function updateRoleDescriptions(UpdateRoleDescription[] calldata roleDescriptions) public onlyDelegateCall {
222:  function _context() internal view returns (LlamaCore core, LlamaPolicy policy) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

```solidity
102:  constructor(
227:  function _deploy(
267:  function _authorizeStrategyLogic(ILlamaStrategy strategyLogic) internal {
273:  function _authorizeAccountLogic(ILlamaAccount accountLogic) internal {
279:  function _setPolicyMetadata(LlamaPolicyMetadata _llamaPolicyMetadata) internal {
285:  function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol

```solidity
// @return missing
104:  function contractURI(string memory name) public pure returns (string memory) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol

```solidity
// @param missing
57:  constructor(LlamaExecutor rootLlamaExecutor) {

// @return missing
74:  function getMetadata(LlamaExecutor llamaExecutor) external view returns (string memory _color, string memory _logo) {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol

```solidity
///  private and internal `functions` should preppend with `underline`
37:    function getAtProbablyRecentTimestamp(History storage self, uint256 timestamp) internal view returns (uint128) {
66:    function push(History storage self, uint256 quantity, uint256 expiration) internal returns (uint128, uint128) {
76:    function push(History storage self, uint256 quantity) internal returns (uint128, uint128) {
83:    function latest(History storage self) internal view returns (uint128) {
92:    function latestCheckpoint(History storage self)
114:    function length(History storage self) internal view returns (uint256) {
223:    function sqrt(uint256 x) internal pure returns (uint256 z) {
215:    function average(uint256 a, uint256 b) private pure returns (uint256) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/LlamaUtils.sol

```solidity
10:  function toUint64(uint256 n) internal pure returns (uint64) {
16:  function toUint128(uint256 n) internal pure returns (uint128) {
22:  function uncheckedIncrement(uint256 i) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
// private and internal `variables` should preppend with `underline`
169:  mapping(uint256 => Action) internal actions;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

```solidity
// private and internal `variables` should preppend with `underline`
100:  mapping(uint256 tokenId => mapping(uint8 role => Checkpoints.History)) internal roleBalanceCkpts;
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol

```solidity
/// loose the ^ for a safer stable version
3:  pragma solidity ^0.8.0;
```


https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

```solidity
/// loose the >= for a safer stable version
3:  pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaSingleUseScript.sol

```solidity
/// loose the ^ for a safer stable version
2:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaBaseScript.sol

```solidity
/// loose the ^ for a safer stable version
2:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
/// loose the ^ for a safer stable version
2:  pragma solidity ^0.8.19;
```

---

### Initialized variables [6]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/Checkpoints.sol

```solidity
/// initialized with the default value
43:        uint256 low = 0;
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

```solidity
/// initialized with the default value
185:    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

```solidity
/// initialized with the default value
636:    for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {
656:    for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

```solidity
/// initialized with the default value
156:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
174:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
189:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
207:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
222:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
237:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
270:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
285:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol

```solidity
/// initialized with the default value
71:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
178:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
186:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
199:    for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {
210:    for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {
217:    for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

```solidity
/// initialized with the default value
177:    for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
185:    for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {
```

