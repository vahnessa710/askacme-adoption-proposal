# ADR-003: Grant of Write Access for AskACME (Pilot Phase)

**Status:** Proposed
**Date:** Day 2, AI Enablement Sprint
**Deciders:** Consulting pod

---

## Context

The dossier frames this as a core design question, not an afterthought: *"where retrieval ends and agency begins."* Read access (retrieval — answering questions) and write access (agency — actually changing something in a live system) carry fundamentally different risk profiles, and conflating them is a classic way pilots get into trouble.

Three facts from discovery bear directly on this decision:

1. **The most obvious candidate for write access is a PartyPal-style action** — e.g., updating a delivery window — since that's the shadow-AI behavior AskACME is ultimately meant to displace. But our own Day 1 Tool-Use & Integration Map already flagged this as **"Later phase — not part of the initial pilot,"** following the stated philosophy of building read-only trust first.
2. **Production changes require CAB approval with a rollback plan** (dossier constraint #4). A write action taken by the assistant *is* a production change happening in real time — which raises the question of who's accountable for it, and whether it can even be "rolled back" the way a code deployment can.
3. **No read-path permission model is fully verified yet** — Salesforce's record-level enforcement (ADR-002) is still pending verification, and even our included pilot sources rely on ingestion-time tagging that hasn't been tested against real ACME data. Granting *write* access before *read* authorization is proven correct would stack an unverified, higher-consequence capability on top of an already-unverified one.

## Options Considered

**Option A — No write access in the pilot; read-only for all sources**
AskACME can only answer questions. It cannot create, update, or delete anything in any connected system.
- *Pros:* Matches the "build trust before power" philosophy already established across Day 1 (every prioritized use case is read-only). Removes an entire category of risk — a wrong answer is embarrassing; a wrong *action* (e.g., double-booking a delivery, corrupting a rental inventory record) is an operational incident.
- *Cons:* Doesn't yet address the actual PartyPal replacement need — Birdie Roadman's team still depends on PartyPal's booking/action capability, which this phase doesn't touch.

**Option B — One narrow, low-risk write action as a pilot exception**
Allow a single, low-consequence write — e.g., the assistant can create a ServiceNow ticket on a user's behalf, but nothing tied to customer-facing operations (deliveries, inventory, bookings).
- *Pros:* Starts building real operational experience with agentic actions and human-in-the-loop review, without touching anything customer-facing or hard to reverse.
- *Cons:* Still requires its own approval workflow, its own RFC, and — per the agentic-governance gap noted in program guidance — human-in-the-loop review for any consequential action. Adds scope and a new risk surface before the read-side is even proven.

**Option C — Grant broader write access now, including delivery-window updates**
Move directly toward replacing PartyPal's action capability.
- *Pros:* Directly addresses the highest-visibility shadow-AI dependency (events ops).
- *Cons:* Highest risk by a wide margin. Combines an unverified permission model, no existing human-in-the-loop pattern, and a customer-facing, hard-to-reverse action (a wrong delivery window affects a real event). Given ACME's own litigation history (the Trebuchet incident) around event-day mishaps, a write-access mistake here isn't hypothetical reputational risk — it's the same category of incident that's already produced a legal claim. Rejected outright for the pilot.

## Decision

**Adopt Option A — no write access in the pilot.** AskACME is read-only across every included source for this phase, with no exceptions.

Reasoning: this isn't a permanent "no" — it's a sequencing decision consistent with everything already established. The read path itself isn't fully trust-verified yet (ADR-002); introducing write access, even narrow (Option B), adds a second unverified system (human-in-the-loop approval, rollback capability, CAB's rollback-plan requirement applied to an AI-initiated change) before the first one has evidence behind it. Option C is rejected outright given the direct parallel to ACME's own litigation history.

**Phase 2 trigger (not a date):** write access becomes a live design question only after (a) the pilot's golden question set has passed consistently across at least one full eval cycle, and (b) a specific, narrow write action is proposed with its own RFC, rollback plan, and human-in-the-loop approval step — evaluated on its own merits, not bundled into this pilot.

## Consequences

- **PartyPal is not replaced by this pilot.** Birdie's team keeps using PartyPal through this phase — this must be stated explicitly in the proposal, not left implied, since "what happens to PartyPal" is a required element of the Day 4 rollout plan.
- **Success metrics for this pilot stay read-only in nature** (e.g., ChatGPT-usage reduction, time-to-answer improvement) — none of the Day 1 use cases can claim credit for reducing PartyPal's booking/action traffic, only its question-answering traffic, if any.
- **This sets up a clean, separate decision point for Phase 2** rather than a vague "eventually we'll add actions" — the trigger conditions above become a concrete milestone for the Day 4 project plan and rollout phasing.
- **Risk-register entry:** "PartyPal's action capability remains unaddressed post-pilot" — owner: Birdie Roadman (as the dependent stakeholder) / Dana Okafor (sponsor) — mitigation: Phase 2 write-access proposal scoped and RFC'd separately, gated on pilot eval success.

---
*This completes the minimum three ADRs required for Day 2, Deliverable 3: (1) model hosting, (2) ACL enforcement point, (3) write-access grant.*
