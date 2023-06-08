File: script/DeployUtils.sol

72:  function readRelativeStrategies(string memory jsonInput)
94:  function readAccounts(string memory jsonInput)
107: function readRoleDescriptions(string memory jsonInput)
115: function readRoleHolders(string memory jsonInput)
129: function readRolePermissions(string memory jsonInput)

In this functions you give input as a JSON. This type of actions is not blockchain specific. You must do all preprocessing of JSON file outside of blockchain with JS and then call needed functions from blockchain with filtered data. In case of big JSON file these functions will cost huge amount of gas. If memory exceeds 724 bytes the gas will increase quadraticaly which will cause high gas cost and even DoS problem. It can also exceeds block gas limit in case if someone will frontrun your transaction and consume some of block gas limit.
