# AskACME — High-Level Architecture (HLA) Package
**Day 2, Deliverable 1** · C4 Container altitude · No product names, per ARB convention

---

## 1. Diagram

```mermaid
C4Container
    title AskACME — High-Level Architecture (Container View)

    Person(employee, "ACME Employee", "Sales, Depot Ops, Events Ops, Legal, Exec, etc.")

    System_Boundary(idp, "Identity Providers") {
        System_Ext(okta, "SSO Provider", "Session auth for employees")
        System_Ext(ad, "Directory of Record", "Source of truth for workforce identity")
        System_Ext(entra, "Cloud Identity Sync", "Synced from directory of record, M365")
    }

    System_Boundary(prod, "AskACME — Production Zone") {
        Container(gateway, "AI Gateway", "Chokepoint", "Central authN, logging, redaction, per-team cost attribution")
        Container(retrieval, "Permission-Aware Retrieval Layer", "ACL enforcement", "Enforces classification tiers via ingestion-time permission tags, checked at query time; content and permission tags refresh on separate cycles (ADR-002)")
        Container(model, "Model Serving", "LLM inference", "Resumes existing hosted model pilot, contingent on legal/DPA status check (ADR-001); self-hosted fallback if unresolved")
    }

    System_Boundary(nonprod, "Non-Production / Test Zone") {
        Container(eval, "Eval Harness", "Golden question set", "Normal + honeypot test cases; must pass before promotion to Production")
    }

    System_Boundary(included, "Data Sources — Included (Pilot Scope)") {
        SystemDb_Ext(hr, "HR Policies", "Internal — all employees")
        SystemDb_Ext(sop, "Depot SOPs", "Internal — depot-ops, events-ops")
        SystemDb_Ext(wiki, "Engineering Wiki", "Internal — it-eng")
        SystemDb_Ext(armory, "Rental Inventory System", "Internal — nightly batch only, freshness disclaimer required")
        SystemDb_Ext(snow, "ITSM Knowledge Base", "Internal — KB articles only, incidents excluded")
    }

    System_Boundary(gated, "Data Sources — Gated (Deferred, Phase 2)") {
        SystemDb_Ext(sfdc, "CRM System", "Confidential — pending ACL verification + external access grant, 4-8wk chain")
        SystemDb_Ext(salesent, "Sales Enablement Docs", "Confidential — pending internal ACL verification only")
        SystemDb_Ext(prodsafety, "Product Safety Docs", "Confidential — pending internal ACL verification only")
    }

    System_Boundary(eu, "EU Data Residency Boundary (GDPR)") {
        SystemDb_Ext(workday, "HRIS", "Restricted — excluded from pilot; EU personal data must never leave this boundary if ever in scope")
    }

    System_Boundary(excluded, "Data Sources — Excluded") {
        SystemDb_Ext(sap, "ERP System", "Excluded — Project Fresh Start migration collision")
        SystemDb_Ext(legal, "Legal / Claims Repository", "Excluded — Restricted, active legal hold, honeypot")
        SystemDb_Ext(corpdev, "Corporate Development Docs", "Excluded — Restricted, honeypot")
    }

    Rel(employee, gateway, "Sends query", "HTTPS, via inspected egress")
    Rel(gateway, idp, "Authenticates / authorizes caller")
    Rel(gateway, retrieval, "Forwards authorized request")
    Rel(retrieval, hr, "Reads (pilot)")
    Rel(retrieval, sop, "Reads (pilot)")
    Rel(retrieval, wiki, "Reads (pilot)")
    Rel(retrieval, armory, "Reads (pilot, batch-fresh only)")
    Rel(retrieval, snow, "Reads (pilot, KBs only)")
    Rel(retrieval, sfdc, "Blocked — pending verification + access grant", "dashed")
    Rel(retrieval, salesent, "Blocked — pending ACL verification", "dashed")
    Rel(retrieval, prodsafety, "Blocked — pending ACL verification", "dashed")
    Rel(retrieval, model, "Sends authorized, grounded context")
    Rel(model, gateway, "Returns generated response")
    Rel(gateway, employee, "Returns answer")
    Rel(eval, gateway, "Gates promotion — must pass before any release")
```

*(Paste into a `.md` file in your repo — GitHub renders Mermaid natively. No image export needed, satisfies "rendered from the repo.")*

---

## 2. One-Page Narrative

**Components.** Four containers carry the system: an **AI Gateway** (the mandatory chokepoint — authN, logging, redaction, per-team cost attribution, satisfying the no-shadow-AI and cost-visibility requirements), a **Permission-Aware Retrieval Layer** (enforces Public→Restricted classification at query time — the model never sees a forbidden chunk), **Model Serving** (the LLM itself — intentionally unnamed at this altitude), and an **Eval Harness** sitting in a separate Non-Production zone, running the golden question set (normal + honeypot cases) before anything promotes to Production.

**Trust boundaries.** Three boundaries matter here: **Production vs. Non-Production** (separates the live assistant from where new versions are tested — a SOX-driven line, since developers cannot deploy their own changes straight to Production); the **EU Data Residency boundary** (GDPR — wraps any source containing EU personal data; currently only the excluded HRIS sits here, but the boundary is drawn regardless of current scope so it's structurally ready if that ever changes); and the **Identity boundary**, where the Gateway authenticates every caller before a query reaches retrieval.

**Data flows.** An employee's query passes through the Gateway (authenticated against the identity boundary), into the Retrieval Layer, which reads only from the five **included** pilot sources, then forwards grounded context to Model Serving, and returns an answer back through the Gateway. All flows are **read-only** in this phase — per ADR-003, no write/action capability is in scope for the pilot. This means PartyPal's booking/action role for Events Ops remains untouched and undisplaced by this phase; that gap is logged as a Phase 2 item, not silently absorbed into this architecture.

**Integration points & Day 1 traceability.** Every Day 1 data source is accounted for:

| Status | Sources | Reason |
|---|---|---|
| **Included** | HR Policies, Depot SOPs, Eng Wiki, Rental Inventory System, ITSM KBs | Low-risk, internal classification, no personal data; supports the three pilot use cases |
| **Gated (Phase 2)** | CRM System, Sales Enablement, Product Safety | ACL enforcement unverified; CRM additionally requires an external access grant (est. 4–8 weeks, gated by security review and change-board cadence — not a committed date) |
| **Excluded** | ERP System, Legal/Claims, Corp Dev docs, HRIS | ERP: territorial team, active migration collision. Legal/Claims & Corp Dev: Restricted, honeypot test cases. HRIS: Restricted personal data, also sits inside the EU boundary |

No integration is a "write" action in this phase — the assistant only reads. Where retrieval ends and agency begins is therefore simple for the pilot: it doesn't begin yet.

---

**Deferred to LLA (next deliverable):** the 5-stage environment promotion path (DEV→SIT→UAT→PREPROD→PROD), specific model-hosting choice, chunking/embedding strategy, and concrete authN/authZ configuration.

**Traceability to ADRs:** Model Serving → ADR-001 (hosting choice, contingent on DPA verification). Retrieval Layer → ADR-002 (ingestion-time tagging, dual refresh cadence). Absence of any write relationship in this diagram → ADR-003 (no write access in pilot).
