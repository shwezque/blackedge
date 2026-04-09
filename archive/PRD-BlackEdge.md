# BlackEdge Product Requirements Document
**Version**: 2.0  
**Date**: February 27, 2026  
**Status**: Draft  
**Figma**: [BlackEdge WebApp](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8106)

---

## Notes

> [!IMPORTANT]
> **Screenshots**: The Figma designs are the source of truth for visual specifications. Screenshots embedded in this document may become outdated over the course of discussions. When there is a discrepancy, **always defer to the Figma screens**. The text descriptions, validations, and flow logic in this document are kept up to date.

> [!NOTE]
> **Currency convention**: BlackEdge operates in USDC (on Solana). This document uses two notations:
> - **`$` prefix** — for values displayed in the app (portfolio balances, allocations, unallocations, P&L, asset values). Example: `$10,000`.
> - **`USDC` suffix** — for on-chain transactions (deposits, withdrawals, fees). Example: `100 USDC`.
> 
> The distinction: `$` represents value within the platform; `USDC` represents actual tokens moving on-chain.

---

## I. Executive Summary

BlackEdge is a custodial autonomous trading platform where users deposit USDC (Solana), select one or more AI Fund Managers (LLMs), allocate capital, and let the AI trade on-chain assets on their behalf. The platform manages real capital across tokenized assets with full transparency, transaction history, and portfolio analytics.

**Core Value Proposition**: Users gain access to sophisticated, always-on AI trading without needing trading expertise, while retaining the ability to allocate, unallocate, and withdraw funds at will.

---

## II. Product Vision

```
User signs up → Chooses AI Fund Manager(s) → Deposits and Allocates funds → 
AI trades autonomously → User monitors & earns
```

**Key Differentiators**
- **True autonomy**: LLMs make all trading decisions; users set guardrails, not strategies
- **Model choice**: 6 frontier LLMs competing transparently on user capital
- **Multi-manager**: Users can allocate funds across multiple AI Fund Managers simultaneously
- **Minimal prompting**: LLMs receive one objective—*maximize returns*—no strategy hints
- **Full transparency**: Every trade includes LLM reasoning, and all transactions are logged

---

## III. User Personas

| Persona | Description | Primary Need |
|---------|-------------|--------------|
| **Passive Investor** | Wants exposure to tokenized assets without active management | Set-and-forget with notifications |
| **AI Curious** | Wants to test which LLM performs best with real stakes | Model comparison, performance history |
| **Risk-Conscious** | Interested but worried about catastrophic loss | Override controls, stop-loss, max drawdown |

---

## IV. Account & Wallet Model

### A. Account Structure

```
User Account
├── Main Wallet (custodial, Solana USDC)
│   ├── Available Cash (unallocated USDC)
│   └── Total Balance = Available Cash + Σ(All AI FM current values)
│
├── AI Fund Manager: ChatGPT
│   ├── Allocated Amount (initial capital assigned)
│   ├── Current Value (mark-to-market of all holdings + cash)
│   ├── P&L (Current Value - Allocated Amount)
│   └── Holdings (tokens the AI has bought/sold)
│
├── AI Fund Manager: Claude
│   └── ... (same structure)
│
└── AI Fund Manager: [any of 6 LLMs]
    └── ... (same structure)
```

### B. Key Financial Calculations

#### User's Total

| Metric | Formula | Where Displayed |
|--------|---------|-----------------|
| **Total Balance** | `Available Cash + Σ(Current Value of each AI Fund Manager)` | Portfolio header (green banner) |
| **Available Cash** | `Σ(Deposits) − Σ(Withdrawals) − Σ(Allocations) + Σ(Unallocations) − Σ(Fees deducted)` | Portfolio header, below Total Balance |
| **Daily Change** | `Total Balance(now) − Total Balance(24h ago)` | Portfolio header |
| **Daily Change %** | `(Total Balance(now) − Total Balance(24h ago)) / Total Balance(24h ago) × 100` | Portfolio header, alongside Daily Change |

#### Per AI Fund Manager

| Metric | Formula | Where Displayed |
|--------|---------|-----------------|
| **Allocated** | `Σ(All allocations to this manager) − Σ(All unallocations from this manager)` | AI Manager table + Detail View |
| **Current Value** | `Cash held by manager + Σ(Quantity × Current Price for each held asset)` | AI Manager table + Detail View |
| **Total P&L** | `Current Value − Allocated` | AI Manager table + Detail View (color-coded) |
| **P&L %** | `(Current Value − Allocated) / Allocated × 100` | AI Manager table + Detail View |
| **24H Change** | `Current Value(now) − Current Value(24h ago)` | AI Manager table + Detail View |
| **Deployed** | `Σ(Value of all open positions)` | AI Manager table + Detail View |
| **Deployed %** | `Deployed / Current Value × 100` | AI Manager table |
| **Cash** | `Current Value − Deployed` | Detail View (implied) |

**Cash** is also sometimes referred to as **Undeployed Balance** in other parts of the document to differentiate this from **Available Balance**, which is at the user level.

#### Per Asset (Holdings Table)

| Metric | Formula | Where Displayed |
|--------|---------|-----------------|
| **Quantity** | Number of tokens currently held (net of buys and sells) | Detail View → Holdings Table |
| **Avg Price** | `Σ(Purchase Price × Purchase Quantity for each buy) / Σ(Purchase Quantity)` | Detail View → Holdings Table |
| **Current Price** | Live mark-to-market price from Tiered Price Discovery | Detail View → Holdings Table |
| **Value** | `Quantity × Current Price` | Detail View → Holdings Table |
| **Cost Basis** | `Quantity × Avg Price` | Detail View (derived) |
| **P&L** | `Value − Cost Basis` i.e. `(Current Price − Avg Price) × Quantity` | Detail View → Holdings Table (color-coded) |
| **P&L %** | `(Current Price − Avg Price) / Avg Price × 100` | Detail View → Holdings Table |

---

## V. User Flows & Screen Specifications

> The flowchart below captures the full onboarding → deposit → allocation loop, including post-success branching. See the [Figma Screens canvas](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8106) for all screens.

![Onboarding, Deposit & Allocation Flow](assets/onboarding-allocation-flow.png)

### Flow 1: Onboarding (New User)

> **Figma**: [Sign Up (mobile)](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=792-8492) · [Sign Up (desktop)](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8566) · [Choose AI Manager](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8635) · [Allocate Funds](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8828) · [Deposit Modal](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8985) · [Confirmation](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-9189)

The onboarding flow is linear with 3 main steps. 
```
Step 1: User signs up
Step 2: User chooses AI Fund Manager(s)
Step 3: User deposits and allocates funds
```
Users can sign out at any point.

#### Step 1: Sign Up

![Sign Up](FigmaScreenshots/01%20Sign%20In.png)

**Desktop (1440px)**: Centered card layout (490px wide)  
**Mobile (390px)**: Full-width layout

| Element | Details |
|---------|---------|
| **Logo** | BlackEdge combination logo (centered) |
| **Headline** | "Let AI invest for you" |
| **Description** | "BlackEdge is a custodial AI trading platform where users deposit funds and let selected LLM models trade on-chain assets with customizable risk." |
| **LLM Icons** | Row of 6 LLM provider logos (ChatGPT, Claude, Grok, Gemini, Deepseek, Qwen) |
| **Auth Options** | 3 full-width buttons stacked vertically: |
| | 1. **Connect Wallet** (with wallet icon) — primary auth method |
| | 2. **Sign up with Google** (with Google icon) |
| | 3. **Sign up with Telegram** (with Telegram icon) |
| **Legal** | "By signing up, you agree to BlackEdge's Terms of Use and Privacy Policy." (linked) |

**Entry point**: Landing page → "Sign Up" or "Log in"  
**Exit point**: On successful registration → Step 2

##### Connect Wallet (alternative sign-in)

> [!IMPORTANT]
> BlackEdge operates on the **Solana blockchain**. The wallet connection library **must** support Solana wallets (not EVM/Ethereum wallets).

**Connection Flow**:
1. User taps "Connect Wallet" 
2. Modal: "Choose a Wallet" → Options: Phantom, Solflare, Backpack, Coinbase Wallet (Solana), Ledger, WalletConnect (Solana)
3. If Coinbase Wallet selected: "Scan to connect" with QR code OR "I don't have a wallet" → options to get Coinbase Wallet
4. Wallet connected → session established

**Mobile**: Native wallet apps or QR code scanning  
**Desktop**: Browser extension wallets or QR code for mobile wallets

#### Step 2: Choose AI Fund Manager

![Choose AI Fund Manager](FigmaScreenshots/02%20Choose%20AI%20FM.png)

> User selects one AI Fund Manager from the grid. Performance data is live from the platform's benchmark portfolios.

**Header**: "AI Fund Managers' Performance"  
**Subheader**: "Each model started with $10,000 and traded autonomously."

**AI Manager Selection Grid** (3 columns desktop, 2 columns mobile):

| AI Manager | Provider | Tagline | Performance |
|------------|----------|---------|-------------|
| ChatGPT | OpenAI | "Most popular AI, strong reasoning" | +22.37% |
| Claude | Anthropic | "Safety-focused, careful analysis" | +22.37% |
| Grok | xAI | "Real-time sentiment from X" | +22.37% |
| Gemini | Google | "Google's frontier AI model" | +22.37% |
| Deepseek | Deepseek | "Quant-backed, cost efficient" | +22.37% |
| Qwen | Alibaba | "Top open-weight model" | +22.37% |

> **Note**: Just show the AI Manager without the version so that we can easily update to the latest models (or not) without having to change the UI.

Each card shows:
- LLM avatar/logo (32×32 rounded)
- Model name (bold, 24px)
- Provider name (secondary text, 16px)
- Historical return % (green, right-aligned)
- One-line description

**Bottom**: "Next" button (centered, 144px wide)

**Exit point**: Select AI Manager → Step 3

**AI Fund Manager Estimated Cost**:

| Provider | Model | Version | Avg Tokens/Cycle | $/1K Tokens | Avg $/Cycle |
|----------|-------|---------|------------------|-------------|-------------|
| OpenAI | ChatGPT | GPT-5.2 | ~5,000 | $0.015 | $0.075 |
| Anthropic | Claude | Opus 4.5 | ~5,000 | $0.015 | $0.075 |
| xAI | Grok | 4 | ~5,000 | $0.010 | $0.050 |
| Google | Gemini | 3 Pro | ~5,000 | $0.007 | $0.035 |
| Deepseek | Deepseek | R1 | ~5,000 | $0.0022 | $0.011 |
| Alibaba | Qwen | 3-Max | ~5,000 | $0.0016 | $0.008 |

> **Note**: ~5K tokens/cycle = system prompt (~1.5K) + portfolio context (~200) + research/tool calls (~2.5K) + response (~800). Costs are blended input/output rates. Costs increase with: user agency overrides, deeper research, and extended asset scopes.

#### Step 3: Deposit and Allocate Funds (Onboarding)

![Allocate Funds](FigmaScreenshots/03%20Allocate%20and%20Deposit.png)

> [!IMPORTANT]
> **Onboarding flow is a single streamlined operation.** The user specifies how much they want to allocate, deposits that exact amount, and the deposit is **automatically allocated** to the selected AI Manager. There is no balance validation against Available Cash because the user has no existing balance during onboarding.

**Screen**: "Allocate Funds To [AI Manager Name]"  
**Back navigation**: Arrow left to return to Step 2

| Element | Details |
|---------|---------|
| **Heading label** | "You've selected" |
| **Selected Manager Card** | Shows selected LLM with logo, name, provider, return % |
| **Amount Label** | "Amount" |
| **Amount Input** | Text field with "$" prefix and placeholder "Enter amount", numeric input |
| **Primary CTA** | "Next" button (full-width, blue, 480px desktop) |
| **Secondary CTA** | "I'll do this later" link/button |

**Validation (onboarding-specific)**:
- Amount must be ≥ 1 USDC (minimum deposit)
- No Available Cash validation (user has no balance yet)
- If user taps "I'll do this later", a skip confirmation dialog appears

**Onboarding deposit + allocation behavior**:
1. User enters desired allocation amount (e.g. $100)
2. User taps "Next" → Deposit modal opens with the amount shown as the deposit target
3. User sends USDC to the displayed wallet address
4. Once deposit is confirmed on-chain, the **full deposited amount is automatically allocated** to the selected AI Manager
5. Confirmation screen shows the combined deposit + allocation result

> There is no intermediate step where the user must separately "allocate" after depositing. The deposit **is** the allocation during onboarding.

> [!CAUTION]
> **Required User Acknowledgments Before Deposit**
> - "This platform uses AI to make trading decisions. AI can and will make mistakes."
> - "Past performance does not guarantee future results."
> - "You may lose some or all of your deposited funds."
> - "BlackEdge does not provide financial advice."
> - "Users bear all trading risk."

#### Step 3.2: Deposit Funds Modal (onboarding)

![Deposit Funds Modal](FigmaScreenshots/04%20Deposit.png)

**Modal overlay** (520px wide desktop, 390px mobile):

| Element | Details |
|---------|---------|
| **Title** | "Deposit funds" |
| **QR Code** | Wallet address as scannable QR code (220×220) |
| **Wallet Address** | Full address displayed with copy icon |
| **Network** | "Solana (SOL)" with SOL icon |
| **Fee** | "0 USDC" |
| **Minimum deposit** | "1 USDC" |
| **Required network confirmations** | "40" |
| **Processing time** | "~20 minutes" |
| **Warning** | ⚠️ "Please note that only supported networks on BlackEdge are shown above, if you deposit via another network, your assets may be lost." |
| **Primary CTA** | "Save and share address" button (blue, full-width) |
| **Secondary CTA** | "I'll do this later" link |

**Skip Deposit Confirmation Dialog** (if user taps "I'll do this later"):

![Skip Deposit Dialog](FigmaScreenshots/03.01%20Skip.png)

- Warning icon (amber triangle)
- "Are you sure you want to skip depositing funds?"
- "This will cancel your allocation to [AI Manager Name]."
- **Primary CTA**: "No, continue" (blue, full-width) — returns to deposit
- **Secondary CTA**: "Yes, skip this step" — cancels allocation and proceeds

#### Step 3.3: Transaction Processing

![Transaction Processing](FigmaScreenshots/04.01%20Tx%20Processing.png)

- Loading spinner animation (80×80)
- "Transaction processing ..."
- No user interaction possible during processing

#### Step 3.4: Transaction Confirmation

![Transaction Confirmation](FigmaScreenshots/04.02%20Success.png)

| Element | Details |
|---------|---------|
| **Success icon** | Green checkmark circle |
| **Amount** | "$10.00" (or actual amount) |
| **Message** | "[AI Manager] is now successfully managing your portfolio." |
| **Transaction Details** | |
| From | "Main Wallet (3K9C...h7iK)" with copy icon |
| To | "[AI Manager] (2L3D...p9aB)" with copy icon |
| Deposit Amount | "10.00 USDC" |
| Fee | "0 USDC" |
| Total Amount | "10.00 USDC" |
| Transaction Date & Time | "19 Feb 2026, 18:44" |
| **Primary CTA** | "Proceed to Portfolio" button (blue, full-width) |
| **Secondary CTA** | "Choose More AI Fund Managers" (outline) |
| **Tertiary CTA** | "Deposit More Funds" (outline) |


---

### Flow 2: Portfolio & AI Fund Manager (Main App Screen)

> **Figma**: [Portfolio – Empty State](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8107) · [Portfolio – With Balance](https://www.figma.com/design/TcSFqX0kRpk93ylkPkShK5/BlackEdge--WebApp?node-id=804-8322)

This is the primary screen after onboarding. It has two states: **Empty State** and **With Balance**. Clicking an AI Manager opens a **Detail View** with three tabs.

#### Global Navigation

**Top Bar** (72px height):
- Left: BlackEdge logo
- Center: Search bar
- Right: Notification bell icon (24×24) with badge dot for unread notifications + User avatar (circular, 40×40, shows initials e.g. "SQ")

#### Portfolio Header (Green Banner)

Common across both states:

| Element | Details |
|---------|---------|
| **Background** | Green gradient (#22C55E style, with dot pattern overlay) |
| **Label** | "Total Balance" (small text, underlined) |
| **Balance** | Large amount display, e.g. "$12,482.16" (bold, ~48px) |
| **Daily Change** | "▲ $476.12 (1.11%)" or "▼ $X.XX (X.XX%)" with up/down arrow icon |
| **Available Cash** | "Available Cash: $3,159.06" (below change indicator) |
| **Action Buttons** | Three pill buttons in a row: |
| | 1. **Deposit** (blue filled, primary) — opens the Deposit Funds modal (same as Flow 1 Step 3.2) |
| | 2. **Withdraw** (white filled) — opens Flow 6 |
| | 3. **Allocate** (white filled) — opens Flow 4 (Allocate More Funds to Existing AI Manager) |

**Header metric formulas** (see Account & Wallet Model for full definitions):
- **Total Balance** = Available Cash + Σ(Current Value of all AI Fund Managers)
- **Daily Change** = Total Balance now − Total Balance 24 hours ago
- **Available Cash** = Total deposits − Total withdrawals − Σ(Allocations) + Σ(Unallocations/returns)

---

#### Empty State (No AI Fund Managers assigned)

**Portfolio - Empty State**
![Portfolio — Empty State (Desktop)](FigmaScreenshots/Portfolio_%20Empty%20State%20-%20Desktop.png)

Displayed when the user has no active AI Fund Managers. Below the header, scrollable:

##### 1. Choose your AI Fund Manager
- Grid of 6 LLM cards (2 columns on mobile, 3 on desktop)
- Same cards as onboarding Step 2 (logo, name, provider, tagline)
- Tapping a card starts the "Adding AI Manager" flow (→ Flow 3)

##### 2. AI Fund Managers' Performance
- Title: "AI Fund Managers' Performance"
- Subtitle: "Each model started with $10,000 and traded autonomously."
- Refresh icon (top-right)

**Toggle controls**:
- $ / % toggle (switch between dollar values and percentages)
- Date range tabs: 7D | 14D | 30D | 90D

**Performance Chart**:
- Multi-line chart showing all 6 LLMs
- Color-coded lines per LLM
- Y-axis: Price/value
- X-axis: Date labels
- Chart lines are smoothened with no data markers
- **Tooltip on hover/tap**: Date, all LLM values
- **Legend**: 6 entries in 2×3 grid showing LLM name + current value
  - GPT-5.2: $10,245
  - Claude 4.5: $10,102
  - Deepseek R1: $10,102
  - Grok 4: $10,102
  - Gemini 3 Pro: $10,102
  - Qwen-3-Max: $10,102

> [!NOTE]
> The benchmark portfolio data reflects **actual trading results** from live platform benchmark portfolios. Each LLM was allocated $10,000 and has traded autonomously since launch. The performance chart covers the period from the start of benchmark trading to the present, and is updated in real-time.

##### 3. Transactions (empty)
- Title: "Transactions"
- Illustration image (empty inbox style)
- "You don't have any transactions yet"
- "Start by depositing funds and choosing an AI to manage them."

##### 4. Portfolio Breakdown (empty)
- Title: "Portfolio Breakdown"
- "No portfolio breakdown yet"
- "Start by depositing funds and choosing an AI to manage them."

---

#### With Balance State (AI Fund Managers active)

**Portfolio - With Balance**
![Portfolio — With Balance (Desktop)](FigmaScreenshots/Portfolio_%20WithBalance%20-%20Desktop.png)

Displayed when the user has one or more active AI Fund Managers.

##### 1. Your AI Fund Managers

- Title: "Your AI Fund Managers"
- "Add More AI Manager" button (top-right, outline style)

**Table Layout**:

| Column | Description | Formula / Formatting |
|--------|-------------|----------------------|
| **AI Fund Manager** | LLM avatar (32×32) + Model name (bold) + Provider name (secondary) | Left-aligned |
| **Allocated** | Total capital assigned to this manager (cumulative) | USD |
| **Current Value** | Cash held by manager + mark-to-market value of all positions | USD |
| **Total P&L** | Current Value − Allocated Amount | Color-coded: green (▲) = profit, red (▼) = loss. Shows both $ and % |
| **24H Change** | Current Value − Value 24 hours ago | Color-coded: green (▲) = up, red (▼) = down |
| **Deployed** | Value of capital currently in open positions (not cash) | Percentage of Current Value |
| **Status** | Active / Paused indicator | Green dot = Active, Orange dot = Paused |
| **Actions** | Three-dot menu (⋯) | See menu below |

**Three-dot menu actions** (contextual):
- **View** → opens AI Manager Detail View (see below)
- **Allocate** → opens Allocate More Funds flow (→ Flow 4)
- **Unallocate** → opens Unallocate Funds flow (→ Flow 5)
- **Pause/Resume** → pauses/resumes AI trading (see Pause/Resume below)

##### 2. Transactions (with data)

- Title: "Transactions"
- List of transaction items in reverse-chronological order

**Transaction Row Anatomy**:
```
[Primary Icon (40px circle)]
   └─ [Subscript Badge (16px, bottom-right)]
[Description]                        [Amount]
[Timestamp] · [Status]               [Currency]
```

**Column Specifications**:

| Column | Format | Available Values |
|--------|--------|------------------|
| **Type** | Enum | `ALLOCATE`, `DEPOSIT`, `WITHDRAW`, `UNALLOCATE`, `TRADE_BUY`, `TRADE_SELL` |
| **Primary Icon** | 40×40 circle | AI Manager avatar (for allocate/unallocate/trade) or User avatar (for deposit/withdraw) |
| **Subscript Badge** | 16×16 icon, bottom-right of primary | See badge table below |
| **Description** | `{action} {preposition} {object}` | See description syntax table below |
| **Amount** | `{sign}${value}` | `+` prefix = inflow (green), `-` prefix = outflow (red) |
| **Timestamp** | Relative | `"Xm ago"`, `"Xh ago"`, `"Xd ago"`, or `"DD MMM YYYY"` if > 7 days |
| **Status** | Enum | `Completed`, `Processing`, `Pending`, `Failed` |

**Subscript Badge Icons**:

| Type | Badge Icon | Badge Color |
|------|-----------|-------------|
| `DEPOSIT` | ↓ Arrow down | Green (#22C55E) |
| `WITHDRAW` | ↑ Arrow up | Red (#EF4444) |
| `ALLOCATE` | → Arrow right | Blue (#3B82F6) |
| `UNALLOCATE` | ← Arrow left | Orange (#F59E0B) |
| `TRADE_BUY` | Cart/buy icon | Green (#22C55E) |
| `TRADE_SELL` | Sell icon | Red (#EF4444) |

**Description Syntax**:

| Type | Template | Example |
|------|----------|---------|
| `ALLOCATE` | `"Allocated to {AI Manager}"` | "Allocated to ChatGPT" |
| `DEPOSIT` | `"Deposit"` | "Deposit" |
| `WITHDRAW` | `"Withdrawn"` | "Withdrawn" |
| `UNALLOCATE` | `"Unallocated from {AI Manager}"` | "Unallocated from Claude" |
| `TRADE_BUY` | `"{AI Manager} bought {Token}"` | "ChatGPT bought APPLx" |
| `TRADE_SELL` | `"{AI Manager} sold {Token}"` | "Claude sold TSLAx" |

**Amount Color Rules**:

| Direction | Sign | Text Color |
|-----------|------|------------|
| Inflow (Deposit, Allocate) | `+` | Green (#22C55E) |
| Outflow (Withdraw, Trade Buy) | `-` | Red (#EF4444) |
| Return (Unallocate, Trade Sell) | `+` | Green (#22C55E) |

**"See more" link** at bottom of list (navigates to full transaction history)

##### 3. Portfolio Breakdown (with data)
- Title: "Portfolio Breakdown"
- **Donut chart**: Visual allocation split across AI Managers
- **Legend**: Color-coded entries, e.g.:
  - ChatGPT: 50%
  - Claude: 50%

##### 4. Choose More AI Fund Managers
- Same grid as empty state, always visible for adding more managers
- Already-added managers show a green "✦ Added" badge

---

#### Pause/Resume AI Manager

Triggered from the three-dot menu on an active AI Manager card.

| State | Behavior |
|-------|----------|
| **Paused** | AI Manager halts all trading activity. All current positions are held as-is — no new BUY/SELL actions. The scan cycle still runs but skips this manager. Status indicator changes to orange "Paused" dot. |
| **Resumed** | AI Manager re-enters the normal scan cycle and may trade on the next cycle. Status indicator returns to green "Active" dot. |

- Pausing does **not** liquidate positions or return funds
- Users can still **unallocate** or **allocate more** while an AI Manager is paused
- If a user pauses and then unallocates, the standard unallocation logic applies (Flow 5)

---

#### AI Manager Detail View

Opened by clicking an AI Manager row in the table or "View" from the three-dot menu. The detail view has three tabs: **Portfolio**, **Activity**, and **Trade History**.

##### Portfolio Tab

**AI Manager - Portfolio Tab** (tapping an active AI Manager row):
![Portfolio — AI Manager Detail View](FigmaScreenshots/Portfolio%20-%20View.png)

**Summary Metrics** (displayed at top of detail view):

> See **IV. Account & Wallet Model → Per AI Fund Manager** for metric formulas and definitions.

**Holdings Table** (lists all tokens currently held by this AI Manager):

| Column | Description | Formatting |
|--------|-------------|------------|
| **Symbol** | Token ticker (e.g., AAPLx, SOL) | Uppercase |
| **Quantity** | Number of tokens held | 2-6 decimals based on token |
| **Avg Price** | Entry price per token | USD with precision |
| **Current Price** | Live mark-to-market price | USD with precision |
| **Value** | Quantity × Current Price | USD |
| **P&L** | (Current Price − Avg Price) × Quantity | USD + % (color-coded) |

>**Crypto-Friendly Price Formatting**:
For tokens with very small prices (e.g., meme coins at $0.000015), the UI uses **subscript zero notation**: `$0.000015` displays as `$0.0₄15` (subscript 4 = four zeros after decimal).

##### Activity Tab

**AI Manager - Activity Tab** (within AI Manager detail):
![Portfolio — Activity Tab](FigmaScreenshots/Portfolio%20-%20Activity.png)

Chronological feed of all agent **meta-events**, grouped by date (sorted descending). Individual BUY/SELL trades are shown in the Trade History tab, not here.

| Event Type | Description | Example |
|------------|-------------|--------|
| **Lifecycle Events** | Allocation, unallocation | "Allocated $500 to ChatGPT" |
| **Trading** | Paused, Resumed | "ChatGPT paused trading" |
| **LLM_CHANGE** | Underlying model version changed | "Model updated: GPT-5.1 → GPT-5.2" |

Each entry shows:
- Action icon + color badge
- Description text
- Timestamp (relative or absolute)

##### Trade History Tab

**AI Manager - Trading History Tab** (within AI Manager detail):
![Portfolio — Trading History Tab](FigmaScreenshots/Portfolio%20-%20Trading%20History.png)

Full trade log with sortable columns:

| Column | Description | Formatting |
|--------|-------------|------------|
| **Timestamp** | When the trade executed | Local timezone (e.g., "19 Feb 2026, 18:44") |
| **Action** | BUY / SELL | Color-coded: green = BUY, red = SELL |
| **Symbol** | Token traded | Uppercase ticker |
| **Quantity** | Number of tokens | Decimal precision |
| **Price** | Execution price per token | USD with precision |
| **Reasoning** | Full LLM rationale for this trade | Expandable text (click to show/hide) |

---

### Flow 3: Adding AI Manager (from Portfolio screen, post-onboarding)

> [!IMPORTANT]
> **Post-onboarding flows differ from onboarding.** After onboarding, the user already has a Main Wallet with a potential balance. The flow is: **specify allocation amount first → validate against Available Cash → if insufficient, either change allocation or deposit more funds → then loop on allocation until sufficient funds are available**. Deposits go to the Main Wallet (not directly to the AI Manager), and allocation is a separate explicit step. See the flowchart at the start of this document.

When tapping an AI Manager card from the "Choose your AI Fund Manager" section:

1. **Allocate Funds To [AI Manager Name]** screen appears
2. User enters desired allocation amount
3. **Validation** (post-onboarding rules apply — see below)
4. If sufficient Available Cash → "Next" → Processing → Confirmation
5. If insufficient Available Cash → user is prompted to deposit more funds first or change the allocation.

**Post-onboarding allocation validation**:
- Amount must be ≥ $1
- Amount must be ≤ Available Cash
- If Amount > Available Cash, show error: "Insufficient funds. You have $[Available Cash] available. Deposit more funds or enter a lower amount."
- User can tap "Deposit" to add funds to their Main Wallet, then return to the allocation screen
- **Deposits land in the Main Wallet as Available Cash** — they are NOT auto-allocated. The user must explicitly complete the allocation step after depositing.

---

### Flow 4: Allocate More Funds to Existing AI Manager

Triggered from the three-dot menu on an active AI Manager card, or from the "Allocate" button on the portfolio header.

> This flow follows the same **post-onboarding** validation rules as Flow 3 (allocate-first, validate against Available Cash).

**Screen**: "Allocate Funds To [AI Manager]"

| Element | Details |
|---------|---------|
| **Title** | "Allocate Funds To [AI Manager]" |
| **AI Manager Card** | Shows current manager with logo, name, provider, return % |
| **Amount Input** | Numeric field with "$" prefix |
| **Available Balance shown** | User sees how much Available Cash they have |
| **Primary CTA** | "Next" |
| **Secondary CTA** | "I'll do this later" |

![Allocate More Funds](FigmaScreenshots/Allocate%20More%20Funds.png)

**Validation** (same as Flow 3):
- Amount must be ≥ $1
- Amount must be ≤ Available Cash
- If insufficient → error message + option to deposit first
- Deposits go to Main Wallet; user must return and complete allocation separately

**Post-allocation**:
- Transaction processing screen (spinner)
- Confirmation screen with details (From, To, Amount, Fee, Total, Date & Time)

---

### Flow 5: Unallocate Funds from AI Manager

![Unallocate Funds Modal](FigmaScreenshots/Unallocate%20Funds.png)

Triggered from the three-dot menu on an active AI Manager card.

**Modal overlay**: "Unallocate Funds"

| Element | Details |
|---------|---------|
| **Unallocate from** | Pre-populated with AI Manager name (e.g. "ChatGPT") |
| **Send to** | Pre-populated with Main Wallet address (e.g. "Main wallet") |
| **Amount Input** | Numeric field with $ Prefix + MAX button (max = Current Value of AI Manager) |
| **Available cash** | Shows available cash of the AI Manager (e.g. "Available cash: $120.00") |
| **Primary CTA** | "Review details" button (blue, full-width) |
| **Secondary CTA** | "Cancel" link |

#### Unallocation Fund-Sourcing Logic

When a user requests to unallocate **$X** from an AI Manager, the system follows this priority order:

```
Step 1: Check AI Manager's Undeployed Balance
        undeployed_balance = AI Manager's Current Value − Value of Open Positions

Step 2: If undeployed_balance >= X
        → Transfer $X from AI Manager to Main Wallet immediately
        → No positions are touched
        → Status: Completed (instant)

Step 3: If undeployed_balance < X
        → Transfer all undeployed_balance to Main Wallet immediately
        → remaining = X − undeployed_balance
        → AI Manager begins unwinding positions to raise remaining amount
        → Positions are sold based on the AI Manager's discretion (AI Manager will need to be prompted)
        → As sells complete, proceeds transfer to Main Wallet automatically
        → Status: Processing (until fully unwound)
```

**Worked Example**:
- AI Manager "ChatGPT" has Current Value = **$5,000**
  - Undeployed Balance: **$1,200**
  - Open Positions: **$3,800** (across 5 tokens)
- User requests to unallocate **$2,500**
- **Step 1**: $1,200 undeployed balance transferred immediately to Main Wallet
- **Step 2**: Remaining $1,300 needed → AI sells positions
- **Step 3**: Sells complete → $1,300 transferred → Total $2,500 returned
- AI Manager's new Current Value: **$2,500** (remaining positions + any undeployed balance from sells)

**Confirmation dialog** (before execution):
- Warning icon (amber)
- "Are you sure you want to unallocate $[amount] from [AI Manager]?"
- If amount ≤ undeployed balance: "This amount will be transferred from undeployed balance. No positions will be affected."
- If amount > undeployed balance: "This amount exceeds your undeployed balance of [undeployedbalance]. The AI manager will sell approximately [remaining] worth of positions to fulfill this request. Actual amounts may vary due to market conditions."
- "Confirm" button
- "Cancel" button

**Post-unallocation**:
- Transaction processing screen (may take longer if positions are being unwound)
- Confirmation with details (From: AI Manager wallet, To: Main Wallet, Breakdown of cash vs. sold positions)

#### Unallocation Edge Cases

| Edge Case | Scenario | Resolution |
|-----------|----------|------------|
| **Amount > Current Value** | User enters $6,000 but AI Manager only has $5,000 Current Value | UI prevents submission. Validation error: "Amount exceeds current value of $5,000. Enter a lower amount or unallocate all." Max input capped to Current Value. |
| **Full unallocation** | User wants to unallocate 100% of Current Value | Equivalent to "Close AI Manager." All positions are liquidated, all cash returned. AI Manager is removed from the user's active managers list. Confirmation dialog should note: "This will close [AI Manager] entirely and return all funds." |
| **Zero undeployed balance** | AI Manager is 100% deployed (no undeployed balance) | Entire requested amount must come from position sales. UI shows: "All requested funds will come from selling positions. This may take several minutes and amounts may vary due to market prices." |
| **Illiquid tokens** | AI Manager holds tokens with low liquidity or wide spreads | System sells what it can and queues remaining sells. If a position cannot be sold within a reasonable time (e.g., 30 min), the system retries with smaller order sizes. If still unsellable after multiple attempts, user is notified: "Unable to fully unallocate. $[remaining] is in illiquid positions. You can retry later or force-close the AI Manager." |
| **Partial fill on sell** | DEX only partially fills a sell order due to liquidity | System accepts partial fill, queues remainder. User sees "Processing" status. Partial amount is transferred to Main Wallet as it becomes available. Transaction log shows individual partial fills. |
| **Rapid price drop during unwind** | Market crashes while positions are being sold to fulfill unallocation | Sells execute at market price (slippage applies). Final amount returned may be less than requested. Confirmation screen shows actual amounts vs. requested. User is not guaranteed the exact requested amount—only the proceeds of the liquidation. |
| **Concurrent unallocation requests** | User submits two unallocation requests in quick succession | Second request is blocked with: "An unallocation is already in progress. Please wait for it to complete." Only one unallocation per AI Manager can be active at a time. |
| **AI Manager has pending trades** | AI Manager has open/pending buy orders when unallocation is requested | Pending buy orders are cancelled first, freeing up cash. Then the standard unallocation logic proceeds. |
| **Unallocation during scan cycle** | User unallocates while the AI is mid-trade-execution | The unallocation request is queued until the current scan cycle completes. The AI will not open new positions if an unallocation is pending. |
| **Network/transaction failure** | Solana transaction fails during the transfer | System retries up to 3 times with exponential backoff. If all retries fail, the unallocation is rolled back: funds remain with the AI Manager, and the user is notified: "Unallocation failed due to a network error. Your funds are safe. Please try again." |
| **Minimum remaining balance** | After unallocation, AI Manager would have < $10 remaining | If remaining Current Value would drop below $10 (minimum viable portfolio), prompt user: "After this unallocation, only $[remaining] would remain—below the $10 minimum. Would you like to unallocate everything instead?" Offer full close as alternative. |

---

### Flow 6: Withdraw

![Withdraw Funds Modal](FigmaScreenshots/Withdraw%20Funds.png)

Triggered from the "Withdraw" button on the portfolio header.

**Modal overlay**: "Withdraw funds"

| Element | Details |
|---------|---------|
| **Withdraw from** | Pre-populated with Main Wallet address (e.g. "Main wallet (8f3K…QrSt)") |
| **Send to** | Text field — "Enter wallet address" |
| **Amount Input** | Numeric field with USDC suffix + MAX button |
| **Available balance** | Shows current Available Cash (e.g. "Available balance: $120.00") |
| **Network** | "Solana (SOL)" with icon |
| **Fee** | "1.00 USDC" |
| **Required network confirmations** | "40" |
| **Processing time** | "~20 minutes" |
| **Warning** | ⚠️ "Please note that only supported networks on BlackEdge are shown above, if you withdraw via another network, your assets may be lost." |
| **Primary CTA** | "Preview" button (blue, full-width) |
| **Secondary CTA** | "Cancel" link |

**Confirmation dialog**:
- Summary of withdrawal details
- "Confirm Withdrawal" button

**Post-withdrawal**:
- Transaction processing screen
- Confirmation with details (Amount, Fee, Net received, Destination wallet, Tx hash)

---

### Flow 7: Profile & Settings

Accessed from the user avatar (top-left of navigation bar).

#### Profile Menu
- User avatar + name displayed
- Menu items:
  1. **Edit Account Details**
  2. **Terms of Service and Privacy Policy**
  3. **Delete Account**
  4. **Log out**

#### Edit Account Details

| Field | Details |
|-------|---------|
| **Display Name** | Editable text field |
| **Email** | Editable text field |
| **Connected Wallet** | Wallet address (with option to disconnect/reconnect) |
| **Save Changes** button | Primary CTA |

#### Delete Account

**Confirmation flow**:
1. Warning screen: "Are you sure you want to delete your account?"
2. Lists consequences:
   - All portfolios will be closed
   - Active AI managers will be stopped
   - Remaining funds will need to be withdrawn first
3. "Delete my account" (destructive red) + "Cancel" buttons
4. Final confirmation: type "DELETE" to confirm

#### Terms of Service & Privacy Policy

Full legal document pages, scrollable:
- **Trading Policy**: Covers trading hours, order types, asset eligibility
- **Privacy Policy**: Covers data collection, usage, retention, user rights

#### Log Out

Simple confirmation → return to sign-in screen.

---

### Flow 8: Notifications

Accessed from the bell icon (top-right of navigation bar).

#### Notification Center

**Header**: "Notifications" with:
- "Mark all as read" link (top-right)
- Filter tabs: **All** | **Private** | **Alert**

#### Empty State
- "No Notification"
- "You don't have notifications yet"

#### With Notifications
Each notification item shows:
- **Icon**: Relevant action icon (trade, alert, etc.)
- **Title**: e.g. "ChatGPT bought 2.5 AAPLx at $178.50"
- **Details**: e.g. "Apple showing strength post-earnings"
- **Timestamp**: e.g. "Last Wednesday at 3:55 AM"
- **Read/unread indicator**: Dot or visual distinction

**Notification Channels**:
- **In-app**: Displayed in the Notification Center (bell icon)
- **Telegram**: Push notifications sent to the user's linked Telegram account (if signed up via Telegram)

**Notification Types**:

| Type | Trigger | Content |
|------|---------|---------|
| **Trade Executed** | AI manager buys/sells | Asset name, quantity, price, manager name |
| **Allocation Confirmed** | Funds allocated | Amount, AI manager name |
| **Unallocation Confirmed** | Funds returned | Amount, AI manager name |
| **Deposit Confirmed** | USDC received | Amount, from wallet |
| **Withdrawal Confirmed** | USDC sent | Amount, to wallet, tx hash |
| **Significant P&L Change** | Threshold breach | Portfolio, change %, amount |


---

## VI. Autonomous Trading Engine

### A. Trading Loop
- LLMs discover tradable assets dynamically (no hardcoded lists)
- Trading loop: Research → Decide → Execute → Report
- Positions tracked per AI Manager per user
- All trades logged with LLM reasoning
- **Scan frequency**: 1× per day (configurable)

> [!IMPORTANT]
> The scan frequency must be implemented as a **configurable backend parameter** (e.g., environment variable or admin config table). The team should be able to change the frequency (e.g., from 1×/day to 2×/day or 1×/hour) without code changes.

### B. LLM System Prompt

All 6 AI Fund Managers receive the **same system prompt**. Performance differences reflect each model's reasoning capabilities, not different instructions.

```
You are an autonomous trading agent managing a real portfolio on the Solana blockchain.

OBJECTIVE: Maximize returns on the capital allocated to you.

AVAILABLE ACTIONS:
- BUY <symbol> <quantity> — Purchase tokens
- SELL <symbol> <quantity> — Sell held tokens
- HOLD — Take no action this cycle

CONSTRAINTS:
- You may only trade assets within your assigned Asset Scope
- Maximum 8 concurrent positions

CONTEXT PROVIDED EACH CYCLE:
- Your current holdings (symbol, quantity, avg price, current price, P&L)
- Available cash balance
- Market data and research tools
- Your last 10 trades with reasoning (for thesis continuity)

RESPONSE FORMAT:
For each action, provide:
1. Action (BUY/SELL/HOLD)
2. Symbol and quantity (if applicable)
3. Reasoning (2-4 sentences explaining your thesis)

RULES:
- Always provide reasoning for every decision, including HOLD
- Do not fabricate prices or market data
- If uncertain, default to HOLD
- You are fully autonomous — no human approval is needed
```

> [!WARNING]
> **Output Guardrails**
>
> The following safeguards must be enforced. Some are best handled by **system-level validation** (code that runs after the LLM responds), while others are **prompt-implicit** (behaviors the LLM should exhibit naturally given the context it receives). We intentionally avoid over-prompting — the prompt stays lean, and the system catches the rest.
>
> **System-enforced (validated in code before execution):**
>
> | Guardrail | Enforcement |
> |-----------|-------------|
> | No selling tokens not held | Reject trade if `symbol` not in current holdings or `quantity` > held quantity |
> | Non-zero quantity | Reject any BUY/SELL action where `quantity ≤ 0` |
> | Valid symbol | Reject trade if `symbol` is not in the allowed asset scope |
> | Trade frequency | System controls scan cycle timing; LLM cannot trigger trades outside the scheduled cadence |
> | Position limits | Reject trade if it would exceed max concurrent positions (8) or violate size constraints |
>
> **Prompt-implicit (expected from LLM behavior; monitored, not hardcoded):**
>
> | Expectation | How We Monitor |
> |-------------|----------------|
> | Reasoning grounded in real data — no hallucinated prices, news, or metrics | Spot-check rationale against market data provided in context; flag divergence |
> | Considers the full asset scope, not just currently held tokens | Track whether research phase references assets beyond current holdings |
> | Actionable responses — avoids excessive HOLD without reasoning | Dashboard metric: HOLD rate per LLM over time; alert if >90% across N cycles |
> 
>**Principle**: Keep the prompt minimal. Let the system enforce hard constraints. Monitor soft expectations and iterate on the prompt only when patterns emerge. User agency parameters (Phase 2) override the system prompt.

**Per-cycle context variables injected**:

| Variable | Description |
|----------|-------------|
| `{{PORTFOLIO}}` | Current holdings with quantities, avg prices, current prices, P&L |
| `{{CASH_BALANCE}}` | Available cash for new trades |
| `{{TRADE_HISTORY}}` | Last 10 trades with full reasoning |
| `{{MARKET_DATA}}` | Pre-fetched prices and research from ScanContext |
| `{{ASSET_SCOPE}}` | Allowed asset universe (MVP: Solana Tokenized RWA) |
| `{{CONSTRAINTS}}` | Active trading constraints (may include user overrides in Phase 2) |

> [!NOTE]
> The prompt above is the **baseline prompt**. It will be iterated on during closed beta based on observed trading behavior. User agency parameters (Phase 2) are injected via the `{{CONSTRAINTS}}` variable.

### C. ScanContext Pattern (API Cost Optimization)

Each scan cycle creates a shared `ScanContext` that all agents reference:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCAN CYCLE START                             │
│  1. Create ScanContext with unique cycle ID                     │
│  2. Pre-fetch baseline prices                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              PARALLEL AGENT EXECUTION                           │
│  - All 6 LLMs run simultaneously (Promise.allSettled)           │
│  - First agent to discover token fetches price → cached         │
│  - Subsequent agents get cached price instantly                 │
│  - Graceful failure: one LLM timeout doesn't block others       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  USER-SPECIFIC EXECUTION                        │
│  - Filter recommendations by user constraints                   │
│  - Apply user overrides (stop-loss, max drawdown, etc.)         │
│  - Calculate position sizing based on user capital              │
│  - Execute trades per portfolio                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SCAN CYCLE END                               │
│  - Clear ScanContext (no stale data carries over)               │
│  - Log cycle metrics (latency, trades executed)                 │
└─────────────────────────────────────────────────────────────────┘
```

**Cost**: O(LLMs × Asset Scopes) instead of O(Users) — research is shared, only execution is per-user.

### D. Tiered Price Discovery

| Priority | Source | Use Case |
|----------|--------|----------|
| 1 | **ScanContext cache** | Prices fetched earlier in same cycle |
| 2 | **Alchemy Token API** | Primary real-time source for Solana tokens |
| 3 | **CoinGecko** | Fallback for tokens not indexed by Alchemy |
| 4 | **DB cache** | Last resort; prevents valuation crashes |

It is imperative that Alchemy fetches the price successfully. If all tiers fail, trade is **rejected** (fake/illiquid token protection).


### E. Agent Memory & Trade Reasoning

Each LLM maintains persistent memory:
- **Last 10 trades** with full reasoning
- Memory informs future decisions (thesis continuity)
- Versioned per model + prompt combination

Every trade stores:
```json
{
  "agentId": "gpt-5.2",
  "action": "BUY",
  "symbol": "AAPLx",
  "quantity": 2.5,
  "price": 178.50,
  "rationale": "Apple showing strength post-earnings. RWA tokenized version trading at slight discount to spot. Macro environment supportive for tech. Adding to existing position.",
  "timestamp": "2026-02-04T10:30:00Z"
}
```

### F. Error Handling & Fallbacks

| Scenario | System Behavior |
|----------|-----------------|
| **LLM API timeout** | Skip agent for this cycle; retry next cycle; no partial trades |
| **LLM returns invalid trade** | Reject silently; log error; continue with other agents |
| **Price feed failure** | Use tiered fallback (Alchemy → CoinGecko → DB cache) |
| **All price sources fail** | Reject trade; do not execute with stale/unknown price |
| **Insufficient balance** | Reject trade; log insufficient funds error |
| **Token not found** | Reject trade; prevents fake token execution |
| **Rate limit hit** | Exponential backoff; queue retry |
| **Allocation > Available Cash** | UI prevents submission; show error message |
| **Withdrawal > Available Cash** | UI prevents submission; show error message |

**Principle**: Fail safe. If uncertain, do nothing rather than risk user capital.

---

## VII. Revenue & Fee Model

### A. Fee Schedule

| Fee Type | Rate | When Collected |
|----------|------|----------------|
| **Management Fee** | 1% of allocated amount, prorated if < 1 year | Upon unallocation or portfolio close |
| **Performance Fee** | 20% of realized profits | When user closes AI manager (on profits only) |
| **Withdrawal Fee** | 1 USDC flat | Per withdrawal |
| **Premium Subscription** | TBD | Monthly; unlocks Phase 2 + Phase 3 features |


**Example:**

User deposits 2,000 USDC, allocates $1,000 to ChatGPT. ChatGPT grows portfolio to $1,200. User closes ChatGPT after 6 months, then withdraws all.
- Management fee: ~$5 (1% × $1,000 × 0.5 years, collected at close)
- Performance fee: $40 (20% × $200 profit, collected at close)
- User receives to main wallet: $1,155, totalling $2,155.
- User withdraws: 2,155 USDC − 1 USDC withdrawal fee = 2,154 USDC.


> [!IMPORTANT]
> **Open Question:**
>- How to prorate management and performance fee on partial unallocation?
>- Will losses offset performance fee applied to gains on a per asset basis?
>
> **Decision needed before engineering begins.**


### B. Fee Hooks (Configurable)

> [!WARNING]
> **All fee values must be implemented as configurable placeholders in code** (e.g. environment variables or a config/fees table), not hardcoded. The following fee hooks must all exist, even if their current default is 0:
>
> | Fee Hook | Current Default | Notes |
> |----------|----------------|-------|
> | **Deposit fee** | 0 USDC flat | Per deposit |
> | **Deposit minimum amount** | 1 USDC | Minimum accepted deposit |
> | **Withdrawal fee** | 1 USDC flat | Per withdrawal |
> | **Withdrawal minimum amount** | 1 USDC | Minimum accepted withdrawal |
> | **Allocation fee** | $0 or 0% | Per allocation transaction (can be fixed or %) |
> | **Unallocation fee** | $0 or 0% | Per unallocation transaction (can be fixed or %) |
> | **Trade execution fee** | $0 or 0% | Per trade deployed by AI Manager (can be fixed or %) |
> | **Performance fee (% of profit)** | 20% | On realized profits at AI Manager close |
>
> These fees are **not mutually exclusive** — for example, if the allocation fee is non-zero, the unallocation fee will likely be zero to avoid double-charging. The goal is to have all hooks wired up so the fee structure can be adjusted without code changes.



---

## VIII. Technical Architecture

### A. Infrastructure
- Responsive web app (mobile-first, 390px mobile, 1440px desktop)
- Real-time price feeds (WebSocket)
- Custodial wallet infrastructure (secure key management)
- Trade execution via Solana DEX aggregators (Jupiter)

### B. Security
- SOC 2 compliance roadmap
- Multi-sig for platform wallets
- Rate limiting, DDoS protection
- Audit logging for all trades
- Wallet addresses truncated in UI (e.g., "3K9C...h7iK")

### C. Data
- Per-trade logging with LLM reasoning
- User activity audit trail
- Performance snapshots (for historical charts)
- Transaction history with status tracking

### D. Responsive Design

| Breakpoint | Width | Layout Notes |
|------------|-------|-------------|
| **Mobile** | 390px | Single column, stacked cards, full-width buttons |
| **Desktop** | 1440px | Multi-column layouts, sidebar possible, centered content areas |

**Key responsive differences from Figma**:
- Onboarding forms: 490px card on desktop, full-width on mobile
- AI Manager grid: 3 columns desktop, 2 columns mobile
- Deposit modal: 520px on desktop, full-screen on mobile
- Allocation confirmation: 522px on desktop, full-screen on mobile
- Performance chart: Full-width on both, but with more horizontal labels on desktop

---

## IX. Roadmap

### Phase 1 - Initial Asset Scope Universe

- **Solana Tokenized RWA Assets**: Tokenized stocks, bonds, ETFs, and commodities on Solana (e.g., xStocks/Backed Finance: AAPLx, TSLAx, NVDAx, SPYx)

### Phase 2 — User Agency Features

> *"Give users a sense of control without micromanaging or overprompting the AI"*

| Parameter | Description |
|-----------|-------------|
| **Max Drawdown Tolerance** | Auto-pause trading if portfolio drops by X% |
| **Stop-Loss per Position** | Individual position loss limit |
| **Min Market Cap** | Exclude tokens below threshold |
| **Min/Max Transaction Size** | Constrain single trade amounts |
| **Avg Deployment %** | Target % of capital deployed vs. cash |

**Override Hierarchy**: AI agents may autonomously set any of these parameters, and should not be prompted to do so. User-defined parameters always take precedence.

### Phase 3 — Extended Asset Scopes

Available as a multi-select when creating/editing AI Manager allocations:
- **All Solana Tokens**: Full Solana ecosystem including meme coins, DeFi tokens
- **Hyperliquid Tokens**: Perpetuals and spot on Hyperliquid
- **ETH Tokens**: Ethereum mainnet DeFi and L2 tokens
- **Prediction Markets**: Polymarket, Kalshi



---

## X. FAQ

### A. Fees

**Q: What fees does BlackEdge charge?**

| Fee | Rate | When Collected |
|-----|------|----------------|
| **Management Fee** | 1% annually | Upon unallocation or portfolio close |
| **Performance Fee** | 20% of profits | Upon unallocation or portfolio close |
| **Withdrawal Fee** | 1 USDC flat | Per withdrawal |
| **Deposit Fee** | 0 USDC | Per deposit |

**Q: How are the management and performance fees calculated?**

Both fees are collected when you unallocate funds from an AI Manager.
- **Management fee** — 1% per year on the amount you allocated, prorated for time. If you close after 6 months, you pay half a year's worth.
- **Performance fee** — 20% of profits only. If the AI lost money, you pay $0 in performance fees.

**Example**: You allocate $1,000 to ChatGPT. Six months later, ChatGPT has grown your portfolio to $1,200.

| | Calculation | Amount |
|--|-------------|--------|
| Profit | $1,200 − $1,000 | $200 |
| Management fee | 1% × $1,000 × 0.5 years | $5 |
| Performance fee | 20% × $200 profit | $40 |
| **You receive back** | $1,200 − $5 − $40 | **$1,155** |

Fees are only charged on what you actually earned. If your AI Manager loses money, the performance fee is zero — you only pay the small management fee.

**Q: What if a portfolio loses money?**

No performance fee is charged on losses. Management fee still applies.

### B. Risk & Disclaimers

**Q: Can I lose money?**

Yes. AI trading carries significant risk. LLMs can and will make mistakes. You may lose some or all of your deposited funds.

**Q: Is BlackEdge giving financial advice?**

No. BlackEdge is a technology platform. We do not provide financial advice.

### C. How LLMs Trade

**Q: What instructions do the LLMs receive?**

> "You are an autonomous trading agent. Your sole objective is to **maximize returns** on the portfolio you manage. You have access to market data, research tools, and trading execution."

**Q: Do different LLMs have different strategies?**

No. All LLMs receive identical prompts, market data, trading constraints, and asset scope. Performance differences reflect each model's reasoning capabilities.

**Q: Can I see what the LLM is thinking?**

Yes. Every trade includes full reasoning/rationale from the LLM, accessible in the Trade History tab.

### D. Capital & Withdrawals

**Q: Can I allocate all my funds to one AI Manager?**

Yes. There are no allocation limits.

**Q: How fast are withdrawals?**

About 20 minutes for Solana network confirmation, minus 1 USDC fee. Funds must be in Available Cash (unallocated) to withdraw.

**Q: What happens if I don't log in?**

Trading continues 24/7. The AI operates autonomously. You'll receive notifications for significant events.

---

*Document maintained by: Product Team*  
*Last updated: March 3, 2026*  
*Next review: After closed beta feedback*

