# ADR-002: ACL Enforcement Point for Permission-Aware Retrieval

**Status:** Proposed
**Date:** Day 2, AI Enablement Sprint
**Deciders:** Consulting pod

---

## Context

The dossier's non-negotiable constraint is blunt: *"the assistant must never reveal anything the asking identity couldn't access in the source system."* The honeypot rule makes this testable and unforgiving — if Mo Alvarez asks about "Project Slingshot" and gets anything back, that's a **failed authorization test**, not a minor bug.

The hard question isn't *whether* to enforce permissions — it's **where in the pipeline** the check happens. Get this wrong and one of two failure modes shows up:
- Enforce too late → the model already "saw" a forbidden chunk before anything filtered it out. Even if the final answer looks fine, the sensitive content already passed through a system that shouldn't have had it — a real exposure, not just a UX issue.
- Enforce too rigidly at the wrong layer → the assistant becomes useless (e.g., blocking entire sources instead of individual documents), or too slow to be usable.

Three complicating facts from Day 1 make this ACME-specific, not generic:
1. **SharePoint permissions are "wildly inconsistent"** across thousands of sites (dossier, Section 4) — so trusting a live, real-time permission check against SharePoint itself is risky; the permissions being checked may themselves be wrong.
2. **ArmoryOS is nightly-batch only** — there is no live API to check against at query time, even if we wanted to.
3. **Salesforce requires record-level sharing rules** — a finer-grained check than "can this role see this SharePoint site," and one we've already flagged (ADR context, Day 1 Tool-Use Map) as unverified.

## Options Considered

**Option A — Enforce live, at query time, by calling each source system's permission API directly**
For every query, check the requester's access against SharePoint/Salesforce/etc. in real time before returning results.
- *Pros:* Always reflects the source system's *current* permission state — no staleness.
- *Cons:* Impossible for ArmoryOS (no live API — batch-only by design). Inherits SharePoint's own inconsistent permission structure directly into every query — if SharePoint's ACLs are wrong, so is AskACME's. Adds real latency per query (multiple live calls before answering).

**Option B — Enforce at retrieval time, using permission metadata captured at ingestion**
When each document is ingested and indexed, tag it with its classification tier and authorized audience (captured from the source system at that point in time). At query time, the retrieval layer filters candidate chunks against the requester's identity/group membership *before* anything reaches the model.
- *Pros:* Works uniformly across all sources, including ArmoryOS (tags come from the same nightly batch that already delivers its content). Fast — filtering is a metadata comparison, not a live API round-trip. Matches the dossier's own framing exactly: "the model never sees forbidden chunks."
- *Cons:* Permission tags can go **stale** — if someone's access is revoked mid-cycle, the index won't reflect that until the next refresh. Requires a defined refresh cadence for *permissions*, separate from content freshness.

**Option C — Enforce only at the application/output layer, after retrieval**
Retrieve broadly, then filter or redact the assistant's final answer based on the requester's access.
- *Pros:* Simplest to build first.
- *Cons:* Directly violates the dossier's explicit design principle — the model has already processed forbidden content by the time filtering happens. This is the failure mode the honeypot test is specifically designed to catch. Rejected outright.

## Decision

**Adopt Option B — enforce at retrieval time via ingestion-time permission metadata**, with one refinement: **separate the refresh cadence for permissions from the refresh cadence for content.**

Reasoning: Option A is disqualified by ArmoryOS alone (no live API exists), and would import SharePoint's own inconsistency problem directly into the assistant. Option C is disqualified outright — it's the exact anti-pattern the dossier warns against. Option B is the only one that works uniformly across every Day 1 source, including the batch-only and the API-driven ones.

**The refinement matters:** content and permissions don't change at the same rate or for the same reasons. A document's *text* might be stable for months; a person's *access* can change the day they leave a team. So:
- **Content refresh** follows each source's natural cadence (already documented in the Day 1 Data Source Inventory — e.g., SharePoint near-real-time, ArmoryOS nightly).
- **Permission refresh** runs on its **own, faster cycle** — re-checking who's authorized for what — independent of whether the underlying content changed at all.

This means the retrieval layer's metadata store needs two timestamps per chunk, not one: last content sync, last permission sync.

## Consequences

- **Staleness window exists and must be disclosed**, same pattern as the ArmoryOS "data freshness" disclaimer from Day 1 — except this time it's a *permission* freshness risk, not a content one. Worth a similar user-facing disclaimer or, better, a tightly-bounded permission refresh interval for higher-sensitivity sources.
- **Salesforce's record-level requirement becomes an ingestion-time design problem**, not a query-time one: the ingestion pipeline must capture record-level sharing rules as metadata tags accurately enough to trust at retrieval time. This is exactly what "verify record-level permission enforcement" (Day 1 Tool-Use Map, ADR context) needs to prove *before* Salesforce can move out of gated status — it's not just "get access," it's "prove the tagging is correct."
- **Golden question set (eval gate) must specifically test staleness**, not just point-in-time correctness — e.g., a test case simulating "this person's access was just revoked" to confirm the permission-refresh cycle catches it within its defined window.
- **Risk-register entry:** "Permission metadata staleness between refresh cycles" — mitigation: defined, source-specific permission refresh cadence (faster than content refresh where source sensitivity is higher); eval gate includes a staleness/revocation test case.

---
*This is ADR-002 of the minimum three required for Day 2, Deliverable 3. Remaining candidate: any grant of write access (currently none proposed — read-only for pilot; see Tool-Use & Integration Map).*
