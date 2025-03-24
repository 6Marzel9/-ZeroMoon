# ZeroMoon Token

![ZeroMoon Logo](zeromoon.png)

---

## Table of Contents
- [Introduction](#introduction)
- [Tokenomics](#tokenomics)
- [Smart Contract](#smart-contract)
- [Functionality](#functionality)
- [Integration with PancakeSwap](#integration-with-pancakeswap)
- [Security and Audits](#security-and-audits)
- [Community and Governance](#community-and-governance)
- [Roadmap and Future Developments](#roadmap-and-future-developments)
- [License](#license)

---

## Introduction
ZeroMoon is a decentralized token on BSC that rewards holders through automatic reflection, reduces supply via burning, and enhances liquidity on PancakeSwap V2. Built with autonomy in mind, it requires no manual intervention post-deployment, leveraging smart contract logic to manage rewards, burns, liquidity, and developer support.

---

## Tokenomics
ZeroMoon’s tokenomics are designed to balance rewards, scarcity, and ecosystem sustainability:

- **Total Supply**: 1,000,000,000 ZRO (1 billion ZRO)
  - Defined as `ORIGINAL_SUPPLY = 1_000_000_000 * 10**18`.

- **Burn Mechanism**: Burns tokens until 200,000,000 ZRO (20% of supply) are removed.
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

- **LP Deposit Limit**: Stops liquidity addition after 800,000,000 ZRO (80% of supply) are deposited into PancakeSwap.
  - Defined as `LP_DEPOSIT_LIMIT = ORIGINAL_SUPPLY * 80 / 100`.

- **Reflection Eligibility**: Holders must have a balance worth at least 0.1 BNB (`MIN_BNB_VALUE_FOR_REFLECTION = 0.1 ether`).

---

## Smart Contract
The ZeroMoon smart contract is written in **Solidity ^0.8.20** and leverages **OpenZeppelin** libraries for security and standardization:

- **Inheritance**:
  - `ERC20`: Standard token functionality.
  - `Ownable`: Ownership controls (e.g., fee exemptions, dev wallet settings).
  - `ReentrancyGuard`: Protection against reentrancy attacks.

- **Key Features**:
  - **Reflection**: Distributes rewards to holders via a scaling factor (`_scalingFactor`) and scaled balances (`_scaledBalances`).
  - **Burning**: Transfers tokens to `deadWallet` (`0x000...dEaD`) until `BURN_LIMIT` is reached.
  - **Liquidity**: Accumulates fees in `_accumulatedLiquidityTokens` and adds them to PancakeSwap V2 via `_addLiquidityAutomatically`.
  - **Developer Fee**: Allocates fees to `devWallet` or up to 10 `devWallets` (set via `setDevWallets`).
  - **Fee Exemptions**: Managed via `_isExcludedFromFee` mapping (e.g., contract, owner, burn, pre-sale addresses).

- **Constants**:
  - `TOTAL_FEE = 10` (10% fee before adjustments).
  - `FEE_DENOMINATOR = 10000` (basis points precision).

---

## Functionality
ZeroMoon operates autonomously with the following core features, illustrated with code examples from the contract:

- **Reflection**:
  - A portion of each transaction is redistributed to holders via `_applyReflectionTokens`, adjusting the `_scalingFactor`:
    ```solidity
    _scalingFactor = _scalingFactor * (_totalSupply - _burnedTokens) /
                     (_totalSupply - _burnedTokens + reflectionPool);
    ```
  - Eligibility requires a balance worth at least 1 BNB, checked via:
    ```solidity
    function isEligibleForReflection(address account) public view returns (bool) {
        uint256 balance = balanceOf(account);
        uint256 zmrPerBNB = getZROPerBNB();
        if (zmrPerBNB == 0) return false;
        uint256 bnbValue = (balance * 1 ether) / zmrPerBNB;
        return bnbValue >= MIN_BNB_VALUE_FOR_REFLECTION;
    }
    ```

- **Burning**:  
  - **Process**:  
    Tokens are burned by transferring them to a designated burn address, known as the `deadWallet` (e.g., `0x000...dEaD`), until the total burned tokens reach the predefined `BURN_LIMIT` of 200 million ZRO. This burning process is integrated into the `_distributeFees` function, which is called during each transaction to handle fee distribution.

  - **How It Works**:  
    The burning mechanism follows a structured logic:  
    - **Condition Check**:  
      Burning only occurs if:  
      - The `_burnStop` flag is `false` (indicating the burn limit has not yet been reached).  
      - There is a positive `burnAmount` (a portion of the transaction fee designated for burning).  
    - **Burn Execution**:  
      The tokens allocated for burning are sent to the `deadWallet` by increasing its scaled balance:  
      ```solidity
      _scaledBalances[deadWallet] += burnAmount * scalingFactor;
      ```  
    - **Tracking**:  
      The total number of burned tokens is tracked using the `_burnedTokens` variable, which increments with each burn:  
      ```solidity
      _burnedTokens += burnAmount;
      ```  
    - **Event Logging**:  
      A `Transfer` event is emitted to publicly log the burn on the blockchain:  
      ```solidity
      emit Transfer(address(this), deadWallet, burnAmount);
      ```  
    - **Burn Limit Check**:  
      After each burn, the contract compares `_burnedTokens` to the `BURN_LIMIT`. If the limit is reached or exceeded:  
      - The `_burnStop` flag is set to `true`, halting further burns.  
      - A `BurnLimitReached` event is emitted:  
        ```solidity
        if (_burnedTokens >= BURN_LIMIT) {
            _burnStop = true;
            emit BurnLimitReached(_burnedTokens);
        }
        ```

- **Post-Burn Limit**:  
    Once the `BURN_LIMIT` is reached, burning ceases entirely. The portion of the transaction fee previously used for burning (2.5%) is then redirected to reflection (increased to 4.5%) and liquidity (increased to 4.75%), while the developer fee is reduced to 0.75%, benefiting the community.

  - **Meaning for Users**:  
    - **Supply Reduction**:  
      Burning permanently removes tokens from circulation, reducing the total supply of ZRO tokens from 1 billion to 800 million. This can increase scarcity and potentially enhance the token’s value over time.  
    - **Controlled Deflation**:  
      The `BURN_LIMIT` caps the amount of tokens that can be burned, ensuring that deflation is controlled and a sufficient token supply remains for trading and utility within the ecosystem.  
    - **Transparency**:  
      The process is fully transparent, with events and variables like `_burnedTokens` publicly accessible, allowing anyone to verify the total burned amount on the blockchain.

- **Automated Liquidity**:
  - **Process**: A portion of each transaction fee is collected and stored in `_accumulatedLiquidityTokens`. When this amount reaches the `liquidityThreshold` and the contract holds enough BNB, the `_addLiquidityAutomatically()` function activates:
    ```solidity
    if (_accumulatedLiquidityTokens >= liquidityThreshold && address(this).balance >= MIN_BNB_BALANCE) {
        _addLiquidityAutomatically();
    }
    ```
  - **How It Works**: 
    - The contract converts half of the accumulated ZRO tokens into BNB.
    - It then pairs the remaining ZRO tokens with this BNB and adds them to the ZRO/BNB liquidity pool on PancakeSwap V2.
    - This continues until the total ZRO deposited into the pool hits the `LP_DEPOSIT_LIMIT` of 800 million ZRO.
  - **LP Tokens Burned**: 
    - After adding liquidity, the resulting LP tokens (which represent ownership of the pool) are sent to the `deadWallet` (`0x000000000000000000000000000000000000dEaD`), a burn address. This process is handled in the `addLiquidityETH()` function:
      ```solidity
      pancakeSwapV2Router.addLiquidityETH{value: bnbReceived}(
          address(this),
          lpTokenAmount,
          0,
          0,
          deadWallet,  // LP tokens sent here
          block.timestamp + 300
      );
      ```
    - Sending LP tokens to this address burns them permanently, as no one can access or control assets sent there. This ensures all liquidity added is locked forever.
  - **Funding Source**: 
    - The ZRO tokens used for these LP deposits come directly from transaction fees. In the contract, `_calculateFees()` allocates a percentage of each trade (e.g., 3% before the burn limit) to `_accumulatedLiquidityTokens`, which is later used for liquidity:
      ```solidity
      _accumulatedLiquidityTokens += liquidityAmount;
      ```
    - This means liquidity is extracted from the fee structure, relying on community trading activity rather than pre-mined tokens.  
    - Since fees fund the liquidity, every transaction helps grow the pool, making it a self-sustaining system driven by the community.

- **Developer Support**:
  - Fees are sent to `devWallet` or split among `devWallets` (if configured) in `_distributeFees`:
    ```solidity
    if (devWallets.length == DEV_WALLETS_COUNT) {
        uint256 devShare = devAmount / DEV_WALLETS_COUNT;
        for (uint256 i = 0; i < DEV_WALLETS_COUNT; i++) {
            _scaledBalances[devWallets[i]] += devShare * scalingFactor;
            emit Transfer(address(this), devWallets[i], devShare);
        }
    } else {
        _scaledBalances[devWallet] += devAmount * scalingFactor;
        emit Transfer(address(this), devWallet, devAmount);
    }
    ```

---

## Integration with PancakeSwap
ZeroMoon integrates seamlessly with **PancakeSwap V2**:
- **Trading Pair**: ZRO/WBNB
- **Automated Liquidity**: Periodically adds liquidity to the pair via `_addLiquidityAutomatically`.
- **Swaps**: Supports trading via PancakeSwap’s DEX with fee-on-transfer mechanics.

---

## Security and Audits
- **OpenZeppelin Libraries**: Uses audited `ERC20`, `Ownable`, and `ReentrancyGuard` for security.
- **Reentrancy Protection**: Applied to key functions like `_addLiquidityAutomatically` and `_manageBalances`.

---

## Community and Governance
ZeroMoon is fully self-sustaining and autonomous.

---

## Roadmap and Future Developments
- **Expansion**: Expand to other networks and integration with additional DeFi platforms.

---

## License
- Licensed under the MIT License
