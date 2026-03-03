# Indie Protocol — Smart Contract Security Audit

**Date**: February 2026
**Scope**: All 12 production contracts + 1 extension
**Method**: Static analysis, pattern matching, cross-contract interaction review
**Contracts Reviewed**: InUSDToken, InUSDCTreasury, TokenExchange, ServiceNFT, ServicePass, Marketplace, BillingExecutor, PlatformConfig, ServiceFactory, ServiceRegistry, ServiceAccountImpl, X402Receiver, ERC20TransferWithAuthorization

---

## Executive Summary

A comprehensive security review of the Indie Protocol smart contracts identified **21 findings** across 5 severity levels. One critical vulnerability in the treasury withdrawal function could allow reserve insolvency. Three high-severity issues affect marketplace safety, executor trust assumptions, and fund recovery. Seven medium-severity findings involve reentrancy risks, CEI violations, and front-running vectors.

The protocol's architecture is sound, with good use of OpenZeppelin v5 primitives, role-based access control, and established ERC/EIP standards. The findings below represent areas for hardening before mainnet deployment.

| Severity | Count |
|----------|-------|
| Critical | 1 |
| High | 3 |
| Medium | 7 |
| Low | 6 |
| Info | 4 |

---

## Critical Findings

### C-1: `adminWithdraw()` Has No Solvency Check

**Contract**: `InUSDCTreasury.sol:165-177`
**Impact**: Admin can drain USDC reserves below the amount needed to honor inUSD redemptions, breaking the 90% backing guarantee.

**Evidence**: The `adminWithdraw` function transfers USDC to an arbitrary address without verifying that remaining reserves can cover outstanding inUSD supply at the redemption rate.

**Recommendation**: Add a solvency guard:
```solidity
uint256 requiredReserve = (inusdToken.totalSupply() * redemptionRate) / RATE_DENOMINATOR;
require(usdc.balanceOf(address(this)) - amount >= requiredReserve, "Would break solvency");
```

---

## High Findings

### H-1: Paused/Delisted Services Can Still Be Purchased

**Contract**: `Marketplace.sol:147-165`
**Impact**: Users can purchase instances of services that creators have paused or that have been delisted from the marketplace.

**Evidence**: The `purchase()` function does not check `serviceNFT.paused()` or the marketplace listing status before processing the purchase and minting a new instance.

**Recommendation**: Add pre-purchase validation:
```solidity
require(isListed[service], "Service not listed");
require(!IServiceNFT(service).paused(), "Service paused");
```

### H-2: Executor Can Lock TBAs and Drain All inUSD

**Contract**: `ServiceAccountImpl.sol` + `ServiceNFT.sol`
**Impact**: The executor address (set in PlatformConfig) can lock any service instance's TBA and then transfer all inUSD to whitelisted addresses with no on-chain spending limit or rate control.

**Evidence**: In locked state, `ServiceAccountImpl.execute()` only checks that `msg.sender == executor` and that the target is a whitelisted address. There is no per-execution cap, daily limit, or relationship to the service's `executionPrice`.

**Recommendation**:
- Add per-execution spending limits derived from `maxExecutionPrice`
- Implement daily withdrawal caps on locked TBAs
- Consider requiring the executor to submit batch settlements through `BillingExecutor` rather than direct TBA transfers

### H-3: Burned NFT Permanently Bricks Its TBA

**Contract**: `ServiceAccountImpl.sol:129-131`
**Impact**: If a ServiceNFT token is burned, its Token-Bound Account becomes permanently inaccessible. Any inUSD or other tokens in the TBA are locked forever.

**Evidence**: `isValidSigner()` calls `IERC721(tokenContract).ownerOf(tokenId)`, which reverts for burned tokens (ERC-721 standard behavior). This means no address can pass the signer check, and the TBA's `execute()` function becomes uncallable.

**Recommendation**:
- Prevent burning NFTs with non-zero TBA balances: add a check in `_update()` that requires the TBA balance to be zero before allowing burns
- Or: add a rescue function that allows fund recovery from TBAs of burned tokens (e.g., controlled by protocol admin)

---

## Medium Findings

### M-1: EIP-3009 Front-Running Can Strand Funds

**Contract**: `TokenExchange.sol`
**Impact**: A `transferWithAuthorization` call can be front-run by a MEV bot that submits the same authorization. The USDC transfers to the exchange contract, but the original swap transaction reverts because the authorization nonce is consumed. The user's USDC is stuck in the contract.

**Recommendation**: Add a sweep/recovery function for stranded USDC, or use `receiveWithAuthorization` which is front-run resistant (only the designated recipient can execute it).

### M-2: Relayer Can Misdirect Gasless Purchase

**Contract**: `Marketplace.sol`
**Impact**: In gasless purchase flows, the user signs the payment authorization (amount + nonce), but the relayer constructs the actual `purchase()` call including the service address. A malicious relayer could redirect the purchase to a different service.

**Recommendation**: Include the service address in the user's signed payload so the relayer cannot substitute it.

### M-3: CEI Violation in `renewSubscription`

**Contract**: `ServiceNFT.sol`
**Impact**: The `renewSubscription` function calls `inusd.transferFrom()` (external call) before updating the subscription expiry state. This violates the Checks-Effects-Interactions pattern.

**Recommendation**: Move the subscription expiry update before the transfer call, or add `nonReentrant` modifier (already imported via OpenZeppelin).

### M-4: `ServiceAccountImpl` Missing ReentrancyGuard

**Contract**: `ServiceAccountImpl.sol`
**Impact**: The `execute()` and `batchExecute()` functions make arbitrary external calls without reentrancy protection. A malicious target contract could re-enter the TBA.

**Recommendation**: Add OpenZeppelin's `ReentrancyGuard` and apply `nonReentrant` to `execute()` and `batchExecute()`.

### M-5: BillingExecutor Batch ID Collision

**Contract**: `BillingExecutor.sol`
**Impact**: Batch IDs are sequential (`nextBatchId++`). If the executor is restarted or redeployed, batch IDs could collide with historical records, overwriting settlement data.

**Recommendation**: Use a hash-based batch ID (e.g., `keccak256(abi.encode(block.number, msg.sender, batchData))`) or include a domain separator.

### M-6: `ServicePass.purchase()` Missing ReentrancyGuard

**Contract**: `ServicePass.sol`
**Impact**: The purchase function performs external calls (token transfers) without reentrancy protection.

**Recommendation**: Add `nonReentrant` modifier to `purchase()`.

### M-7: `treasuryBurn` Can Burn From Arbitrary Address

**Contract**: `InUSDToken.sol`
**Impact**: An address with the TREASURY role can call `burnFrom` on any address that has granted an allowance, not just the treasury's own balance.

**Recommendation**: Restrict `treasuryBurn` to only burn from `msg.sender` or from the treasury address itself.

---

## Low Findings

### L-1: Backing Check Formula Mismatch

**Contract**: `InUSDToken.sol`
**Impact**: The mint-time backing check validates 90% USDC backing, but users expect 90% redemption at any time. If minting continues while reserves are drawn down by exits, the actual backing ratio can fall below 90%.

**Recommendation**: Consider a real-time solvency check on mint that accounts for outstanding supply, not just the current mint amount.

### L-2: Admin Can Neutralize Flash Loan Protections

**Contract**: `InUSDToken.sol`
**Impact**: `setMaxMintPerTransaction` and `setMaxDailyMint` have no lower bounds. Admin could set these to very high values, effectively removing flash loan size limits.

**Recommendation**: Add minimum floor values for mint caps.

### L-3: First-Come-First-Served Redemption Race

**Contract**: `InUSDCTreasury.sol`
**Impact**: With 90% backing, if all holders attempt to redeem simultaneously, late redeemers may find insufficient reserves. This is a bank-run scenario inherent to the fractional reserve design.

**Recommendation**: Document this risk clearly. Consider implementing a redemption queue or cooldown period for large redemptions.

### L-4: `listService`/`delistService` Lack `nonReentrant`

**Contract**: `Marketplace.sol`
**Impact**: These functions modify state and interact with external contracts (ServicePass burn) without reentrancy protection.

**Recommendation**: Add `nonReentrant` modifier.

### L-5: `boostService` Unbounded External Calls

**Contract**: `Marketplace.sol`
**Impact**: If boost logic involves loops with external calls, gas consumption could exceed block limits, causing transactions to fail.

**Recommendation**: Cap iteration count or use pull-based patterns.

### L-6: External Registry Calls During NFT Transfer Could Freeze Tokens

**Contract**: `ServiceNFT.sol` + `ServiceRegistry.sol`
**Impact**: The `_update()` hook in ServiceNFT calls the external ServiceRegistry to update ownership. If the registry reverts (e.g., due to a bug or pause), all NFT transfers are blocked.

**Recommendation**: Wrap the registry call in a try/catch, or use an event-based pattern where the registry updates asynchronously.

---

## Informational Findings

### I-1: Subscription Payments Bypass Platform Fee

**Contract**: `ServiceNFT.sol`
**Impact**: `renewSubscription` sends the full subscription price directly from the buyer to the creator. Unlike marketplace purchases, there is no platform fee deduction. This is likely intentional but should be documented.

### I-2: No Refund Mechanism on Subscription Cancellation

**Contract**: `ServiceNFT.sol`
**Impact**: If a user's subscription is active and they stop using the service, there is no pro-rata refund mechanism. The full 30-day payment is non-refundable.

### I-3: ServicePass Tokens Are Transferable

**Contract**: `ServicePass.sol`
**Impact**: Listing and Feature passes are standard ERC-1155 tokens with no transfer restrictions. A secondary market could emerge, potentially undermining the protocol's pricing model for marketplace access.

### I-4: PlatformConfig Has No Timelock or Multi-Sig

**Contract**: `PlatformConfig.sol`
**Impact**: A single admin address can change all protocol contract addresses (executor, treasury, marketplace, etc.) in a single transaction with no delay. This creates a single point of failure and trust assumption.

**Recommendation**: Implement a timelock (e.g., 48-hour delay) for configuration changes, and require multi-sig approval.

---

## Recommendations Summary

### Before Mainnet (Must Fix)

1. **Add solvency check to `adminWithdraw()`** (C-1)
2. **Check service pause/listing status in `Marketplace.purchase()`** (H-1)
3. **Add spending limits for executor on locked TBAs** (H-2)
4. **Prevent burning NFTs with non-zero TBA balances** (H-3)
5. **Add `ReentrancyGuard` to `ServiceAccountImpl`** (M-4)
6. **Fix CEI violation in `renewSubscription`** (M-3)

### Before Mainnet (Recommended)

7. **Use `receiveWithAuthorization` to prevent front-running** (M-1)
8. **Include service address in gasless purchase signatures** (M-2)
9. **Add `nonReentrant` to `ServicePass.purchase()`** (M-6)
10. **Use hash-based batch IDs in `BillingExecutor`** (M-5)
11. **Restrict `treasuryBurn` target address** (M-7)

### Post-Launch Hardening

12. **Add timelock + multi-sig to `PlatformConfig`** (I-4)
13. **Add `nonReentrant` to `listService`/`delistService`** (L-4)
14. **Add minimum floor values for mint caps** (L-2)
15. **Document fractional reserve redemption risk** (L-3)
16. **Wrap registry calls in try/catch** (L-6)

---

*This audit was performed through static analysis and cross-contract interaction review. A formal audit by a specialized firm (e.g., OpenZeppelin, Trail of Bits, Cyfrin) is recommended before mainnet deployment with significant TVL.*
