# ZeroMoon EchoPinger Bot Documentation

## 1. Introduction

The ZeroMoon EchoPinger Bot is an off-chain application designed to monitor the ZeroMoon smart contract and the Binance Smart Chain (BSC) network. Its primary purpose is to ensure the timely execution of critical contract functions, particularly related to automated liquidity provision and other maintenance tasks defined within the ZeroMoon contract.

This document outlines the bot's core responsibilities, the contract functions it interacts with, its configuration, and its operational logic.

**Note on Security**: While this document describes the bot's functionality, the full source code of the operational bot may not be made public to protect specific operational strategies, advanced error handling, and security measures related to the bot's own wallet and gas management.

## 2. Core Responsibilities

The EchoPinger Bot has the following main responsibilities:

1.  **Monitor ZeroMoon Contract State**: Regularly fetches data from the ZeroMoon contract, such as:
    *   Accumulated tokens for liquidity (`getAccumulatedLiquidityTokens()`)
    *   Total tokens already deposited into liquidity (`getTotalLpDeposited()`)
    *   The ZeroMoon contract's own BNB balance.
    *   Current burn status (`_burnedTokens()`, `_burnStop()`)
    *   Current LP fee collection status (`_lpStop()`)
2.  **Trigger Liquidity Provision**:
    *   If conditions within the ZeroMoon contract are met for adding liquidity, the bot will call the `triggerAutoLiquidity()` function on the ZeroMoon contract. 
3.  **Self-Funding for Gas Fees**:
    *   The bot's wallet address is designated as one of the developer fee recipients in the ZeroMoon smart contract.
    *   It receives its share of developer fees in ZeroMoon (0Moon) tokens.
    *   The bot automatically swaps a portion of these received 0Moon tokens for BNB to cover its ongoing operational gas costs for interacting with the network (e.g., calling `triggerAutoLiquidity()`).
4.  **Manage Bot's Own Excess BNB**: Ensures the bot's wallet maintains an optimal BNB balance, transferring any significant excess BNB (accumulated beyond operational needs) to a designated vault address.

## 3. ZeroMoon Contract Interaction

The bot primarily interacts with the following functions on the `ZeroMoon.sol` smart contract:

**Read Functions (Views):**
*   `getAccumulatedLiquidityTokens()`: To check tokens ready for LP.
*   `getTotalLpDeposited()`: To check against `LP_DEPOSIT_LIMIT`.
*   `_burnedTokens()`: To monitor burn progress.
*   `_burnStop()`: To check if the burn mechanism has ceased.
*   `_lpStop()`: To check if LP fee collection has ceased.
*   `balanceOf(address)`: To check the bot's own ZeroMoon token balance (received as dev fees).
*    The bot also queries the ZeroMoon contract's BNB balance via the RPC provider.

**Write Functions (Transactions - requiring gas):**
*   `triggerAutoLiquidity()`: Called by the bot to initiate the ZeroMoon contract's internal `_addLiquidityAutomatically` process if on-chain conditions are met. This function is also publicly callable by any user, serving as a decentralized backup for liquidity processing.
    *   **Note**: The bot is configured as the authorized `echoPingerAddress`; it call `executeLpFromPoke()`.
*   `approve(address spender, uint256 amount)`: Called by the bot on the ZeroMoon contract to approve the PancakeSwap Router to spend the bot's *own* ZeroMoon tokens (its dev fee share) when swapping them for BNB.

**PancakeSwap Router Interaction (for bot's self-funding):**
*   `WETH()`: To get the WBNB address.
*   `swapExactTokensForETHSupportingFeeOnTransferTokens(...)`: Used by the bot to swap its received ZeroMoon dev fees into BNB.

## 4. Configuration

**Bot BNB Management (after self-funding from 0Moon swaps):**
*   `MIN_BOT_BNB_BALANCE`: Minimum BNB the bot should retain for operations (e.g., "0.1").
*   `MAX_BOT_BNB_BALANCE`: If the bot's BNB (from 0Moon swaps) exceeds this, it transfers the excess to `VAULT_ADDRESS` (e.g., "0.5").

**ZeroMoon Contract Parameters (for bot's checks before calling `triggerAutoLiquidity`):**
*   `LP_DEPOSIT_LIMIT_FROM_CONTRACT`: The `LP_DEPOSIT_LIMIT` value from the ZeroMoon contract.
*   `MIN_TOKENS_FOR_LP_FROM_CONTRACT`: The `MIN_TOKENS_TO_PROCESS_PER_CYCLE` from ZeroMoon.
*   `MIN_BNB_IN_CONTRACT_FOR_LP`: The `MIN_BNB_BALANCE` required in the ZeroMoon contract for LP ops.

**Bot's ZeroMoon Swap for Self-Funding:**
*   `MIN_ZEROMOON_TO_INITIATE_SWAP`: Minimum 0Moon balance (from dev fees) the bot must have to trigger its swap-to-BNB (e.g., "1000000").
*   `ZEROMOON_SWAP_AMOUNT`: Amount of 0Moon the bot will swap for BNB if the minimum is met (e.g., "500000").

**Operational:**
*   `CHECK_INTERVAL_MS`: How often the bot performs its checks (e.g., "60000" for 60 seconds).

## 5. Operational Logic

The bot runs in a loop with the following main steps:

1.  **`checkAndTriggerLp()`**:
    *   Reads relevant state from the ZeroMoon contract.
    *   If conditions indicate an LP addition can be triggered, the bot estimates gas and calls `triggerAutoLiquidity()` on the ZeroMoon contract.
    *   Logs the outcome.

2.  **`manageBotZeroMoonBalanceAndSwap()` (Self-Funding Mechanism)**:
    *   Checks the bot wallet's own ZeroMoon token balance (which accumulates from its share of dev fees).
    *   If this balance exceeds `MIN_ZEROMOON_TO_INITIATE_SWAP`, it approves the PancakeSwap Router and then swaps a configured `ZEROMOON_SWAP_AMOUNT` of its ZeroMoon tokens for BNB. This BNB is then available in the bot's wallet for gas fees.

3.  **`manageBotBnbBalance()`**:
    *   Checks the bot wallet's current BNB balance (which is replenished by the 0Moon swaps).
    *   If it exceeds `MAX_BOT_BNB_BALANCE`, it transfers the excess BNB (amount above `MIN_BOT_BNB_BALANCE`) to the `VAULT_ADDRESS`.

4.  **Loop**: Repeats these checks at the interval defined by `CHECK_INTERVAL_MS`. The order of operations ensures 0Moon is swapped for BNB before checking if excess BNB needs to be vaulted.

## 6. Funding and Maintenance

The EchoPinger Bot is designed to be self-sustaining in terms of operational gas fees.
*   **Primary Funding Source**: The bot's wallet address is designated as one of the recipients of the developer fee share from the ZeroMoon smart contract. It receives these fees in ZeroMoon (0Moon) tokens.
*   **Automated Gas Replenishment**: The bot includes logic (`manageBotZeroMoonBalanceAndSwap()`) to automatically convert a portion of its received 0Moon tokens into BNB. This BNB is then used to pay for the network transaction fees incurred when it calls functions like `triggerAutoLiquidity()` on the ZeroMoon contract or manages its own BNB balance.
*   This automated cycle ensures the bot can continue to operate and support the ZeroMoon ecosystem's liquidity mechanisms without requiring manual BNB top-ups, provided the developer fee share generates sufficient 0Moon tokens to cover gas costs over time.

## 7. Disclaimer

This bot is provided as a utility to support the ZeroMoon ecosystem. Users should understand that off-chain components like this bot rely on the infrastructure they run on (servers, RPC nodes) and the security of the bot's wallet. The ZeroMoon smart contract itself is designed to function autonomously based on transaction flow, but this bot aims to ensure and secure that certain functions are triggered proactively when conditions are optimal. The self-funding mechanism relies on the ongoing collection of developer fees by the ZeroMoon contract and the market conditions for swapping 0Moon to BNB.

---
