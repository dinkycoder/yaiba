# ADR 0001: Learned model as the engine, rules as the safety envelope

- **Status:** Accepted
- **Date:** Oct 16, 2026
- **Detail:** [AI-centred plan](../capstone/ai-centered-plan.md)

## Context

Yaiba answers tapd's Portfolio Pilot `ResolveRequest` calls. Through the pinned proto (commit `9db270f`) it
can set a rate, an `expiry_timestamp` and an `accepted_max_amount`, or reject. Those levers price the free
American call option. The Phase 3 plan prices it with fixed rules: short expiries, volatility-based spreads
and per-peer walk-away tracking. Fixed rules can't adapt to each peer, and they charge honest peers the same
as opportunists. Yaiba is also the founder's Quantic MSAIE capstone (Aug 16 to Nov 21, 2027), which must show
AI engineering throughout. A learned model has its own risks: little real data, no real attacker data
before mainnet, possible leakage, and silent failure.

## Decision

1. A learned free-option detector is the pricing engine. It predicts the chance that a quote ends as an
   exercised free option, and Yaiba turns that into terms.
2. The Phase 3 rules guard stays as a three-part envelope:
   - the **baseline** the model must beat;
   - the **fallback** when the model is down, slow, out of distribution or uncertain;
   - **hard limits** the model can never exceed: maximum expiry, minimum spread and per-peer exposure caps.
3. The model climbs offline, then shadow, then live, with an exit gate at each rung.
4. An operator risk agent (an LLM with read-only tools) explains decisions and proposes policy. A human
   approves every change, and the agent never trades or edits policy.
5. The detector (through shadow mode), the operator agent, the evaluation report and a signet-only demo are
   the capstone deliverables.

## Alternatives considered

- **Rules only.** Simple and auditable, but the AI component would be weak, and honest peers would pay for
  opportunists. Rejected; the rules are kept as the envelope.
- **Model only, no envelope.** Most flexible, but one bad model or feature bug could lose real funds without
  limit. Rejected.
- **LLM decides on quotes.** Too slow and too hard to calibrate for a per-quote price, and it can be steered
  by peer-supplied text. Rejected; the LLM is kept advisory.

## Consequences

- **Good:** losses can fall without overcharging honest peers, a bad model can never exceed operator
  limits, and every decision records which path made it.
- **Cost:** two decision paths to test and maintain, plus a label pipeline, a market simulator and MLOps.
- **Risk:** training data is simulated or scripted until mainnet, so results must be reported as such.
- **Plan changes:** the README phase table now names the AI deliverables. The operator agent moves from
  Alpha to the capstone so that it is new capstone work.
