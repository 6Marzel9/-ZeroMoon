# ZeroMoon

![ZeroMoon Logo](zeromoon.png)

ZeroMoon is an advanced reflection (auto-staking) token deployed on the Binance Smart Chain (BSC), inspired by SafeMoon but reengineered for enhanced safety, autonomy, and scalability. It introduces automated liquidity management, dynamic fee adjustments, and a secure, decentralized framework to eliminate centralized vulnerabilities.

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
ZeroMoon is a decentralized token on BSC that rewards holders through automatic reflection (auto-staking), reduces supply via burning, and enhances liquidity on PancakeSwap V2. Built with autonomy in mind, it requires no manual intervention post-deployment, leveraging smart contract logic to manage rewards, burns, liquidity, and developer support.

---

## Key Features
- **Dynamic Fee Structure:** Automatically adjusts fees based on burn and liquidity milestones for balanced rewards and sustainability.
- **Automated Liquidity Addition:** Adds liquidity to PancakeSwap when fee thresholds are met, supporting a stable trading ecosystem.
- **Burn and Liquidity Caps:** Limits token burning to 20% of the total supply and liquidity addition from fees to 2.6 billion tokens.
- **Scalability:** Supports up to 20 million users with O(1) time complexity operations for efficiency at scale.
- **No Exploitable Assets:** Ensures the contract holds no drainable assets (BNB, tokens, or LP tokens).
- **Scalability** for +20 million users

---

## Tokenomics
ZeroMoon’s tokenomics are designed to balance rewards, scarcity, and ecosystem sustainability:

- **Total Supply**: 1,000,000,000 0MOON (1 billion 0MOON)
  - Defined as `ORIGINAL_SUPPLY = 1_000_000_000 * 10**18`.

- **Burn Mechanism**: Burns tokens until 200,000,000 0MOON (20% of supply) are removed.
  - Set via `BURN_LIMIT = ORIGINAL_SUPPLY * 20 / 100`.

- **Transaction Fees** (in basis points, where 100 = 1%):
  - **Before Burn Limit (10% Total Fee)**:
    - **Reflection**: 3.5% (`REFLECTION_FEE_BEFORE = 350`)
    - **Liquidity**: 3% (`LIQUIDITY_FEE_BEFORE = 300`)
    - **Burn**: 2.5% (`BURN_FEE_BEFORE = 250`)
    - **Developer**: 1.0% (`DEV_FEE_BEFORE = 100`)

  - **After Burn Limit, Before LP Limit (10% Total Fee)**:
    - **Reflection**: 4.5% (`REFLECTION_FEE_BEFORE_LP = 450`)
    - **Liquidity**: 4.75% (`LIQUIDITY_FEE_BEFORE_LP = 475`)
    - **Burn**: 0% (burn stops)
    - **Developer**: 0.75% (`DEV_FEE_BEFORE_LP = 75`)

  - **After Both Limits (6% Total Fee)**:
    - **Reflection**: 5.5% (`REFLECTION_FEE_AFTER = 550`)
    - **Liquidity**: 0% (liquidity addition stops)
    - **Burn**: 0%
    - **Developer**: 0.5% (`DEV_FEE_AFTER = 50`)

- **LP Deposit Limit**: Stops liquidity addition after 2,600,000,000 0MOON (Total Supply * 2.6) are deposited into PancakeSwap.
  - Defined as `LP_DEPOSIT_LIMIT = 2_600_000_000 * 10**18`.

- **Reflection (Auto-Staking) Eligibility**: No restrictions or minimum token holding limits.

---

## Safety Mechanisms
The ZeroMoon contract mitigates risks inherent in earlier reflection tokens through:

- **Ownership Renouncement:** After the Initial Fair Offering (IFO), ownership can be renounced, disabling all onlyOwner functions for true decentralization.
- **Immutable Addresses:** Critical addresses (pancakeSwapV2Pair, vaultAddress) are fixed at deployment, preventing manipulation.
- **Reentrancy Protection:** Uses the nonReentrant modifier to secure external calls, thwarting reentrancy attacks.


---

## Contract Overview
- **Inheritance**: `ZeroMoon` inherits from:
  - `ERC20` (OpenZeppelin): Provides standard ERC20 token functionality (e.g., `transfer`, `balanceOf`).
  - `Ownable` (OpenZeppelin): Adds ownership controls (e.g., `onlyOwner` modifier).
  - `ReentrancyGuard` (OpenZeppelin): Adds reentrancy protection (e.g., `nonReentrant` modifier).


---

## List of All Functions


**1. Constructor**
- **Function**: `constructor`
  - **Visibility**: Implicitly public (only called during deployment).
  - **Modifiers**: `payable`
  - **Parameters**:
    - `address _devWallet`: Address for dev fees.
    - `address _pancakeSwapRouterAddress`: PancakeSwap router address.
    - `address _pancakeSwapFactoryAddress`: PancakeSwap factory address.
    - `address _deadWalletAddress`: Address for burning tokens.
    - `address payable _vaultAddress`: Address to receive BNB transfers.
  

**2. Public Functions**

- **Function**: `isContract`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Parameters**:
    - `address _addr`: Address to check.
  - **Returns**: `bool`: True if the address is a contract, false otherwise.
  - **Purpose**: Checks if an address is a contract by inspecting its code size.

- **Function**: `getZMOPerBNB`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Parameters**: None
  - **Returns**: `uint256`: Number of tokens per BNB based on PancakeSwap pair reserves.
  - **Purpose**: Calculates the token price in BNB using the PancakeSwap pair’s reserves.

- **Function**: `getBNBPerZMO`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Parameters**: None
  - **Returns**: `uint256`: Amount of BNB per token based on PancakeSwap pair reserves.
  - **Purpose**: Calculates the BNB price of one token using the PancakeSwap pair’s reserves.

- **Function**: `transfer`
  - **Visibility**: `public`
  - **Modifiers**: `override` (overrides `ERC20`)
  - **Parameters**:
    - `address recipient`: Address to send tokens to.
    - `uint256 amount`: Amount of tokens to transfer (in wei).
  - **Returns**: `bool`: True if successful.
  - **Purpose**: Transfers tokens from the sender to the recipient, applying fees and updating holder counts.

- **Function**: `transferFrom`
  - **Visibility**: `public`
  - **Modifiers**: `override` (overrides `ERC20`)
  - **Parameters**:
    - `address sender`: Address to transfer tokens from.
    - `address recipient`: Address to send tokens to.
    - `uint256 amount`: Amount of tokens to transfer (in wei).
  - **Returns**: `bool`: True if successful.
  - **Purpose**: Transfers tokens from a sender to a recipient on behalf of the caller (using allowance), applying fees, and updating holder counts.

- **Function**: `balanceOf`
  - **Visibility**: `public`
  - **Modifiers**: `view`, `override` (overrides `ERC20`)
  - **Parameters**:
    - `address account`: Address to check the balance of.
  - **Returns**: `uint256`: Balance of the account in tokens (adjusted for reflections).
  - **Purpose**: Returns the token balance of an account, accounting for reflection scaling.

- **Function**: `totalSupply`
  - **Visibility**: `public`
  - **Modifiers**: `view`, `override` (overrides `ERC20`)
  - **Parameters**: None
  - **Returns**: `uint256`: Total supply minus burned tokens.
  - **Purpose**: Returns the total supply of tokens, excluding burned tokens.

- **Function**: `getTotalLpDeposited`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Parameters**: None
  - **Returns**: `uint256`: Total tokens deposited to the LP from fees.
  - **Purpose**: Returns the total amount of tokens added to the liquidity pool via automatic liquidity addition.

**3. External Functions**

- **Function**: `setDevWallets`
  - **Visibility**: `external`
  - **Modifiers**: `onlyOwner` (from `Ownable`)
  - **Parameters**:
    - `address[] memory wallets`: Array of 3 addresses to receive dev fees.
  - **Returns**: None
  - **Purpose**: Allows the owner to set the dev wallets for fee distribution (must be 3 EOAs).

**4. Private Functions**
These functions are internal to the contract and handle core logic.

- **Function**: `calculateMinAmount`
  - **Visibility**: `private`
  - **Modifiers**: `pure`
  - **Parameters**:
    - `uint256 expectedAmount`: Expected amount to adjust for slippage.
  - **Returns**: `uint256`: Minimum amount after applying slippage tolerance.
  - **Purpose**: Calculates the minimum acceptable amount for swaps, applying a 10% slippage tolerance.

- **Function**: `_calculateFees`
  - **Visibility**: `private`
  - **Modifiers**: None
  - **Parameters**:
    - `uint256 amount`: Amount to calculate fees on.
  - **Returns**: `Fees memory`: Struct containing reflection, liquidity, burn, and dev fees.
  - **Purpose**: Calculates the fees to be applied to a transfer based on the current phase (before/after burn stop, before/after LP stop).

- **Function**: `_transferWithFees`
  - **Visibility**: `private`
  - **Modifiers**: None
  - **Parameters**:
    - `address sender`: Address sending tokens.
    - `address recipient`: Address receiving tokens.
    - `uint256 amount`: Amount to transfer (in wei).
  - **Returns**: None
  - **Purpose**: Internal function to handle token transfers, applying fees, exclusions, and reflections.

- **Function**: `_distributeFees`
  - **Visibility**: `private`
  - **Modifiers**: None (previously had `nonReentrant`, now removed)
  - **Parameters**:
    - `uint256 reflectionAmount`: Reflection fee amount.
    - `uint256 liquidityAmount`: Liquidity fee amount.
    - `uint256 burnAmount`: Burn fee amount.
    - `uint256 devAmount`: Dev fee amount.
  - **Returns**: None
  - **Purpose**: Distributes fees to reflections, liquidity, burn, and dev wallets, triggers liquidity addition, and sends BNB to the vault if conditions are met.

- **Function**: `_applyReflectionTokens`
  - **Visibility**: `private`
  - **Modifiers**: None
  - **Parameters**: None
  - **Returns**: None
  - **Purpose**: Distributes accumulated reflection tokens by adjusting `_scalingFactor` for non-excluded holders.

- **Function**: `_addLiquidityAutomatically`
  - **Visibility**: `private`
  - **Modifiers**: `nonReentrant`
  - **Parameters**: None
  - **Returns**: None
  - **Purpose**: Automatically swaps tokens for BNB and adds liquidity to the PancakeSwap pool if conditions (threshold, BNB balance) are met.

**5. Inherited Functions from `ERC20` (OpenZeppelin)**

- **Function**: `name`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Returns**: `string`: "ZeroMoon"
  - **Purpose**: Returns the token’s name.

- **Function**: `symbol`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Returns**: `string`: "0Moon"
  - **Purpose**: Returns the token’s symbol.

- **Function**: `decimals`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Returns**: `uint8`: 18
  - **Purpose**: Returns the number of decimals (standard for ERC20).

- **Function**: `allowance`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Parameters**:
    - `address owner`: Address owning tokens.
    - `address spender`: Address allowed to spend tokens.
  - **Returns**: `uint256`: Amount the spender is allowed to spend.
  - **Purpose**: Returns the remaining allowance for a spender.

- **Function**: `approve`
  - **Visibility**: `public`
  - **Parameters**:
    - `address spender`: Address to approve.
    - `uint256 amount`: Amount to approve.
  - **Returns**: `bool`: True if successful.
  - **Purpose**: Approves a spender to transfer tokens on behalf of the caller.

- **Function**: `increaseAllowance`
  - **Visibility**: `public`
  - **Parameters**:
    - `address spender`: Address to increase allowance for.
    - `uint256 addedValue`: Amount to increase allowance by.
  - **Returns**: `bool`: True if successful.
  - **Purpose**: Increases the allowance for a spender.

- **Function**: `decreaseAllowance`
  - **Visibility**: `public`
  - **Parameters**:
    - `address spender`: Address to decrease allowance for.
    - `uint256 subtractedValue`: Amount to decrease allowance by.
  - **Returns**: `bool`: True if successful.
  - **Purpose**: Decreases the allowance for a spender.

**6. Inherited Functions from `Ownable` (OpenZeppelin)**

- **Function**: `owner`
  - **Visibility**: `public`
  - **Modifiers**: `view`
  - **Returns**: `address`: Address of the owner.
  - **Purpose**: Returns the current owner of the contract.

- **Function**: `renounceOwnership`
  - **Visibility**: `public`
  - **Modifiers**: `onlyOwner`
  - **Parameters**: None
  - **Returns**: None
  - **Purpose**: Renounces ownership, setting the owner to `address(0)`.

- **Function**: `transferOwnership`
  - **Visibility**: `public`
  - **Modifiers**: `onlyOwner`
  - **Parameters**:
    - `address newOwner`: Address to transfer ownership to.
  - **Returns**: None
  - **Purpose**: Transfers ownership to a new address.

**7. Fallback Function**
- **Function**: `receive`
  - **Visibility**: `external`
  - **Modifiers**: `payable`
  - **Parameters**: None
  - **Returns**: None
  - **Purpose**: Allows the contract to receive BNB, emitting a `BNBReceived` event.

---


## Reflection

**Process**
Reflection, or auto-staking, is a mechanism where holders automatically receive a portion of the transaction fees without needing to stake their tokens manually. In `ZeroMoon`, this is achieved by adjusting a scaling factor (`_scalingFactor`) to distribute reflection fees to non-excluded holders, effectively increasing their token balances over time without explicit transfers.

**How It Works**
The reflection mechanism follows a structured logic integrated into the contract’s transfer and fee distribution process:

- **Condition Check**:
  - Reflection occurs when there are accumulated reflection tokens (`_accumulatedReflectionTokens > 0`) in the contract, which are collected as part of the transaction fees.
  - This check happens in `_distributeFees`:
    ```solidity
    if (_accumulatedReflectionTokens > 0) {
        _applyReflectionTokens();
    }
    ```
  - `_accumulatedReflectionTokens` is incremented during transfers when a `reflectionAmount` is calculated:
    ```solidity
    if (reflectionAmount > 0) {
        _accumulatedReflectionTokens += reflectionAmount;
        emit ReflectionShared(msg.sender, reflectionAmount);
    }
    ```

- **Reflection Fee Rates**:
  - Before burn stop: 3.5% (`REFLECTION_B4_BST`).
  - After burn stop, before LP stop: 4.5% (`REFLECTION_B4_LPS`).
  - After LP stop: 5.5% (`REFLECTION_AFTER`).
  - Example: A 1 million token transfer (1,000,000 * 10^18) before burn stop generates a reflection fee of 35,000 * 10^18 tokens.

- **Scaling Factor Adjustment**:
  - The `_applyReflectionTokens` function distributes these accumulated tokens by adjusting `_scalingFactor`:
    ```solidity
    function _applyReflectionTokens() private {
        uint256 reflectionPool = _accumulatedReflectionTokens;
        if (reflectionPool > 0 && _totalScaledBalance > 0) {
            uint256 S = _totalScaledBalance / _scalingFactor;
            if (S > 0) {
                uint256 denominator = S + reflectionPool;
                require(denominator > S, "Integer overflow in reflection calculation");
                if (denominator > 0) {
                    uint256 newScalingFactor = (_scalingFactor * S) / denominator;
                    require(newScalingFactor < _scalingFactor, "Invalid scaling factor adjustment");
                    _scalingFactor = newScalingFactor;
                    _accumulatedReflectionTokens = 0;
                }
            }
        }
    }
    ```
  - **Logic**:
    - `_totalScaledBalance` is the sum of scaled balances for non-excluded holders.
    - `S` represents the total unscaled balance of non-excluded holders (`_totalScaledBalance / _scalingFactor`).
    - The new `_scalingFactor` is adjusted to distribute the `reflectionPool` proportionally across holders by reducing the scaling factor, effectively increasing each holder’s balance when viewed through `balanceOf`.

- **Balance Adjustment**:
  - The `balanceOf` function adjusts balances based on `_scalingFactor`:
    ```solidity
    function balanceOf(address account) public view override returns (uint256) {
        if (_scaledBalances[account] == 0) return 0;
        if ((_exclusionFlags[account] & EXCLUDED_FROM_REFLECTION) != 0) {
            return _scaledBalances[account];
        } else {
            return _scaledBalances[account] / _scalingFactor;
        }
    }
    ```
  - Non-excluded holders see their balance increase as `_scalingFactor` decreases, reflecting the distribution of reflection fees.

- **Exclusion**:
  - Certain addresses (e.g., `contractAddress`, `deadWallet`, `pancakeSwapV2Pair`) are excluded from reflections via `_exclusionFlags`:
    ```solidity
    _exclusionFlags[contractAddress] = EXCLUDED_FROM_FEE | EXCLUDED_FROM_REFLECTION;
    ```
  - Contracts are automatically excluded during transfers:
    ```solidity
    if (isContract(recipient) && (_exclusionFlags[recipient] & EXCLUDED_FROM_REFLECTION) == 0) {
        _exclusionFlags[recipient] |= EXCLUDED_FROM_REFLECTION;
        // ...
    }
    ```

- **Event Logging**:
  - When reflection fees are accumulated, a `ReflectionShared` event is emitted:
    ```solidity
    emit ReflectionShared(msg.sender, reflectionAmount);
    ```

**Post-LP Stop**
- Once `_lpStop` is true (after `LP_DEPOSIT_LIMIT` is reached), the reflection (auto-staking) fee increases to 5.5%, enhancing rewards for holders as liquidity fees cease. 

**Meaning for Users**

- **Passive Income**:
  - Holders automatically earn more tokens simply by holding, as each transaction redistributes fees to non-excluded holders via `_scalingFactor` adjustments.
- **Increased Rewards Over Time**:
  - As burn and LP limits are reached, reflection fees increase (from 3.5% to 5.5%), providing greater rewards to long-term holders.
- **Fair Distribution**:
  - Excluding contracts and specific addresses ensures reflections benefit real users, not bots or contracts, promoting fairness in the ecosystem.
- **Scalability**:
  - The reflection mechanism is O(1) (constant time), making it scalable for +20 million users, as it avoids iterating over holders.

---

## Burning

**Process**
Tokens are burned by transferring them to a designated burn address (`deadWallet`, address `0x000...dEaD`), until the total burned tokens reach the predefined `BURN_LIMIT` of 200 million 0Moon tokens. This burning process is integrated into the `_distributeFees` function, which handles fee distribution during each transaction. While the tokens are sent to `deadWallet`, a `Transfer` event is emitted with `address(0)` as the recipient to signal the burn, following ERC20 conventions.

**How It Works**
The burning mechanism follows a structured logic:

- **Condition Check**:
  - Burning occurs only if:
    - `_burnStop` is `false` (burn limit not reached).
    - `burnAmount` (calculated fee portion for burning) is positive.
  - Code in `_distributeFees`:
    ```solidity
    if (!_burnStop && burnAmount > 0) {
        // Burning logic
    }
    ```

- **Burn Fee Rates**:
  - Before burn stop: 2.5% (`BURN_B4_BST`).
  - After burn stop: 0%.
  - Example: A 1 million token transfer (1,000,000 * 10^18) generates a burn fee of 25,000 * 10^18 tokens.

- **Burn Execution**:
  - Tokens are "burned" by increasing the scaled balance of `deadWallet`:
    ```solidity
    _scaledBalances[deadWallet] += burnAmount;
    ```
  - Note: The `deadWallet` is excluded from reflections (`EXCLUDED_FROM_REFLECTION`), so its balance is not scaled by `_scalingFactor`. The balance of `deadWallet` can be checked via `balanceOf(deadWallet)`, which will return the raw `_scaledBalances[deadWallet]` value, representing the total burned tokens.

- **Tracking**:
  - The total burned tokens are tracked in `_burnedTokens`:
    ```solidity
    _burnedTokens += burnAmount;
    ```

- **Event Logging**:
  - A `Transfer` event logs the burn on the blockchain, using `address(0)` as the recipient to follow ERC20 burn conventions:
    ```solidity
    emit Transfer(contractAddress, address(0), burnAmount);
    ```
  - Note: While the event uses `address(0)` to signal a burn, the actual tokens are held in `deadWallet`, and `balanceOf(address(0))` will always be 0 since no tokens are sent there.

- **Burn Limit Check**:
  - After each burn, the contract checks if `_burnedTokens` has reached `BURN_LIMIT`:
    ```solidity
    if (_burnedTokens >= BURN_LIMIT) {
        _burnStop = true;
    }
    ```

- **Liquidity Threshold Adjustment**:
  - Burning also affects `liquidityThreshold`, which controls the frequency of liquidity addition:
    ```solidity
    if (_burnedTokens < BURN_LIMIT) {
        uint256 burnSteps = _burnedTokens / (ORIGINAL_SUPPLY / 10);
        liquidityThreshold = BASE_LIQUIDITY_THRESHOLD;
        for (uint256 i = 0; i < burnSteps; i++) {
            liquidityThreshold = (liquidityThreshold * 11) / 10;
        }
    } else {
        liquidityThreshold = FIXED_LIQUIDITY_THRESHOLD;
    }

    ```
  - This ties burn progress to liquidity addition frequency.

**Post-Burn Limit**
- Once `_burnedTokens` reaches `BURN_LIMIT` (200 million tokens), burning stops:
  - `_burnStop` is set to `true`.
  - The burn fee becomes 0%.
  - The 2.5% burn fee is redirected: reflection fee increases to 4.5%, liquidity to 4.75%, and dev fee reduces to 0.75%.

**Meaning for Users**
- **Supply Reduction**:
  - Burning removes tokens from circulation, reducing the effective supply from 1 billion to 800 million tokens (including the initial 200M burned during IFO). This increases scarcity, potentially enhancing token value.
- **Controlled Deflation**:
  - `BURN_LIMIT` caps burning at 200 million tokens, ensuring a balance between deflation and maintaining enough tokens for trading and utility.
- **Transparency**:
  - `_burnedTokens` and `Transfer` events provide public visibility into the burn process, allowing users to verify the total burned amount on the blockchain.

---

## Automated Liquidity

**Process**
Automated liquidity addition ensures the PancakeSwap liquidity pool grows over time by using a portion of transaction fees. The contract accumulates liquidity fees in `_accumulatedLiquidityTokens`, and when certain conditions are met, it swaps half of these tokens for BNB and adds them to the pool, up to an `LP_DEPOSIT_LIMIT` of 2.6 billion tokens.

**How It Works**
The automated liquidity mechanism is integrated into the `_distributeFees` and `_addLiquidityAutomatically` functions:

- **Condition Check**:
  - Liquidity addition is triggered in `_distributeFees`:
    ```solidity
    if (_accumulatedLiquidityTokens >= liquidityThreshold && contractAddress.balance >= MIN_BNB_BALANCE) {
        _addLiquidityAutomatically();
    }
    ```
  - **Conditions**:
    - `_accumulatedLiquidityTokens` must meet or exceed `liquidityThreshold` (starts at 0.01 * 10^18, adjusts with burns).
    - Contract BNB balance must be at least `MIN_BNB_BALANCE` (0.001 BNB).

- **Liquidity Fee Rates**:
  - Before burn stop: 3%.
  - After burn stop, before LP stop: 4.75%.
  - After LP stop: 0%.
  - Example: A 1 million token transfer generates 30,000 * 10^18 tokens in liquidity fees before the burn stop.

- **Accumulation**:
  - Liquidity fees are added to `_accumulatedLiquidityTokens`:
    ```solidity
    _accumulatedLiquidityTokens += liquidityAmount;
    _scaledBalances[contractAddress] += liquidityAmount;
    ```

- **Liquidity Addition** (`_addLiquidityAutomatically`):
  - **Check Limits**:
    ```solidity
    if (_lpStop) {
        return;
    }
    uint256 tokenAmount = _accumulatedLiquidityTokens;
    if (tokenAmount == 0) return;
    if (contractAddress.balance < MIN_BNB_BALANCE) {
        emit LiquidityAdditionFailed(tokenAmount, "Insufficient BNB");
        return;
    }
    ```
  - **Calculate Amounts**:
    - Split `tokenAmount` into `swapAmount` (to swap for BNB) and `lpTokenAmount` (to pair with BNB):
      ```solidity
      uint256 swapAmount = tokenAmount / 2;
      uint256 lpTokenAmount = tokenAmount - swapAmount;
      ```
  - **Limit Check**:
    - If `_totalLpDeposited + lpTokenAmount` exceeds `LP_DEPOSIT_LIMIT`, adjust amounts:
      ```solidity
      if (_totalLpDeposited + lpTokenAmount > LP_DEPOSIT_LIMIT) {
          lpTokenAmount = LP_DEPOSIT_LIMIT - _totalLpDeposited;
          if (lpTokenAmount == 0) {
              _lpStop = true;
              return;
          }
          swapAmount = lpTokenAmount;
          tokenAmount = lpTokenAmount * 2;
      }
      ```
  - **Approve and Swap**:
    - Approve the router to spend tokens:
      ```solidity
      _approve(contractAddress, address(pancakeSwapV2Router), tokenAmount);
      ```
    - Swap half for BNB:
      ```solidity
      try pancakeSwapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(
          swapAmount,
          amountOutMin,
          path,
          contractAddress,
          block.timestamp + 300
      ) {
          uint256 bnbReceived = contractAddress.balance - initialBalance;
          // ...
      }
      ```
  - **Add Liquidity**:
    - Pair the remaining tokens with BNB:
      ```solidity
      try pancakeSwapV2Router.addLiquidityETH{value: bnbReceived}(
          contractAddress,
          lpTokenAmount,
          0,
          0,
          deadWallet,
          block.timestamp + 300
      ) {
          _accumulatedLiquidityTokens -= tokenAmount;
          _totalLpDeposited += lpTokenAmount;
          // ...
      }
      ```

- **Event Logging**:
  - `LiquidityAdded` event if successful:
    ```solidity
    emit LiquidityAdded(lpTokenAmount, bnbReceived);
    ```
  - `LiquidityAdditionFailed` event if failed:
    ```solidity
    emit LiquidityAdditionFailed(tokenAmount, "Swap failed");
    ```

**Post-LP Stop**
- Once `_totalLpDeposited` reaches `LP_DEPOSIT_LIMIT` (2.6 billion tokens), `_lpStop` is set to `true`, stopping further liquidity addition. Liquidity fees become 0%, and the total fee drops to 6%.

**Meaning for Users**

- **Stable Trading**:
  - Automated liquidity ensures the PancakeSwap pool grows, reducing slippage and improving trading stability.
- **Long-Term Growth**:
  - 2.6 billion tokens added over time (260% of `ORIGINAL_SUPPLY`) creates a deep pool, supporting high trading volumes.
- **Fairness**:
  - LP tokens are sent to `deadWallet`, ensuring no one can withdraw liquidity, protecting users from rug pulls.
  - Funding liquidity growth solely through market volume ensures that the pool’s expansion reflects actual trading activity, promoting a fair and transparent ecosystem.


---

## Developer Support

**Process**
The **Developer Support** mechanism in `The_ZeroMoon` contract allocates a portion of each transaction’s fees to support the development team, ensuring the project’s sustainability. These fees are initially sent to a single developer wallet (`devWallet`) or distributed across multiple wallets (`devWallets`) if set. Over time, as the contract progresses through its phases (burn stop and LP stop), the developer fee decreases, and a portion of this reduction is redirected to enhance reflection (auto-staking) rewards for the community. This reflects the developers’ commitment to giving back by sharing 50% of their fee allocation with holders.

**How It Works**
The developer fee mechanism is embedded in the `_distributeFees` function, and its interaction with reflection rewards evolves across three phases: before burn stop, after burn stop but before LP stop, and after LP stop. Let’s break this down:

- **Developer Fee Rates Across Phases**:
  - **Before Burn Stop**: 1% (`DEV_B4_BST` = 100 basis points).
  - **After Burn Stop, Before LP Stop**: 0.75% (`DEV_B4_LPS` = 75 basis points).
  - **After LP Stop**: 0.5% (`DEV_AFTER` = 50 basis points).
  - Total transaction fees remain at 10% until LP stop, then drop to 6%.

- **Fee Distribution Logic**:
  - In `_distributeFees`, the developer fee (`devAmount`) is calculated based on the transaction amount and the current phase’s rate:
    ```solidity
    fees.dev = (amount * DEV_B4_BST) / FEE_DENOMINATOR; // Before burn stop: 1%
    // After burn stop: DEV_B4_LPS (0.75%)
    // After LP stop: DEV_AFTER (0.5%)
    ```
  - The fee is then distributed:
    - If `devWallets` is set (3 addresses), the fee is split equally among them.
    - Otherwise, it goes to `devWallet`:
      ```solidity
      if (devWallets.length == DEV_WALLETS_COUNT) {
          uint256 devShare = devAmount / DEV_WALLETS_COUNT;
          for (uint256 i = 0; i < DEV_WALLETS_COUNT; i++) {
              address wallet = devWallets[i];
              // Distribute devShare to wallet
              emit Transfer(contractAddress, wallet, devShare);
          }
      } else {
          // Distribute devAmount to devWallet
          emit Transfer(contractAddress, devWallet, devAmount);
      }
      ```

- **Reduced Impact and Sharing**:
  - **Fee Reduction**: The developer fee decreases over time:
    - From 1% to 0.75% after burn stop (a 25% reduction in dev share).
    - From 0.75% to 0.5% after LP stop (a further 33.33% reduction from 0.75%, or 50% from the original 1%).
  - **Sharing with Community**: The developers allocate 50% of their initial fee share to reflection rewards by the end of the contract’s lifecycle:
    - **Initial Dev Fee**: 1% (100 basis points).
    - **Final Dev Fee**: 0.5% (50 basis points).
    - **Reduction**: 1% - 0.5% = 0.5% (50 basis points), which is 50% of the initial 1% dev fee.
    - This 0.5% (50 basis points) reduction is redirected to increase the reflection fee, enhancing auto-staking rewards for holders:
      - The reflection fee starts at 3.5% (`REFLECTION_B4_BST`).
      - Increases to 4.5% (`REFLECTION_B4_LPS`) after burn stop (a 1% increase, of which 0.25% comes from the dev fee reduction).
      - Increases to 5.5% (`REFLECTION_AFTER`) after LP stop (a further 1% increase, of which 0.25% comes from the dev fee reduction).
    - **Total Reflection Increase**: From 3.5% to 5.5% = 2% (200 basis points).
    - **Dev Contribution to Reflection**: The dev fee reduction (0.5%) contributes to this increase, alongside reductions in burn and liquidity fees:
      - Burn fee: 2.5% → 0% (250 basis points redirected).
      - Liquidity fee: 3% → 4.75% → 0% (net 300 basis points redirected).
      - Total redirected to reflection: 250 (burn) + 300 (liquidity) - 175 (new liquidity) + 50 (dev) = 425 basis points, but reflection only increases by 200 basis points (the rest adjusts other fees).

- **Calculation of Sharing**:
  - **Initial Dev Fee**: 1% = 100 basis points.
  - **Final Dev Fee**: 0.5% = 50 basis points.
  - **Reduction**: 50 basis points = 50% of the initial dev fee.
  - **Reflection Gain from Dev**: Of the 200 basis points added to reflection (3.5% to 5.5%), 50 basis points come from the dev fee reduction:
    - First phase (burn stop): 25 basis points (1% to 0.75%) → 25/100 = 25% of initial dev fee.
    - Second phase (LP stop): 25 basis points (0.75% to 0.5%) → 25/100 = 25% of initial dev fee.
    - Total: 25% + 25% = 50% of the initial dev fee is redirected to reflection.

- **Event Logging**:
  - Each dev fee distribution emits a `Transfer` event:
    ```solidity
    emit Transfer(contractAddress, devWallet, devAmount);
    ```

**Meaning for Users**
- **Transparency**:
  - Dev fees are transparently allocated, with events logging each distribution, and no hidden pre-allocations exist at deployment.
- **Sustainability**:
  - Fees fund development, ensuring the project can maintain and improve its ecosystem, but these funds come entirely from market activity, not from reserved tokens.
- **Reduced Impact**:
  - The gradual reduction of the developer fee from 1% to 0.5% over the contract’s lifecycle minimizes the portion of transaction fees taken by the developers, leaving more value for the community.
- **Community Sharing**:
  - Developers give back by allocating 50% of their initial fee share (0.5% of the 1%) to reflection rewards, increasing the auto-staking benefits for holders from 3.5% to 5.5%. This demonstrates a commitment to the community, ensuring holders receive greater passive rewards as the project matures.
- **Enhanced Rewards**:
  - The redirected 0.5% contributes to a 2% total increase in reflection fees, boosting auto-staking rewards and incentivizing long-term holding.
- **Fairness**:
  - By not reserving tokens for developers or third parties at deployment, the project ensures a fair distribution of the initial supply, with all allocations (including developer support) derived from market volume, fostering trust and alignment with user interests.

---

## Conclusion
The **Reduced Impact** aspect of the Developer Support mechanism in `ZeroMoon` contract reflects the developers’ commitment to the community. By reducing their fee from 1% to 0.5%—a 50% reduction—they redirect this portion to enhance reflection (auto-staking) rewards, increasing the reflection fee from 3.5% to 5.5% over time. This sharing mechanism benefits users by providing greater passive income, fostering trust, and ensuring the project’s long-term sustainability while maintaining transparency through events and clear fee structures, all while maintaining scalability for +20 million users. 

## Community and Governance
ZeroMoon is fully self-sustaining and autonomous.

---

## Roadmap and Future Developments
- **Expansion**: Expand to other networks and integration with additional DeFi platforms.

---

## License
- Licensed under the MIT License

