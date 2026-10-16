# Yaiba

Risk tooling for Lightning edge nodes that convert stablecoins to and from bitcoin.

Yaiba plugs into an edge node's Taproot Assets daemon (`tapd`) as its **Portfolio Pilot**, the external
service tapd consults before it accepts any trade. On every quote, Yaiba prices and guards against the
**free American call option**, and it can screen the counterparty. It starts on Taproot Assets and is
built so an RGB adapter can follow.

> **Status:** Phase 0 (Discovery). No code yet. This README is the build plan and will be updated as each
> phase closes.

## Why it exists

On Taproot Assets, only the first and last hops of a payment know which asset is moving. Every hop in
between sees plain bitcoin; this is called *translated forwarding*. The two edge nodes at either end do
the currency conversion, earn a spread, and carry the risk.

The biggest of those risks is the free option. An edge node locks an exchange rate when it offers an
HTLC, but the receiver decides later, up to the timeout, whether to reveal the secret that settles the
payment. If the rate moves in the receiver's favor, they settle; if not, they let the payment expire at
no cost. The edge node absorbs the loss. Yaiba's job is to find that gap between price lock and final
decision, and charge for it.

## Glossary (plain English)

| Term | Meaning |
|---|---|
| Edge node | A Lightning node that holds stablecoin inventory and converts between that asset and bitcoin at the edge of the network. |
| tapd | The Taproot Assets daemon: the background program that holds assets, builds transactions and talks to peers. |
| RFQ | Request for quote: a peer asks the edge node what rate it will give for a trade. |
| Portfolio Pilot | An external service tapd asks to accept or reject each quote (added in tapd v0.8; still marked experimental). |
| HTLC | Hash time lock contract: a payment that completes only if the receiver reveals a secret before a deadline. |
| Free option | The receiver's costless right to walk away from a locked rate until the HTLC times out. |
| Shadow mode | Yaiba scores live traffic and records decisions but changes nothing. Operators see what it *would* have done. |
| regtest / signet / mainnet | A private Bitcoin network on your own machine, a public test network, and the real network. |

## How Yaiba is built: two ladders

Every phase moves up two ladders, one rung at a time, and never skips a rung. Each rung up makes a
mistake more expensive.

- **Network ladder:** regtest, then signet, then capped mainnet, then uncapped mainnet.
- **Model ladder:** rules baseline, then offline evaluation on logged quotes, then shadow mode, then live
  decisions.

## Phase plan

Each phase ends with an artifact and a pass/fail exit gate. The next phase doesn't start until the gate
passes. Dates line up with the Quantic MSAIE Class of December 2027 schedule, so each course feeds the
phase running alongside it.

| Phase | Dates | Goal | Artifact | Exit gate | MSAIE course alongside |
|---|---|---|---|---|---|
| **0. Discovery** | Oct 13 – Oct 25, 2026 | Fix the problem, the user and the test for success | `docs/prd.md`: user, problem, success metrics, non-goals, kill threshold | Founder signs the PRD and the kill threshold before any code | Managing AI Engineering: AI-Assisted Software Development; Managing AI Application Development I |
| **1. Design** | Oct 26 – Nov 8, 2026 | Decide how it works before coding | `docs/architecture.md`, `docs/adr/` (one page per major decision), `docs/threat-model.md`, `docs/lab/` (bet list and the five lab-agent definitions; see [Lab practice](#lab-practice)) | Every attack and failure mode has an owner and a mitigation, and the bet list has a trigger for each open upstream dependency | Managing AI Application Development II; User-Centered Design; Cloud Foundations |
| **2. Spike** | Nov 2 – Nov 15, 2026 | Prove the riskiest unknown on regtest | Throwaway Docker regtest network (bitcoind, 2× LND, 2× tapd), a stub pilot, `docs/spikes/001-portfolio-pilot.md` | tapd calls the stub pilot on every quote, and one test-stablecoin payment is priced by Yaiba | Cloud Foundations; Professionalism and Ethics |
| *Light weeks* | Nov 16 – Dec 6, 2026 | Protect the exam and course project | Draft quote-log and decision table schemas | None; no new build work | Managing AI Engineering Exam and Project |
| **3. MVP** | Dec 7, 2026 – Feb 21, 2027 | The smallest real product, test-first | Python Portfolio Pilot, rules-based free-option guard (short expiries, volatility-based spreads, per-peer walk-away tracking), PostgreSQL quote log, operator dashboard, CI, regtest benchmark baseline | All tests and CI green, and the guard measurably cuts edge-node losses in a replayed regtest scenario | Web Application and Interface Design (dashboard, databases, back end); Software Testing and CI/CD (tests, CI, performance monitoring) |
| **4. Alpha** | Feb 22 – Apr 11, 2027 | Real operators on a test network | Integration guide, open-source reference pilot, design-partner list (names kept private), signet deployment in shadow mode that logs every quote and its outcome as detector training data (the LLM operator agent moves to the capstone; see [ADR 0001](docs/adr/0001-ai-engine-rules-envelope.md)) | Kill threshold met: the demo is shown to at least 5 operators by Mar 31, 2027, and at least 1 runs Yaiba in shadow mode on signet by Apr 11. Otherwise, pivot | AI Engineering Techniques and Architectures (prompting, LLM apps, agents) |
| *Hardening* | Apr 12 – May 16, 2027 | Make it adapter-ready | ADR for the protocol-adapter layer; RGB adapter design once Utexo's Lightning support ships | Core logic has no tapd-specific code | Software Design and Architecture |
| *Model* | May 17 – Jul 11, 2027 | Make a learned detector the engine and keep the rules as its safety envelope ([ADR 0001](docs/adr/0001-ai-engine-rules-envelope.md)) | Regtest market simulator, labelled quote dataset (simulator plus alpha signet shadow logs), first offline free-option detector, offline evaluation report | Offline only, and it affects no decision: on a time-based split, saves more sats than the rules baseline at a fixed false-penalty rate, is calibrated, and passes the leakage check | Machine Learning to AI Model Fine-Tuning |
| **5. Beta** | Jul 12 – Sep 26, 2027 | Real money, deliberately limited | Security review, release checklist, incident runbook, pricing and go-to-market plan, containerised deployment (rules guard decides on mainnet). From Aug 16, capstone AI work: production detector pipeline and operator risk agent v1 | Capped mainnet with at least 1 operator, monitoring live, no unexplained losses through the Sep 20 capstone check-in. AI: the production detector re-passes the offline gate by Sep 20 | AI and Organizational Transformation (go-to-market); Capstone Kickoff Aug 16; Microservices Architectures (deploy, test, scale) |
| *Capstone* | Aug 16 – Nov 21, 2027 (overlaps Beta) | Build and prove the AI components as new team work ([plan](docs/capstone/ai-centered-plan.md)) | Detector in signet shadow mode beside the rules guard, operator risk agent (read-only tools, grounding checker, human-approved proposals) with its eval set, evaluation report, signet-only demo, written report and presentation | Detector shadow gate reported (met or not), agent grounds 100% of numbers on its eval set, and the work is submitted by Nov 21, 2027, 11:59 p.m. PT. No graded result depends on mainnet | Specialization Period; Capstone check-ins Sep 20 and Oct 18 |
| **6. General availability** | From Dec 2027 | Scale | Service-level agreement, pricing, support process, second protocol adapter | Repeatable operator onboarding, and revenue covers running costs | After graduation (Dec 23, 2027) |

Winter break (Dec 21, 2026 – Jan 3, 2027) is not counted as build time.

## Lab practice

Yaiba's platform moves under it: the Portfolio Pilot only arrived in tapd v0.8 and is still experimental,
RGB's Lightning support hasn't shipped, and stablecoin issuers are backing more than one Bitcoin protocol.
So Yaiba runs a small lab alongside the phase plan, inspired by the "operate like a lab team" idea. The lab
feeds the phase plan; it never replaces it. Setting it up is a Phase 1 task.

- **Budget:** about one day a week. The rest goes to the current phase.
- **Bet list (`docs/lab/bets.md`):** every prediction is written as a bet with a trigger condition, for
  example "when RGB's Lightning support ships, build the RGB adapter" or "when the Portfolio Pilot loses its
  experimental label, harden Yaiba's interface to it". The trigger turns a forecast into an action.
- **Five lab agents (`docs/lab/agents/`):**
  - *Scout:* a weekly scheduled task that reads tapd and RGB release notes, the BOLTs and bLIPs repositories
    and Delving Bitcoin, writes a short "what just became possible" brief, and flags any bet whose trigger
    fired.
  - *Spike engineer:* builds time-boxed, throwaway prototypes on regtest, each in its own worktree.
  - *Red team:* attacks the free-option guard to find gaps.
  - *Evaluator:* read-only; scores every spike against a fixed rubric. The agent that builds never grades.
  - *Demo builder:* turns a surviving spike into a two-minute demo.
- **Graduation rule:** every spike ends one of two ways. It becomes an ADR plus a phase-plan item, or it is
  archived with a one-line note on what it taught. Only the founder decides which.
- **Limit:** agents find what's newly possible; only operators can say what they'll pay for.

## Running alongside the phases

- **Co-founder search** from Phase 3: ideally a Go engineer who already runs Lightning infrastructure.
- **Funding:** an open-source grant application once the reference pilot is public (Phase 4); a strategic
  pitch to stablecoin and Bitcoin-infrastructure investors after Alpha shows operator usage.
- **Community:** Tampa Bay BitDevs, the taproot-assets GitHub repository, and Delving Bitcoin.

## Non-goals (for now)

- No custom smart contracts, and nothing on non-Bitcoin chains.
- Yaiba never makes legal conclusions. Counterparty screening defaults to *advise mode* (accept, record a
  signed REVIEW_REQUIRED decision, alert the operator). Rejecting trades is an opt-in *enforce mode* the
  operator turns on.
- Compliance screening is the second feature. Yaiba leads with free-option risk, which helps a single
  operator from day one.

## Pinned upstream

| Component | Reference |
|---|---|
| Taproot Assets | [lightninglabs/taproot-assets](https://github.com/lightninglabs/taproot-assets), Portfolio Pilot interface at `taprpc/portfoliopilotrpc/portfolio_pilot.proto`, commit `9db270f` (Oct 9, 2026) |
