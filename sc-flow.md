# Match Flow Diagrams for Trading.sol

This document provides comprehensive visual diagrams showing how the three match types work in the CTF Exchange.

## Table of Contents
- [1. COMPLEMENTARY Match Flow](#1-complementary-match-flow-buy-vs-sell)
- [2. MINT Match Flow](#2-mint-match-flow-buy-vs-buy)
- [3. MERGE Match Flow](#3-merge-match-flow-sell-vs-sell)
- [4. Complete Match Orders Flow](#4-complete-match-orders-flow)
- [5. Asset Flow by Match Type](#5-asset-flow-by-match-type)
- [6. Fee Collection Flow](#6-fee-collection-flow)

---

## 1. COMPLEMENTARY Match Flow (BUY vs SELL)

```
┌─────────────────────────────────────────────────────────────────┐
│                  COMPLEMENTARY MATCH                            │
│              (BUY Order vs SELL Order)                          │
└─────────────────────────────────────────────────────────────────┘

Input:
  Taker Order: BUY YES @ price P
  Maker Order: SELL YES @ price P

┌──────────────┐
│  STEP 1:     │  Pull Taker's collateral
│  Initialize  │
└──────┬───────┘
       │
       │  Taker gives: USDC
       ▼
┌─────────────────┐
│   EXCHANGE      │◄──── USDC from Taker
│   (Holds USDC)  │
└─────────┬───────┘
          │
┌─────────▼────────────────────────────────────────────┐
│  STEP 2: Fill Maker Order                           │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [a] Maker → Exchange: YES tokens          │     │
│  │     (Maker provides YES tokens to sell)   │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [b] CTF Operation: COMPLEMENTARY          │     │
│  │     NO operation needed - direct swap     │     │
│  │     Exchange already has USDC from Taker  │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [c] Exchange → Maker: USDC - fee          │     │
│  │     (Maker receives payment for YES)      │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [d] Fee Collection: fee → Operator        │     │
│  └───────────────────────────────────────────┘     │
└───────────────────────────────────────────────────┘
          │
          │  Exchange now has YES tokens
          ▼
┌─────────────────┐
│  STEP 3:        │  Calculate final amounts
│  Finalize       │  taking = surplus amount
└─────────┬───────┘
          │
          │  Send YES tokens to Taker
          ▼
┌─────────────────┐
│  STEP 4:        │  Exchange → Taker: YES - fee
│  Distribute     │  Fee → Operator
└─────────┬───────┘  Refund excess USDC
          │
          ▼
┌─────────────────┐
│   RESULT:       │
│   Taker: -USDC  │  Taker paid USDC
│          +YES   │  Taker got YES tokens
│   Maker: +USDC  │  Maker got USDC
│          -YES   │  Maker sold YES tokens
└─────────────────┘

Fund Flow Summary:
═══════════════════
Taker:  USDC ──────► Exchange ──────► Maker (payment)
Maker:  YES  ──────► Exchange ──────► Taker (tokens)
Fees:   From both USDC & YES ──────► Operator
```

**Key Characteristics:**
- Direct token swap between buyer and seller
- No CTF operations needed (no minting or merging)
- Exchange acts as intermediary holding funds temporarily
- Both parties trade the same outcome token

---

## 2. MINT Match Flow (BUY vs BUY)

```
┌─────────────────────────────────────────────────────────────────┐
│                       MINT MATCH                                │
│          (Both BUY Orders - Different Tokens)                   │
└─────────────────────────────────────────────────────────────────┘

Input:
  Taker Order: BUY YES @ price P
  Maker Order: BUY NO @ price P

┌──────────────┐
│  STEP 1:     │  Pull Taker's collateral
│  Initialize  │
└──────┬───────┘
       │
       │  Taker gives: USDC
       ▼
┌─────────────────┐
│   EXCHANGE      │◄──── USDC from Taker
│   (Holds USDC)  │
└─────────┬───────┘
          │
┌─────────▼────────────────────────────────────────────┐
│  STEP 2: Fill Maker Order                           │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [a] Maker → Exchange: USDC                │     │
│  │     (Maker provides collateral to buy NO) │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [b] CTF Operation: MINT                   │     │
│  │     Exchange calls CTF.splitPosition()    │     │
│  │     USDC → [YES tokens + NO tokens]       │     │
│  │                                            │     │
│  │     Exchange splits its USDC balance      │     │
│  │     Creates both outcome tokens           │     │
│  └───────────────────────────────────────────┘     │
│                     ↓                                │
│              ┌──────────────┐                       │
│              │  YES tokens  │                       │
│              │   +  +  +    │                       │
│              │  NO tokens   │                       │
│              └──────────────┘                       │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [c] Exchange → Maker: NO - fee            │     │
│  │     (Maker receives NO tokens bought)     │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [d] Fee Collection: fee → Operator        │     │
│  └───────────────────────────────────────────┘     │
└───────────────────────────────────────────────────┘
          │
          │  Exchange now has YES tokens
          ▼
┌─────────────────┐
│  STEP 3:        │  Calculate final amounts
│  Finalize       │  taking = YES tokens available
└─────────┬───────┘
          │
          │  Send YES tokens to Taker
          ▼
┌─────────────────┐
│  STEP 4:        │  Exchange → Taker: YES - fee
│  Distribute     │  Fee → Operator
└─────────┬───────┘  Refund excess USDC
          │
          ▼
┌─────────────────┐
│   RESULT:       │
│   Taker: -USDC  │  Taker paid USDC
│          +YES   │  Taker got YES tokens
│   Maker: -USDC  │  Maker paid USDC
│          +NO    │  Maker got NO tokens
└─────────────────┘

Fund Flow Summary:
═══════════════════
Taker:  USDC ──────► Exchange
Maker:  USDC ──────► Exchange
                     │
Exchange: MINT ──────┤
                     ├──► YES tokens ──► Taker
                     └──► NO tokens  ──► Maker
Fees:   From YES & NO tokens ──────► Operator
```

**Key Characteristics:**
- Both parties are buyers wanting different outcome tokens
- Exchange performs CTF splitPosition to create complete set
- Collateral (USDC) is split into YES + NO outcome tokens
- Each buyer receives their desired outcome token
- Net effect: Complete set is purchased and distributed

---

## 3. MERGE Match Flow (SELL vs SELL)

```
┌─────────────────────────────────────────────────────────────────┐
│                      MERGE MATCH                                │
│         (Both SELL Orders - Different Tokens)                   │
└─────────────────────────────────────────────────────────────────┘

Input:
  Taker Order: SELL YES @ price P
  Maker Order: SELL NO @ price P

┌──────────────┐
│  STEP 1:     │  Pull Taker's outcome tokens
│  Initialize  │
└──────┬───────┘
       │
       │  Taker gives: YES tokens
       ▼
┌─────────────────┐
│   EXCHANGE      │◄──── YES from Taker
│  (Holds YES)    │
└─────────┬───────┘
          │
┌─────────▼────────────────────────────────────────────┐
│  STEP 2: Fill Maker Order                           │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [a] Maker → Exchange: NO tokens           │     │
│  │     (Maker provides NO tokens to sell)    │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [b] CTF Operation: MERGE                  │     │
│  │     Exchange calls CTF.mergePositions()   │     │
│  │     [YES tokens + NO tokens] → USDC       │     │
│  │                                            │     │
│  │     Exchange merges its outcome tokens    │     │
│  │     Burns both tokens, returns collateral │     │
│  └───────────────────────────────────────────┘     │
│                     ↓                                │
│              ┌──────────────┐                       │
│              │     USDC     │                       │
│              │  (collateral)│                       │
│              └──────────────┘                       │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [c] Exchange → Maker: USDC - fee          │     │
│  │     (Maker receives payment for NO)       │     │
│  └───────────────────────────────────────────┘     │
│                                                       │
│  ┌───────────────────────────────────────────┐     │
│  │ [d] Fee Collection: fee → Operator        │     │
│  └───────────────────────────────────────────┘     │
└───────────────────────────────────────────────────┘
          │
          │  Exchange now has USDC
          ▼
┌─────────────────┐
│  STEP 3:        │  Calculate final amounts
│  Finalize       │  taking = USDC available
└─────────┬───────┘
          │
          │  Send USDC to Taker
          ▼
┌─────────────────┐
│  STEP 4:        │  Exchange → Taker: USDC - fee
│  Distribute     │  Fee → Operator
└─────────┬───────┘  Refund excess YES
          │
          ▼
┌─────────────────┐
│   RESULT:       │
│   Taker: +USDC  │  Taker got USDC
│          -YES   │  Taker sold YES tokens
│   Maker: +USDC  │  Maker got USDC
│          -NO    │  Maker sold NO tokens
└─────────────────┘

Fund Flow Summary:
═══════════════════
Taker:  YES tokens ──────► Exchange
Maker:  NO tokens  ──────► Exchange
                           │
Exchange: MERGE ───────────┤
                           └──► USDC ──► Taker & Maker
Fees:   From USDC payments ──────► Operator
```

**Key Characteristics:**
- Both parties are sellers with different outcome tokens
- Exchange performs CTF mergePositions to redeem collateral
- YES + NO tokens are burned to recover USDC collateral
- Each seller receives their share of the collateral
- Net effect: Complete set is redeemed for collateral

---

## 4. Complete Match Orders Flow

```
┌══════════════════════════════════════════════════════════════════┐
║              MATCH ORDERS ORCHESTRATION FLOW                     ║
╚══════════════════════════════════════════════════════════════════╝

Entry Point: _matchOrders(takerOrder, makerOrders[], takerFillAmount, makerFillAmounts[])

┌─────────────────────────────────────────────────────────────────┐
│ PHASE 0: Input Validation & Logging                            │
├─────────────────────────────────────────────────────────────────┤
│ • Log all order parameters (taker + all makers)                 │
│ • Display sides, tokens, amounts, fees                          │
│ • Validate taker order (hash, signature, nonce, expiration)     │
│ • Calculate taker's taking amount                               │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: Initialize                                             │
├─────────────────────────────────────────────────────────────────┤
│ Pull funds from Taker:                                          │
│   Taker.maker → Exchange                                        │
│   Amount: takerFillAmount                                       │
│   Token: makerAssetId (USDC if BUY, outcome token if SELL)     │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: Fill Maker Orders (Loop through each maker)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ For each makerOrder in makerOrders[]:                          │
│                                                                  │
│ ┌────────────────────────────────────────────────────────┐    │
│ │ 2.1: Determine Match Type                              │    │
│ │      • COMPLEMENTARY: Taker BUY + Maker SELL (or vice) │    │
│ │      • MINT: Both BUY (different tokens)               │    │
│ │      • MERGE: Both SELL (different tokens)             │    │
│ └────────────────────────────────────────────────────────┘    │
│                                                                  │
│ ┌────────────────────────────────────────────────────────┐    │
│ │ 2.2: Validate Orders Can Match                         │    │
│ │      • Check price crossing                            │    │
│ │      • Validate token IDs match or complement          │    │
│ │      • Verify maker order (hash, sig, nonce, exp)      │    │
│ └────────────────────────────────────────────────────────┘    │
│                                                                  │
│ ┌────────────────────────────────────────────────────────┐    │
│ │ 2.3: Calculate Amounts & Fees                          │    │
│ │      making = makerFillAmounts[i]                      │    │
│ │      taking = (making * takerAmount) / makerAmount     │    │
│ │      fee = calculated based on side and fee rate       │    │
│ └────────────────────────────────────────────────────────┘    │
│                                                                  │
│ ┌────────────────────────────────────────────────────────┐    │
│ │ 2.4: Fill Facing Exchange                              │    │
│ │                                                         │    │
│ │  [a] Pull from Maker                                   │    │
│ │      Maker → Exchange: makerAssetId (making amount)    │    │
│ │                                                         │    │
│ │  [b] Execute CTF Operation                             │    │
│ │      • COMPLEMENTARY: No-op (tokens already available) │    │
│ │      • MINT: Exchange calls _mint(conditionId, amount) │    │
│ │              Splits USDC → YES + NO tokens             │    │
│ │      • MERGE: Exchange calls _merge(conditionId,amount)│    │
│ │               Merges YES + NO → USDC                   │    │
│ │                                                         │    │
│ │  [c] Verify Sufficient Tokens                          │    │
│ │      Check: _getBalance(takerAssetId) >= taking        │    │
│ │                                                         │    │
│ │  [d] Send to Maker (net of fee)                        │    │
│ │      Exchange → Maker: takerAssetId (taking - fee)     │    │
│ │                                                         │    │
│ │  [e] Collect Fee                                       │    │
│ │      Exchange → Operator: fee                          │    │
│ └────────────────────────────────────────────────────────┘    │
│                                                                  │
│ Repeat for all maker orders...                                  │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 3: Calculate Final Amounts                               │
├─────────────────────────────────────────────────────────────────┤
│ • Get actual balance: actualTaking = _getBalance(takerAssetId)  │
│ • Verify: actualTaking >= expectedTaking (from Phase 0)         │
│ • Calculate taker fee: fee = f(takerOrder.feeRateBps, amounts)  │
│ • Update taking to actual surplus amount                        │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 4: Distribute to Taker                                   │
├─────────────────────────────────────────────────────────────────┤
│ • Exchange → Taker: takerAssetId (taking - fee)                 │
│ • Exchange → Operator: fee                                      │
│ • Calculate refund: _getBalance(makerAssetId)                   │
│ • If refund > 0: Exchange → Taker: makerAssetId (refund)        │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 5: Emit Events & Log Results                             │
├─────────────────────────────────────────────────────────────────┤
│ • OrderFilled event for taker                                   │
│ • OrdersMatched event                                           │
│ • Log final summary (gave, got, fees, refunds)                  │
└─────────────────────────────────────────────────────────────────┘

Exit: Match complete
```

**Important Notes:**
- The Exchange acts as the central coordinator
- All maker orders are filled sequentially in a loop
- The taker's position is filled by aggregating all maker fills
- Surplus tokens from MINT/MERGE operations go to the taker
- All fees are collected before distributing assets to users

---

## 5. Asset Flow by Match Type

```
═══════════════════════════════════════════════════════════════════
                    ASSET DERIVATION RULES
═══════════════════════════════════════════════════════════════════

BUY Order:
  makerAsset = 0 (USDC/collateral)
  takerAsset = order.tokenId (YES or NO outcome token)

  User provides: USDC collateral
  User receives: Outcome tokens

SELL Order:
  makerAsset = order.tokenId (YES or NO outcome token)
  takerAsset = 0 (USDC/collateral)

  User provides: Outcome tokens
  User receives: USDC collateral

═══════════════════════════════════════════════════════════════════
              MATCH TYPE DETERMINATION MATRIX
═══════════════════════════════════════════════════════════════════

Taker Side │ Maker Side │ Match Type     │ CTF Operation
───────────┼────────────┼────────────────┼──────────────────────
BUY        │ BUY        │ MINT           │ Split collateral
           │            │                │ (create YES + NO)
───────────┼────────────┼────────────────┼──────────────────────
SELL       │ SELL       │ MERGE          │ Merge positions
           │            │                │ (burn YES + NO)
───────────┼────────────┼────────────────┼──────────────────────
BUY        │ SELL       │ COMPLEMENTARY  │ None (direct swap)
───────────┼────────────┼────────────────┼──────────────────────
SELL       │ BUY        │ COMPLEMENTARY  │ None (direct swap)
═══════════════════════════════════════════════════════════════════

Additional Rules:
• COMPLEMENTARY: Both orders trade the SAME outcome token
• MINT: Both orders trade COMPLEMENTARY tokens (YES vs NO)
• MERGE: Both orders trade COMPLEMENTARY tokens (YES vs NO)
```

**Code Reference:**
```solidity
// From Trading.sol:397-406
function _deriveMatchType(Order memory takerOrder, Order memory makerOrder)
    internal pure returns (MatchType) {
    if (takerOrder.side == Side.BUY && makerOrder.side == Side.BUY)
        return MatchType.MINT;
    if (takerOrder.side == Side.SELL && makerOrder.side == Side.SELL)
        return MatchType.MERGE;
    return MatchType.COMPLEMENTARY;
}

function _deriveAssetIds(Order memory order)
    internal pure returns (uint256 makerAssetId, uint256 takerAssetId) {
    if (order.side == Side.BUY) return (0, order.tokenId);
    return (order.tokenId, 0);
}
```

---

## 6. Fee Collection Flow

```
┌═══════════════════════════════════════════════════════════════┐
║                    FEE COLLECTION FLOW                        ║
╚═══════════════════════════════════════════════════════════════╝

Fees are charged on the "receiving" side of each order:
• BUY orders: Fee on outcome tokens received
• SELL orders: Fee on USDC received

┌──────────────────────────────────────────────────────────────┐
│ Maker Order Fee Collection                                   │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│ In _fillFacingExchange():                                    │
│   [c] Exchange → Maker: (takingAmount - fee)                 │
│   [d] Exchange → Operator: fee                               │
│                                                               │
│ Fee is deducted before sending to maker                      │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│ Taker Order Fee Collection                                   │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│ In _matchOrders() Phase 4:                                   │
│   Exchange → Taker: (taking - fee)                           │
│   _chargeFee(Exchange, Operator, takerAssetId, fee)          │
│                                                               │
│ Fee is deducted before sending to taker                      │
└──────────────────────────────────────────────────────────────┘

Fee Calculation:
  feeAmount = (receivingAmount * feeRateBps) / 10000

  Where:
    • BUY: receivingAmount = outcome tokens received
    • SELL: receivingAmount = USDC received
    • feeRateBps: Fee rate in basis points (1 bps = 0.01%)
```

**Example Fee Calculation:**
```
BUY Order with 1% fee (100 bps):
  User buys 100 YES tokens
  Fee = (100 * 100) / 10000 = 1 YES token
  User receives: 100 - 1 = 99 YES tokens
  Operator receives: 1 YES token

SELL Order with 1% fee (100 bps):
  User sells 100 YES tokens for 100 USDC
  Fee = (100 * 100) / 10000 = 1 USDC
  User receives: 100 - 1 = 99 USDC
  Operator receives: 1 USDC
```

**Code Reference:**
```solidity
// From Trading.sol:463-473
function _chargeFee(address payer, address receiver, uint256 tokenId, uint256 fee)
    internal {
    if (fee > 0) {
        _transfer(payer, receiver, tokenId, fee);
        emit FeeCharged(receiver, tokenId, fee);
    }
}

// Fee calculation in CalculatorHelper.sol
function calculateFee(
    uint256 feeRateBps,
    uint256 proceeds,
    uint256 makerAmount,
    uint256 takerAmount,
    Side side
) internal pure returns (uint256 fee)
```

---

## Summary

The CTF Exchange trading system supports three distinct match types, each optimized for different trading scenarios:

1. **COMPLEMENTARY**: Direct swaps between buyers and sellers of the same token
2. **MINT**: Enables two buyers to split a complete set of outcome tokens
3. **MERGE**: Allows two sellers to redeem their complementary tokens for collateral

All flows follow the same 5-phase orchestration pattern, with the Exchange acting as a trusted intermediary that:
- Validates all orders before execution
- Performs CTF operations when needed (MINT/MERGE)
- Collects fees from both makers and takers
- Ensures atomic execution of all transfers
- Returns surplus tokens to the taker

This design enables efficient orderbook-based trading while leveraging the Conditional Tokens Framework for position management.

