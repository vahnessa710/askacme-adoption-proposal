# ADR-001: Model Hosting Approach for AskACME

**Status:** Proposed — pending confirmation of Azure OpenAI pilot's DPA/legal status
**Date:** Day 2, AI Enablement Sprint
**Deciders:** Consulting pod. Note: Dana Okafor (sponsor) and Marcus Reyes (CISO) are named as the eventual approvers/verifiers of the assumption below, but no actual stance from either has been obtained — see Assumption note.

---

## Context

AskACME needs an LLM to generate answers from retrieved, permission-filtered context. Three paths exist, and this is one of the highest-stakes decisions in the whole engagement — it determines where ACME's data physically goes, what compliance obligations attach, and how fast the pilot can actually launch.

Two facts from discovery directly shape this decision:

1. **An existing, partially-built asset already exists**: the dossier notes a stalled Azure OpenAI pilot in `eastus2` — one of the "stalled AI experiments" under the current shadow-AI/governance gap.
2. **A real legal blocker exists somewhere in this space**: "an enterprise LLM agreement is stalled in legal over DPA terms and a no-training-on-our-data clause." It is **not yet confirmed** whether this stalled agreement is the same one covering the Azure OpenAI pilot, a separate vendor negotiation, or both.

**Assumption logged — unconfirmed, not sourced from any stakeholder statement:** we are *assuming*, for planning purposes only, that the Azure OpenAI pilot's underlying agreement is separate from the stalled third-party LLM negotiation and already satisfies the DPA/no-training requirement. Neither the dossier nor any stakeholder interview actually states this — it does not say whose deal is stalled, or whether it's the same one covering the Azure OpenAI pilot. This is a gap we filled to keep planning moving, and it is the first thing to verify with Dana/Legal (and likely Ingrid Voss, as DPO) before any real data flows through this component.

**Decision trigger — bounded, not open-ended:**
- **Status check (1–2 weeks):** ask Legal/Procurement to confirm whether the Azure OpenAI pilot's agreement is or isn't part of the stalled negotiation. This is a lookup against an existing contract, not a new negotiation, so it should be fast.
- **If confirmed clean:** proceed with Option B.
- **If confirmed entangled:** pivot to Option A (self-hosted) **immediately** — do not wait further. The stalled negotiation is already tied to governance efforts that have been stuck for months with no signal of resolving soon; waiting on it further is an open-ended bet, not a bounded risk.

## Options Considered

**Option A — Self-hosted, open-weight model**
Runs entirely inside ACME's own infrastructure (on-prem VMware or AWS/Azure tenancy). No data ever leaves ACME's network boundary.
- *Pros:* Strongest data-residency and DPA story — nothing to negotiate, satisfies Marcus's "no data leaves without a DPA" mantra by construction. No dependency on the stalled legal agreement.
- *Cons:* Requires GPU serving infrastructure, MLOps capability, and model-quality tuning ACME doesn't currently have in-house. Slowest to stand up. Adds a new, unfamiliar operational burden on a team already stretched (S/4 migration, WMS, etc.).

**Option B — Existing Azure OpenAI pilot (eastus2)**
Resumes and builds on the already-started Azure OpenAI pilot.
- *Pros:* Not starting from zero — infrastructure and Azure-tenant integration partially exist. Sits inside the Azure boundary ACME already trusts for M365/Entra, simplifying identity integration. Enterprise cloud AI offerings typically include contractual no-training commitments, which may resolve Marcus's condition without new negotiation.
- *Cons:* Contingent on the assumption above — if this pilot is in fact entangled with the stalled DPA negotiation, this option is blocked until Legal resolves it. Data does leave ACME's own infrastructure (though staying within the Azure tenant boundary).

**Option C — External API (direct third-party provider)**
Calls a model provider's API directly, outside any existing ACME cloud tenancy.
- *Pros:* Fastest to integrate technically; no infrastructure to stand up.
- *Cons:* Weakest data-residency position — data leaves ACME's trusted boundary entirely. Most exposed to the exact concern Marcus has already flagged. No existing contractual relationship to lean on. Almost certainly the option most entangled with "an enterprise LLM agreement... stalled in legal."

## Decision

**Adopt Option B — resume and build on the existing Azure OpenAI pilot** — contingent on Legal/Procurement confirming (before any production data flows through it) that this specific pilot's agreement already satisfies the DPA and no-training requirements, and is distinct from the stalled third-party negotiation.

**Fallback:** if the status check comes back "entangled with the stalled negotiation," or if no clear answer is obtained within the 1–2 week status-check window, fall back to **Option A (self-hosted)** rather than Option C — self-hosted is the only option that doesn't depend on an external legal resolution at all, even though it costs more time and effort up front.

Option C is rejected outright for this engagement: it carries the highest data-exposure risk for no corresponding speed advantage over Option B, which already has a head start.

## Consequences

- **If confirmed:** AskACME's model-serving component lives inside ACME's existing Azure tenancy, simplifying the identity/auth story (Entra-native) and giving Priya's cost-attribution requirement an existing Azure billing/tagging structure to hook into.
- **If not confirmed:** the project pivots to Option A, which pushes out the pilot timeline (GPU serving stand-up, MLOps tooling) and becomes its own Day 3 cost driver (self-hosted GPU serving is one of the largest possible line items in the cost model).
- **Either way:** this decision must be re-verified, not assumed permanent — public/cloud model versions change without notice (noted separately in program guidance), so any future model version bump is itself a change-managed event, not a silent update.
- **Risk-register entry:** "Azure OpenAI pilot DPA status unconfirmed" — owner: Dana Okafor (sponsor) — mitigation: 1–2 week status check with Legal before any pilot data flows through the model; if entangled or unresolved past that window, trigger self-hosted fallback immediately, no further wait.

---
*This is ADR-001 of the minimum three required for Day 2, Deliverable 3. Candidates for the remaining two: (a) ACL-enforcement point (where in the retrieval layer permission checks happen), (b) any grant of write access (currently none — read-only for pilot).*
