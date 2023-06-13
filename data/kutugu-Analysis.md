# Introduction

Llama is a governance system for onchain organizations. It uses non-transferable NFTs to encode access control, features programmatic control of funds, and includes a modular framework to define action execution rules.  

# About LLAMA

The [video](https://www.loom.com/share/2cd1513d4ee34aa8ace6df51e402c259) explainer provides a high-level overview of the Llama system and the [docs](https://github.com/code-423n4/2023-06-llama/tree/main/docs) describe the core components.

# Architecture

![Architecture](https://raw.githubusercontent.com/code-423n4/2023-06-llama/main/diagrams/llama-overview.png)

# Observations

The core concept of LLAMA is to represent a unique user through the "Soulbound Token", note that the token can't be used as a DID and can still be attacked by witches. Therefore, LLAMA exercises various rights by granting role to tokenId, similar to the reputation system, and approximates the concept of DID in the form of `tokenId` + `role`.    
Different governance systems are isolated from each other, and the contract code is reused through `clone`.    
The system works through the decentralized voting mechanism for the execution of `action`, for the execution of various things, support `delegatecall` and `call`.    

# Threat Model

## Centralization risks

According to the protocol, there are no privileged roles and powers, but LLAMA as a general framework, doesn't make clear specifications. The implementation depends on the specific governance system, see `Specification` below.  

## Systemic risks

### Specification

As a general governance system, LLAMA implements the basic framework, so there are no clear specification. Including the following problems:
- If the off-chain DAO is not decentralized enough, the system can still be manipulated.
- Some user roles are centralized granted directly at initialization, rather than by voting. Although this may be determined by off-chain voting, but as I said earlier, this is not clear specification.
- How are voting weights determined? Can whales manipulate elections if they rely on a token count?
- How often the vote weights change dynamically, an attack vector against `LlamaAbsolutePeerReview` is to dynamically change the creator's vote weights.  
- The action cannot be controlled. If the action target and selector are the same at the same time, the action guard may conflict.

### Contract clone

- `LLAMA_POLICY_LOGIC` / `LLAMA_CORE_LOGIC` is immutable, if there is a bug, factory contract need be abandoned
- `constant` / `immutable` value is hardcoded and will be cloned synchronously
- clone may be frontrun when it is profitable, for example, policy and llamaCore's `name` could be used for fraud on NFT

### Contract call

- `delegatecall`: Whether the untrusted contract can be invoked to modify the contract storage and control the contract
    - `LlamaExecutor`: no storage, works well
    - `LlamaAccount`: can change storage and reset later
- `call`: Focus on contract function calls within this protocol, calling external contracts have no effect
    - Find the function that can bypass the timelock(voting). The easiest way is to look for a missing function that is not with a `onlyLlama` modifier.   

### NFT metadata

- arbitrary scripts can be injected into svg to steal user data.   
- arbitrary data can be injected to name to corrupt the hard-coded data.   
- name length has no limit, may destroy NFT display.    

## Multichain

According to the documentation, LLAMA can be deployed on EVM compatible chains and Rollup. There are some related issues to be aware of:
- `push0` is still not supported by many chains, like Arbitrum and might be problematic for projects compiled with a version of Solidity `>= 0.8.20` (when it was introduced). 
- Signature `replay` / `malleability` attack, and they are handled well by LLAMA
- zkSync Era `opcode`

## EIP

- Whether the EIP is implemented correctly, including `EIP712` and `EIP721`.  
    - The `tokenURI` is not implemented as `EIP721` required.

## Bug Fix

LLAMA had been [audited](https://github.com/code-423n4/2023-06-llama/blob/main/audits/Llama-Spearbit-Audit.pdf) before and needed to check if the bugs had been fixed correctly.

Scope of change:
* New `ILlamaAccount` interface
* Replacing all occurences of `LlamaAccount` with `ILlamaAccount`
throughout the codebase.
* `LlamaAccount` now implements `ILlamaAccount`
* `authorizedAccountLogics` mapping in `LlamaFactory`
* Account Logic contracts authorized in `LlamaFactory` as part of the
factory construction and `authorizeAccountLogic()` function which sets
`authorizedAccountLogics` mapping to true and emits
`AccountLogicAuthorized` event.
* `_deployAccounts` in `LlamaCore` now has a check to see if the Llama
Account logic contract is an authorized one and reverts with a
`UnauthorizedAccountLogic` error if not.
* Updated `computeLlamaAccountAddress` function signature in
`LlamaLens`.
* Relevant Tests and updated existing tests to reflect changes in event
and function signatures.
* Updated initial parameters in input jsons used for initial deploy and
action creation.
* Removing all occurences of `payable` for `ILlamaAccount`
* `ILlamaAccount` `initialize()` function takes `bytes memory config` as
input parameter instead of `string memory name` to make it generalizable
for future Account Logic contracts.
* Path change: `src/LlamaAccount.sol` -> `src/accounts/LlamaAccount.sol`
* Removing certain indexed fields on `StrategyAuthorized`,
`AccountCreated` and `ScriptAuthorized` events which are unique and have
no need to be indexed.

### Time spent:
24 hours