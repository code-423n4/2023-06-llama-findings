# Report

## Gas Optimizations

|        | Issue                                                                                                                      |
| ------ | :------------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | MISSING DEADLINE CHECKS ALLOW PENDING TRANSACTIONS TO BE MALICIOUSLY EXECUTED                                              |
| GAS-2  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                                 |
| GAS-3  | Using CREATE2 is cheaper than Clones                                                                                       |
| GAS-4  | Division or multiply by two should use bit shifting                                                                        |
| GAS-5  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                          |
| GAS-6  | INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS                                     |
| GAS-7  | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                                                               |
| GAS-8  | OPTIMIZE NFT DELEGATE DEPLOYMENTS BY USING PROXY                                                                           |
| GAS-9  | Use a more recent version of solidity                                                                                      |
| GAS-10 | Reorder structure layout                                                                                                   |
| GAS-11 | Use shift Right/Left instead of division/multiplication if possible                                                        |
| GAS-12 | STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING |
| GAS-13 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                        |
| GAS-14 | USE BYTES32 INSTEAD OF STRING                                                                                              |

### [GAS-1] MISSING DEADLINE CHECKS ALLOW PENDING TRANSACTIONS TO BE MALICIOUSLY EXECUTED

#### Description:

AMMs should provide their users with an option to limit the execution of their pending actions, such as swaps or adding and removing liquidity. The most common solution is to include a deadline timestamp as a parameter (for example see Uniswap V2). If such an option is not present, users can unknowingly perform bad trades:

#### **Proof Of Concept**

```solidity
File: src/LlamaPolicyMetadata.sol

111:       '. Visit https://app.llama.xyz to learn more.", "image":"https://llama.xyz/policy-nft/llama-profile.png", "external_link": "https://app.llama.xyz", "banner":"https://llama.xyz/policy-nft/llama-banner.png" }';

111:       '. Visit https://app.llama.xyz to learn more.", "image":"https://llama.xyz/policy-nft/llama-profile.png", "external_link": "https://app.llama.xyz", "banner":"https://llama.xyz/policy-nft/llama-banner.png" }';

```

### [GAS-2] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

242:     return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));

529:     bytes32 permissionId = keccak256(abi.encode(permission));

708:       abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

```

### [GAS-3] Using CREATE2 is cheaper than Clones

#### Description:

Using clone contracts requires extra proxy call, increasing the cost of all pair functions. Using CREATE2, although will increase cost of pair creation, will make all pair interactions cheaper.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

4: import {Clones} from "@openzeppelin/proxy/Clones.sol";

638:       ILlamaStrategy strategy = ILlamaStrategy(Clones.cloneDeterministic(address(llamaStrategyLogic), salt));

658:       ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), salt));

```

```solidity
File: src/LlamaFactory.sol

4: import {Clones} from "@openzeppelin/proxy/Clones.sol";

250:       LlamaPolicy(Clones.cloneDeterministic(address(LLAMA_POLICY_LOGIC), keccak256(abi.encodePacked(name))));

253:     llamaCore = LlamaCore(Clones.cloneDeterministic(address(LLAMA_CORE_LOGIC), keccak256(abi.encodePacked(name))));

```

### [GAS-4] Division or multiply by two should use bit shifting

#### Description:

`<x> / 2` is the same as `<x> >> 1`. While the compiler uses the `SHR` opcode to accomplish both, the version that uses division incurs an overhead of 20 gas due to `JUMP`s to and from a compiler utility function that introduces checks which can be avoided by using `unchecked {}` around the division by two.

#### **Proof Of Concept**

```solidity
File: src/lib/Checkpoints.sol

216:         return (a & b) + (a ^ b) / 2; // (a + b) / 2 can overflow.

```

### [GAS-5] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

81:   modifier onlyLlama() {

```

```solidity
File: src/LlamaFactory.sol

32:   modifier onlyRootLlama() {

```

```solidity
File: src/LlamaPolicy.sol

68:   modifier onlyLlama() {

75:   modifier nonTransferableToken() {

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

19:   modifier onlyAuthorized(LlamaExecutor llamaExecutor) {

```

```solidity
File: src/accounts/LlamaAccount.sol

103:   modifier onlyLlama() {

```

```solidity
File: src/llama-scripts/LlamaBaseScript.sol

11:   modifier onlyDelegateCall() {

```

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

14:   modifier unauthorizeAfterRun() {

```

### [GAS-6] INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS

#### Description:

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

#### **Proof Of Concept**

```solidity
File: src/lib/Checkpoints.sol

37:     function getAtProbablyRecentTimestamp(History storage self, uint256 timestamp) internal view returns (uint128) {

83:     function latest(History storage self) internal view returns (uint128) {

114:     function length(History storage self) internal view returns (uint256) {

223:     function sqrt(uint256 x) internal pure returns (uint256 z) {

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

62:   function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {

142:   function _burn(uint256 id) internal virtual {

163:   function _safeMint(address to, uint256 id) internal virtual {

174:   function _safeMint(address to, uint256 id, bytes memory data) internal virtual {

```

```solidity
File: src/lib/LlamaUtils.sol

10:   function toUint64(uint256 n) internal pure returns (uint64) {

16:   function toUint128(uint256 n) internal pure returns (uint128) {

22:   function uncheckedIncrement(uint256 i) internal pure returns (uint256) {

```

### [GAS-7] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### **Proof Of Concept**

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

14:   modifier unauthorizeAfterRun() {

```

### [GAS-8] OPTIMIZE NFT DELEGATE DEPLOYMENTS BY USING PROXY

#### Description:

The cost of NFT delegate deployments can be significantly reduced by deploying proxies instead of clones of the implementation.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

638:       ILlamaStrategy strategy = ILlamaStrategy(Clones.cloneDeterministic(address(llamaStrategyLogic), salt));

658:       ILlamaAccount account = ILlamaAccount(Clones.cloneDeterministic(address(llamaAccountLogic), salt));

```

```solidity
File: src/LlamaFactory.sol

250:       LlamaPolicy(Clones.cloneDeterministic(address(LLAMA_POLICY_LOGIC), keccak256(abi.encodePacked(name))));

253:     llamaCore = LlamaCore(Clones.cloneDeterministic(address(LLAMA_CORE_LOGIC), keccak256(abi.encodePacked(name))));

```

#### Recommended Mitigation Steps:

Consider using the Clones library from OpenZeppelin–it deploys and absolutely minimal non-upgradable proxy contract. Such proxies, however, cannot be verified on Etherscan. Some more info.

### [GAS-9] Use a more recent version of solidity

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaExecutor.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaFactory.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadata.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/accounts/LlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaAccount.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaActionGuard.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Checkpoints.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

3: pragma solidity >=0.8.0;

```

```solidity
File: src/lib/Enums.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/LlamaUtils.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/Structs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/lib/UDVTs.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/llama-scripts/LlamaBaseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/llama-scripts/LlamaSingleUseScript.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/strategies/LlamaAbsolutePeerReview.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteQuorum.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

2: pragma solidity 0.8.19;

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

2: pragma solidity 0.8.19;

```

### [GAS-10] Reorder structure layout

#### Description:

Structures could be optimized moving the position of certain values in order to save a lot slots.

#### **Proof Of Concept**

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

27:   struct Config {
        uint64 approvalPeriod; // The length of time of the approval period.
    uint64 queuingPeriod; // The length of time of the queuing period. The disapproval period is the queuing period when
      // enabled.
    uint64 expirationPeriod; // The length of time an action can be executed before it expires.
    uint128 minApprovals; // Minimum number of total approval quantity.
    uint128 minDisapprovals; // Minimum number of total disapproval quantity.
    bool isFixedLengthApprovalPeriod; // Determines if an action be queued before approvalEndTime.
    uint8 approvalRole; // Anyone with this role can cast approval of an action.
    uint8 disapprovalRole; // Anyone with this role can cast disapproval of an action.
    uint8[] forceApprovalRoles; // Anyone with this role can single-handedly approve an action.
    uint8[] forceDisapprovalRoles; // Anyone with this role can single-handedly disapprove an action.
  }

```

### [GAS-11] Use shift Right/Left instead of division/multiplication if possible

#### **Proof Of Concept**

```solidity
File: src/lib/Checkpoints.sol

216:         return (a & b) + (a ^ b) / 2; // (a + b) / 2 can overflow.

```

### [GAS-12] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

225:     string memory _name,

265:     string memory description

291:     string memory description,

333:     (bool success, bytes memory result) =

471:   function getAction(uint256 actionId) external view returns (Action memory) {

523:     string memory description

528:     PermissionData memory permission = PermissionData(target, bytes4(data), strategy);

544:     ActionInfo memory actionInfo = ActionInfo(actionId, policyholder, role, strategy, target, value, data);

565:   function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)

576:   function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)

721:     string memory description

```

```solidity
File: src/LlamaExecutor.sol

31:     returns (bool success, bytes memory result)

```

```solidity
File: src/LlamaFactory.sol

108:     string memory name,

109:     bytes[] memory initialStrategies,

110:     bytes[] memory initialAccounts,

111:     RoleDescription[] memory initialRoleDescriptions,

112:     RoleHolderData[] memory initialRoleHolders,

113:     RolePermissionData[] memory initialRolePermissions

155:     string memory name,

158:     bytes[] memory initialStrategies,

159:     bytes[] memory initialAccounts,

160:     RoleDescription[] memory initialRoleDescriptions,

161:     RoleHolderData[] memory initialRoleHolders,

162:     RolePermissionData[] memory initialRolePermissions,

163:     string memory color,

164:     string memory logo

206:   function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)

209:     returns (string memory)

211:     (string memory color, string memory logo) = LLAMA_POLICY_METADATA_PARAM_REGISTRY.getMetadata(llamaExecutor);

211:     (string memory color, string memory logo) = LLAMA_POLICY_METADATA_PARAM_REGISTRY.getMetadata(llamaExecutor);

218:   function contractURI(string memory name) external view returns (string memory) {

228:     string memory name,

231:     bytes[] memory initialStrategies,

232:     bytes[] memory initialAccounts,

233:     RoleDescription[] memory initialRoleDescriptions,

234:     RoleHolderData[] memory initialRoleHolders,

235:     RolePermissionData[] memory initialRolePermissions

285:   function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {

```

```solidity
File: src/LlamaPolicy.sol

268:   function roleBalanceCheckpoints(address policyholder, uint8 role) external view returns (Checkpoints.History memory) {

282:     returns (Checkpoints.History memory)

290:     Checkpoints.Checkpoint[] memory checkpoints = new Checkpoints.Checkpoint[](sliceLength);

346:   function tokenURI(uint256 tokenId) public view override returns (string memory) {

352:   function contractURI() public view returns (string memory) {

```

```solidity
File: src/LlamaPolicyMetadata.sol

17:   function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)

20:     returns (string memory)

22:     string[21] memory parts;

23:     string memory policyholder = LibString.toHexString(address(uint160(tokenId)));

76:     string memory output1 =

78:     string memory output2 =

80:     string memory output = string.concat(output1, output2, parts[18], parts[19], parts[20]);

82:     string memory json = Base64.encode(

104:   function contractURI(string memory name) public pure returns (string memory) {

105:     string[5] memory parts;

112:     string memory json = Base64.encode(bytes(string.concat(parts[0], parts[1], parts[2], parts[3], parts[4])));

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

61:     string memory rootColor = "#6A45EC";

62:     string memory rootLogo =

74:   function getMetadata(LlamaExecutor llamaExecutor) external view returns (string memory _color, string memory _logo) {

82:   function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {

90:   function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {

```

```solidity
File: src/accounts/LlamaAccount.sol

130:   function initialize(bytes memory config) external initializer {

132:     Config memory accountConfig = abi.decode(config, (Config));

301:     returns (bytes memory)

304:     bytes memory result;

```

```solidity
File: src/interfaces/ILlamaAccount.sol

18:   function initialize(bytes memory config) external;

```

```solidity
File: src/interfaces/ILlamaStrategy.sol

29:   function initialize(bytes memory config) external;

```

```solidity
File: src/lib/Checkpoints.sol

106:             Checkpoint memory ckpt = _unsafeAccess(self._checkpoints, pos - 1);

132:             Checkpoint memory last = _unsafeAccess(self, pos - 1);

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

30:   function tokenURI(uint256 id) public view virtual returns (string memory);

62:   function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {

174:   function _safeMint(address to, uint256 id, bytes memory data) internal virtual {

```

```solidity
File: src/llama-scripts/LlamaGovernanceScript.sol

65:     returns (bytes[] memory returnData)

75:       (bool success, bytes memory response) = targets[i].call(data[i]);

```

```solidity
File: src/strategies/LlamaAbsoluteStrategyBase.sol

153:   function initialize(bytes memory config) external virtual initializer {

154:     Config memory strategyConfig = abi.decode(config, (Config));

273:     Action memory action = llamaCore.getAction(actionInfo.id);

279:     Action memory action = llamaCore.getAction(actionInfo.id);

285:     Action memory action = llamaCore.getAction(actionInfo.id);

295:     Action memory action = llamaCore.getAction(actionInfo.id);

```

```solidity
File: src/strategies/LlamaRelativeQuorum.sol

156:   function initialize(bytes memory config) external initializer {

157:     Config memory strategyConfig = abi.decode(config, (Config));

283:     Action memory action = llamaCore.getAction(actionInfo.id);

289:     Action memory action = llamaCore.getAction(actionInfo.id);

296:     Action memory action = llamaCore.getAction(actionInfo.id);

306:     Action memory action = llamaCore.getAction(actionInfo.id);

```

### [GAS-13] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: src/accounts/LlamaAccount.sol

324:    if (originalStorage != _readSlot0()) revert Slot0Changed();
        } else {
         (success, result) = target.call{value: msg.value}(callData);
        }

```

```solidity
File: src/lib/Checkpoints.sol

48:        if (_timestamp < _unsafeAccess(self._checkpoints, mid).timestamp) {
                high = mid;
            } else {
                low = mid + 1;
            }

167:       if (_unsafeAccess(self, mid).timestamp > timestamp) {
                high = mid;
            } else {
                low = mid + 1;
            }

191:       if (_unsafeAccess(self, mid).timestamp < timestamp) {
                low = mid + 1;
            } else {
                high = mid;
            }

```

### [GAS-14] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: src/LlamaCore.sol

99:     string description

123:   event ApprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);

126:   event DisapprovalCast(uint256 id, address indexed policyholder, uint8 indexed role, uint256 quantity, string reason);

143:     keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

147:     "CreateAction(address policyholder,uint8 role,address strategy,address target,uint256 value,bytes data,string description,uint256 nonce)"

152:     "CastApproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"

157:     "CastDisapproval(address policyholder,uint8 role,ActionInfo actionInfo,string reason,uint256 nonce)ActionInfo(uint256 id,address creator,uint8 creatorRole,address strategy,address target,uint256 value,bytes data)"

183:   string public name;

225:     string memory _name,

265:     string memory description

291:     string memory description,

365:   function castApproval(uint8 role, ActionInfo calldata actionInfo, string calldata reason) external {

382:     string calldata reason,

398:   function castDisapproval(uint8 role, ActionInfo calldata actionInfo, string calldata reason) external {

415:     string calldata reason,

523:     string memory description

565:   function _castApproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)

576:   function _castDisapproval(address policyholder, uint8 role, ActionInfo calldata actionInfo, string memory reason)

721:     string memory description

750:     string calldata reason

772:     string calldata reason

```

```solidity
File: src/LlamaFactory.sol

44:     string indexed name,

108:     string memory name,

155:     string memory name,

163:     string memory color,

164:     string memory logo

206:   function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)

209:     returns (string memory)

211:     (string memory color, string memory logo) = LLAMA_POLICY_METADATA_PARAM_REGISTRY.getMetadata(llamaExecutor);

218:   function contractURI(string memory name) external view returns (string memory) {

228:     string memory name,

285:   function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {

```

```solidity
File: src/LlamaPolicy.sol

144:     string calldata _name,

149:     __initializeERC721MinimalProxy(_name, string.concat("LL-", LibString.replace(LibString.upper(_name), " ", "-")));

346:   function tokenURI(uint256 tokenId) public view override returns (string memory) {

352:   function contractURI() public view returns (string memory) {

```

```solidity
File: src/LlamaPolicyMetadata.sol

17:   function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)

20:     returns (string memory)

22:     string[21] memory parts;

23:     string memory policyholder = LibString.toHexString(address(uint160(tokenId)));

33:     parts[3] = string.concat(LibString.slice(policyholder, 0, 6), "...", LibString.slice(policyholder, 38, 42));

76:     string memory output1 =

77:       string.concat(parts[0], parts[1], parts[2], parts[3], parts[4], parts[5], parts[6], parts[7], parts[8]);

78:     string memory output2 =

79:       string.concat(parts[9], parts[10], parts[11], parts[12], parts[13], parts[14], parts[15], parts[16], parts[17]);

80:     string memory output = string.concat(output1, output2, parts[18], parts[19], parts[20]);

82:     string memory json = Base64.encode(

84:         string.concat(

97:     output = string.concat("data:application/json;base64,", json);

104:   function contractURI(string memory name) public pure returns (string memory) {

105:     string[5] memory parts;

112:     string memory json = Base64.encode(bytes(string.concat(parts[0], parts[1], parts[2], parts[3], parts[4])));

113:     return string.concat("data:application/json;base64,", json);

```

```solidity
File: src/LlamaPolicyMetadataParamRegistry.sol

31:   event ColorSet(LlamaExecutor indexed llamaExecutor, string color);

34:   event LogoSet(LlamaExecutor indexed llamaExecutor, string logo);

47:   mapping(LlamaExecutor => string) public color;

50:   mapping(LlamaExecutor => string) public logo;

61:     string memory rootColor = "#6A45EC";

62:     string memory rootLogo =

74:   function getMetadata(LlamaExecutor llamaExecutor) external view returns (string memory _color, string memory _logo) {

82:   function setColor(LlamaExecutor llamaExecutor, string memory _color) public onlyAuthorized(llamaExecutor) {

90:   function setLogo(LlamaExecutor llamaExecutor, string memory _logo) public onlyAuthorized(llamaExecutor) {

```

```solidity
File: src/accounts/LlamaAccount.sol

30:     string name; // Name of the Llama account.

118:   string public name;

```

```solidity
File: src/lib/ERC721NonTransferableMinimalProxy.sol

26:   string public name;

28:   string public symbol;

30:   function tokenURI(uint256 id) public view virtual returns (string memory);

62:   function __initializeERC721MinimalProxy (string memory _name, string memory _symbol) internal {

```
