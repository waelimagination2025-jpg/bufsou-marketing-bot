# BUFSOU Live Storefront Forensic Audit

**Audit date:** 2026-11-15  
**Live URL:** https://bufsou-atelier-store.vercel.app/  
**Repository context:** `waelimagination2025-jpg/bufsou-marketing-bot`  
**Audit branch:** `agcf/phase0-forensic-audit`  
**Method:** Live page scrape plus isolated browser interaction. Only the first purchase CTA was clicked to inspect navigation. No customer data was entered, no form was submitted, no payment action was completed, and no order was placed.

## 1. Executive finding

A live BUFSOU Atelier Paris landing page exists at the Vercel URL referenced by the repository's marketing workflow.

The live page is a polished, single-product skincare landing page for a four-formula set at a displayed price of **$54.95 USD** with a displayed comparison price of **$145.00 USD**.

However, the observed purchase flow is **not a verified e-commerce checkout**.

The "Add to Cart" and "Reserve" buttons scroll to a registration form. The browser-side JavaScript:
1. Validates contact and US shipping fields in the browser.
2. Generates a client-side order identifier.
3. Sets an order status to `PENDING_FULFILLMENT`.
4. Saves the order/customer data to browser `localStorage` under the configured BUFSOU orders key.
5. Displays a confirmation modal.
6. Starts a five-second redirect to `https://payoneer.com`.

No backend order API, server-side order persistence, payment session, payment authorization, verified payment webhook, supplier order submission, inventory reservation, or tracking flow was identified in the inspected purchase logic.

**Critical distinction:** A browser-local registration record is not a paid order. A redirect to the Payoneer homepage is not a merchant payment checkout. A client-generated pending status is not evidence of payment or fulfillment.

## 2. Live storefront features observed

- Brand presentation: BUFSOU · Atelier Paris.
- Single featured offer: four-formula skincare set.
- Displayed price: $54.95 USD.
- Displayed comparison price: $145.00 USD.
- Product names: Clinical Cleanser, Hydrating Toner, Advanced Bio-Tech Serum, Volufiline Moisturizer.
- Long-form marketing and editorial sections.
- Promotional videos and image assets hosted on external domains.
- Product, shipping, and refund information accordions.
- Registration form fields: full name, email, street, city, US state, ZIP.
- CTA buttons: Reserve, Add to Cart, Reserve the Set.
- Payment destination in the client script: `https://payoneer.com`.

## 3. Purchase-flow verification

| Step | Observed behavior | Assessment |
|---|---|---|
| Click Add to Cart | Scrolls to registration form | Not a cart |
| Product selection | One hardcoded set | No catalog/variants |
| Quantity | No selector observed | Not implemented |
| Customer details | Collected in browser form | Present |
| Validation | Client-side JavaScript | Not server validation |
| Order ID | Generated in browser | Not authoritative |
| Order persistence | Browser localStorage | No central order database |
| Order status | Client sets PENDING_FULFILLMENT | Premature/unverified |
| Payment handoff | Redirect to Payoneer homepage | Not a payment session |
| Payment confirmation | No server webhook identified | Not implemented/verified |
| Supplier handoff | No live integration identified | Not implemented/verified |
| Shipment tracking | No live integration identified | Not implemented/verified |

The form's HTML default action points to the current page, while a JavaScript submit handler prevents normal submission and handles the data locally.

The browser code calls a purchase tracking hook after local registration, before any verified payment. If the tracking hook is active, this can count registrations as purchases and corrupt conversion and revenue analytics.

## 4. Critical privacy and transaction risks

### P0 — Customer personal data is not sent to an order backend
The inspected script stores names, email addresses, and shipping addresses in browser localStorage. This creates no durable merchant-side order record and exposes sensitive data to scripts running in the same origin/browser context.

Required remediation:
- Stop collecting customer addresses until a secure server-side order path exists.
- Do not store customer PII in localStorage.
- Use a server-side order service with access controls, encryption, retention limits, and audit logging.
- Collect only data required for fulfillment and payment.

### P0 — No verified payment transaction
The redirect goes to the Payoneer homepage, not a BUFSOU-specific hosted checkout/payment URL or verified payment session.

Required remediation:
- Choose a payment provider that approves the owner's legal entity and payout country.
- Create a real hosted checkout session server-side.
- Treat signed provider webhooks as the source of truth for payment state.
- Do not mark an order paid or route it to a supplier before verified payment.

### P0 — Misleading order status
The browser creates a `PENDING_FULFILLMENT` record before payment. This status is semantically unsafe because fulfillment should not be pending until the merchant has a confirmed paid order and an accepted supplier workflow.

Required state model:
`CART → CHECKOUT_PENDING → PAYMENT_PENDING → PAID → SUPPLIER_SUBMISSION_PENDING → SUPPLIER_ACCEPTED → FULFILLMENT_PENDING → SHIPPED → DELIVERED`

Include explicit failure, cancellation, refund, dispute, and manual-review states.

### P1 — False purchase analytics risk
The purchase event is called at registration confirmation, not after payment confirmation. Separate `lead/checkout_started`, `payment_pending`, and `purchase` events. Emit `purchase` only after verified payment.

## 5. Product, advertising, and trust-claim audit

The live page makes claims that require documentary substantiation before continued use in paid advertising or regulated markets.

Examples observed:
- French/Parisian atelier and award-winning dermatology laboratory origin.
- "Medical-grade" and "medical PDRN".
- PDRN at 3% and Volufiline at 5%.
- Alpine botanical sourcing and 100% certified vegan/eco-luxe ingredients.
- Dermatologically validated, hypoallergenic, and "verified completely safe" claims.
- Claims about restoring structural volume, reversing lines, draining congestion, erasing dark circles, and tightening.
- 10,000+ community reviews, 1,439 reviews, and 4.9/5 rating.
- $145 comparison price and 62% discount.
- Free US shipping, 24-hour dispatch, 3–6 business-day delivery, and full insurance.
- 30-day money-back guarantee.

This audit does not establish that these statements are false. It establishes that no supporting certificates, lab reports, ingredient dossiers, supplier records, authentic review dataset, historical price evidence, carrier contracts, or refund reserve evidence were supplied in the inspected repository or live page.

Until evidence is supplied:
- Remove or suspend unsubstantiated medical, clinical, origin, certification, safety, efficacy, review, price-comparison, shipping-speed, and guarantee claims.
- Use conservative cosmetic appearance/experience wording that is supportable.
- Do not invent reviews, identities, clinical validation, certifications, or scarcity.
- Retain only real, verifiable customer reviews with valid consent and source records.

## 6. Market and payment feasibility — preliminary research

The owner operates from Jordan. The following is an initial feasibility screen, not payment-provider approval or legal advice.

### Stripe
Stripe's official global availability page lists supported merchant-account countries. Jordan does not appear in the published supported-country list at the time of this audit.

Source: https://stripe.com/global

Do not design the launch around a Jordan-registered Stripe account unless Stripe confirms eligibility directly. Do not use a borrowed, nominee, or fictitious foreign entity to bypass onboarding restrictions.

### Shopify Payments
Shopify's official supported-country page says businesses located outside its listed countries need a third-party payment provider.

Source: https://help.shopify.com/en/manual/payments/shopify-payments/supported-countries

Jordan does not appear in the country list retrieved during this audit. If Shopify is selected, third-party payment-provider eligibility and transaction fees must be confirmed before build commitment.

### PayPal
The official PayPal global country directory was located, but the precise Jordan account features, commercial receiving, withdrawal route, limitations, and merchant eligibility were not conclusively verified in this audit.

Source: https://www.paypal.com/en/webapps/mpp/country-worldwide

Do not assume the live Payoneer homepage redirect constitutes Payoneer checkout integration. Payoneer merchant acceptance, checkout product availability, account eligibility, and payout conditions require direct confirmation.

### CJdropshipping
CJ publishes API documentation covering product synchronization, order synchronization, webhooks, sandbox, logistics, disputes, shops, and supplier functions.

Source: https://developers.cjdropshipping.com/en/api/start/

CJ is a credible candidate for a supplier integration proof-of-concept, subject to account/API access, product eligibility, cosmetics documentation, landed cost, destination coverage, and sample quality.

### DSers / AliExpress
DSers publishes developer/API information for supplier and channel applications. Access, API scope, supported workflows, and commercial plan requirements must be confirmed before selecting it as the core integration.

Source: https://www.dsers.dev/api/channel_app_api

### Jordan cosmetics compliance
The Jordan Trade Portal search surfaced a JFDA cosmetics-registration/import procedure, but the official page returned HTTP 503 during retrieval. The exact current requirements therefore remain unverified.

Source: https://tradeportal.customs.gov.jo/procedure/1801/step/11876?l=en&embed=true&includeSearch=true

Do not sell/import skincare into Jordan until current JFDA classification, registration/import requirements, labeling, ingredient, and responsible-party obligations are confirmed with the competent authority or qualified local specialist.

## 7. Problem tree — live site

### Root problem
The live page presents a checkout-like experience without a verified server-side commerce transaction system.

### Cluster A — Conversion and checkout
- CTA wording implies a cart, but it only scrolls to a form.
- No quantity/variant selection.
- No verified payment checkout.
- No durable order confirmation.

### Cluster B — Customer data
- Personal information is held in browser localStorage.
- No verified server-side privacy, retention, or access controls.
- No confirmed order-management back office.

### Cluster C — Fulfillment
- No supplier order API.
- No inventory or price synchronization.
- No carrier/tracking integration.
- No verified delivery or return workflow.

### Cluster D — Trust and compliance
- Multiple clinical, medical, efficacy, origin, review, and shipping claims need evidence.
- The current form is US-only.
- Cosmetic destination-market eligibility is not established.

### Cluster E — Analytics
- Purchase event may fire before payment.
- No verified transaction ID from a payment provider.
- Revenue and ROAS reporting would be unreliable if based on this event.

## 8. Immediate remediation priorities

### P0 — Protect customers and brand
1. Disable the current fake checkout/Payoneer redirect before accepting customer data or representing the flow as payment.
2. Stop storing customer names, emails, and addresses in localStorage.
3. Stop labeling browser registrations as paid/pending-fulfillment orders.
4. Remove unsupported medical, safety, certification, review, discount, shipping, and scarcity claims until substantiated.
5. Preserve a read-only copy of the current live page/source assets before changing deployment.

### P1 — Establish the actual business model
1. Decide seller-of-record and initial target countries.
2. Confirm the owner's legal business/payment eligibility.
3. Confirm actual product identity, manufacturer, ingredient list, safety documentation, packaging, origin, and stock source.
4. Obtain supplier landed-cost and sample evidence.
5. Establish shipping, refund, and returns economics.

### P2 — Build a real transaction system
1. Server-side catalog and product records.
2. Cart with quantity and variant handling.
3. Secure hosted payment checkout.
4. Signed payment webhooks and idempotency.
5. Server-side order database and audit trail.
6. Supplier adapter and order state machine.
7. Tracking synchronization and customer notifications.
8. Refund and reconciliation operations.

### P3 — Expand catalog
Only after the first product passes supplier, compliance, payment, fulfillment, and contribution-margin gates should the system expand toward 20–50 products and later the 1,000-product target.

## 9. Evidence classifications

- `VERIFIED-LIVE`: landing page and registration UI are publicly accessible; CTA scroll behavior was observed.
- `VERIFIED-LIVE-CODE`: browser JavaScript locally stores submitted registration details and redirects to Payoneer homepage after a countdown; observed without submitting any real data.
- `NOT FOUND`: real cart, payment session, server-side order backend, supplier integration, and fulfillment integration in the inspected purchase flow.
- `UNKNOWN`: actual payment-provider merchant account, actual supplier agreement, product manufacturing provenance, substantiation of claims, live fulfillment capability, and legal approvals.
- `OWNER-GATED`: payment account selection, spending, supplier commitments, live deployment, and commercial publication.

## 10. Final conclusion

BUFSOU has a live, visually developed single-product landing page and a small marketing-automation repository. It does **not** yet demonstrate a complete functioning online store.

The immediate priority is not adding 999 more products. It is replacing the false checkout with a secure, real, auditable transaction lifecycle, validating the underlying product and supplier, and proving one legitimate order from payment through delivery and financial reconciliation.

No live form was submitted, no customer information was entered, no payment was made, no supplier was contacted, and no production deployment was changed during this audit.
