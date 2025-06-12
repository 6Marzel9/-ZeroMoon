# ZeroMoon Token

![ZeroMoon Logo](zeromoon.png)

---

## Table of Contents
- [Introduction](#introduction)
- [Key Features](#key-features)
- [Tokenomics](#tokenomics)
- [Safety Mechanisms](#safety-mechanisms)
- [Contract Overview](#contract-overview)
- [List of All Functions](#list-of-all-functions)
- [Reflection (Auto-Staking)](#reflection)
- [Burning](#burning)
- [Automated Liquidity](#automated-liquidity)
- [Developer Support: Reduced Impact and Community Sharing](#developer-support)
- [Conclusion](#conclusion)
- [Community and Governance](#community-and-governance)
- [Roadmap and Future Developments](#roadmap-and-future-developments)
- [License](#license)

---

## Introduction
ZeroMoon (0Moon) is an advanced reflection (auto-staking) token deployed on the Binance Smart Chain (BSC). It builds upon foundational concepts from earlier reflection tokens but is re-engineered with a primary focus on enhanced safety, programmatic autonomy, and robust scalability. ZeroMoon introduces automated liquidity management, a dynamic fee system that adapts to ecosystem milestones, and a secure, decentralized framework designed to minimize centralized vulnerabilities.

The ZeroMoon contract is optimized for scalability, primarily utilizing constant-time O(1) operations for critical functions like reflections and fee distribution. This design ensures that gas costs remain low and predictable, allowing the system to efficiently support a very large number of users and transactions.

---

## Key Features
- **Dynamic Fee Structure:** Transaction fees automatically adjust based on the achievement of burn and liquidity provision milestones, ensuring a balanced approach to rewards, deflation, and long-term sustainability.
- **Automated Liquidity Addition:** The contract programmatically collects a portion of fees to add liquidity to a PancakeSwap V2 pair when defined thresholds are met, fostering a stable and healthy trading ecosystem. LP tokens generated are sent to a dead address, locking the liquidity.
- **Burn and Liquidity Caps:**
    - **Burn Limit**: Token burning from fees ceases once 25% of the original total supply has been burned.
    - **Liquidity Deposit Limit**: Automated liquidity addition from fees stops once 50% of the original total supply has been designated for or added to the liquidity pool.
- **Scalability:** Designed with O(1) time complexity for core operations like reflection distribution, enabling efficient performance even with a large and active user base.
- **Programmatically Managed Assets:** The contract holds 0Moon tokens (for liquidity/reflection) and BNB (for gas). These assets are managed by the smart contract's predefined logic for their intended purposes (e.g., adding liquidity, fee distribution, sending to `boxAddress` under specific conditions) and are not arbitrarily drainable by the owner or external parties through direct withdrawal functions.

---

## Tokenomics
ZeroMoon’s tokenomics are meticulously designed to balance holder rewards, token scarcity through burning, and the overall sustainability of the ecosystem.

- **Token Name**: ZeroMoon
- **Token Symbol**: 0Moon <!-- CRITICAL: Ensure ERC20 constructor is ERC20("ZeroMoon", "0Moon") -->
- **Total Supply (`ORIGINAL_SUPPLY`)**: 100,000,000,000 0Moon (100 Billion 0Moon)
  - Defined in contract as `ORIGINAL_SUPPLY = 100_000_000_000 * 10**18`.

- **Burn Mechanism (`BURN_LIMIT`)**: Deflationary burning of tokens from transaction fees continues until 25,000,000,000 0Moon (25% of `ORIGINAL_SUPPLY`) are removed from circulation.
  - Defined in contract as `BURN_LIMIT = ORIGINAL_SUPPLY * 25 / 100`.

- **Transaction Fees (Dynamic Phases)**:
  *(Fees are percentages of the transaction amount, denominator is 10000)*
  - **Phase 1: Before Burn Limit Reached (Total Fee: 10%)**:
    - **Reflection**: 3.5% (`REFLECTION_B4_BST = 350`)
    - **Liquidity**: 3.0% (`LIQUIDITY_B4_BST = 300`)
    - **Burn**: 2.5% (`BURN_B4_BST = 250`)
    - **Developer**: 1.0% (`DEV_B4_BST = 100`)

  - **Phase 2: After Burn Limit, Before LP Limit (Total Fee: 10%)**:
    - **Reflection**: 4.5% (`REFLECTION_B4_LPS = 450`)
    - **Liquidity**: 4.75% (`LIQUIDITY_B4_LPS = 475`)
    - **Burn**: 0% (Burn fee stops)
    - **Developer**: 0.75% (`DEV_B4_LPS = 75`)

  - **Phase 3: After Both Burn & LP Limits Reached (Total Fee: 6%)**:
    - **Reflection**: 5.5% (`REFLECTION_AFTER = 550`)
    - **Liquidity**: 0% (Liquidity fee stops)
    - **Burn**: 0% (Burn fee remains stopped)
    - **Developer**: 0.5% (`DEV_AFTER = 50`)

- **LP Deposit Limit (`LP_DEPOSIT_LIMIT`)**: Automated liquidity addition from fees ceases after 50,000,000,000 0Moon (50% of `ORIGINAL_SUPPLY`) have been cumulatively designated for or added to the PancakeSwap liquidity pool.
  - Defined in contract as `LP_DEPOSIT_LIMIT = ORIGINAL_SUPPLY * 50 / 100`.

- **Reflection (Auto-Staking) Eligibility**: All holders are eligible for reflections provided they are not explicitly excluded (e.g., contract addresses, dead wallet, LP pair). There are no minimum token holding requirements to receive reflections.

---

## Safety Mechanisms
The ZeroMoon contract incorporates several design choices to enhance security and mitigate risks:

- **Ownership Renouncement (Planned)**: Post-launch, contract ownership is intended to be renounced. This action would make owner-restricted functions (`onlyOwner`) permanently inaccessible, moving the contract towards greater decentralization and immutability for those functions.
- **Immutable Addresses**: Critical infrastructure addresses such as the `pancakeSwapV2Pair`, `deadWallet`, primary `devWallet`, and `boxAddress` are set at deployment and are immutable, preventing malicious changes.
- **Reentrancy Protection**: Utilizes OpenZeppelin's `ReentrancyGuard` modifier (`nonReentrant`) on key functions involved in external calls or state changes related to liquidity operations (`executeLpFromPoke`, `triggerAutoLiquidity`, `_addLiquidityAutomatically`), protecting against reentrancy attacks.
- **Locked Liquidity**: LP tokens generated from automated liquidity additions are sent directly to the `deadWallet`. This effectively burns the LP tokens, permanently locking the underlying 0Moon and BNB in the PancakeSwap liquidity pool and preventing a "rug pull" of this contract-generated liquidity.
- **No Arbitrary Withdrawals**: The contract does not contain functions allowing the owner or any other address to arbitrarily withdraw 0Moon tokens or BNB held within it. Asset movements are governed by the predefined programmatic logic of the tokenomics (fee distribution, liquidity addition, specific transfers to `boxAddress`).

---

## Contract Overview
- **Contract address**: [0xeb92bc0fd1af01d156142ae379004ed681b5c34c](https://testnet.bscscan.com/address/0xeb92bc0fd1af01d156142ae379004ed681b5c34c)
- **Contract Name**: `ZeroMoon`
- **SPDX License**: MIT
- **Solidity Version**: 0.8.30
- **Inheritance**: `ZeroMoon` inherits from:
  - `ERC20` (OpenZeppelin): Provides standard ERC20 token functionality.
  - `Ownable` (OpenZeppelin): Implements ownership control for administrative functions.
  - `ReentrancyGuard` (OpenZeppelin): Provides protection against reentrancy attacks.

---

## List of All Functions

*(This list summarizes key functions. For full details, refer to the contract code and NatSpec comments.)*

**1. Constructor**
- **Function**: `constructor`
  - **Visibility**: Deployment-time only.
  - **Modifiers**: `payable`
  - **Parameters**: `_devWallet`, `_pancakeSwapRouterAddress`, `_pancakeSwapFactoryAddress`, `_deadWalletAddress`, `_boxAddress`.
  - **Purpose**: Initializes the contract, sets up immutable addresses, creates the PancakeSwap pair, mints `ORIGINAL_SUPPLY` to the deployer, and sets initial exclusions.

**2. Public View Functions (Data Retrieval)**
- `name()`, `symbol()`, `decimals()`, `totalSupply()`, `balanceOf(address)`, `allowance(address, address)`: Standard ERC20 view functions. `totalSupply()` returns `ORIGINAL_SUPPLY - _burnedTokens`. `balanceOf(address)` accounts for reflections.
- `isContract(address _addr)`: Checks if an address has deployed code.
- `get0MoonPerBNB()`: Returns the current 0Moon price per 1 BNB from PancakeSwap.
- `getBNBPer0Moon()`: Returns the current BNB price per 1 0Moon from PancakeSwap.
- `getAccumulatedLiquidityTokens()`: Shows tokens currently held by the contract for LP addition.
- `getTotalLpDeposited()`: Shows total 0Moon tokens deposited into LP by the contract.
- `owner()`: Returns the current contract owner.
- `echoPingerAddress()`: Returns the configured EchoPinger address.
- `holdersCount()`: Returns the current number of token holders.
- `_burnStop()`, `_lpStop()`: Public boolean flags indicating status of burn/LP mechanisms.

**3. Public State-Changing Functions (Transactions)**
- `transfer(address, uint256)`, `transferFrom(address, address, uint256)`, `approve(address, uint256)`, `increaseAllowance(address, uint256)`, `decreaseAllowance(address, uint256)`: Standard ERC20 functions. `transfer` and `transferFrom` trigger fee logic.
- `triggerAutoLiquidity()`: `external nonReentrant`. Allows any user to trigger liquidity addition if conditions are met.

**4. Owner-Restricted Functions (`onlyOwner`)**
- `setEchoPingerAddress(address _pingerAddress)`: Sets the authorized EchoPinger contract address.
- `setDevWallets(address[] memory wallets)`: Sets/updates the array of 3 developer wallets for fee splitting.
- `renounceOwnership()`: Allows the current owner to renounce ownership.
- `transferOwnership(address newOwner)`: Allows the current owner to transfer ownership.

**5. External Authorized Functions**
- `executeLpFromPoke()`: `external nonReentrant`. Callable only by `echoPingerAddress`. Triggers liquidity processing and maintenance.

**6. Internal Core Logic Functions (Private/Internal)**
- `_update(address sender, address recipient, uint256 amount)`: Core internal function overriding ERC20, handles all token movements, fee application, reflection accounting, and holder counts.
- `_calculateFees(uint256 amount)`: Calculates fee breakdown based on current contract phase.
- `_distributeFees(...)`: Distributes collected fees to reflection, liquidity, burn, and dev; triggers `_applyReflectionTokens` and `_addLiquidityAutomatically`.
- `_applyReflectionTokens()`: Adjusts `_scalingFactor` to distribute reflection rewards.
- `_addLiquidityAutomatically()`: `private nonReentrant`. Swaps tokens for BNB and adds to LP.
- `handleLpAddFailure(uint256 tokensIntendedForLp)`: Manages repeated LP addition failures.
- `_sendAccumulatedLpTokensToBox()`: Sends leftover `_accumulatedLiquidityTokens` to `boxAddress` under specific conditions.

**7. Fallback Function**
- `receive() external payable`: Allows the contract to receive BNB directly, emitting `BNBReceived`.

---

## Reflection (Auto-Staking)

**Process**
Reflection, or auto-staking, is ZeroMoon's mechanism for rewarding holders. A portion of every transaction fee is allocated to reflections. Instead of direct token transfers to each holder (which is gas-intensive), ZeroMoon uses a sophisticated `_scalingFactor` that is adjusted when reflections are processed. This effectively increases the value of tokens held by eligible participants over time.

**How It Works**

- **Fee Collection**: Reflection fees (3.5%, 4.5%, or 5.5% depending on the contract phase) are taken from transactions and added to `_accumulatedReflectionTokens`.
    ```solidity
    // In _distributeFees, after fees are calculated:
    if (reflectionAmount != 0) {
        _accumulatedReflectionTokens += reflectionAmount;
        emit ReflectionShared(contractAddress, reflectionAmount); // contractAddress is the source of pooled reflection
    }
    ```
- **Distribution via Scaling Factor**: The `_applyReflectionTokens` function is called (typically from `_distributeFees`) to process these accumulated tokens:
    ```solidity
    // Simplified logic from _applyReflectionTokens:
    // uint256 S = _totalScaledBalance / _scalingFactor; // Total actual tokens of participants
    // uint256 newScalingFactor = (_scalingFactor * S) / (S + reflectionPool);
    // _scalingFactor = newScalingFactor;
    // _accumulatedReflectionTokens = 0;
    ```
  A decrease in `_scalingFactor` means that when a user's balance is calculated (`_userAccountData[user].scaledBalance / _scalingFactor`), their effective token count increases.
- **Balance Calculation**: The public `balanceOf(address account)` function reflects these rewards:
    ```solidity
    // Simplified from balanceOf:
    // if (account is excluded from reflection) return _userAccountData[account].scaledBalance;
    // else return _userAccountData[account].scaledBalance / _scalingFactor;
    ```
- **Exclusions**: The contract itself, the `deadWallet`, the `pancakeSwapV2Pair`, and other smart contracts (auto-detected via `isContract()`) are typically excluded from receiving reflections. This ensures rewards are concentrated among actual users (EOAs), maximizing their share and preventing dilution. This user-centric approach is a key design principle.
- **Event**: `ReflectionShared` event logs additions to the reflection pool.

**Meaning for Users**
- **Passive Earnings**: Hold 0Moon and automatically earn more from transaction activity.
- **Increasing Reward Rates**: As the project matures (burn and LP limits reached), the reflection fee percentage increases, enhancing rewards.
- **Fairness**: By excluding most contract addresses, reflections primarily benefit individual holders.
- **Efficiency**: The O(1) scaling factor method is gas-efficient and highly scalable.

---

## Burning

**Process**
ZeroMoon incorporates a token burn mechanism to create deflationary pressure. A portion of transaction fees is designated as `burnAmount`. These tokens are effectively removed from circulation by allocating them to the `deadWallet`'s balance until the `BURN_LIMIT` (25% of `ORIGINAL_SUPPLY`) is reached.

**How It Works**
- **Fee Collection**: The burn fee (2.5% before `_burnStop`) is calculated in `_calculateFees`.
- **Execution**: In `_distributeFees`:
    ```solidity
    if (!_burnStop && burnAmount != 0) {
        _userAccountData[deadWallet].scaledBalance += burnAmount; // deadWallet is excluded, so this is actual balance
        _burnedTokens += burnAmount;
        emit Transfer(contractAddress, deadWallet, burnAmount); // Transfer to the deadWallet

        if (_burnedTokens >= BURN_LIMIT) {
            _burnStop = true;
        }
    }
    ```
- **Tracking**: `_burnedTokens` tracks cumulative burns. The `deadWallet`'s balance, queryable via `balanceOf(deadWallet)`, also reflects total tokens burned this way.
- **Event**: `Transfer` event with `deadWallet` as the recipient logs the burn.
- **Post-Burn Limit**: Once `_burnStop` is true, the burn fee becomes 0%, and those fee points are reallocated to other components like reflection and liquidity.

**Meaning for Users**
- **Increased Scarcity**: Burning reduces the circulating supply, which can positively impact token value.
- **Defined Limit**: The `BURN_LIMIT` ensures a predictable end to the burn phase.
- **Transparency**: Burned amounts are publicly auditable via `_burnedTokens` and `Transfer` events to the `deadWallet`.

---

## Automated Liquidity

**Process**
To ensure a robust and deep trading pool on PancakeSwap, ZeroMoon automatically adds liquidity using a portion of transaction fees. These fees are collected in `_accumulatedLiquidityTokens`. When thresholds are met (sufficient tokens accumulated, sufficient BNB in the contract for gas), the contract swaps half these tokens for BNB and pairs them with the remaining half to add liquidity. This process continues until `LP_DEPOSIT_LIMIT` (50% of `ORIGINAL_SUPPLY`) is reached.

**How It Works**
- **Fee Collection**: Liquidity fees (3.0% or 4.75% depending on phase) are added to `_accumulatedLiquidityTokens` and the contract's own token balance (`_userAccountData[contractAddress].scaledBalance`).
- **Trigger Conditions**: Liquidity addition is triggered from `_distributeFees` (or `executeLpFromPoke`/`triggerAutoLiquidity`) if:
    - `_accumulatedLiquidityTokens >= MIN_TOKENS_TO_PROCESS_PER_CYCLE` (currently 10 million tokens with 18 decimals).
    - `contractAddress.balance >= MIN_BNB_BALANCE` (currently 0.001 BNB).
    - `_totalLpDeposited < LP_DEPOSIT_LIMIT`.
- **Execution (`_addLiquidityAutomatically`)**:
    1.  Calculates `tokensToProcessThisCycle`, respecting `MAX_LIQUIDIFY_PERCENT_OF_RESERVES_BPS` and `ABSOLUTE_MAX_TOKENS_TO_PROCESS_PER_CYCLE`.
    2.  Adjusts amounts if nearing `LP_DEPOSIT_LIMIT` to add exactly the remaining capacity.
    3.  Approves PancakeSwap Router to spend tokens.
    4.  Swaps half (`swapAmount`) for BNB using `pancakeSwapV2Router.swapExactTokensForETH(...)`.
    5.  Adds the received BNB and the other half of tokens (`lpTokenAmount`) to liquidity using `pancakeSwapV2Router.addLiquidityETH{value: bnbReceived}(...)`.
    6.  **LP tokens received from `addLiquidityETH` are sent to the `deadWallet`, permanently locking this liquidity.**
    7.  Updates `_accumulatedLiquidityTokens` and `_totalLpDeposited`.
    8.  Handles potential failures in swap or LP addition, with retries and fallback mechanisms (see `handleLpAddFailure`).
- **Events**: `LiquidityAdded` on success, `LiquidityAdditionFailed` on failure.
- **Post-LP Limit**: Once `_lpStop` is true, the liquidity fee becomes 0%.

**Meaning for Users**
- **Improved Trading Stability**: A growing liquidity pool reduces price slippage and supports larger trades.
- **Locked Liquidity**: Sending LP tokens to the `deadWallet` is a critical safety feature, preventing rug pulls of contract-generated liquidity.
- **Sustainable Growth**: Liquidity is built organically from transaction volume.
- **Transparency**: Operations are logged via events.

---

## Developer Support: Reduced Impact and Community Sharing

**Process**
A portion of transaction fees is allocated to developer wallet(s) to fund ongoing project maintenance, innovation, and marketing. ZeroMoon's approach aims for sustainability while demonstrating a commitment to the community by reducing the developer fee share over time and redirecting a portion of that to enhance holder reflections.

**How It Works**
- **Fee Rates**:
    - Phase 1 (Before Burn Stop): 1.0% (`DEV_B4_BST`)
    - Phase 2 (After Burn Stop, Before LP Stop): 0.75% (`DEV_B4_LPS`)
    - Phase 3 (After Both Stops): 0.5% (`DEV_AFTER`)
- **Distribution**: Fees are sent to the primary `devWallet` or split among the `devWallets` array if configured.
- **Sharing Mechanism**: The developer fee reduces from an initial 1.0% to a final 0.5%. This 0.5% reduction is effectively reallocated, contributing to the increase in the reflection fee percentage as the contract matures.
    - Reflection starts at 3.5%.
    - After burn stop, reflection becomes 4.5% (Dev fee reduced by 0.25%, Burn fee (2.5%) also reallocated).
    - After LP stop, reflection becomes 5.5% (Dev fee reduced by another 0.25%, Liquidity fee (4.75%) also reallocated).
    - In total, the developer fee's 0.5% reduction directly contributes to the overall 2% increase in the reflection rate enjoyed by holders in the final phase.

**Meaning for Users**
- **Project Sustainability**: Ensures resources for the team to continue building and supporting ZeroMoon.
- **Reduced Dev Impact Over Time**: The decreasing fee percentage lessens the portion taken by developers as the ecosystem matures.
- **Enhanced Holder Rewards**: The reallocation of part of the dev fee to reflections directly benefits long-term holders by increasing their passive earnings.
- **Transparency**: Dev fee distributions are logged via `Transfer` events. No pre-mined team tokens; dev funding comes from ongoing transaction activity.

---

## Conclusion
ZeroMoon (0Moon) is engineered as a community-focused reflection token with robust, automated mechanisms for liquidity provision and token burning. Its dynamic fee structure adapts to project milestones, progressively increasing rewards for holders. Key safety features, including locked liquidity and programmatically managed contract assets, aim to build trust and long-term value. The developer support model also shares benefits with the community by redirecting a portion of their fees to reflections over time. ZeroMoon's design prioritizes fairness, transparency, and scalability.

---

## Community and Governance
- ZeroMoon is designed for autonomous operation. Post-launch, with ownership renouncement (planned), governance may evolve through community consensus for future initiatives outside the immutable contract logic.
- Join our community channels: [Telegram](https://t.me/) and [X](https://x.com/)

---

## Roadmap and Future Developments
- **Phase 1 (Launch & Growth)**: Successful token launch, initial marketing, community building, listings on tracking sites.
- **Phase 2 (Ecosystem Expansion)**: Expansion to another chains.
- **Phase 3 (Utility & Innovation)**: Exploration of utility cases for the 0Moon token, and its strategic integration into further DeFi developments by leveraging its unique functionalities.

---

## License
This project, including the ZeroMoon smart contract, is licensed under the MIT License.
