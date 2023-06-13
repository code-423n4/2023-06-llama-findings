## GAS-1: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* Checkpoints.sol (Line: 135)
* LlamaAbsoluteStrategyBase.sol (Line: 268, 274, 280)
* LlamaPolicy.sol (Line: 227)
* LlamaRelativeQuorum.sol (Line: 278, 284, 291)

## GAS-2: Cache the mapping values rather than fetch it every time

### Affected file

* LlamaAbsoluteStrategyBase.sol (Line: 213, 215)
* LlamaPolicy.sol (Line: 443, 448)
* LlamaRelativeQuorum.sol (Line: 221, 223)

## GAS-3: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* LlamaExecutor.sol (Line: 16)
* LlamaPolicyMetadataParamRegistry.sol (Line: 59)

## GAS-4: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* LlamaCore.sol (Line: 142, 146, 151, 156, 161)

## GAS-5: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* Checkpoints.sol (Line: 167, 191)
* ERC721NonTransferableMinimalProxy.sol (Line: 45, 90, 128, 145, 166, 177)
* LlamaAbsolutePeerReview.sol (Line: 43, 46, 56, 67, 68, 68, 79, 80, 81, 81)
* LlamaAbsoluteQuorum.sol (Line: 30, 33, 36, 49, 49, 60, 61, 61)
* LlamaAbsoluteStrategyBase.sol (Line: 179, 187, 213, 213, 230, 230, 254, 254, 254, 254, 260, 305)
* LlamaAccount.sol (Line: 104)
* LlamaCore.sol (Line: 298, 298, 389, 389, 422, 422, 607, 611, 629, 649)
* LlamaPolicy.sol (Line: 69, 237, 414, 467, 470, 473, 473, 476, 476, 491)
* LlamaRelativeQuorum.sol (Line: 179, 187, 202, 205, 216, 216, 221, 221, 231, 231, 240, 240, 264, 264, 264, 264, 270, 325)

## GAS-6: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* ERC721NonTransferableMinimalProxy.sol (Line: 62, 142, 163, 174)

## GAS-7: Multiple accesses of a mapping/array should use a local variable cache

### Description

Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.
To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

### Affected file

* ERC721NonTransferableMinimalProxy.sol (Line: 130, 137)

## GAS-8: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* LlamaAccount.sol (Line: 103)
* LlamaBaseScript.sol (Line: 11)
* LlamaCore.sol (Line: 81)
* LlamaFactory.sol (Line: 32)
* LlamaPolicy.sol (Line: 68, 75)
* LlamaPolicyMetadataParamRegistry.sol (Line: 19)
* LlamaSingleUseScript.sol (Line: 14)

## GAS-9: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* LlamaCore.sol (Line: 242)
* LlamaPolicyMetadataParamRegistry.sol (Line: 64, 65)

## GAS-10: Storage pointer to a structure is cheaper than copying each value of the structure into memory, same for array and mapping

### Description

It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.

### Affected file

* Checkpoints.sol (Line: 106, 132)
* LlamaAbsoluteStrategyBase.sol (Line: 154, 273, 279, 285, 295)
* LlamaAccount.sol (Line: 132, 304)
* LlamaCore.sol (Line: 333, 528, 544)
* LlamaFactory.sol (Line: 211)
* LlamaGovernanceScript.sol (Line: 75)
* LlamaPolicy.sol (Line: 290)
* LlamaPolicyMetadata.sol (Line: 22, 23, 76, 78, 80, 90)
* LlamaPolicyMetadataParamRegistry.sol (Line: 61, 62)
* LlamaRelativeQuorum.sol (Line: 157, 283, 289, 296, 306)

## GAS-11: The result of a function call should be cached rather than re-calling the function

### Description

External calls are expensive. Consider caching.

### Affected file

* Checkpoints.sol (Line: 132, 139, 143, 147, 270, 270, 270, 270, 270, 270, 271, 272, 273, 274, 275, 276)
* LlamaAbsoluteStrategyBase.sol (Line: 177, 180, 185, 188)
* LlamaCore.sol (Line: 528, 549)
* LlamaFactory.sol (Line: 250, 253)
* LlamaPolicy.sol (Line: 151, 151, 155, 161, 285, 288)
* LlamaRelativeQuorum.sol (Line: 177, 180, 185, 188)

## GAS-12: Use ```assembly``` to write address storage values

### Affected file

* ERC721NonTransferableMinimalProxy.sol (Line: 63, 64)
* LlamaAbsoluteStrategyBase.sol (Line: 155, 156, 157, 158, 159, 160, 166, 167, 171, 174)
* LlamaAccount.sol (Line: 131, 133)
* LlamaBaseScript.sol (Line: 21)
* LlamaCore.sol (Line: 232, 233, 234, 235, 305, 319, 349, 490, 553, 559)
* LlamaExecutor.sol (Line: 16)
* LlamaFactory.sol (Line: 115, 116, 133, 263, 280)
* LlamaPolicy.sol (Line: 150, 183, 465, 508, 522)
* LlamaPolicyMetadataParamRegistry.sol (Line: 58, 59)
* LlamaRelativeQuorum.sol (Line: 158, 159, 160, 161, 162, 163, 166, 167, 171, 174)
* LlamaSingleUseScript.sol (Line: 25)

## GAS-13: Use assembly to check for address(0)

### Description

Saves 6 gas per instance if using assembly to check for address(0).

### Remediation

```
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```

### Affected file

* ERC721NonTransferableMinimalProxy.sol (Line: 41, 45, 90, 128, 130, 145)

## GAS-14: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* Checkpoints.sol (Line: 77)
* LlamaAbsolutePeerReview.sol (Line: 57, 79)
* LlamaAbsoluteQuorum.sol (Line: 36, 60)
* LlamaAbsoluteStrategyBase.sol (Line: 215, 232)
* LlamaCore.sol (Line: 617)
* LlamaFactory.sol (Line: 244)
* LlamaRelativeQuorum.sol (Line: 223, 242)
* LlamaUtils.sol (Line: 11, 17)

## GAS-15: Use elementary types or a user-defined type instead of a struct that has only one member

### Affected file

* Checkpoints.sol (Line: 21)
* LlamaAccount.sol (Line: 29)

## GAS-16: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* LlamaAccount.sol (Line: 200, 249, 258)
* LlamaBaseScript.sol (Line: 12, 21)
* LlamaCore.sol (Line: 445, 455, 708)
* LlamaGovernanceScript.sol (Line: 223)

## GAS-17: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* LlamaAbsolutePeerReview.sol (Line: 56, 68, 81)
* LlamaAbsoluteQuorum.sol (Line: 36, 49, 61)
* LlamaAbsoluteStrategyBase.sol (Line: 213, 230)
* LlamaCore.sol (Line: 629, 649)
* LlamaGovernanceScript.sol (Line: 74)
* LlamaPolicy.sol (Line: 467, 470, 473, 476)
* LlamaPolicyMetadataParamRegistry.sol (Line: 20)
* LlamaRelativeQuorum.sol (Line: 216, 221, 231, 240)

## GAS-18: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* Checkpoints.sol (Line: 130)
* LlamaCore.sol (Line: 629, 649)

## GAS-19: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* LlamaBaseScript.sol (Line: 18)
* LlamaExecutor.sol (Line: 12)
* LlamaFactory.sol (Line: 71, 74, 77)
* LlamaPolicyMetadataParamRegistry.sol (Line: 41, 44)
* LlamaSingleUseScript.sol (Line: 22)

## GAS-21: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* LlamaAccount.sol (Line: 323, 326)
* LlamaExecutor.sol (Line: 34)
* LlamaGovernanceScript.sol (Line: 75)

## GAS-22: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* LlamaCore.sol (Line: 242, 529, 708, 728, 753, 775, 791)