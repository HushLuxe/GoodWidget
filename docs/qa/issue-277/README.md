# QA Review & Defect Audit Report: Bridge-Widget Correctness & Styling

- **Target Bounty:** [Issue #277: [GoodBounty-QA]: Bridge-widget QA](https://github.com/GoodDollar/GoodWidget/issues/277)
- **Target PR:** [PR #266: Bridge widget fixes for issue #257](https://github.com/GoodDollar/GoodWidget/pull/266)
- **Base Issue:** [Issue #257: Bridging widget correctness and styling issues](https://github.com/GoodDollar/GoodWidget/issues/257)
- **Audited Commit SHA:** `8feadd7d59948c924a7d2195a1f556d9355ad4fd`
- **Installed Bridging SDK:** `@goodsdks/bridging-sdk@1.0.6`
- **Reviewer / QA Auditor:** `@HushLuxe`
- **Automated Test Results:** **38 / 38 Passed (100%)**
- **Live On-Chain Transfer:** **100% Confirmed** on Celo Mainnet ➔ Fuse via LayerZero
- **Final Verdict:** **CHANGES REQUESTED** (Core functional routing and contract interactions succeed, but 11 verified styling, contrast, and resilience defects require remediation).

---

## 1. Executive Summary & QA Verification Matrix (QA01–QA12)

Every acceptance criteria scenario defined in Issue #277 and base Issue #257 was systematically audited across desktop, tablet, and mobile viewports in both Light and Dark themes.

| Case ID | Scenario Name | Test Environment / Fixture | Status | Audit Findings & Verification Summary |
| :--- | :--- | :--- | :--- | :--- |
| **QA01** | First Use & Connection | Disconnected Fixture (`--no-wallet`) | **PASS** | Widget renders centered; placeholder `Balance: — G$` prevents flash of zero balance; `Connect Wallet` CTA is prominent; tabs switch cleanly. |
| **QA02** | Routes & Providers | Injected Wallet / Runtime Fixtures | **PASS (BUG-05)** | Axelar is strictly isolated to Celo ↔ Ethereum; LayerZero is active across all supported chains. Zero premature network switch dialogs on route selection. *(Provider selection card inherits pill radius and distorts into a pinched oval).* |
| **QA03** | Amounts & Fee Segregation | Injected Wallet / Runtime Fixtures | **PASS (BUG-07)** | Protocol fee is strictly segregated from native relayer fee (CELO). Route-specific rates verified: Fuse ➔ Celo quotes apply a 0.5% protocol fee (seen in review fixtures), while Celo ➔ Fuse quotes apply a 0.1% protocol fee (seen in live transfer); both calculations are accurate to contract config. *(16-decimal native provider fee string overflows container on small screens).* |
| **QA04** | Review & Confirm Flow | Injected Wallet / Runtime Fixtures | **PASS (BUG-03)** | Review drawer presents complete fee breakdown and deferred network switch notice. *(Modal title wraps to two lines on 320px screens).* |
| **QA05** | Rejection & Recovery | Injected Wallet / Mocked SDK | **PASS (BUG-08)** | User wallet rejections and on-chain reverts halt at the exact failed step, render error banner, and provide a working `Retry` button. *(Misleading approval error copy states bridge transaction reverted).* |
| **QA06** | Stale State & Interruptions | Automated Test Suite | **PASS** | In-flight request ID guards prevent stale quote overwrites. Rapid route and amount switching cancels pending requests cleanly without UI desynchronization. |
| **QA07** | Progress & Delivery | Injected Wallet / Live Fixtures | **PASS (BUG-01)** | Stepper tracks token approval and cross-chain relay progression. Clickable explorer links for source and provider. *(In Light Theme, explorer link titles fail to render, appearing only as empty underlines with icons).* |
| **QA08** | History Tab & Detail Modal | Injected Wallet / Live Fixtures | **PASS (BUG-06)** | Populated history tab displays live completed transactions. *(In Light Theme, secondary `"Refresh"` button text resolves pure white-on-white (`#FFFFFF` on `#FFFFFF`), invisible until hovered).* |
| **QA09** | Mobile & Accessibility | Mobile Viewports (390px, 320px) | **PARTIAL** | On 320px viewports, `Amount` and `Balance:` collide (`AmountBalance:`), modal title wraps to 2 lines, and provider cards pinch. Yellow warning text fails WCAG AA contrast. |
| **QA10** | Lead Design Comparison | Visual Design Comparison | **DOCUMENTED** | Structural layout complies with GoodWalletV2 specs; component radius inconsistencies and color token inheritance gaps documented. |
| **QA11** | Independent Live Transfer | Live Celo Mainnet (`0x6175...b585`) | **PASS (Live Confirmed)** | Broadcasted 10.00 G$ transfer on Celo Mainnet via LayerZero. Confirmed on Celoscan(`0xeb49c8...`) and Fuse Explorer (`0xa5343b...`). *(Testing XDC route uncovered destination RPC failure cascading to zero balance BUG-09).* |
| **QA12** | Live Rejection & Retry | Live Injected Wallet | **PASS (Live Confirmed)** | Live wallet rejection halts cleanly at `Approve G$ Needs attention`, leaving downstream bridging pending. Working `Retry` button restarts flow without page reload (See Section 4 & BUG-08 visual proof). |

---

## 2. Master Bug & Defect Tracker (11 Verified Defects)

| Bug ID | Severity | Category | Description | Steps to Reproduce | Expected vs. Actual Result |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **BUG-01** | High | Accessibility | Light Theme invisible explorer link text in status modal | Open status modal in Light Theme (`defaultTheme="light"`) | **Expected:** High-contrast, readable explorer link labels.<br>**Actual:** Explorer link titles fail to render, leaving only blank underlines and external link icons. |

| **BUG-03** | Low | Mobile Layout | "Review Bridge" title breaks across 2 lines on 320px mobile | Set viewport to 320px width and open Review drawer | **Expected:** Heading scales or fits on 1 line.<br>**Actual:** Header wraps onto two lines (`Review` / `Bridge`), compressing vertical drawer space. |
| **BUG-04** | Medium | Mobile Layout | Amount & Balance header label collision (`AmountBalance:`) on 320px mobile | Open Bridge tab on 320px mobile viewport | **Expected:** Labels wrap or maintain gap.<br>**Actual:** Labels collide with 0px spacing into `AmountBalance: 190.00 G$ MAX`. |
| **BUG-05** | Medium | UI / Styling | Provider `ActionButton` distorts into pinched oval/egg capsule | View provider selection tile with multi-line subtitle | **Expected:** Clean rounded rectangular card (`$3` / `$4`).<br>**Actual:** Inherits `borderRadius="$full"`. At ~85px height, ends deform into a bulbous egg that encroaches on text padding. |
| **BUG-06** | High | Accessibility | Secondary `"Refresh"` button text resolves pure white-on-white in Light Theme | View empty history state in Light Theme | **Expected:** Text is readable in unhovered state.<br>**Actual:** Button text is 100% white-on-white invisible; only becomes readable when mouse cursor hovers and fills background blue. |
| **BUG-07** | Medium | UI / Precision | 16-decimal native provider fee string overflow on 320px & sticker shock | Enter bridge amount and view provider fee on 320px screen | **Expected:** Fee rounded to 4 decimals.<br>**Actual:** Displays 16 unrounded decimals (`17.6294004463614488 CELO`), clipping at container edge on 320px (`CELO (native toke`). |
| **BUG-08** | Medium | UX / Copy | Misleading error copy ("Bridge transaction reverted on-chain") on token approval failure | Reject token approval in wallet or trigger approval revert | **Expected:** Copy states token approval was canceled or failed.<br>**Actual:** Step copy under `Approve G$ Needs attention` states `"Bridge transaction reverted on-chain. Try a smaller amount."` |
| **BUG-09** | High | Resilience / UX | Destination RPC rate-limit / failure cascades into false zero balance (`0.00 G$`) | Select route where destination RPC fails or rate-limits (e.g. XDC public RPC) | **Expected:** Destination error surfaced while source token balance remains accurate.<br>**Actual:** Widget catches error, clears `bridgeConfig`, and falsely wipes user balance to `Balance: 0.00 G$`. |
| **BUG-10** | Medium | UX / Code Leak | Raw Solidity revert string `"minAmount"` rendered directly to UI | Enter amount below on-chain contract threshold (e.g. `1` or `9` G$) | **Expected:** Localized copy (`"Minimum bridge amount is 10 G$"`).<br>**Actual:** Widget prints raw unformatted Solidity code token `minAmount` in red. |
| **BUG-11** | Medium | Accessibility | Light Theme warning notice text fails WCAG AA contrast on white card | View token approval notice in Light Theme | **Expected:** Warning text meets WCAG AA contrast ($\ge 4.5:1$).<br>**Actual:** Bright yellow `$warning` text on `#FFFFFF` card yields ~1.8:1 contrast, rendering it washed out and illegible. |
| **BUG-12** | Medium | Layout / CLS | Header layout shifts erratically (CLS) based on network name length | Switch "From network" between Celo and Ethereum | **Expected:** Stable layout with badge pinned to title and wallet aligned right.<br>**Actual:** Unbundled `XStack flexWrap="wrap"` causes `Ethereum` badge to drop to line 2 with wallet on right, while `Celo` badge stays on line 1 pushing wallet alone to line 2 on the left. |

---

## 3. Visual Evidence for All Documented Defects

> **Submission Evidence Note:** Following strict QA audit standards, only defect captures are embedded below to maintain report conciseness. All captures document live verified bugs in the audited codebase.

### BUG-01: Light Theme Invisible Explorer Link Titles in Status Modal
In the transaction progress/status modal when running in Light Theme (`defaultTheme="light"`), all three transaction explorer link labels (`Source transaction`, `Track on LayerZero`, `Destination transaction`) fail to render their text, appearing only as empty underlines with external link icons.
![BUG-01 Light Theme Modal Invisibility](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/qa07-light-theme-completed-links-invisible.jpg)

---

### BUG-03: "Review Bridge" Title Wrap on 320px Mobile
On narrow 320px viewports, the modal title breaks across two lines (`Review` / `Bridge`), compressing vertical layout above drawer content.
![BUG-03 Review Title Wrap](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/qa04-mobile-review-font-wrap.jpg)

---

### BUG-04: Amount & Balance Header Label Collision on 320px Mobile
Labels run together without spacing into `AmountBalance: 190.00 G$ MAX` due to missing gap and responsive wrap handling on narrow viewports.
![BUG-04 Mobile Collided Labels Dark](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug04-bug05-mobile-320px-dark.jpg)
![BUG-04 Mobile Collided Labels Light](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug04-bug05-mobile-320px-light.jpg)

---

### BUG-05: Provider Action Button Distortion on Multi-Line Subtitles
LayerZero provider card inherits `$full` border radius (`9999px`) from `ButtonFrame`. When subtitle copy wraps across multiple lines, the component expands in height (~85px on mobile), causing the curved ends to severely deform into a pinched oval / egg capsule that encroaches onto text and checkmark padding.
![BUG-05 Distorted Stadium Capsule](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug05-layerzero-mobile-320px-xdc.jpg)

---

### BUG-06: Secondary "Refresh" Button White-on-White Text in Light Theme
Empty history state in Light Theme renders the `"Refresh"` button text in pure white on a near-white button background (`#FFFFFF` on `#F5F7FA`), making the button label completely invisible until hovered.
![BUG-06 Refresh Unhovered Invisible](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug06-history-refresh-unhovered-invisible.png)
![BUG-06 Refresh Hovered Visible](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug06-history-refresh-hovered-visible.png)

---

### BUG-07: 16-Decimal Native Provider Fee String Overflow on 320px
Native provider fee displays 16 unrounded decimals (`17.6294004463614488 CELO`) on standard/desktop viewports, and on 320px narrow mobile screens it overflows and clips off the end of the text (`CELO (native toke`).
![BUG-07 Desktop 16-Decimal Unrounded](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-10-dollars-insufficient-fee.jpg)
![BUG-07 Mobile 320px Fee Overflow Clipping](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug07-fee-overflow-320px.jpg)

---

### BUG-08: Misleading Token Approval Failure Copy
When token approval reverts or fails, the step description under `Approve G$ Needs attention` displays `"Bridge transaction reverted on-chain. Try a smaller amount."`, misleading the user into thinking the bridge transaction executed and reverted on-chain when the bridge transaction was never actually submitted.
![BUG-08 Approval Misleading Error](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/qa05-approval-failed-misleading-error.jpg)

---

### BUG-09: Cascading Destination RPC Rate-Limit Leading to False Zero Balance
When a destination chain RPC is rate-limited or fails (such as public XDC RPC `https://rpc.xdcrpc.com` returning `call rate limit exhausted`), `useBridgeConfig` catches the error, renders the red banner `"Unable to load balance and limits."`, and resets `bridgeConfig = null`. Because the user's G$ balance is coupled directly to `bridgeConfig` instead of an independent balance query, `AmountEntry` wipes the user's real balance and displays `Balance: 0.00 G$`.
![BUG-09 Destination RPC Zero Balance Cascade](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-xdc-rpc-zero-balance.png)

---

### BUG-10: Raw Smart Contract Revert String `"minAmount"` Rendered to UI
When entered amount is below the on-chain contract limit ($< 10$ G$), `canBridge` returns `"minAmount"`. The widget outputs the raw unformatted string `minAmount` in red text without converting it into a helpful message (e.g. `"Minimum bridge amount is 10 G$"`).
![BUG-10 Desktop Raw minAmount](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug10-raw-minAmount-desktop.jpg)
![BUG-10 Mobile 320px Raw minAmount](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-bug10-raw-minAmount-mobile320.jpg)

---

### BUG-11: Light Theme Warning Text Severe Contrast Failure
In Light Theme, warning copy (`"This amount needs a one-time G$ approval before it can bridge."`) is styled with `$warning` (bright yellow/gold) against a pure white `#FFFFFF` surface. The resulting ~1.8:1 contrast ratio severely violates WCAG AA standards and renders the text washed out and illegible.
![BUG-11 Light Theme Low Contrast Warning](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-insufficient-native-light-mobile.jpg)

---

### BUG-12: Header Layout Erratic Wrap & CLS on Desktop / Standard Viewports
In `BridgingWidgetView.tsx`, `<Heading>`, `<Badge>`, and `<WalletControls>` are unbundled flat siblings inside `<XStack justifyContent="space-between" flexWrap="wrap">`. 
- **On standard container widths (desktop/tablet):**
  - **With "Ethereum" (8 letters):** The badge cannot fit on line 1 and wraps to line 2 (far left), while the wallet sits on line 2 (far right).
  - **With "Celo" (4 letters):** The shorter badge squeezes onto line 1 next to the title, forcing the wallet chip down alone to line 2 (far left).
  - This creates an erratic Cumulative Layout Shift (CLS) on desktop whenever the user changes the source chain. (On narrow 320px mobile screens, elements stack uniformly).
![BUG-12 Header Wrap Ethereum](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-header-wrap-ethereum.png)
![BUG-12 Header Wrap Celo](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-header-wrap-celo.png)

---

## 4. Live On-Chain Transfer Audit (QA11 & QA12)

- **Connected Wallet:** [`0x61757e66712CEDc6c53E3fa71b1F7B6580CBB585`](https://celoscan.io/address/0x61757e66712CEDc6c53E3fa71b1F7B6580CBB585) (Celo Mainnet)
- **Token:** GoodDollar (`G$`) — Celo Contract: [`0x62B8B11039FcfE5aB0C56E502b1C372A3d2a9c7A`](https://celoscan.io/token/0x62B8B11039FcfE5aB0C56E502b1C372A3d2a9c7A)
- **Initial Balance:** 200.00 G$ | 20.00 CELO
- **Transfer Route:** Celo Mainnet (Chain ID: 42220) ➔ Fuse (Chain ID: 122) via LayerZero
- **Amount Transferred:** 10.00 G$
- **Amount Received on Fuse:** 9.99 G$ (reflecting 0.1% protocol fee deduction of 0.01 G$)
- **On-Chain Evidence:**
  - **Celo Source Transaction:** [0xeb49c84d01f311e669072646611398cd234aa8e1242af6316f9a4f78743a66bf](https://celoscan.io/tx/0xeb49c84d01f311e669072646611398cd234aa8e1242af6316f9a4f78743a66bf)
  - **Fuse Destination Transaction:** [0xa5343b320a89748fc073cb18789012df9d4342283b133cb9e24e8d027e8699b8](https://explorer.fuse.io/tx/0xa5343b320a89748fc073cb18789012df9d4342283b133cb9e24e8d027e8699b8)
  - **Relay Tracking:** LayerZero Scan confirmed cross-chain delivery
- **Post-Transfer Balance:** 190.00 G$ | 5.491 CELO
- **History Tab Verification:** Live transfer persisted in widget history:
  `Celo → Fuse, completed, LayerZero · 10.00 G$`
  ![Live Celo to Fuse Transfer in History Tab](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/pass2-history-completed-transfer.jpg)
- **Live Network Resilience Finding:** Tested Celo ➔ XDC route. Discovered that `@goodsdks/bridging-sdk` relies on the default viem public XDC RPC (`https://rpc.xdcrpc.com`), which exhausted its call rate limit (`code -32005`), exposing BUG-09.

### Live Wallet Rejection & Error Interception (QA12)
- **Verified Behavior:** When token approval was rejected in the live injected wallet on Celo Mainnet, the UI intercepted the error cleanly without crashing or freezing. It transitioned to `Approve G$ Needs attention`, left downstream bridging steps safely pending, and rendered an active `Retry` button allowing immediate re-submission.
- **Visual Evidence:** (See BUG-08 screenshot: [`qa05-approval-failed-misleading-error.jpg`](https://raw.githubusercontent.com/HushLuxe/GoodWidget/qa/issue-277-bridge-audit/docs/qa/issue-277/pass2_evidence/qa05-approval-failed-misleading-error.jpg)).

---

## 5. Actionable Developer Recommendations for PR Author (`@edehvictor`)

1. **Fix Light Theme Text Invisibility (BUG-01, BUG-06):**
   - In `packages/ui/src/theme.ts`, `Anchor.tsx`, and `Button.tsx`, ensure `ButtonText`, secondary buttons, and explorer link labels explicitly consume dynamic contextual `$textColor` / `$color` rather than failing to render contrast on light surfaces.
2. **Fix Provider ActionButton Distortion (BUG-05):**
   - In `ActionButton` (`packages/bridging-widget/src/components/shared.tsx`), override `borderRadius="$3"` or `"$4"` instead of inheriting `$full` (pill) so multi-line text cards do not distort into pinched egg capsules.
3. **Decouple Source Balance from Route Config (BUG-09):**
   - In `useBridgeConfig`, separate the source token balance query from the destination fee/limit query. A failure or rate-limit on a destination RPC must never wipe out the user's source token balance or display `0.00 G$`.
4. **Translate Contract Revert Codes (BUG-10):**
   - In `primaryRequirementMessage` (`errors.ts`), map contract strings like `"minAmount"` to friendly copy: `"Minimum bridge amount is 10 G$"`.
5. **Fix Label Collisions on 320px Mobile (BUG-04):**
   - In `AmountEntry.tsx`, add `flexWrap="wrap"` or responsive gap to the header row so `Amount` and `Balance: ... G$ MAX` never collide into `AmountBalance:`.
6. **Correct Misleading Approval Error Copy (BUG-08):**
   - In `useBridgeSubmission.ts`, distinguish approval failures from bridge execution failures. Approval rejection/revert should state `"Token approval was rejected or failed"`, not `"Bridge transaction reverted on-chain"`.
7. **Fix Header Layout Jumps (BUG-12):**
   - In `BridgingWidgetView.tsx`, nest `<Heading>` and `<Badge>` together inside an inner `<XStack gap="$2" alignItems="center">` so the network badge stays permanently pinned to the title, preventing erratic header wrapping and CLS when switching chains.

---

## 6. Automated Test Execution Evidence

All automated Playwright interaction tests and mocked SDK integration suites passed without regression on commit `8feadd7d59948c924a7d2195a1f556d9355ad4fd`:

```bash
# 1. Vitest Unit Suite
pnpm --filter @goodwidget/bridging-widget test
# Result: 4/4 test files passed, 32/32 tests passed (1.42s)

# 2. Playwright UI States Suite
pnpm exec playwright test tests/widgets/bridging-widget/states.spec.ts
# Result: 26 passed (33.4s)

# 3. Playwright Real / Mocked Adapter Integration Suite
pnpm exec playwright test tests/widgets/bridging-widget/realAdapter.spec.ts
# Result: 12 passed (55.1s)

# Total Automated Suite: 38/38 Playwright tests passed (88.5s)
```

---

## 7. Final Sign-Off Statement

The foundational bridging architecture introduced in PR #266 represents high-quality work: route filtering (Axelar vs LayerZero), cross-chain batch event queries, and two-step approval/relayer submissions function reliably and passed on-chain mainnet validation.

However, based on independent human verification, **11 defects** are currently blocking full production readiness—primarily regarding **Light Theme text contrast (WCAG AA failures)**, **320px small-viewport layout collisions**, and **destination RPC error cascading (false zero balance display)**.

**Verdict:** **CHANGES REQUESTED** pending the visual contrast, mobile layout, and balance resilience fixes detailed above.

- **Auditor:** `@HushLuxe`
- **Audit Deliverable:** Single Unified Bounty Review
- **Audit Status:** Complete & Evidence-Locked
