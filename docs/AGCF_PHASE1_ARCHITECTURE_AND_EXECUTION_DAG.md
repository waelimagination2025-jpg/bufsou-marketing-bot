# AGCF Phase 1 — Architecture Decisions and Execution DAG

**Date:** 2026-11-15  
**Branch:** `agcf/phase0-forensic-audit`  
**Purpose:** Convert the forensic findings into an evidence-led implementation plan for the existing live BUFSOU storefront.

## 1. Product decision

Build BUFSOU first as a **single-retailer dropshipping storefront**. Do not begin as an open multi-vendor marketplace.

Why:
- BUFSOU currently owns the customer-facing brand and one featured skincare set.
- A marketplace would require seller onboarding, seller verification/KYC, split settlement, seller payouts, dispute ownership, seller policies, and marketplace-specific legal analysis.
- A single-retailer model is materially simpler for proving one real customer-to-delivery transaction.
- Supplier adapters can be abstracted so a marketplace model can be reconsidered after product-market fit.

The business is the retailer/merchant of record only if its legal and payment arrangements actually establish that role. Confirm with qualified local counsel and the payment provider.

## 2. Commercial strategy

### Stage A — Repair and prove the existing BUFSOU offer
Do not immediately replace the existing premium skincare positioning or discard campaign assets. First verify:
- Actual legal product identity and SKU.
- Manufacturer and country of origin.
- Complete ingredient list and formula concentrations.
- Product safety substantiation and test reports.
- Label artwork and responsible-person information.
- Actual supplier and available inventory.
- Landed cost to each destination.
- Shipping service, tracking, delivery time, and return route.
- Historical/reference price supporting any comparison price.
- Authentic review records and permission to publish.
- Evidence supporting every clinical, dermatological, vegan, origin, safety, and efficacy claim.

If these records are not available, suspend the relevant claims or product launch rather than inventing proof.

### Stage B — Prove one complete transaction
The first commercial milestone is one testable product and one destination, with:
- A live product record.
- A secure server-side cart and checkout.
- Payment provider approval and a hosted payment session.
- Signed payment webhook and idempotent payment state.
- Durable server-side order record.
- Supplier order submission after payment confirmation.
- Supplier acceptance.
- Tracking synchronization.
- Customer notification.
- Refund/exception path.
- Financial reconciliation.

### Stage C — Expand to 20–50 products
Only add products with verified supplier identity, stock/price freshness, destination eligibility, compliance evidence, shipping/return economics, and positive risk-adjusted contribution margin.

### Stage D — Build the 1,000-product catalog
Treat 1,000 as a qualified catalog target, not a launch-day listing quota. Do not use filler products or unverified sales/ranking claims.

## 3. Target architecture

```text
BUFSOU Storefront (Vercel or selected host)
        |
        v
Commerce API / Application Backend
        |
        +--> Product Catalog & Compliance Registry
        +--> Cart & Checkout
        +--> Payment Adapter
        |       +--> Approved Jordan merchant PSP
        |
        +--> Order State Machine
        |       +--> Durable SQL Database
        |       +--> Idempotency / Audit Log
        |
        +--> Supplier Adapter Layer
        |       +--> CJdropshipping (candidate)
        |       +--> DSers/AliExpress (candidate)
        |       +--> Other approved suppliers
        |
        +--> Fulfillment & Tracking
        +--> Refunds / Disputes / Exceptions
        +--> Customer Notifications
        +--> Financial Ledger & Reconciliation
        |
        v
Admin Console / Analytics / AI Decision Support
```

### Design rules

- The frontend must never be the authority for payment or order state.
- Never store customer names, email addresses, or shipping addresses in localStorage.
- Never store card data; use provider-hosted checkout/tokenization.
- A signed server-side payment webhook is the authority for paid status.
- A supplier order may be submitted only after verified payment and policy checks.
- All order submissions must be idempotent to prevent duplicate supplier orders.
- Supplier APIs are isolated behind adapters; do not couple the storefront to one supplier.
- Product publication requires compliance status and claim substantiation.
- All prices must be computed from verified cost components and freshness windows.
- Customer data must have access control, retention policy, and deletion/export process.
- Use sandbox and test transactions before live production.
- Owner approval is required for spending, production release, supplier commitments, paid advertising, and external commercial obligations.

## 4. Payment feasibility decision

### Initial candidates

| Candidate | Evidence | Decision |
|---|---|---|
| PayTabs Jordan / MEPS | Official Jordan-specific merchant page, local settlement in local currency, multiple currencies, international cards, API and plugins | FIRST TO QUALIFY |
| Tap Payments | Official MENA payment acceptance information and source describing Jordan service | SECOND TO QUALIFY |
| HyperPay | MENA payment gateway and regional presence; Jordan merchant eligibility and payout details not established here | HOLD FOR DIRECT CONFIRMATION |
| PayPal | Country directory located; exact Jordan merchant receiving, withdrawal, and limitations not confirmed | HOLD |
| Stripe | Jordan absent from official supported merchant-country list inspected | NOT A JORDAN-DOMICILED DEFAULT |
| Shopify Payments | Jordan absent from official supported-country list inspected | THIRD-PARTY PSP REQUIRED IF SHOPIFY IS CHOSEN |

Before selecting any PSP, obtain written answers to:
1. Can a Jordan-registered individual/company open a merchant account for this business category?
2. Can it accept Visa/Mastercard/Amex payments from US customers?
3. Which currencies can be presented and settled?
4. Which Jordanian bank accounts can receive settlement?
5. What are setup, monthly, transaction, FX, refund, chargeback, and reserve fees?
6. Does the provider permit cross-border dropshipping of skincare/cosmetics?
7. Does it support hosted checkout, APIs, signed webhooks, refunds, and recurring/one-time payments?
8. What documents and minimum operating history are required?

No account creation, subscription, or payment provider commitment is authorized by this plan.

## 5. Supplier strategy

### Candidate 1: CJdropshipping
CJ's official developer documentation exposes product synchronization, order synchronization, webhooks, sandbox, logistics, dispute, shop, and supplier API sections.

Documentation: https://developers.cjdropshipping.com/en/api/start/

Use it as the first technical proof-of-concept candidate, subject to:
- Account/API access.
- Cosmetics product eligibility.
- Manufacturer and ingredient documentation.
- Sample quality.
- Stock/price freshness.
- Destination coverage.
- Landed cost.
- Delivery service levels.
- Return/refund route.
- Contractual terms.

### Candidate 2: DSers / AliExpress
DSers developer/API documentation exists, but the actual available API scopes, account access, and supported automation must be confirmed before committing.

Documentation: https://www.dsers.dev/api/channel_app_api

Do not assume a supplier listing is independently verified. Supplier platform badges and advertised metrics are signals, not a substitute for due diligence and sample testing.

## 6. Compliance gates

### United States
The live page is US-focused. Before selling cosmetics into the US:
- Review MoCRA facility registration and product listing duties and applicable exemptions.
- Identify the responsible person and labeling obligations.
- Maintain ingredient, safety substantiation, adverse-event, and traceability records as applicable.
- Review claims carefully: claims to affect skin structure/function or treat disease can cause a cosmetic to be regulated as a drug or both.
- Confirm importer and customs responsibilities for the actual shipping model.

Official FDA references:
- https://www.fda.gov/cosmetics/registration-listing-cosmetic-product-facilities-and-products
- https://www.fda.gov/cosmetics/cosmetics-laws-regulations/modernization-cosmetics-regulation-act-2022-mocra
- https://www.fda.gov/cosmetics/cosmetics-labeling-regulations/cosmetics-labeling-guide
- https://www.fda.gov/cosmetics/cosmetics-international-activities/cosmetics-importers

### Jordan
The Jordan Trade Portal identifies JFDA cosmetics registration/import procedures, including JFDA approvals and importer-related process steps. The detailed procedure page returned a 503 during one retrieval; confirm current requirements directly with JFDA/qualified local counsel before importing or selling into Jordan.

Official portal:
- https://tradeportal.customs.gov.jo/procedure/1801/step/11876?l=en&embed=true&includeSearch=true
- https://tradeportal.customs.gov.jo/procedure/1818/step/12691?l=en&includeSearch=true&embed=true

## 7. First implementation backlog

| ID | Priority | Work item | Acceptance evidence | Gate |
|---|---|---|---|---|
| AGCF-001 | P0 | Connect Vercel project and identify deployment source | Project ID, linked Git repository/ref or source export, deployment settings | OWNER-GATED |
| AGCF-002 | P0 | Preserve current live deployment/source/assets | Timestamped source snapshot and hashes | Required before edits |
| AGCF-003 | P0 | Disable unsafe localStorage PII and fake Payoneer redirect | Verified safe form behavior; no live personal data collection before backend | OWNER-GATED production change |
| AGCF-004 | P0 | Remove unsupported claims or map each to evidence | Claims-to-evidence register and approved copy | OWNER-GATED publication |
| AGCF-005 | P0 | Verify exact product and manufacturer | SKU dossier, ingredients, labels, safety and origin documents | OWNER-GATED |
| AGCF-006 | P0 | Qualify PSP | Written merchant eligibility, fees, settlement, API/webhook docs | OWNER-GATED |
| AGCF-007 | P0 | Qualify supplier and sample | Supplier dossier, sample inspection, landed-cost quote | OWNER-GATED |
| AGCF-008 | P1 | Design commerce backend and database | Reviewed schema, API contract, threat model | Technical |
| AGCF-009 | P1 | Implement product catalog and compliance registry | Automated tests and admin import | Technical |
| AGCF-010 | P1 | Implement cart and server-side order state machine | Unit/integration tests; idempotency proof | Technical |
| AGCF-011 | P1 | Implement PSP sandbox adapter | Signed webhook tests; no live charges | PSP access needed |
| AGCF-012 | P1 | Implement supplier sandbox adapter | Product/order/stock test fixtures | Supplier access needed |
| AGCF-013 | P1 | Implement tracking, exceptions, refunds, reconciliation | End-to-end sandbox test evidence | External APIs needed |
| AGCF-014 | P2 | Pilot 20–50 qualified products | Complete product and supplier dossiers | Compliance gate |
| AGCF-015 | P3 | Expand catalog to 1,000 qualified candidates | Source-backed product records and scoring evidence | Data gate |

## 8. First-release acceptance criteria

The first release is not "ready" until all are true:
- A customer can add a real product and quantity to a cart.
- Checkout creates a server-side order in a non-paid state.
- Payment provider creates a valid hosted checkout session.
- Payment status changes only after a verified webhook.
- Replayed webhooks do not duplicate payment/order transitions.
- Supplier order is not submitted before verified payment.
- Supplier order submission is idempotent.
- Tracking updates are recorded and visible.
- Refunds and failures have explicit states.
- PII is never stored in browser localStorage.
- Customer privacy and retention controls are documented.
- Product and destination compliance gates pass.
- Unit economics are positive after variable costs in the approved base case.
- A full sandbox transaction is reproducible from logs.
- Production activation receives explicit owner approval.

## 9. Immediate blocker

The live website is hosted at Vercel, but the inspected GitHub repository contains only the marketing scripts and promotional assets. The frontend source and deployment configuration are not present in its recursive tree.

Therefore the next technical action is to connect the Vercel workspace so the actual project, linked source repository, environment configuration names, build settings, domains, and deployments can be inspected.

Until that source is found, modifying the marketing repository cannot safely update the live storefront.
