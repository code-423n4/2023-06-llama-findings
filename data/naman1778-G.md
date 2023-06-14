## [G-01] Use assembly to write address storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

There are 5 instances of this contract in 5 files:

    File: src/LlamaExecutor.sol

    16: LLAMA_CORE = msg.sender;

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaExecutor.sol

    File: src/LlamaPolicyMetadataParamRegistry.sol	

    59: LLAMA_FACTORY = msg.sender;
    
https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol

    File: src/accounts/LlamaAccount.sol

    131: llamaExecutor = address(LlamaCore(msg.sender).executor());

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

    File: src/LlamaPolicy.sol

    183: llamaExecutor = _llamaExecutor;

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

    File: src/llama-scripts/LlamaBaseScript.sol	

    21: SELF = address(this);

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaBaseScript.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }
    
    contract Contract0 {
    
        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }
    
    }
    
    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }
    
    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-02] Use assembly to check for address(0)

Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

There are 17 instances of this issue in 4 files:

    File: src/accounts/LlamaAccount.sol

    148: if (nativeTokenData.recipient == address(0)) revert ZeroAddressNotAllowed();

    166: if (erc20Data.recipient == address(0)) revert ZeroAddressNotAllowed();

    199: if (erc721Data.recipient == address(0)) revert ZeroAddressNotAllowed();

    247: if (erc1155Data.recipient == address(0)) revert ZeroAddressNotAllowed();

    256: if (erc1155BatchData.recipient == address(0)) revert ZeroAddressNotAllowed();

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

    File: src/LlamaPolicy.sol

    181: if (llamaExecutor != address(0)) revert AlreadyInitialized();

    405: if (llamaExecutor == address(0)) return; // Skip check during initialization.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

    File: src/LlamaCore.sol	

    298: if (signer == address(0) || signer != policyholder) revert InvalidSignature();

    330: if (guard != ILlamaActionGuard(address(0))) guard.validatePreActionExecution(actionInfo);

    339: if (guard != ILlamaActionGuard(address(0))) guard.validatePostActionExecution(actionInfo);

    389: if (signer == address(0) || signer != policyholder) revert InvalidSignature();

    422: if (signer == address(0) || signer != policyholder) revert InvalidSignature();

    550: if (guard != ILlamaActionGuard(address(0))) guard.validateActionCreation(actionInfo);

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

    File: src/lib/ERC721NonTransferableMinimalProxy.sol
 
    41: require((owner = _ownerOf[id]) != address(0), "NOT_MINTED");

    45: require(owner != address(0), "ZERO_ADDRESS");

    90: require(to != address(0), "INVALID_RECIPIENT");

    128: require(to != address(0), "INVALID_RECIPIENT");

    130: require(_ownerOf[id] == address(0), "ALREADY_MINTED");

    145: require(owner != address(0), "NOT_MINTED");

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(0x0000000000000000000000000000000000000001);
            c1.optimized(0x0000000000000000000000000000000000000001);
        }
    }
    
    contract Contract0 {
    
         function not_optimized(address addr) public pure{
             if(addr == address(0)){
                revert();
             }
         }
    }
    
    contract Contract1 {
    
         function optimized(address addr) public pure{
             assembly {
               if iszero(addr) {
                   revert(0x00, 0x20)
               }
            }
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 41893                                     | 240             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 258             | 258 | 258    | 258 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 37687                                     | 219             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 252             | 252 | 252    | 252 | 1       |

## [G-03] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 25 instances of this issue in 6 files:

    File: src/strategies/LlamaRelativeQuorum.sol	    

    177: for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

    185: for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

    File: src/llama-scripts/LlamaGovernanceScript.sol	

    71: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    178: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    186: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    199: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    210: for (uint256 i = 0; i < _revokePolicies.length; i = LlamaUtils.uncheckedIncrement(i)) {

    217: for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol  

    File: src/accounts/LlamaAccount.sol

    156: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    174: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    189: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {    

    207: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    222: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    237: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    270: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

    285: for (uint256 i = 0; i < length; i = LlamaUtils.uncheckedIncrement(i)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

    File: src/LlamaPolicy.sol

    151: for (uint256 i = 0; i < roleDescriptions.length; i = LlamaUtils.uncheckedIncrement(i)) {

    155: for (uint256 i = 0; i < roleHolders.length; i = LlamaUtils.uncheckedIncrement(i)) {

    161: for (uint256 i = 0; i < rolePermissions.length; i = LlamaUtils.uncheckedIncrement(i)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

    File: src/LlamaCore.sol

    636: for (uint256 i = 0; i < strategyLength; i = LlamaUtils.uncheckedIncrement(i)) {

    656: for (uint256 i = 0; i < accountLength; i = LlamaUtils.uncheckedIncrement(i)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

    File: src/strategies/LlamaAbsoluteStrategyBase.sol	

    177: for (uint256 i = 0; i < strategyConfig.forceApprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

    185: for (uint256 i = 0; i < strategyConfig.forceDisapprovalRoles.length; i = LlamaUtils.uncheckedIncrement(i)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-04] Use nested if and, avoid multiple check combinations

Using nested if, is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 20 instances of this issue in 8 files:

    File: src/LlamaPolicyMetadataParamRegistry.sol	

    21: msg.sender != address(llamaExecutor) && msg.sender != address(ROOT_LLAMA_EXECUTOR) && msg.sender != LLAMA_FACTORY

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol

    File: src/strategies/LlamaAbsoluteQuorum.sol	

    36: if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

    49: if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

    61: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol

    File: src/strategies/LlamaAbsolutePeerReview.sol

    57: minDisapprovals != type(uint128).max
    58:   && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty

    68: if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

    81: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol

    File: src/strategies/LlamaRelativeQuorum.sol	    

    216: if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

    221: if (role != approvalRole && !forceApprovalRole[role]) return 0;

    231: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

    240: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

    File: src/llama-scripts/LlamaGovernanceScript.sol	

    74: if (!addressIsCore && !addressIsPolicy) revert UnauthorizedTarget(targets[i]);

https://github.com/code-423n4/2023-06-llama/blob/main/src/llama-scripts/LlamaGovernanceScript.sol  

    File: src/LlamaPolicy.sol

    467: if (hadRole && !willHaveRole) {

    470: } else if (!hadRole && willHaveRole) {

    473: } else if (hadRole && willHaveRole && initialQuantity > quantity) {

    476: } else if (hadRole && willHaveRole && initialQuantity < quantity) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

    File: src/LlamaCore.sol	

    629: if (address(factory).code.length > 0 && !factory.authorizedStrategyLogics(llamaStrategyLogic)) {

    649: if (address(factory).code.length > 0 && !factory.authorizedAccountLogics(llamaAccountLogic)) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

    File: src/strategies/LlamaAbsoluteStrategyBase.sol	

    213: if (role != approvalRole && !forceApprovalRole[role]) return 0;

    230: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }
    
    contract Contract0 {
    
        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |

## [G-05] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There are 7 instances of this issue in 1 file:

    File: src/LlamaCore.sol	

    242: return keccak256(abi.encode(PermissionData(address(policy), bytes4(selector), bootstrapStrategy)));

    529: bytes32 permissionId = keccak256(abi.encode(permission));

    708: abi.encode(EIP712_DOMAIN_TYPEHASH, keccak256(bytes(name)), keccak256(bytes("1")), block.chainid, address(this))

    728: abi.encode(

    753: abi.encode(

    775: abi.encode(

    791: abi.encode(

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-06] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

There are 10 instances of this issue in 6 files:

    File: src/LlamaPolicyMetadata.sol	

    17: function tokenURI(string memory name, uint256 tokenId, string memory color, string memory logo)

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol

    File: src/strategies/LlamaRelativeQuorum.sol	    

    156: function initialize(bytes memory config) external initializer {

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

    File: src/LlamaFactory.sol	

    154: function deploy(
    155:   string memory name,
    156:   ILlamaStrategy strategyLogic,
    157:   ILlamaAccount accountLogic,
    158:   bytes[] memory initialStrategies,
    159:   bytes[] memory initialAccounts,
    160:   RoleDescription[] memory initialRoleDescriptions,
    161:   RoleHolderData[] memory initialRoleHolders,
    162:   RolePermissionData[] memory initialRolePermissions,
    163:   string memory color,
    164:   string memory logo
    165: ) external onlyRootLlama returns (LlamaExecutor executor, LlamaCore core) {

    206: function tokenURI(LlamaExecutor llamaExecutor, string memory name, uint256 tokenId)

    218: function contractURI(string memory name) external view returns (string memory) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol

    File: src/accounts/LlamaAccount.sol

    130: function initialize(bytes memory config) external initializer {

https://github.com/code-423n4/2023-06-llama/blob/main/src/accounts/LlamaAccount.sol

    File: src/LlamaCore.sol	

    224: function initialize(
    225:   string memory _name,

    259: function createAction(
    260:   uint8 role,
    261:   ILlamaStrategy strategy,
    262:   address target,
    263:   uint256 value,
    264:   bytes calldata data,
    265:   string memory description
    266: ) external returns (uint256 actionId) {

    284: function createActionBySig(
    285:   address policyholder,
    286:   uint8 role,
    287:   ILlamaStrategy strategy,
    288:   address target,
    289:   uint256 value,
    290:  bytes calldata data,
    290:   string memory description,
    291:   uint8 v,
    292:   bytes32 r,
    293:   bytes32 s
    294: ) external returns (uint256 actionId) {

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

    File: src/strategies/LlamaAbsoluteStrategyBase.sol	

    153: function initialize(bytes memory config) external virtual initializer {

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }

    contract Contract0 {

         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }

    contract Contract1 {

         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-07] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

There are 21 instances of this issue in 7 files:

    File: src/strategies/LlamaAbsoluteQuorum.sol	

    /// @audit approvalRole
    29: uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(approvalRole);
    30: if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);
    
    /// @audit disapprovalRole
    32: uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(disapprovalRole);
    33: if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);

    /// @audit minDisapprovals 
    36: if (minDisapprovals != type(uint128).max && minDisapprovals > disapprovalPolicySupply) {

    /// @audit approvalRole
    49: if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);

    /// @audit disapprovalRole
    61: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteQuorum.sol

    File: src/strategies/LlamaAbsolutePeerReview.sol

    /// @audit approvalRole
    42: uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(approvalRole);
    43: if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);
    52: uint256 actionCreatorApprovalRoleQty = llamaPolicy.getQuantity(actionInfo.creator, approvalRole);

    /// @audit disapprovalRole
    45: uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(disapprovalRole);
    46: if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);
    55: uint256 actionCreatorDisapprovalRoleQty = llamaPolicy.getQuantity(actionInfo.creator, disapprovalRole);

    /// @audit minDisapprovals 
    57: minDisapprovals != type(uint128).max
    58: && minDisapprovals > disapprovalPolicySupply - actionCreatorDisapprovalRoleQty

    /// @audit approvalRole
    68: if (role != approvalRole && !forceApprovalRole[role]) revert InvalidRole(approvalRole);
 
    /// @audit disapprovalRole
    81: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsolutePeerReview.sol

    File: src/strategies/LlamaRelativeQuorum.sol	    

    /// @audit approvalRole
    201: uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsNumberOfHolders(approvalRole);
    202: if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);

    /// @audit disapprovalRole
    204: uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsNumberOfHolders(disapprovalRole);
    205: if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);

    /// @audit forceApprovalRole[role]
    221: if (role != approvalRole && !forceApprovalRole[role]) return 0;
    223: return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;
    
    /// @audit disapprovalRole
    231: if (role != disapprovalRole && !forceDisapprovalRole[role]) revert InvalidRole(disapprovalRole);

    /// @audit forceDisapprovalRole[role]
    240: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
    241: return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

    File: src/LlamaFactory.sol	

    /// @audit llamaCount 
    261: llamaCount, name, address(llamaCore), address(llamaExecutor), address(policy), block.chainid
    263: llamaCount = LlamaUtils.uncheckedIncrement(llamaCount);

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaFactory.sol    

    File: src/LlamaPolicy.sol

    /// @audit llamaExecutor 
    405: if (llamaExecutor == address(0)) return; // Skip check during initialization.
    406: address llamaCore = LlamaExecutor(llamaExecutor).LLAMA_CORE();

    /// @audit roleBalanceCkpts[tokenId][role]
    443: uint128 initialQuantity = roleBalanceCkpts[tokenId][role].latest();
    448: roleBalanceCkpts[tokenId][role].push(willHaveRole ? quantity : 0, expiration);

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

    File: src/LlamaCore.sol

    /// @audit actionsCount
    542: actionId = actionsCount;
    559: actionsCount = LlamaUtils.uncheckedIncrement(actionsCount); // Safety: Can never overflow a uint256 by incrementing.

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

    File: src/strategies/LlamaAbsoluteStrategyBase.sol	

    /// @audit forceApprovalRole[role]
    213: if (role != approvalRole && !forceApprovalRole[role]) return 0;
    215: return quantity > 0 && forceApprovalRole[role] ? type(uint128).max : quantity;

    /// @audit forceDisapprovalRole[role]
    230: if (role != disapprovalRole && !forceDisapprovalRole[role]) return 0;
    232: return quantity > 0 && forceDisapprovalRole[role] ? type(uint128).max : quantity;

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
         uint8 num=0;
         function not_optimized() public returns(uint8){
            if(num==0)
            {
               return num;
            }
         }
    }

    contract Contract1 {
         uint8 num=0;
         function optimized() public returns(uint8){
            uint8 num1 = num;
            if(num1==0)
            {
               return num1;
            }
         }
    }

#### Gas Report 

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 32299                                     | 190             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2415            | 2415 | 2415   | 2415 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 31699                                     | 187             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2312            | 2312 | 2312   | 2312 | 1       |

## [G-08] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 7 instances of this issue in 4 files:

    File: src/strategies/LlamaRelativeQuorum.sol	    

    265: state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
    266:   || state == ActionState.Failed // @audit bitwise operator

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol

    File: src/LlamaCore.sol

    445: if (target == address(this) || target == address(policy)) revert RestrictedAddress();

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol

    File: src/lib/ERC721NonTransferableMinimalProxy.sol
 
    92: require(msg.sender == from || isApprovedForAll[from][msg.sender] || msg.sender == getApproved[id], "NOT_AUTHORIZED");

    118: return interfaceId == 0x01ffc9a7 // ERC165 Interface ID for ERC165
    119:  || interfaceId == 0x80ac58cd // ERC165 Interface ID for ERC721
    120:  || interfaceId == 0x5b5e139f; // ERC165 Interface ID for ERC721Metadata

    167: to.code.length == 0
    168:   || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, "")
    169:     == ERC721TokenReceiver.onERC721Received.selector,

    178: to.code.length == 0
    179:   || ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, data)
    180:     == ERC721TokenReceiver.onERC721Received.selector,

https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol    

    File: src/strategies/LlamaAbsoluteStrategyBase.sol	

    255: state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
    256:   || state == ActionState.Failed

https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaAbsoluteStrategyBase.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-09] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There is 1 instance of this issue in 1 file:

    File: src/LlamaPolicy.sol

    289: uint256 sliceLength = end - start;

https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }
    
    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |