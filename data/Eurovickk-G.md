src/LlamaPolicyMetadataParamRegistry.sol

Gas optimization is an important consideration when writing smart contracts. One area to potentially optimize gas usage is the storage of the color and logo mappings. Currently, the mappings are defined as mapping(LlamaExecutor => string), which consumes a significant amount of gas for each storage update. Instead, you could consider using a mapping with a separate struct to store the color and logo, which would reduce the gas costs for storage updates. For example:
struct Metadata {
    string color;
    string logo;
}

mapping(LlamaExecutor => Metadata) public metadata;

src/strategies/LlamaAbsoluteQuorum.sol

In the validateActionCreation function, the variables actionCreatorApprovalRoleQty and actionCreatorDisapprovalRoleQty are computed using the getQuantity function multiple times. You can optimize gas usage by computing these quantities once and storing them in local variables.

src/LlamaPolicyMetadata.sol

Consider using fixed-size arrays instead of dynamic arrays where possible. Dynamic arrays have higher gas costs for operations like appending elements. If the size of the array is known and fixed, using a fixed-size array can reduce gas consumption.

// FixedSizeArrayExample.sol

pragma solidity ^0.8.0;

contract FixedSizeArrayExample {
    uint256[5] public fixedArray;
    uint256[] public dynamicArray;

    function appendToFixedArray(uint256 value) public {
        // This operation is possible because the size of the array is fixed at 5
        for (uint256 i = 0; i < fixedArray.length; i++) {
            if (fixedArray[i] == 0) {
                fixedArray[i] = value;
                break;
            }
        }
    }

    function appendToDynamicArray(uint256 value) public {
        // This operation is more expensive due to the dynamic array resizing
        dynamicArray.push(value);
    }
}

In this example, we have a contract with two arrays: fixedArray, which has a fixed size of 5 elements, and dynamicArray, which is a dynamic array.

The appendToFixedArray function appends an element to the fixedArray by iterating over the array and finding an empty slot to fill. Since the size of the array is known and fixed, we can easily append elements without incurring additional gas costs.

On the other hand, the appendToDynamicArray function appends an element to the dynamicArray using the push function. This operation involves resizing the dynamic array, which incurs higher gas costs compared to the fixed-size array.

By observing the gas costs of calling these functions, you'll notice that appendToFixedArray consumes less gas compared to appendToDynamicArray. This demonstrates the gas optimization potential of using fixed-size arrays when the size of the array is known and fixed.

src/strategies/LlamaRelativeQuorum.sol

In the initialize function, the for-loops for processing forceApprovalRoles and forceDisapprovalRoles could potentially be optimized to use a bytes parameter instead of uint8[]. This would allow the contract to accept compressed data, reducing the gas cost for initialization.

src/LlamaFactory.sol

In the _setDeploymentMetadata function, you can consider using a single storage write operation instead of two separate writes. Instead of calling LLAMA_POLICY_METADATA_PARAM_REGISTRY.setColor and LLAMA_POLICY_METADATA_PARAM_REGISTRY.setLogo separately, you can combine the color and logo values into a struct and write the struct in a single storage operation.

function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {
    LLAMA_POLICY_METADATA_PARAM_REGISTRY.setMetadata(llamaExecutor, Metadata(color, logo));
}

struct Metadata {
    string color;
    string logo;
}


Certainly! Here's a modified version of the _setDeploymentMetadata function that combines the color and logo values into a struct and performs a single storage write operation:

solidity
Copy code
function _setDeploymentMetadata(LlamaExecutor llamaExecutor, string memory color, string memory logo) internal {
    LLAMA_POLICY_METADATA_PARAM_REGISTRY.setMetadata(llamaExecutor, Metadata(color, logo));
}

struct Metadata {
    string color;
    string logo;
}
In this updated version, we create a new Metadata struct that holds the color and logo values. Instead of invoking LLAMA_POLICY_METADATA_PARAM_REGISTRY.setColor and LLAMA_POLICY_METADATA_PARAM_REGISTRY.setLogo separately, we call a single function LLAMA_POLICY_METADATA_PARAM_REGISTRY.setMetadata and pass the llamaExecutor address along with the Metadata struct containing the color and logo values.

By combining the two values into a single struct and performing a single storage write operation, we can potentially save gas costs compared to two separate writes. This optimization reduces the number of storage operations, resulting in improved gas efficiency.

src/llama-scripts/LlamaGovernanceScript.sol

Loop Optimization: In functions like initializeRoles, setRoleHolders, setRolePermissions, revokePolicies, and updateRoleDescriptions, using a for loop with an incremental counter can be gas-intensive. Consider using a uint256 variable initialized outside the loop and increment it within the loop using a post-increment (i++) operator.

function initializeRoles(RoleDescription[] calldata descriptions) public onlyDelegateCall {
    (, LlamaPolicy policy) = _context();
    uint256 length = descriptions.length;
    uint256 i;

    for (i; i < length; i++) {
        policy.initializeRole(descriptions[i]);
    }
}

The i variable is initialized outside the loop and incremented using the post-increment operator i++ within the loop. This optimization reduces the gas cost associated with the loop counter incrementation.

Storage Reads and Writes: In functions like setRoleHolders, setRolePermissions, and updateRoleDescriptions, multiple storage reads and writes are performed within the loop. You can consider optimizing these operations by using local variables to store intermediate values and perform a single storage write operation after the loop.

function setRoleHolders(RoleHolderData[] calldata _setRoleHolders) public onlyDelegateCall {
    (, LlamaPolicy policy) = _context();
    uint256 length = _setRoleHolders.length;

    for (uint256 i = 0; i < length; i++) {
        RoleHolderData calldata data = _setRoleHolders[i];
        policy.setRoleHolder(data.role, data.policyholder, data.quantity, data.expiration);
    }
}

Instead of performing separate storage reads and writes within the loop, the RoleHolderData struct is assigned to a local variable data. The setRoleHolder function is then called with the values from data. This optimization reduces the number of storage read and write operations, potentially resulting in gas savings.

Struct Packing: In the UpdateRoleDescription struct, the role field is defined as uint8. If the total number of roles is small (e.g., less than 256), you can consider using a smaller data type like uint8 or enum to save storage space and reduce gas costs.

struct UpdateRoleDescription {
    Role role; // New role description.
    RoleDescription description; // New role description.
}

enum Role {
    Admin,
    Manager,
    User
}

In this optimization, an enum Role is used instead of uint8 for the role field in the UpdateRoleDescription struct. By using an enum, you can save storage space and reduce gas costs if the total number of roles is small.

src/accounts/LlamaAccount.sol 

 Loop Optimization: In functions like batchTransferNativeToken, batchTransferERC20, batchApproveERC20, batchTransferERC721, batchApproveERC721, batchApproveOperatorERC721, batchTransferMultipleERC1155, and batchApproveOperatorERC1155, using a for loop with an incremental counter can be gas-intensive. Consider using a uint256 variable initialized outside the loop and increment it within the loop using a post-increment (i++) operator. This can help reduce the gas cost of executing these functions.

function batchTransferNativeToken(NativeTokenData[] calldata nativeTokenData) external onlyLlama {
    uint256 length = nativeTokenData.length;
    uint256 i;
    
    for (i; i < length; i++) {
        transferNativeToken(nativeTokenData[i]);
    }
}

By initializing the loop counter i outside the loop and using the post-increment (i++) operator, we can reduce the gas cost of executing the loop.

Storage Reads and Writes: In functions like batchTransferNativeToken, batchTransferERC20, batchApproveERC20, batchTransferERC721, batchApproveERC721, batchApproveOperatorERC721, batchTransferMultipleERC1155, and batchApproveOperatorERC1155, multiple storage reads and writes are performed within the loop. You can consider optimizing these operations by using local variables to store intermediate values and perform a single storage write operation after the loop. This can help reduce the gas cost of executing these functions.

function batchTransferERC20(ERC20Data[] calldata erc20Data) external onlyLlama {
    uint256 length = erc20Data.length;
    uint256 i;

    address[] memory recipients = new address[](length);
    uint256[] memory amounts = new uint256[](length);
    
    for (i; i < length; i++) {
        recipients[i] = erc20Data[i].recipient;
        amounts[i] = erc20Data[i].amount;
    }
    
    for (i = 0; i < length; i++) {
        erc20Data[i].token.safeTransfer(recipients[i], amounts[i]);
    }
}

In this example, we use two separate loops. The first loop initializes arrays to store the intermediate values (recipients and amounts) instead of performing multiple storage reads and writes within the loop. Then, in the second loop, we use these arrays to execute the desired operations, resulting in a single storage write operation after the loop.

src/LlamaPolicy.sol

Gas Optimization:

Consider using the unchecked keyword for safe arithmetic operations to avoid unnecessary gas consumption in certain scenarios.
Use fixed-size data types like uint128 and uint64 where appropriate to save gas costs, especially for storage variables that don't require large sizes.


