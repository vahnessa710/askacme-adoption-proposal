# AskACME — High-Level Architecture (HLA) Package
**Day 2, Deliverable 1** · C4 Container altitude · No product names, per ARB convention

---

## 1. Diagram

```mermaid
C4Container
    title AskACME - High-Level Architecture

    Person(employee, "ACME Employee", "All roles")

    System_Boundary(idp, "Identity Providers") {
        System_Ext(okta, "SSO Provider", "Session auth")
        System_Ext(ad, "Directory of Record", "Identity source")
        System_Ext(entra, "Cloud Identity Sync", "M365 identity")
    }

    System_Boundary(prod, "Production Zone") {
        Container(gateway, "AI Gateway", "Chokepoint", "AuthN, logging, cost attribution")
        Container(retrieval, "Retrieval Layer", "ACL enforcement", "Tag-based, ADR-002")
        Container(model, "Model Serving", "LLM inference", "Hosted pilot, ADR-001")
    }

    System_Boundary(nonprod, "Non-Production Zone") {
        Container(eval, "Eval Harness", "Golden question set", "Promotion gate")
    }

    System_Boundary(included, "Included Sources") {
        SystemDb_Ext(hr, "HR Policies", "Internal")
        SystemDb_Ext(sop, "Depot SOPs", "Internal")
        SystemDb_Ext(wiki, "Engineering Wiki", "Internal")
        SystemDb_Ext(armory, "Rental Inventory System", "Batch only")
        SystemDb_Ext(snow, "ITSM Knowledge Base", "KBs only")
    }

    System_Boundary(gated, "Gated Sources") {
        SystemDb_Ext(sfdc, "CRM System", "Phase 2")
        SystemDb_Ext(salesent, "Sales Enablement Docs", "Phase 2")
        SystemDb_Ext(prodsafety, "Product Safety Docs", "Phase 2")
    }

    System_Boundary(eu, "EU Data Residency") {
        SystemDb_Ext(workday, "HRIS", "Excluded")
    }

    System_Boundary(excluded, "Excluded Sources") {
        SystemDb_Ext(sap, "ERP System", "Excluded")
        SystemDb_Ext(legal, "Legal Claims Repository", "Excluded")
        SystemDb_Ext(corpdev, "Corporate Development Docs", "Excluded")
    }

    Rel(employee, gateway, "Sends query")
    Rel(gateway, idp, "Authenticates")
    Rel(gateway, retrieval, "Forwards request")
    Rel(retrieval, hr, "Reads")
    Rel(retrieval, sop, "Reads")
    Rel(retrieval, wiki, "Reads")
    Rel(retrieval, armory, "Reads")
    Rel(retrieval, snow, "Reads")
    Rel(retrieval, sfdc, "Blocked")
    Rel(retrieval, salesent, "Blocked")
    Rel(retrieval, prodsafety, "Blocked")
    Rel(retrieval, model, "Sends context")
    Rel(model, gateway, "Returns response")
    Rel(gateway, employee, "Returns answer")
    Rel(eval, gateway, "Gates promotion")
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
