https://github.com/code-423n4/2023-06-llama/blob/main/src/lib/ERC721NonTransferableMinimalProxy.sol

In this contract, there are 3 transfers like this emit Transfer(owner, address(0), id); in lines 106,139,156 this should use the safeTransfer function.