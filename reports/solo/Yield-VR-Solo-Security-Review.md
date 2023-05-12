# Yield VR Security Review

A security review of the [Yield Variable Rate](https://yieldprotocol.com/) smart contract protocol was done by [Christos Pap](https://twitter.com/christos_eth). \
This security review report includes all the vulnerabilities, issues and code improvements found during the security review.

## Disclaimer

"Audits are a time, resource and expertise bound effort where trained experts evaluate smart
contracts using a combination of automated and manual techniques to find as many vulnerabilities
as possible. Audits can show the presence of vulnerabilities **but not their absence**."

\- Secureum

## About Christos
Christos Pap is an independent security researcher who specializes in Ethereum smart contract security. He currently works as a Junior Security Researcher at Spearbit and is an alumnus of the yAcademy fellowship program. Additionally, he is undergraduate student of Electrical and Computer Engineering and he has extensive experience in offensive security, having worked as a Penetration Tester and holding the OSCP certification. You can connect with him on Twitter at [@christos_eth](https://twitter.com/christos_eth).

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
| Repository    | [https://github.com/yieldprotocol/vault-v2](https://github.com/yieldprotocol/vault-v2)                                              |
| Commit hash   | [1d1602a06fda352f463b6f126c8a90e05e221541](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541) |
| Documentation | [https://docs.yieldprotocol.com/](https://docs.yieldprotocol.com/)                                                                  |
| Methods       | Manual review                                                                                                                       |
|               |

### Issues found

| Severity      | Count |
| :------------ | ----: |
| Critical risk |     0 |
| High risk     |     2 |
| Medium risk   |     1 |
| Low risk      |     3 |
| Informational |     5 |

### Scope

| File                                                                                                                                                                             | nSLOC |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---- |
| Contracts (6)                                                                                                                                                                    |
| [src/variable/VRCauldron.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol)                               | 297   |
| [src/variable/VRLadle.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol)                                     | 210   |
| [src/variable/VRRouter.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRRouter.sol)                                   | 18    |
| [src/variable/VRWitch.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRWitch.sol)                                     | 57    |
| [src/variable/VYToken.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol)                                     | 154   |
| [src/oracles/VariableInterestRateOracle.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol) | 149   |
| Interfaces (2)                                                                                                                                                                   |
| [src/variable/interfaces/IVRCauldron.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/interfaces/IVRCauldron.sol)       | 22    |
| [src/variable/interfaces/IVRWitch.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/interfaces/IVRWitch.sol)             | 78    |
| Total (8)                                                                                                                                                                        | 985   |

# Findings

| ID     | Title                                                                       | Severity      | Resolution   |
| :----- | :-------------------------------------------------------------------------- | :------------ | :----------- |
| [H-01] | Precision loss in the `VariableInterestRateOracle:get` function affects the interest rate of the `Yield Variable Rate` Protocol              | High        | Fixed        |
| [H-02] | Inaccurate refund logic causes incorrect accounting in underlying Join contract | High        | Fixed        |
| [M-01] | `Name`, `Symbol` and `Decimals` will have the default values in the `VyToken`| Medium        | Fixed |
| [L-01] | If the spot oracle goes down, most of the `Yield Variable Rate Protocol`, including liquidations, can be frozen | Low           | Acknowledged        |
| [L-02] | Deviation from `EIP3156` standard may affect composability                  | Low           | - |
| [L-03] | An attacker can force the `Redeemed` event to be emitted with the wrong `holder` argument   | Low | Acknowledged        |
| [I-01] | No allowance front-running mitigation                                       | Informational | Acknowledged        |
| [I-02] | `TransferHelper` library does not verify token code size before executing transfers                                                     | Informational | Acknowledged        |
| [I-03] | Lack of input validation in function parameters                              | Informational | Acknowledged        |
| [I-04] | Typographical errors and missing data in comments & NatSpec docs            | Informational | Fixed        |
| [I-05] | Function ordering does not follow the Solidity style guide                  | Informational | Acknowledged        |


## High severity
### [H-01] Precision loss in the `VariableInterestRateOracle:get` function affects the interest rate of the Yield Variable Rate Protocol
**Context:** [VariableInterestRateOracle.sol#L200-L204](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L200-L204), [VariableInterestRateOracle.sol#L206-L212](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L206-L212), [VariableInterestRateOracle.sol#L194-L196](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L194-L196)

**Description:** The interest rate calculation formula used in the code is based on the formula used by [AAVE](https://docs.aave.com/risk/liquidity-risk/borrow-interest-rate#interest-rate-model).

If [`utilizationRate <= rateParameters.optimalUsageRate`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L199), the interest rate is calculated as:

```solidity
interestRate = rateParameters.baseVariableBorrowRate +
 utilizationRate.wmul(rateParameters.slope1).wdiv(rateParameters.optimalUsageRate
```
However, since [`wmul`](https://github.com/yieldprotocol/yield-utils-v2/blob/3f0d8528686aa2e671077018c9616d212818ad60/src/utils/Math.sol#L7) and [`wdiv`](https://github.com/yieldprotocol/yield-utils-v2/blob/3f0d8528686aa2e671077018c9616d212818ad60/src/utils/Math.sol#L25) function are used together, the calculation is performed as follows:
`interestRate = rateParameters + utilizationRate * rateParameters.slope1 / 1e18 * 1e18 / rateParameters.optimalUsageRate`.

This leads to precision loss, since a division (`/ 1e18`) is performed _before_ the multiplication (`* 1e18`). 

A similar issue occurs also when [`utilizationRate > rateParameters.optimalUsageRate`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L205).

**Recommended Mitigation Steps:** To avoid precision loss in the `get` function, it is recommended to adjust the calculations as follows:
`interestRate = rateParameters.baseVariableBorrowRate + utilizationRate * rateParameters.slope1 / rateParameters.optimalUsageRate;`

**Yield Team:** We have now removed the usage of the [`wmul`](https://github.com/yieldprotocol/yield-utils-v2/blob/3f0d8528686aa2e671077018c9616d212818ad60/src/utils/Math.sol#L7) and [`wdiv`](https://github.com/yieldprotocol/yield-utils-v2/blob/3f0d8528686aa2e671077018c9616d212818ad60/src/utils/Math.sol#L25) functions.

**Christos Pap:** Verified.

### [H-02] Inaccurate refund logic causes incorrect accounting in underlying Join contract
**Context:** [VRLadle.sol#L381](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L381), [Join.sol#L44](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/Join.sol#L44)

**Description:** The [`repay`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#LL367C14-L367C19) function in the `VRLadle` contract can be used by a user to repay all the debt in a vault. According to the function comments, the surplus base will be returned to `msg.sender`. 

However, the refund logic in the [`repay`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L367) function is flawed. The user is refunded while the [`storedBalance`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/Join.sol#L51) from the underlying `Join` is reduced, leading to broken accounting in the `Join` contract because the [`storedBalance`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/Join.sol#L51) will get smaller when the surplus funds are refunded.

An attacker could potentially exploit this issue by sending a large amount to the Join contract, then calling the [repay](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L367) function, causing the [storedBalance](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/Join.sol#L51) to decrease significantly.

The [`exit`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/Join.sol#L44) function of the `Join` contract:
```solidity
    /// @dev Transfer `amount` `asset` to `user`
    function exit(address user, uint128 amount) external virtual override auth returns (uint128) {
        return _exit(user, amount);
    }

    /// @dev Transfer `amount` `asset` to `user`
    function _exit(address user, uint128 amount) internal virtual returns (uint128) {
        IERC20 token = IERC20(asset);
        storedBalance -= amount;
        token.safeTransfer(user, amount);
        return amount;
    }
```

**Recommended Mitigation Steps:** To address this issue, it is recommended to add a new function to the Join contract that retrieves the _unaccounted amount_. This function should be called instead of the `exit` function to ensure that the correct accounting is maintained.

**Yield Team:** Good catch. I think the Join needs a skim function that allows to take the difference between the actual balance and the stored balance, similar to [this](https://github.com/yieldprotocol/yieldspace-tv/blob/5dd1216cbac3e492defd4473a04d02b0c6d991e2/src/Pool/Pool.sol#L1477).

**Christos Pap:** Verified. The issue was fixed by introducing a `skim` function that  retrieves the _unaccounted amount_.

## Medium severity
### [M-01] `Name`, `Symbol` and `Decimals` will have the default values in the `VyToken`

**Context:** [VYToken.sol#L41](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L41)

**Description:** The [yield-utils-v2 ERC20](https://github.com/yieldprotocol/yield-utils-v2/blob/main/src/token/ERC20.sol) implementation does not declare the _decimals_, _string_, and _symbol_ as immutables. As the [`VYToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16) contract is intended to be upgradeable, the constructor code of the ERC20 implementation will not be executed during initialization, resulting in default values. Consequently, [`VYToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16) has 0 decimals, which differs from the underlying token.

Due to a requirement of the proxy-based upgradeability system, no [`constructors`](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers) can be used in upgradeable contracts. Immutable will work, since they aren't stored in the storage, and the compiler will place them into the bytecode in the deployment. 

If we ran the following snippet we can see that the [`VYToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L41) has `0` decimals, which is different than the intended behaviour.

```solidity
  function testDecimals() public {
        console.log("underlying is:", vyToken.underlying());
        console.log(vyToken.decimals());
        console.log("Name is", vyToken.name());
 }
```

**Recommended Mitigation Steps:** It is recommended to also mark the _decimals_, _string_, and _symbol_ as immutables in the [yield-utils-v2 ERC20](https://github.com/yieldprotocol/yield-utils-v2/blob/main/src/token/ERC20.sol) token. Alternatively, the [`initialize` function](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L54) could be used to set those values.

**Yield Team:** Fixed by using the `initialize()` function to set the  _decimals_, _string_, and _symbol_ variables.

**Christos Pap:** Verified.

## Low severity
### [L-01]  If the spot oracle goes down, most of the `Yield Variable Rate Protocol`, including liquidations, can be frozen
**Severity:** Low

**Context:** [VRCauldron.sol#L139](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L139), [ChainlinkMultiOracle.sol#L16](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/chainlink/ChainlinkMultiOracle.sol#L16)

**Description:** The  [`setSpotOracle`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L139) function in the `VRCauldron` contract sets an instance of the  [`ChainlinkMultiOracle` contract](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/chainlink/ChainlinkMultiOracle.sol#L16) contract as the `IOracle`. However, the [`ChainlinkMultiOracle` contract](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/chainlink/ChainlinkMultiOracle.sol#L16) uses a single oracle to obtain the latest price feed.

According to OpenZeppelin's [Smart Contract Security Guidelines #3: The Dangers of Price Oracles](https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles/),
> While currently there’s no whitelisting mechanism to allow or disallow contracts from reading prices, powerful multisigs can tighten these access controls. In other words, the multisigs can immediately block access to price feeds at will. Therefore, to prevent denial of service scenarios, it is recommended to query ChainLink price feeds using a defensive approach with Solidity’s try/catch [structure](https://docs.soliditylang.org/en/latest/control-structures.html#try-catch). In this way, if the call to the price feed fails, the caller contract is still in control and can handle any errors safely and explicitly.

Chainlink has taken oracles offline in extreme cases, such as during the UST collapse when [it paused the UST/ETH price oracle](https://ambcrypto.com/chainlink-how-a-price-discrepancy-resulted-in-millions-lost-from-defi-protocols/) to prevent protocols from receiving inaccurate data.


**Recommended Mitigation Steps:** To protect against the possibility of access to Chainlink feeds being denied, it is recommended to implement a safeguard such as a fallback oracle or an alternative approach that can act when needed.

**Yield Team:** In extreme cases a proposal could be executed to change the `spotOracle`. Since it is a risk in an extreme case we would stick with the current mitigation methodology.

**Christos Pap:** Acknowledged.

### [L-02]  Deviation from `EIP3156` standard may affect composability
**Context:** [VYToken.sol#L248](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L248), [VYToken.sol#L181-L195](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L181-L195)

**Description:** [`VYToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16) isn't fully compatible with the [`EIP3156`](https://eips.ethereum.org/EIPS/eip-3156) standard. 

According to the [`Lender Specification`](https://eips.ethereum.org/EIPS/eip-3156#lender-specification), it's noted that:

> After the callback, the flashLoan function MUST take the amount + fee token from the receiver, or revert if this is not successful.

However, due to the custom [`_burn`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L181-L195) function in the `VYToken` contract, the amount is taken from the contract instead, if there are some funds inside it.

```solidity
function _burn(address holder, uint256 principalAmount) internal override returns (bool) {
        // First use any tokens locked in this contract
        uint256 available = _balanceOf[address(this)];
        if (available >= principalAmount) {
            return super._burn(address(this), principalAmount);
        } else {
            if (available > 0) super._burn(address(this), available);
            unchecked {
                _decreaseAllowance(holder, principalAmount - available);
            }
            unchecked {
                return super._burn(holder, principalAmount - available);
            }
        }
    }
``` 

**Recommended Mitigation Steps:** To make the `VYToken` contract fully compatible with the [EIP3156 standard](https://eips.ethereum.org/EIPS/eip-3156), it is recommended to remove the custom `_burn` function and to modify the `VYToken` accordingly.

**Yield Team:** Hi, you are right, this is a deviation from the standard as written. However, when I wrote it I intended that this would be allowed (same as is in 4626).

Meaning that if the borrower approves the repayment, that should always work. However, if the lender also has an alternative repayment method and the borrower decides to use it, there should be no impediment.

Thanks for raising it, I'll revise the wording in 3156 instead of changing the code here.

**Christos Pap:** Verified.

### [L-03] An attacker can force the Redeemed event to be emitted with the wrong holder argument

**Context:** [VYToken.sol#L111](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L111),  [VYToken.sol#L143](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L143)

**Description:** The custom [`_burn`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L181) function allows a user to transfer `vyToken` to the contract to enable a `burn`, potentially saving the cost of `approve` or `permit`. This is achieved by using any tokens that are present in the `VyToken` contract.

However, this functionality can be exploited by an attacker because the [`withdraw`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L137) and [`redeem`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L105) functions in the contract allow a user to input a `holder` address as an argument. An attacker can send tokens directly to the contract and then pass any address as the holder argument in the [`withdraw`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L137) /[`redeem`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L105) function, which forces that address to be emitted at the [`Redeemed`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L22) event. 

**Recommended Mitigation Steps:** It is difficult to mitigate this issue with the current setup. A possible solution could be to check the approval in the `withdaw`/`redeem` function. 

**Yield Team:** Our users are instructed to strictly use the frontend. So, we are not going to change this.

**Christos Pap:** Acknowledged.

## Informational
### [i-01] No allowance front-running mitigation
**Context:** [VYToken.sol#L16](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16)

**Description:**  The [`VYToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16) contract is inherited from [`ERC20Permit`](https://github.com/yieldprotocol/yield-utils-v2/blob/3f0d8528686aa2e671077018c9616d212818ad60/src/token/ERC20Permit.sol#L16), which, in turn, is inherited from the [`ERC20`](https://github.com/yieldprotocol/yield-utils-v2/blob/3f0d8528686aa2e671077018c9616d212818ad60/src/token/ERC20.sol#L28) contract. These contracts are part of the [`yield-utils-v2`](https://github.com/yieldprotocol/yield-utils-v2/tree/main) GitHub repository.

None of these contracts provide protection against the [`allowance front-running attack`](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit). This attack can occur when a token owner authorizes another account to transfer a specific amount of tokens on their behalf. If the token owner decides to change the allowance amount, the spender can spend both allowances by front-running the allowance-changing transaction.

**Recommendation** To mitigate this issue, you could consider using the [`OpenZeppelin ERC20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/8d633cb7d169f2f8595b273660b00b69e845c2fe/contracts/token/ERC20/ERC20.sol#L38) implementation. This implementation includes the [increaseAllowance](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/8d633cb7d169f2f8595b273660b00b69e845c2fe/contracts/token/ERC20/ERC20.sol#L177) and [decreaseAllowance](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/8d633cb7d169f2f8595b273660b00b69e845c2fe/contracts/token/ERC20/ERC20.sol#L197) functions, which protect against the allowance front-running attack. Although an attacker may not have any incentive to perform this attack, adding these functions can be seen as an added layer of protection.

**Yield Team:** We aknowledge this one and we prefer to not fix it.

### [i-02] `TransferHelper` library does not verify token code size before executing transfers
**Context:** [TransferHelper.sol](https://github.com/yieldprotocol/yield-utils-v2/blob/main/src/token/TransferHelper.sol)

**Description:** The [`TransferHelper`](https://github.com/yieldprotocol/yield-utils-v2/blob/main/src/token/TransferHelper.sol) library fails to verify whether a token has code before executing a token transfer. Consequently, if the token address is empty, the transfer will succeed without crediting the contract with any tokens. While the tokens added to the VRLadle are **whitelisted**, this can still disrupt internal accounting in the following ways:
- If a token is self-destructed, `.transferFrom` and `.transfer` will succeed, breaking the internal accounting.
- If the `Yield Variable Rate` is deployed on multiple chains, such as on `Mainnet` and on `Arbitrum`, and an admin mistakenly adds a token to `Arbitrum` that does not exist only exists on Mainnet, it could create an opportunity for a honeypot attack similar to the [`Qubit Finance hack`](https://halborn.com/explained-the-qubit-hack-january-2022/).


**Recommended Mitigation Steps:** To mitigate this issue, it is recommended to check if the contract has code before executing `.transfer` and `.transferFrom` functions. OpenZeppelin's [`SafeERC20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ab2604ac5b791adf3c5e2397e65128cb56954edd/contracts/token/ERC20/utils/SafeERC20.sol#L19) library verifies this condition [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ab2604ac5b791adf3c5e2397e65128cb56954edd/contracts/utils/Address.sol#L205) and can be used as a reference.

**Yield Team:** 
> If a token is self-destructed, `.transferFrom` and `.transfer` will succeed, breaking the internal accounting.

The same and worse effects can happen if one of the tokens is upgraded, which is the same kind of attack vector but is much more likely.

> If the `Yield Variable Rate` is deployed on multiple chains, such as on `Mainnet` and on `Arbitrum`, and an admin mistakenly adds a token to `Arbitrum` that does not exist only exists on Mainnet, it could create an opportunity for a honeypot attack similar to the [`Qubit Finance hack`](https://halborn.com/explained-the-qubit-hack-january-2022/).

With regards to all governance errors, we prefer to use a [robust governance review process](https://hackernoon.com/how-to-review-a-governance-action) as it offers a more comprehensive protection.

**Christos Pap:** Acknowledged.

### [i-03] Lack of input validation in function parameters
**Context:** [VRCauldron.sol#L114](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L114), [VRCauldron.sol#L149](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L149), [VYToken.sol#L66](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L66)

**Description:** Some functions of the Variable Rate Yield project lack threshold checks, which could lead to unexpected behaviour in certain situations. 

**Recommended Mitigation Steps:** To address these issues, it is recommended to apply the following mitigations:
- In the [`setDebtLimits`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L114) you could add a require statement that `min > max` or `min >= max`.
- In the [`setSpotOracle`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L149) function could add a require statement that `ratio <= 1000000`.
- In the [`setFlashFeeFactor`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L66) function a threshold could be added as an upper bound for the `fee` value.

**Yield Team:** We rely on our mature governance process to prevent issues for the above.

**Christos Pap:** Acknowledged.

### [i-04] Typographical errors and missing data in comments & NatSpec docs
**Context:** [VRCauldron.sol#L110](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L110), [VRLadle.sol#L220](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L220), [VRLadle.sol#L111](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L111), [VariableInterestRateOracle.sol#L12](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L12), [VYToken.sol#L20](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L20), [VYToken.sol#L21](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L21), [VRLadle.sol#L366](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L366)

**Description:** During the audit, several typographical errors and missing data in comments and NatSpec documentation have been identified. Please refer to the `Recommended Mitigation Steps` section for the detailed list.

**Recommended Mitigation Steps:** 
- The `address -> address` casting is unnecessary in [VRCauldron.sol#L110](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L110).
- Typo in [VRLadle.sol#L220](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L220): `implemnting` should be `implementing`.
- Since the [`addToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L104) can also be used to remove a token you could consider renaming the [`TokenAdded`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#LL111C14-L111C24) event to `TokenStatusChanged`.
- The [VariableInterestRateOracle.sol](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/oracles/VariableInterestRateOracle.sol#L12) contract is missing `NatSpec documentation`.
- The [`Point`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L20) event isn't used in the [`VYToken`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16). You could consider removing it.
- You could consider also emitting also the old fee value in the [FlashFeeFactorSet](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L21) event.
- The [`comment`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L366) in the [`repay`](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L367) function is wrong. Consider replacing it with: `The surplus base will be returned to the refundTo address, if refundTo is different than address(0)`.

**Yield Team:** Fixed.

**Christos Pap:** Verified.

### [i-05] Function ordering does not follow the Solidity style guide
**Context:** [VRWitch.sol#L15](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRWitch.sol#L15), [VRCauldron.sol#L13](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRCauldron.sol#L13), [VRLadle.sol#L21](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L21), [VYToken.sol#L16](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VYToken.sol#L16)

**Description:** The recommended order of functions in Solidity, as outlined in the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), is as follows: `constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal` and `private`. However, this ordering isn't enforced through the [`Variable Rate`](https://github.com/yieldprotocol/vault-v2/tree/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable) codebase.

**Recommended Mitigation Steps:** It is recommended to follow the recommended order of functions in Solidity, as outlined in the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions).

**Yield Team:**  We are going to stick to our current style.

**Christos Pap:** Acknowledged.

## Additional Notes
- During the security review, it was discovered that some sections [of the code](https://github.com/yieldprotocol/vault-v2/blob/1d1602a06fda352f463b6f126c8a90e05e221541/src/variable/VRLadle.sol#L340) did not follow the Check Effects Interaction (CEI) pattern. These functions do not have the protection of the [`nonReentrant()` modifier](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ab2604ac5b791adf3c5e2397e65128cb56954edd/contracts/security/ReentrancyGuard.sol#LL50C5-L50C28). If any [`ERC777`](https://eips.ethereum.org/EIPS/eip-777) tokens are supported, a re-entrancy (either a same-function or a cross-function) could be introduced.
The protocol team responded:
> Tokens are supported in a case-by-case basis. We haven't supported any erc777 tokens so far, and have no immediate plans to do so. If we would, we would investigate the consequences at that time.
