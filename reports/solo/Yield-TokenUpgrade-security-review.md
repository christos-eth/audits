# Yield TokenUpgrade Security Review by Christos Pap 
A security review of the `TokenUpgrade` smart contract developed by the [Yield](https://yieldprotocol.com/) protocol was performed by [Christos Pap](https://twitter.com/christos_eth). \
This security review report includes all the vulnerabilities, issues and code improvements found during the security review.

## Disclaimer

"Audits are a time, resource and expertise bound effort where trained experts evaluate smart
contracts using a combination of automated and manual techniques to find as many vulnerabilities
as possible. Audits can show the presence of vulnerabilities **but not their absence**."

\- Secureum

# About Christos 
Christos Pap is an independent security researcher who specializes in Ethereum smart contract security. He currently works as a Junior Security Researcher at Spearbit and is an alumnus of the yAcademy fellowship program. Additionally, he is an undergraduate student of Electrical and Computer Engineering and he has extensive experience in offensive security, having worked as a Penetration Tester and holding the OSCP certification. You can connect with him on Twitter at [@christos_eth](https://twitter.com/christos_eth).

# About the `TokenUpgrade` contract
The `TokenUpgrade` contract is specifically designed to facilitate the token upgrade process for users who wish _to upgrade_ their old strategy tokens to new strategy tokens at a fixed rate. This functionality is particularly relevant to users who were part of the Yield strategies affected by the Euler hack. The Yield team will construct the Merkle Tree off-chain, incorporating the holders of the old strategy tokens. This Merkle Tree serves as a proof of ownership for the tokens during the upgrade process and users have to call the `upgrade` function with the corresponding `from`, `tokenInAmount` and `proof` arguments and upgrade their tokens based on the calculated `ratio`.  

## Risk classification

| Severity           | Impact: High | Impact: Medium | Impact: Low |
| :----------------- | :----------: | :------------: | :---------: |
| Likelihood: High   |   Critical   |      High      |   Medium    |
| Likelihood: Medium |     High     |     Medium     |     Low     |
| Likelihood: Low    |    Medium    |      Low       |     Low     |

### Impact

- **High** - leads to a significant material loss of assets in the protocol or significantly harms a group of users.
- **Medium** - only a small amount of funds can be lost (such as leakage of value) or a core functionality of the protocol is affected.
- **Low** - can lead to any kind of unexpected behaviour with some of the protocol's functionalities that's not so critical.

### Likelihood

- **High** - attack path is possible with reasonable assumptions that mimic on-chain conditions and the cost of the attack is relatively low to the amount of funds that can be stolen or lost.
- **Medium** - only conditionally incentivized attack vector, but still relatively likely.
- **Low** - has too many or too unlikely assumptions or requires a huge stake by the attacker with little or no incentive.

### Actions required by severity level

- **Critical** - client **must** fix the issue.
- **High** - client **must** fix the issue.
- **Medium** - client **should** fix the issue.
- **Low** - client **could** fix the issue.

## Executive summary

### Overview

|               |                                                                                                                                     |
| :------------ | :---------------------------------------------------------------------------------------------------------------------------------- |
| Project Name  | Yield Protocol                                                                                                                      |
| Repository    | [https://github.com/yieldprotocol/yield-utils-v2/tree/audit/token-upgrade](https://github.com/yieldprotocol/yield-utils-v2/tree/audit/token-upgrade)                |
| Commit hash   | [`2569790d2c5c98e0badce1edd8066f38e7f66850`](https://github.com/yieldprotocol/yield-utils-v2/commit/2569790d2c5c98e0badce1edd8066f38e7f66850)    |
| Documentation | Inline Documentation                                                                                                                |
| Methods       | Manual review, Static Analysis                                                                                                                       |
|               |

### Issues found

| Severity      | Count |
| :------------ | ----: |
| Critical risk |     1 |
| High risk     |     1 |
| Medium risk   |     0 |
| Low risk      |     2 |
| Informational |     1 |

### Scope

| File                                                                                                                                                                             | nSLOC |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---- |
| Contracts (1)                                                                                                                                                                    |
| [src/token/TokenUpgrade.sol](https://github.com/yieldprotocol/yield-utils-v2/blob/feat/merkle-distributor/src/token/TokenUpgrade.sol)                                | 89   |
| Interfaces (1)                                                                                                                                                                   |
| [src/interfaces/ITokenUpgrade.sol](https://github.com/yieldprotocol/yield-utils-v2/blob/feat/merkle-distributor/src/interfaces/ITokenUpgrade.sol)                             | 14   |
| Total (2)                                                                                                                                                                        | 103   |

# Findings

| ID     | Title                                                                       | Severity      | Resolution   |
| :----- | :-------------------------------------------------------------------------- | :------------ | :----------- |
| [C-01] | An attacker can frontrun the `swap` function to steal users' tokens          | Critical      | Fixed |
| [H-01] | Users can submit multiple times the same Merkle Tree leaf                   | High          | Fixed |
| [L-01] | Single-step ownership change is risky                                       | Low           | Acknowledged            |
| [L-02] | Insufficient precision when dealing with tokens with different decimals or varying valuations | Low           | Fixed |
| [I-01] | Prefer `MerkleProof.verifyCalldata` over `MerkleProof.verify` for calldata arguments  | Low           | Fixed |


## Critical severity
### [C-01] An attacker can frontrun the `swap` function to steal users' tokens
**Context:** [TokenUpgrade.sol#L125](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L125)

**Description:** The [`swap`](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L125) function, now renamed `upgrade`, is designed for users to upgrade their tokens to their respective replacements based on a registered ratio. To initiate the token upgrade process, users need to approve the `TokenUpgrade` contract and then call the `swap` function.

However, the `to` parameter provided in the `swap` function isn't validated. As a result, an attacker can frontrun the call to the `swap` function, pass his/hers address as the `to` parameter alongside the intended arguments (`from`, `tokenInAmount` and `proof`) and eventually steal users' tokens.

The [`swap`](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L125) function:
```solidity
function swap(IERC20 tokenIn_, address from, address to, uint256 tokenInAmount, bytes32[] calldata proof)
        external
    {
        TokenIn memory tokenIn = tokensIn[tokenIn_];
        if (address(tokenIn.reverse) == address(0)) revert TokenInNotRegistered(address(tokenIn_));
        IERC20 tokenOut_ = tokenIn.reverse;

        bytes32 leaf = keccak256(abi.encodePacked(from, tokenInAmount));
        bool isValidLeaf = MerkleProof.verify(proof, tokenIn.merkleRoot, leaf);
        if (!isValidLeaf) revert NotInMerkleTree();

        tokenIn_.safeTransferFrom(from, address(this), tokenInAmount);
        uint256 tokenOutAmount = tokenInAmount.wmul(tokenIn.ratio);

        tokensIn[tokenIn_].balance += tokenInAmount;
        tokensOut[tokenOut_].balance -= tokenOutAmount;

        tokenOut_.safeTransfer(to, tokenOutAmount);

        emit Swapped(tokenIn_, tokenOut_, tokenInAmount, tokenOutAmount);
    }
```

Also, depending on the number of the previous holders, an attacker may be able to reconstruct the merkle tree and supply the `from`, `tokenInAmount`, and `proof` arguments for each user, while setting their own address as the `to` argument.

**Recommended Mitigation Steps:** To address this security issue, one of the following two approaches is recommended:
- Remove the `from` parameter and replace it with `msg.sender`. This modification ensures that the Merkle Tree node is constructed based on the original sender, and the old tokens are transferred from the user calling the `swap` function.
- Eliminate the `to` parameter from the `swap` function and send the replacement tokens to the `from` address used to calculate the merkle tree node. This alternative approach is preferable, since it doesn't restrict the `swap` function invocation only to the contracts/EOAs that own the old strategy tokens and it may be beneficial for a contract that owns the original tokens but cannot call the swap function due to its design.

**Yield Team:** Fixed by removing the `to` parameter in [Commit](https://github.com/yieldprotocol/yield-utils-v2/pull/74/commits/97e08cc8c2932cf909d1fea993aeb70cb9b3b85e).

**Christos Pap:** Verified.

## High severity
### [H-01] Users can submit multiple times the same Merkle tree leaf
**Context:** [TokenUpgrade.sol#L132-L134](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L132-L134)

**Description:** The [`swap`](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L125) function, now renamed `upgrade`, is designed for users to upgrade their tokens to their respective replacements based on a registered ratio. To initiate the token upgrade process, users need to approve the `TokenUpgrade` contract and then call the `swap` function.

The Mekle tree is constructed off-chain and users provide the `from`, `tokenInAmount` (used to calculate the leaf of the Merkle tree) and `proof` parameters in the `swap` function. 
If the leaf is verified based on the `proof`, the old tokens are transferred from the user in the contract and the user receives the new tokens based on the registered ratio.
However, the submitted leaves are not stored in the smart contract. As a result, it is possible for someone with a sufficient number of tokens (possibly obtained through a direct transfer from another user) to submit the **same leaf** multiple times. This renders the use of Merkle trees useless for the smart contract, as the same leaf can be submitted multiple times.

The [`swap`](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L125) function:
```solidity
function swap(IERC20 tokenIn_, address from, address to, uint256 tokenInAmount, bytes32[] calldata proof)
        external
    {
        . . . 

        bytes32 leaf = keccak256(abi.encodePacked(from, tokenInAmount));
        bool isValidLeaf = MerkleProof.verify(proof, tokenIn.merkleRoot, leaf);
        if (!isValidLeaf) revert NotInMerkleTree();

        tokenIn_.safeTransferFrom(from, address(this), tokenInAmount);
        ...
    }
```

**Recommended Mitigation Steps:** To address this issue, you can introduce a new mapping called `isClaimed`, with the key type of `bytes32` and value type `bool`. When a leaf is used, you can set the corresponding value in `isClaimed` to `true`. Additionally, you can include a require statement like the following after calculating the leaf: `require(!isClaimed[leaf], 'Already claimed.');`

For gas optimization, you can adopt a similar approach to the [`Uniswap's Merkle Distributor repository`](https://github.com/Uniswap/merkle-distributor/blob/master/contracts/MerkleDistributor.sol), where you include the index of the balance array in the proof itself. By doing so, when storing the claimed status, you only need to update a single bit in the new `uint256 => uint256` mapping.

**Yield Team:** Fixed by introducing a `mapping(bytes32 => bool)` in [Commit](https://github.com/yieldprotocol/yield-utils-v2/pull/74/commits/bd5a58c484dd14fa1be943b6fc3f13c5f28eb097).

**Christos Pap:** Verified.

## Low severity
### [L-01]  Single-step ownership change is risky
**Context:** [TokenUpgrade.sol#L14](https://github.com/yieldprotocol/yield-utils-v2/blob/2569790d2c5c98e0badce1edd8066f38e7f66850/src/token/TokenUpgrade.sol#L14)

**Description:** This is a standard deviation from best practices, which poses a security risk. This approach is not recommended, as if the new owner's address is mistakenly set to the wrong address, it can result in the `auth` modifier's associated functions (`register`, `auth`, `extract`, `recover`) becoming inaccessible. 

To mitigate this risk, it is advisable to adopt a two-step approach when changing privileged roles:
- The current privileged role should propose a new address for the ownership change.
- In a separate transaction, the proposed new address can then claim the privileged role.

**Recommended Mitigation Steps:** Consider implementing a two-step ownership transfer process for the owner of the `TokenUpgrade` smart contract, similar to [OpenZeppelin Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) approach.

**Yield Team:** Acknowledged.

### [L-02]  Insufficient precision when dealing with tokens with different decimals or varying valuations 
**Context:** [TokenUpgrade.sol#L62](https://github.com/yieldprotocol/yield-utils-v2/blob/2569790d2c5c98e0badce1edd8066f38e7f66850/src/token/TokenUpgrade.sol#L62)

**Description:** The `TokenUpgrade` contract aims to facilitate the token upgrade process for users who want to upgrade their old strategy tokens to new strategy tokens at a fixed rate, as defined in the register function.

The fixed rate is calculated using the following formula:
`uint96 ratio = tokenOut_.balanceOf(address(this)).wdiv(tokenIn_.totalSupply()).u96();`.

This calculation determines the ratio based on the old token's supply and the new token's supply, which are distributed through the `TokenUpgrade` contract.

However, problems may arise with this calculation when the old token has more decimals, a higher valuation, or a larger total supply than the new token.

For instance, if we want to trade `10 WBTC` with `$PEPE` that have the same value, the ratio will be 0 and they cannot be swapped with through the `TokenUpgrade` contract.

```bash
➜ uint256 wbtc = (10 * 1e8)
➜ uint256 pepe = (171591194968 * 1e18)
➜ uint256 ratio = wbtc * 1e18 / pepe
➜ ratio
Type: uint
├ Hex: 0x0
└ Decimal: 0
```
**Recommended Mitigation Steps:** It is advisable to document that the ratio calculation with a precision of `1e18` is suitable for tokens with the same number of decimals and similar valuations.

**Yield Team:** We will add a Natspec warning that for extreme cases 18 decimals won't be enough.

**Christos Pap:** Verified. 

## Informational
### [i-01] Prefer `MerkleProof.verifyCalldata` over `MerkleProof.verify` for calldata arguments
**Context:** [TokenUpgrade.sol#L133](https://github.com/yieldprotocol/yield-utils-v2/blob/2569790d2c5c98e0badce1edd8066f38e7f66850/src/token/TokenUpgrade.sol#L133)

**Description:**  The `TokenUpgrade` smart contract uses the [`OpenZeppelin MerkleProof library`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/1642b6639b93e3b97be163d49827e1f56b81ca11/contracts/utils/cryptography/MerkleProof.sol#L20) to verify the Merkle Proofs submitted by users. However, the [`swap`](https://github.com/yieldprotocol/yield-utils-v2/blob/2569790d2c5c98e0badce1edd8066f38e7f66850/src/token/TokenUpgrade.sol#LL125C14-L125C18) function uses the [`verify`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/1642b6639b93e3b97be163d49827e1f56b81ca11/contracts/utils/cryptography/MerkleProof.sol#L27) function for calldata arguments. Starting from version `v4.7` of the `MerkleProof library`, the [`verifyCalldata`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/1642b6639b93e3b97be163d49827e1f56b81ca11/contracts/utils/cryptography/MerkleProof.sol#L36) function is available specifically for dealing with calldata arguments.

**Recommendation** It is recommended to utilize the [`verifyCalldata`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/1642b6639b93e3b97be163d49827e1f56b81ca11/contracts/utils/cryptography/MerkleProof.sol#L36) function instead of the verify function when dealing with `calldata` arguments.


**Yield Team:** Fixed in [commit](https://github.com/yieldprotocol/yield-utils-v2/pull/74/commits/c93837531978413ed7a1b272512abfd237a9ff93).

**Christos Pap:** Verified.

## Additional Notes
-  The `TokenUpgrade` contract does not specify how the new tokens will be used in the strategies implemented in the updated version of the Yield protocol.
- The [`swap`](https://github.com/yieldprotocol/yield-utils-v2/blob/764405dee286f49308262df66e893ee8107ee25b/src/token/TokenUpgrade.sol#L125) function has been renamed to `upgrade` after the security review.
- As a result of the `TokenUpgrade` design and the inclusion of the `unregister` function, the owner of the `TokenUpgrade` contract has the ability to unregister a token and subsequently extract all tokens held by the `TokenUpgrade` contract.
