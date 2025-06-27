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
  - [Post-LP Limit: Token Management and Fair Distribution of Residuals](#post-lp-limit-token-management-and-fair-distribution-of-residuals)
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
    - **Burn Limit**: Token burning from fees ceases once 50% of the original total supply has been burned.
    - **Liquidity Deposit Limit**: Automated liquidity addition from fees stops once the original total supply has been designated for or added to the liquidity pool.
- **Scalability:** Designed with O(1) time complexity for core operations like reflection distribution, enabling efficient performance even with a large and active user base.
- **Programmatically Managed Assets:** The contract holds 0Moon tokens (for liquidity/reflection) and BNB (for gas). These assets are managed by the smart contract's predefined logic for their intended purposes (e.g., adding liquidity, fee distribution, sending to `boxAddress` under specific conditions) and are not arbitrarily drainable by the owner or external parties through direct withdrawal functions.
- **Configurable FairLaunch Address**: Owner can set a `fairLaunchAddress` (e.g., for Initial Farm Offerings - IFOs), which is automatically excluded from transaction fees and reflection rewards.
- **Fair Distribution of Residual Tokens**: After liquidity goals are met, any non-earmarked 0Moon tokens remaining in the contract are distributed to holders via reflection, ensuring no tokens are "stuck" or misused.

---

## Tokenomics
ZeroMoon’s tokenomics are meticulously designed to balance holder rewards, token scarcity through burning, and the overall sustainability of the ecosystem.

- **Token Name**: ZeroMoon
- **Token Symbol**: 0Moon
- **Total Supply (`ORIGINAL_SUPPLY`)**: 100,000,000,000 0Moon (100 Billion 0Moon)
  - Defined in contract as `ORIGINAL_SUPPLY = 100_000_000_000 * 10**18`.

- **Burn Mechanism (`BURN_LIMIT`)**: Deflationary burning of tokens from transaction fees continues until 50,000,000,000 0Moon (50% of `ORIGINAL_SUPPLY`) are removed from circulation.
  - Defined in contract as `BURN_LIMIT = ORIGINAL_SUPPLY * 50 / 100`.

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

- **LP Deposit Limit (`LP_DEPOSIT_LIMIT`)**: Automated liquidity addition from fees ceases after 100,000,000,000 0Moon (the `ORIGINAL_SUPPLY`) have been cumulatively designated for (`_totalGrossLpCollected` reaching `LP_DEPOSIT_LIMIT * 2`) or added to (`_totalLpDeposited` reaching `LP_DEPOSIT_LIMIT`) the PancakeSwap liquidity pool.
  - Defined in contract as `LP_DEPOSIT_LIMIT = ORIGINAL_SUPPLY`.

- **Reflection (Auto-Staking) Eligibility & Exclusions**:
    - All holders are eligible for reflections provided they are not explicitly excluded.
    - Excluded by default: The contract itself, the `deadWallet`, the `pancakeSwapV2Pair`.
    - A configurable `fairLaunchAddress`, if set by the owner, is also automatically excluded from both fees and reflections.
    - General smart contracts (identified using `isContract()`) receiving tokens are typically auto-excluded from reflections to concentrate rewards among user EOAs.
    - There are no minimum token holding requirements to receive reflections.

---

## Safety Mechanisms
The ZeroMoon contract incorporates several design choices to enhance security and mitigate risks:

- **Ownership Renouncement (Planned)**: Post-launch, contract ownership is intended to be renounced. This action would make owner-restricted functions (`onlyOwner`) permanently inaccessible, moving the contract towards greater decentralization and immutability for those functions.
- **Immutable Addresses**: Critical infrastructure addresses such as the `pancakeSwapV2Pair`, `deadWallet`, primary `devWallet`, and `boxAddress` are set at deployment and are immutable, preventing malicious changes.
- **Reentrancy Protection**: Utilizes OpenZeppelin's `ReentrancyGuard` modifier (`nonReentrant`) on key functions involved in external calls or state changes related to liquidity operations (`executeLpFromPoke`, `triggerAutoLiquidity`), protecting against reentrancy attacks. Note: `_addLiquidityAutomatically` is private and not directly `nonReentrant` but is called by `nonReentrant` functions.
- **Locked Liquidity**: LP tokens generated from automated liquidity additions are sent directly to the `deadWallet`. This effectively burns the LP tokens, permanently locking the underlying 0Moon and BNB in the PancakeSwap liquidity pool and preventing a "rug pull" of this contract-generated liquidity.
- **No Arbitrary Withdrawals**: The contract does not contain functions allowing the owner or any other address to arbitrarily withdraw 0Moon tokens or BNB held within it. Asset movements are governed by the predefined programmatic logic of the tokenomics (fee distribution, liquidity addition, specific transfers to `boxAddress`, and distribution of residual tokens to holders via reflection).
- **User-Triggerable Liquidity Addition (Decentralized Backup)**:
    - The `triggerAutoLiquidity()` function is public and can be called by *any user or external actor*.
    - If the ZeroMoon contract has accumulated sufficient tokens (`MIN_TOKENS_TO_PROCESS_PER_CYCLE`) and BNB (`MIN_BNB_BALANCE`) for an LP event, and the `LP_DEPOSIT_LIMIT` has not been reached, any call to `triggerAutoLiquidity()` will initiate the contract's automated liquidity addition process.
    - This permissionless trigger serves as a crucial decentralized backup, ensuring that the liquidity mechanism can always be activated by the community if the primary EchoPinger bot is offline or if there's any delay. It guarantees that accumulated liquidity reserves will be processed and not left indefinitely unused within the contract. This feature underscores the contract's autonomy and resilience, further securing against tokens becoming "stuck."

---

## Contract Overview
- **Contract address**: [0x928a3a7dcce6d6e168734490577ee772024add15](https://bscscan.com/token/0x928a3a7dcce6d6e168734490577ee772024add15)
- **Contract Name**: `ZeroMoon`
- **SPDX License**: MIT
- **Solidity Version**: 0.8.30
- **Inheritance**: `ZeroMoon` inherits from:
  - `ERC20` (OpenZeppelin): Provides standard ERC20 token functionality.
  - `Ownable` (OpenZeppelin): Implements ownership control for administrative functions.
  - `ReentrancyGuard` (OpenZeppelin): Provides protection against reentrancy attacks.

---

## List of All Functions

*(This list summarizes key functions and public state variables accessible like view functions. For full details, refer to the contract code and NatSpec comments.)*

**1. Constructor**
- **Function**: `constructor`
  - **Visibility**: Deployment-time only.
  - **Modifiers**: `payable`
  - **Parameters**: `_devWallet`, `_pancakeSwapRouterAddress`, `_pancakeSwapFactoryAddress`, `_deadWalletAddress`, `_boxAddress`.
  - **Purpose**: Initializes the contract, sets up immutable addresses, creates the PancakeSwap pair, mints `ORIGINAL_SUPPLY` to the deployer, and sets initial exclusions.

**2. Public View Functions & State Variables (Data Retrieval)**
- `name()`, `symbol()`, `decimals()`, `allowance(address, address)`: Standard ERC20 view functions.
- `totalSupply()`: Returns `ORIGINAL_SUPPLY - _burnedTokens`.
- `balanceOf(address account)`: Returns token balance, accounting for reflections.
- `isContract(address _addr)`: Checks if an address has deployed code.
- `get0MoonPerBNB()`: Returns the current 0Moon price per 1 BNB from PancakeSwap.
- `getBNBPer0Moon()`: Returns the current BNB price per 1 0Moon from PancakeSwap.
- `owner()`: Returns the current contract owner (from Ownable).
- `echoPingerAddress`: Public state variable. Stores the configured EchoPinger address.
- `fairLaunchAddress`: Public state variable. Stores the configured FairLaunch address.
- `holdersCount`: Public state variable. Returns the current number of token holders.
- `_accumulatedLiquidityTokens`: Public state variable. Shows tokens currently held by the contract for LP addition.
- `_totalLpDeposited`: Public state variable. Shows total net 0Moon tokens deposited into the LP by the contract.
- `_totalGrossLpCollected`: Public state variable. Shows the total gross 0Moon tokens ever collected from fees for LP purposes.
- `_totalReflectionTokensCollected`: Public state variable. Shows the total gross 0Moon tokens ever collected from fees for reflection.
- `_burnedTokens`: Public state variable. Shows total tokens burned via fees.
- `_burnStop`, `_lpStop`: Public boolean state variables indicating status of burn/LP mechanisms.
- `pancakeSwapV2Pair`: Public immutable state variable. Address of the PancakeSwap LP pair.
- `pancakeSwapV2Router`: Public immutable state variable. Address of the PancakeSwap Router.
- `deadWallet`: Public immutable state variable. Address of the burn/dead wallet.

**3. Public State-Changing Functions (Transactions)**
- `transfer(address, uint256)`, `transferFrom(address, address, uint256)`, `approve(address, uint256)`, `increaseAllowance(address, uint256)`, `decreaseAllowance(address, uint256)`: Standard ERC20 functions. `transfer` and `transferFrom` trigger fee logic via `_update`.
- `triggerAutoLiquidity()`: `external nonReentrant`. Allows any user to trigger liquidity addition if conditions are met.

**4. Owner-Restricted Functions (`onlyOwner`)**
- `setEchoPingerAddress(address _pingerAddress)`: Sets the authorized EchoPinger contract address.
- `setFairLaunchAddress(address _newFairLaunchAddress)`: Sets the FairLaunch contract address and excludes it from fees/reflection.
- `setDevWallets(address[] memory wallets)`: Sets/updates the array of 3 developer wallets for fee splitting.
- `renounceOwnership()`: Allows the current owner to renounce ownership (from Ownable).
- `transferOwnership(address newOwner)`: Allows the current owner to transfer ownership (from Ownable).

**5. External Authorized Functions**
- `executeLpFromPoke()`: `external nonReentrant`. Callable only by `echoPingerAddress`. Triggers liquidity processing and maintenance.

**6. Internal Core Logic Functions (Private/Internal)**
- `_update(address sender, address recipient, uint256 amount)`: Core internal function overriding ERC20, handles all token movements, fee application, reflection accounting, and holder counts.
- `_calculateFees(uint256 amount)`: Calculates fee breakdown based on current contract phase.
- `_distributeFees(...)`: Distributes collected fees to reflection, liquidity, burn, and dev; triggers `_applyReflectionTokens` and `_addLiquidityAutomatically`. **Crucially, this function also handles the fair distribution of residual contract tokens to holders via reflection after LP goals are met (see [Automated Liquidity](#post-lp-limit-token-management-and-fair-distribution-of-residuals)).**
- `_applyReflectionTokens()`: Adjusts `_scalingFactor` to distribute reflection rewards.
- `_addLiquidityAutomatically()`: `private`. Swaps tokens for BNB and adds to LP. (Called by `nonReentrant` functions).
- `handleLpAddFailure(uint256 tokensIntendedForLp)`: Manages repeated LP addition failures.
- `_sendAccumulatedLpTokensToBox()`: Sends leftover `_accumulatedLiquidityTokens` to `boxAddress` under specific conditions.

**7. Fallback Function**
- `receive() external payable`: Allows the contract to receive BNB directly, emitting `BNBReceived`.

---

## Reflection (Auto-Staking)

**Process**
Reflection, or auto-staking, is ZeroMoon's mechanism for rewarding holders. A portion of every transaction fee is allocated to reflections. Instead of direct token transfers to each holder (which is gas-intensive), ZeroMoon uses a sophisticated `_scalingFactor` that is adjusted when reflections are processed. This effectively increases the value of tokens held by eligible participants over time.

**How It Works**

- **Sources of Reflection Tokens**:
    - **Transaction Fees**: The primary source is the reflection fee (3.5%, 4.5%, or 5.5% depending on the contract phase) taken from transactions and added to `_accumulatedReflectionTokens`.
    - **Residual Contract Tokens (Fair Distribution)**: After liquidity provision goals (`LP_DEPOSIT_LIMIT`) are fully met and LP operations have stopped (`_lpStop` is true), any remaining 0Moon tokens held directly by the contract (that are not part of `_accumulatedLiquidityTokens` and thus not earmarked for other specific purposes like LP or box transfer) are also added to `_accumulatedReflectionTokens`. This ensures these tokens are fairly distributed to all eligible holders. (See details under [Automated Liquidity](#post-lp-limit-token-management-and-fair-distribution-of-residuals)).
    - **Failed LP Tokens (Fallback)**: In rare cases where repeated attempts to add liquidity fail (exceeding `MAX_CONSECUTIVE_LP_ADD_FAILURES`), tokens intended for LP might be diverted to the reflection pool as a fallback mechanism.
    ```solidity
    // In _distributeFees, after fees are calculated:
    if (reflectionAmount != 0) {
        _accumulatedReflectionTokens += reflectionAmount;
        _totalReflectionTokensCollected += reflectionAmount; // Track gross total
        emit ReflectionShared(contractAddress, reflectionAmount);
    }
    // ... plus logic for residual contract tokens and failed LP tokens adding to _accumulatedReflectionTokens
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
- **Exclusions**: The contract itself, the `deadWallet`, the `pancakeSwapV2Pair`, a configured `fairLaunchAddress`, and other smart contracts (auto-detected via `isContract()` upon receiving tokens) are typically excluded from receiving reflections. This ensures rewards are concentrated among actual users (EOAs), maximizing their share and preventing dilution. This user-centric approach is a key design principle.
- **Event**: `ReflectionShared` event logs additions to the reflection pool.

**Meaning for Users**
- **Passive Earnings**: Hold 0Moon and automatically earn more from transaction activity.
- **Increasing Reward Rates**: As the project matures (burn and LP limits reached), the reflection fee percentage increases, enhancing rewards.
- **Fairness**: By excluding most contract addresses, reflections primarily benefit individual holders. Furthermore, residual contract tokens are also channeled back to holders.
- **Efficiency**: The O(1) scaling factor method is gas-efficient and highly scalable.

---

## Burning

**Process**
ZeroMoon incorporates a token burn mechanism to create deflationary pressure. A portion of transaction fees is designated as `burnAmount`. These tokens are effectively removed from circulation by allocating them to the `deadWallet`'s balance until the `BURN_LIMIT` (50% of `ORIGINAL_SUPPLY`) is reached.

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
To ensure a robust and deep trading pool on PancakeSwap, ZeroMoon automatically adds liquidity using a portion of transaction fees. These fees are collected in `_accumulatedLiquidityTokens`. When thresholds are met (sufficient tokens accumulated, sufficient BNB in the contract for gas), the contract swaps half these tokens for BNB and pairs them with the remaining half to add liquidity. This process continues until `LP_DEPOSIT_LIMIT` is reached (tracked by `_totalLpDeposited` for net tokens added, or `_totalGrossLpCollected` for gross tokens designated for LP).

**How It Works**
- **Fee Collection**: Liquidity fees (3.0% or 4.75% depending on phase) are added to `_accumulatedLiquidityTokens` and the contract's own token balance (`_userAccountData[contractAddress].scaledBalance`). `_totalGrossLpCollected` is also incremented.
- **Trigger Conditions & Execution**: Liquidity addition can be initiated in several ways:
    - **Internally**: From `_distributeFees` after a regular transaction, if conditions are met.
    - **By EchoPinger Bot**: Via the authorized `executeLpFromPoke()` call (if configured and called).
    - **By Any User**: Via the public `triggerAutoLiquidity()` function.
  Regardless of the trigger, the core conditions checked by the contract before proceeding with `_addLiquidityAutomatically` are:
    - `_accumulatedLiquidityTokens >= MIN_TOKENS_TO_PROCESS_PER_CYCLE`
    - `contractAddress.balance >= MIN_BNB_BALANCE`
    - `_totalLpDeposited < LP_DEPOSIT_LIMIT` (and `_lpStop` is false)
- **Execution (`_addLiquidityAutomatically`)**:
    1.  The amount processed per cycle (`tokensToProcessThisCycle`) is determined based on `_accumulatedLiquidityTokens`, capped by `MAX_LIQUIDIFY_PERCENT_OF_RESERVES_BPS` of the LP's token reserves (if reserves exist) and `ABSOLUTE_MAX_TOKENS_TO_PROCESS_PER_CYCLE`. The process continues in chunks until `_lpStop` is true (e.g., `LP_DEPOSIT_LIMIT` is met or `_totalGrossLpCollected` reaches its target).
    2.  Approves PancakeSwap Router to spend tokens.
    3.  Swaps half (`swapAmount`) for BNB using `pancakeSwapV2Router.swapExactTokensForETH(...)`.
    4.  Adds the received BNB and the other half of tokens (`lpTokenAmount`) to liquidity using `pancakeSwapV2Router.addLiquidityETH{value: bnbReceived}(...)`.
    5.  **LP tokens received from `addLiquidityETH` are sent to the `deadWallet`, permanently locking this liquidity.**
    6.  Updates `_accumulatedLiquidityTokens` (decremented) and `_totalLpDeposited` (incremented with `lpTokenAmount`).
    7.  Handles potential failures in swap or LP addition, with retries and fallback mechanisms (see `handleLpAddFailure`). If failures persist, tokens might be moved to reflection.
- **Events**: `LiquidityAdded` on success, `LiquidityAdditionFailed` on failure. `LpProcessingStopSet` when `_lpStop` becomes true.

### Post-LP Limit: Token Management and Fair Distribution of Residuals
This section details how ZeroMoon handles tokens once the primary liquidity provision goals are met, emphasizing fairness to holders.

-   **Cessation of Liquidity Fees**: Once `_lpStop` is true (triggered when `_totalLpDeposited` reaches `LP_DEPOSIT_LIMIT` or `_totalGrossLpCollected` reaches its corresponding target, typically `LP_DEPOSIT_LIMIT * 2`), the liquidity fee component in transactions becomes 0%.
-   **Sweeping Small Accumulated LP Tokens**: If `_lpStop` is true and there are still `_accumulatedLiquidityTokens` (tokens specifically collected for LP) but the amount is below `MIN_TOKENS_TO_PROCESS_PER_CYCLE` (i.e., too small for a full LP cycle), these specific tokens are sent to the `boxAddress`. This transfer is typically triggered by `executeLpFromPoke` or `triggerAutoLiquidity` after an LP attempt or check confirms the conditions.
    ```solidity
    // Logic in _sendAccumulatedLpTokensToBox (called under specific conditions):
    // ERC20.transfer(boxAddress, tokensToTransfer);
    ```
-   **CRITICAL UPDATE: Fair Distribution of Other Residual Contract Tokens to Holders**:
    To ensure maximum fairness and prevent any 0Moon tokens from being indefinitely "stuck" or misused within the contract after its primary liquidity functions are complete, a specific mechanism within the `_distributeFees` function handles **all other 0Moon tokens held directly by the contract** (i.e., its `_userAccountData[contractAddress].scaledBalance` that are *not* part of `_accumulatedLiquidityTokens`).

    If **all** the following conditions are met:
    1.  `_lpStop` is `true` (liquidity fee collection and automated LP addition have permanently stopped).
    2.  `_totalLpDeposited` is greater than or equal to `LP_DEPOSIT_LIMIT` (the net liquidity target for the PancakeSwap pool has been successfully met).
    3.  `_accumulatedLiquidityTokens` is `0` (there are no more tokens specifically earmarked or collected for liquidity purposes).
    4.  The contract itself (`address(this)`) still holds a balance of 0Moon tokens (e.g., these could be remnants from previous operations, tiny unprocessable amounts from fee calculations, or tokens directly transferred to the contract address for any reason).

    Then, these residual 0Moon tokens are automatically added to the `_accumulatedReflectionTokens` pool.
    ```solidity
    // Snippet from _distributeFees:
    // if (_lpStop &&
    //     _totalLpDeposited >= LP_DEPOSIT_LIMIT &&
    //     _accumulatedLiquidityTokens == 0 &&
    //     _userAccountData[contractAddress].scaledBalance > 0) {
    //
    //     uint256 currentContractActualBalance = _userAccountData[contractAddress].scaledBalance;
    //     _accumulatedReflectionTokens += currentContractActualBalance;
    //     _totalReflectionTokensCollected += currentContractActualBalance;
    //     _userAccountData[contractAddress].scaledBalance = 0;
    //     emit ReflectionShared(contractAddress, currentContractActualBalance);
    // }
    ```
    This ensures they are distributed to all eligible token holders through the standard reflection mechanism, rather than being sent to the `boxAddress` or remaining idle. This design prioritizes returning all possible residual value directly to the community once the contract's primary liquidity objectives are fulfilled.

**Meaning for Users**
- **Improved Trading Stability**: A growing liquidity pool reduces price slippage and supports larger trades.
- **Locked Liquidity**: Sending LP tokens to the `deadWallet` is a critical safety feature, preventing rug pulls of contract-generated liquidity.
- **Sustainable Growth**: Liquidity is built organically from transaction volume.
- **Transparency**: Operations are logged via events.
- **Ultimate Fairness**: After LP goals, all non-earmarked residual tokens in the contract are given back to holders as reflections.

---

## Developer Support: Reduced Impact and Community Sharing

**Process**
A portion of transaction fees is allocated to developer wallet(s) to fund ongoing project maintenance, innovation, and marketing. ZeroMoon's approach aims for sustainability while demonstrating a commitment to the community by reducing the developer fee share over time and redirecting a portion of that to enhance holder reflections.

**How It Works**
- **Fee Rates**:
    - Phase 1 (Before Burn Stop): 1.0% (`DEV_B4_BST`)
    - Phase 2 (After Burn Stop, Before LP Stop): 0.75% (`DEV_B4_LPS`)
    - Phase 3 (After Both Stops): 0.5% (`DEV_AFTER`)
- **Distribution**: Fees are sent to the primary `devWallet` or split among the `devWallets` array if configured via `setDevWallets`.
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
ZeroMoon (0Moon) is engineered as a community-focused reflection token with robust, automated mechanisms for liquidity provision and token burning. Its dynamic fee structure adapts to project milestones, progressively increasing rewards for holders. Key safety features, including locked liquidity, programmatically managed contract assets, and configurable exclusions (like `fairLaunchAddress`), aim to build trust and long-term value. **Crucially, its design ensures that after primary liquidity goals are met, any residual 0Moon tokens held by the contract are fairly distributed back to holders via reflection.** The developer support model also shares benefits with the community by redirecting a portion of their fees to reflections over time. ZeroMoon's design prioritizes fairness, transparency, and scalability.

---

## Community and Governance
- ZeroMoon is designed for autonomous operation. Post-launch, with ownership renouncement (planned), governance may evolve through community consensus for future initiatives outside the immutable contract logic.
- Join our community channels: [Telegram](https://t.me/) and [X](https://x.com/)

---

## Roadmap and Future Developments
- **Phase 1 (Launch & Growth)**: Successful token launch, initial marketing, community building, listings on tracking sites.
- **Phase 2 (Ecosystem Expansion)**: Expansion to other chains.
- **Phase 3 (Utility & Innovation)**: Exploration of utility cases for the 0Moon token, and its strategic integration into further DeFi developments by leveraging its unique functionalities.

---

## License
This project, including the ZeroMoon smart contract, is licensed under the MIT License.
