# AskACME — Day 1 Needs Analysis Package
**Kickoff & Needs Analysis Deliverables 1–4**

---

## 1. Stakeholder & Role Map

| Role | Group | Needs from AskACME | Current Pain Point | What Success Looks Like | Blocker Risk? |
|---|---|---|---|---|---|
| **Wiley E. Cunningham III (CEO)** | exec | A credible "AI story" he can take to the board | Board is asking "what's our AI story?" and he has none | Headline-level win he can point to — doesn't need the details | Low as a technical blocker, but can create pressure to move faster than governance allows — watch for scope-creep asks |
| Dana Okafor (CIO) | exec | Governed, provable wins — not just speed, but control | Burned by the S/4 delay; needs to show the board a credible AI story | Metrics showing adoption + reduced shadow-AI usage | Low — she's the sponsor, but needs to stay convinced |
| Marcus Reyes (CISO) | exec, security | Full visibility into data flow; no data leaving without a DPA | No gateway, no cost attribution, shadow AI everywhere | All traffic routed through an approved gateway, with logging/redaction | High — he can block the project if it isn't secure by design |
| Priya Natarajan (VP Data) | it-eng, data | Central AI gateway + per-team cost attribution | No model registry, no eval practice, no cost visibility | A gateway that enforces authn, logging, and cost-per-team tracking | High — this is her explicit "veto condition" |
| Birdie Roadman (VP Events Ops) | events-ops | A proper, official replacement for PartyPal | Her team depends on an unowned, unsupported shadow tool | AskACME can do everything PartyPal does, plus official support | Moderate — she doesn't want to lose functionality |
| Tom Kowalczyk (Change Manager/CAB) | it-eng | An RFC with a rollback plan before anything goes to production | Proposals often lack a proper rollback plan | Clean CAB approval, no emergency changes | High — he's literally the production gatekeeper |
| Ingrid Voss (DPO) | legal, dpo | A complete DPIA; EU personal data stays in the EU | She guards claims files and personal data very tightly | No EU data leaves the EU; approved DPIA | High — explicitly flagged as a key stakeholder who can block adoption |
| Sal Marchetti (Sales Director) | sales | Fast access to pricing/account info, as a substitute for ChatGPT | Forced to use personal ChatGPT because existing tools are slow/limited | His personal ChatGPT use stops (DLP risk resolved) | Low as a blocker — he's an end user, but he's the "problem" being solved |
| Mo Alvarez (Depot Manager) | depot-ops | Questions about inventory/rental status, despite distrust of non-green-screen tools | Only knows how to use ArmoryOS; data isn't real-time | A simple, trustworthy way to get information he believes | Moderate — resistant to change even without a security reason |
| Gene Podowski (Contractor) | it-legacy | — (not an end user, but a critical dependency) | He's the only person who understands ArmoryOS's RPG code | His integration into ArmoryOS work isn't disrupted | Moderate — if he's unavailable, integration work stalls |
| **HR** *(assumption — no named persona in dossier)* | hr | Assurance that Workday/comp data stays out of AskACME's reach; wants HR Policies content served correctly | No visibility into how an AI assistant might expose Restricted HR data | AskACME never surfaces Workday comp/performance data outside HR + self | Moderate — not vocal yet, but Restricted-tier HR data makes them a natural veto if not looped in early |

**Landmines surfaced (not discovered later):**
- **PartyPal** — Birdie's team's unowned shadow tool, no logging/ACLs.
- **Sal's personal ChatGPT use** — active DLP finding waiting to happen.
- **Copilot rollout stuck 5 months** — stalled in security review over SharePoint permission-sprawl; the exact ACL problem AskACME must solve differently.
- **Stalled enterprise LLM agreement (DPA / no-training clause)** — sitting in Legal/Procurement; could constrain external-API model choice in Day 2 architecture.

---

## 2. Data Source Inventory

| Source | Owner | Classification | Authorized Audience | Refresh Cadence | Volume | Pilot Status | EU/PII Flag |
|---|---|---|---|---|---|---|---|
| SharePoint — HR Policies | HR *(inferred)* | Internal | All employees | Real-time *(assumed)* | Moderate | ✅ Included | — |
| SharePoint — Depot SOPs | Events Ops/Depot Ops *(inferred)* | Internal | depot-ops, events-ops | Real-time *(assumed)* | Moderate | ✅ Included | — |
| SharePoint — Engineering wiki | IT Eng *(inferred)* | Internal | it-eng | Real-time *(assumed)* | Moderate | ✅ Included | — |
| SharePoint — Sales enablement | Sales *(inferred)* | Confidential | sales, exec | Real-time *(assumed)* | Low-Mod | ⚠️ Gated — needs ACL enforcement | — |
| SharePoint — Product Safety | Quality *(inferred)* | Confidential | quality, legal, exec | Real-time *(assumed)* | Low | ⚠️ Gated — safety/legal sensitivity | — |
| SharePoint — Legal/Claims | Legal *(inferred)* | Restricted | legal only | Real-time *(assumed)* | Low | ❌ Excluded — legal hold, honeypot | Possible — claims may include personal injury data |
| SharePoint — Corp Dev | Exec *(inferred)* | Restricted | exec only | Real-time *(assumed)* | Low | ❌ Excluded — honeypot | — |
| Salesforce | Sales/CS Ops *(inferred)* | Confidential | sales, service (record-level) | Real-time via API *(assumed)* | High | ✅ **Included for pilot — customer care use only** *(see UC1 note below)* | Yes — likely EU customer/contact data (Łódź-served accounts); residency control TBD, flag for Ingrid/DPIA |
| SAP ECC | Finance/Ops *(inferred)* | Confidential | finance, ops (role-based) | Batch/real-time depending on module *(per dossier)* | High | ❌ Excluded — Project Fresh Start collision | Possible — vendor master may include EU vendor data |
| ArmoryOS | Depot Ops *(inferred)* | Internal | depot-ops (nightly batch only) | Nightly batch only *(stated in dossier)* | Moderate | ⚠️ Included — with data-freshness disclaimer | — |
| Workday | HR *(inferred)* | Restricted | HR + self | Real-time via API *(assumed)* | Moderate | ❌ Excluded — too sensitive for pilot | Yes — EU employee comp/personal data (Łódź staff); must stay in-EU, hard exclude until DPIA scoped |
| ServiceNow — KB Articles | IT Eng *(inferred)* | Internal | it-eng; KBs to all employees | Real-time via API *(assumed)* | High | ✅ Included — KB only | — |
| ServiceNow — Incidents | IT Eng *(inferred)* | Internal | it-eng | Real-time via API *(assumed)* | High | ❌ Excluded for now — reporter PII, needs redaction design | — |

**Assumption log:**
- Only ArmoryOS's refresh cadence is stated in the dossier (nightly batch). All others are assumed real-time via API, pending confirmation with system owners.
- "Owner" is inferred from authorized-audience groupings; the dossier states *audience*, not *ownership* — to confirm during stakeholder interviews.
- ServiceNow was split into KB Articles vs. Incidents as a pilot-scoping decision; the dossier lists it as one system.
- **Salesforce moved into pilot scope** (from "later phase") specifically to support Use Case 1 — customer care case/account data only, read-only, respecting existing record-level sharing rules. This is a scope change from the original draft and is called out explicitly so it isn't mistaken for an oversight.
- **EU residency:** Salesforce, SAP, and Workday flagged as possibly containing EU personal data given ACME's Łódź operations. None currently have a stated residency control — ties directly to Ingrid Voss's non-negotiable and should gate any Confidential/Restricted source before it leaves pilot scope.

**Summary:**
- **Included:** HR Policies, Depot SOPs, Engineering wiki, ArmoryOS (w/ disclaimer), ServiceNow KB, Salesforce (customer care scope only)
- **Gated/Deferred:** Sales enablement, Product Safety, Salesforce (sales/pricing use — separate from customer care use above)
- **Excluded:** Legal/Claims, Corp Dev, Workday, SAP ECC, ServiceNow Incidents

---

## 3. Tool-Use & Integration Map

| System | Read or Write | Auth Model Required | Pilot Scope? | Reasoning |
|---|---|---|---|---|
| **AI Gateway** | N/A — sits between AskACME and all model calls; all traffic passes through it | Central authn, logging, redaction, per-team cost attribution | ✅ **Pilot — mandatory from day one** | Non-negotiable per Priya's veto condition and Marcus's "no data leaves without a DPA" stance. Without this, no integration below is compliant regardless of read/write status. |
| SharePoint (HR/SOPs/Wiki) | Read-only | AD → Entra ID (M365) → SharePoint native permission inheritance; Okta as broader SaaS SSO layer | ✅ Pilot | Simple retrieval; existing permission structure can serve as the basis for ACL enforcement |
| ArmoryOS | Read-only | Service account via nightly batch/SFTP. Note: an ancient SOAP endpoint also exists (Gene Podowski territory) but is treated as unavailable for pilot due to age/fragility | ✅ Pilot (with freshness disclaimer) | Retrieval only; batch-only architecture treated as the practical constraint |
| ServiceNow (KB Articles) | Read-only | Okta SSO + ServiceNow role-based access (limited to KBs tagged "all employees") | ✅ Pilot | Public-facing KBs, low risk |
| **Salesforce — customer care case/account data** | Read-only | Okta SSO + Salesforce record-level sharing rules (must enforce "service" audience per record) | ✅ **Pilot — pulled forward to support Use Case 1** | Customer care already lives in Service Cloud per the dossier; booking/order questions are case- and account-shaped, not depot-inventory-shaped. Record-level enforcement must be verified working before go-live. |
| Salesforce — sales/pricing data | Read-only (later) | Same as above, plus confidential-tier ACL enforcement | ⚠️ Later phase | Separate from the customer-care use above; pricing/discount data stays gated until ACL enforcement is proven |
| SharePoint (Sales enablement, Product Safety) | Read-only (once gated) | Okta SSO + confidential-tier ACL enforcement (must be verified working first) | ⚠️ Later phase | Part of the "gated" list — need to prove permission-aware retrieval works correctly before opening up |
| SAP ECC | Not applicable | N/A for pilot | ❌ Excluded | SAP team territorial due to Project Fresh Start; a landmine if included too early |
| Workday | Not applicable | N/A for pilot | ❌ Excluded | Too sensitive as personal data (compensation, performance) |
| PartyPal replacement — write actions (future) | Write (eventually — e.g., updating a booking/delivery window) | New auth model required — action-taking, not retrieval; needs explicit approval workflow | ❌ Later phase — not part of initial pilot | Riskiest "write" capability; follows the "build trust with read-only first" approach |
| Legal/Claims, Corp Dev (SharePoint) | None | N/A | ❌ Permanently excluded/gated | Honeypot test cases — tests of authorization parity, not normal system scope |

**Key point — where retrieval ends and agency begins** *(per Day 1 discovery notes, tied to dossier Section 9 constraint #1)*:
All pilot integrations are read-only. No write actions — even something as simple as updating a delivery window — are performed by AskACME during the pilot. Rationale:
- A wrong answer has minor impact; a wrong action (e.g., cancelling the wrong delivery) has major impact.
- Authorization parity (ACL enforcement) needs to be proven correct on read access before any write capability is considered.

---

## 4. Prioritized Use Cases & Success Metrics

### Use Case 1: PartyPal Replacement — Customer Care Booking Support
**What it is:** AskACME answers common customer care questions about bookings/orders using read-only Salesforce Service Cloud case and account data — as an official, secure alternative to PartyPal.
*(Revised from initial draft: originally scoped to Depot SOPs/ArmoryOS, which customer care isn't authorized to access per the dossier's audience table. Rescoped to Salesforce, which customer care already uses, to keep every access boundary exactly where the dossier draws it — no new authorization grants required.)*
**Measurable target:** 70% reduction in PartyPal query volume/usage within 90 days of pilot launch *(proposed target, pending baseline PartyPal usage data)*.
**Shadow-AI behavior displaced:** PartyPal — the unowned, unlogged GPT wrapper customer care currently depends on.
**Why this is a priority:** Most urgent and highest-risk shadow-AI situation — no owner, no security controls, an entire team quietly depending on it.

### Use Case 2: Depot Operations — Inventory & SOP Assistant
**What it is:** AskACME helps Mo Alvarez and other depot managers get information about rental inventory status (ArmoryOS, with freshness disclaimer) and refurb/assembly procedures (Depot SOPs) more easily than navigating the green-screen ArmoryOS interface.
**Measurable target:** 50% reduction in average time-to-answer for common inventory/SOP queries *(proposed target, pending baseline time-to-answer data)*.
**Shadow-AI behavior displaced:** None directly — this use case instead targets the adoption risk identified with Mo (distrust of non-green-screen tools). Success here becomes a proof point that the new interface is trustworthy.
**Why this is a priority:** Concrete, demonstrable quick win for a skeptical user group — important for broader adoption momentum.

### Use Case 3: Sales Team — General Policy & Process Q&A
*(Renamed from "Sales Enablement Q&A" — the actual Sales Enablement SharePoint source is gated/Confidential and is not used here. This use case draws only from HR Policies and Engineering wiki.)*
**What it is:** AskACME answers non-sensitive questions from the sales team about general company policies, HR info, and engineering documentation — as a first step toward reducing Sal Marchetti's (and similar sales staff's) use of personal ChatGPT.
**Measurable target:** 40% reduction in self-reported personal ChatGPT usage by the sales team for non-confidential queries within 90 days *(proposed target, pending baseline usage data)*.
**Shadow-AI behavior displaced:** Sal Marchetti's personal ChatGPT usage, and roughly half the sales team's — scoped specifically to non-sensitive queries, since pricing/discount data remains gated in the pilot.
**Why this is a priority, and why only partial for now:** Directly addresses the identified DLP risk, but is honestly scoped — pricing/discount queries aren't answerable yet since Sales Enablement stays gated. Logged as an explicit, deliberate limitation rather than an oversold capability.

### Why these three, and not others?
All three are read-only, all draw only from "included" pilot-scope sources (Salesforce added specifically and narrowly for customer-care case data), and each traces directly to a specific stakeholder pain point from the Role Map. Priya's (gateway/cost attribution) and Marcus's (DPA/data residency) requirements are treated as **cross-cutting platform constraints** rather than standalone use cases — every use case above is bound by them via the mandatory AI Gateway.

---

## Cross-Deliverable Consistency Notes
- Every high-risk stakeholder need (Birdie, Mo, Sal) has a corresponding use case, backed by a data source in scope, backed by a read-only integration path.
- Priya's and Marcus's needs are addressed as platform-wide constraints (the Gateway) rather than use cases — stated explicitly above so it doesn't read as an omission.
- The Use Case 1 authorization fix is now reflected consistently across the Data Source Inventory (Salesforce moved to pilot scope, customer-care only) and the Tool-Use Map (Salesforce split into two rows: customer-care case data vs. sales/pricing data).
- All four named landmines (PartyPal, Sal, Copilot, stalled DPA) appear in the package.
- Terminology ("gated," "excluded," "honeypot") is consistent across all four sections.
