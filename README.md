# Gale James - Senior Technical Writer

**API Documentation Specialist** | 9+ Years Experience | 73+ Endpoints Documented

I specialize in developer-facing API documentation for complex developer platforms. I documented over 73 REST API endpoints, led docs-as-code migrations, and built scalable documentation systems across multiple API versions. My domain background is in ecommerce and payment systems, where precise documentation directly affects integration success.

The examples provided here are original documentation projects created for this portfolio. This portfolio uses automated GitHub Actions workflows to validate links and Markdown on every push.

## 📚 Featured documentation projects

### BookHub Publisher API (complete API documentation set)

A complete documentation set for a RESTful publishing platform API, including OpenAPI spec, integration guide, versioning architecture decision, and release management.

**[View complete project →](./book-api-docs)**

- **[OpenAPI 3.0 specification](./book-api-docs/openapi-spec/bookhub_publisher_api_guide_v2_path.yaml)**: The YAML specification lets publishers manage their book inventory on BookHub via a RESTful interface. It includes authentication, versioning, and error handling.
- **[API integration guide](./book-api-docs/api-guide/bookhub_publisher_api_guide_v2_path.md)**: This guide covers path-based versioning, the new `hitCount` analytics feature, the removal of title sorting, migration steps from v1, authentication requirements, key endpoints, best practices, available support, and the deprecation policy.
- **[Versioning strategy](./book-api-docs/architecture/versioning-rationale.md)**: This document explains the rationale for API versioning in BookHub Publisher API v2. After evaluating header-based (e.g., api-version: 2024-12-30) and path-based (e.g., /v2/books) approaches, I chose path-based versioning, which diverged from the course instructor's recommendation; the rationale document explains why.
- **[Release management](./book-api-docs/release-management/release_management_v2_path.md)**: This document highlights the major changes, migration strategy, support resources, and key dates. This summary covers the focus on analytics, performance improvements, breaking changes, and what publishers and developers should do to migrate successfully to v2.

---

## 📖 Documentation by type (Diátaxis framework)

### Tutorials (learning-oriented)

- **[Making your first API call with Postman](https://app.guidde.com/share/playbooks/j2GEH6v4pUaJ2vo8ErNt9c?origin=hPx2XmP4HZbFEVrCFFzGjtJsDSB2)** *(video, 1:41)*: How to make a simple API call using Postman.
- **[Getting started with payments API](./tutorials/payment-api-getting-started.md)**: How to authenticate, navigate test vs. production environments, and make your first payment call in 15 minutes.
- **[Creating a webhook](./tutorials/webhook-creating-your-first.md)**: How to create, secure, and test webhook notifications.

### How-to guides (task-oriented)

- **[Implement retry logic for failed payments](./how-to-guides/payment-retry-logic.md)**: How to set up automatic retries for failed payments.
- **[Process a refund](./how-to-guides/payment-process-refunds.md)**: How to process a refund.
- **[Handle authorization failures](handling-authorization-failures.md)**: Implement retry logic for soft declines and handle hard decline notifications.
- **[Manage security for webhook data transmission](./how-to-guides/webhooks-data-transmission.md)**: How to protect webhook data.
- **[Manage webhooks using the Admin UI](./how-to-guides/webhooks-Admin-UI.md)**: How to create, edit, enable, disable, or delete webhooks, and manage secrets.
- **[Retrieve an anonymous customer's order and product information](./how-to-guides/webhooks-anonymous-shopper.md)**: How to retrieve an anonymous customer's order and product information.
- **[Manage API errors](./how-to-guides/manage-api-errors.md)**: How to identify, troubleshoot, and resolve common API error codes.
- **[Manage rate limits](./how-to-guides/manage-rate-limits.md)**: How to avoid rate limits.
- **[Prevent duplicate charges with idempotency keys](./how-to-guides/prevent-duplicate-charges-with-idempotency-keys.md)**: How to use idempotency keys to prevent duplicate charges.

### Explanation (understanding-oriented)

- **[Understanding payment authorization and capture](understanding-authorization-capture.md)**: Explains the two-step payment process, authorization holds, and decline types.
- **[Webhooks](./explanation/webhooks.md)**: Covers the key webhook concepts, how they work, and when to use them.

### Reference (information-oriented)

- **[Managing API rate limits](./reference/payment-api-rate-limits.md)**: Covers rate limit tiers and headers, and handling rate limits.
- **[Preventing duplicate charges with idempotency keys](./reference/payment-idempotency-keys.md)**: Covers how idempotency keys work and why they are essential for preventing accidental duplicate charges.
- **[Payment decline codes](decline-codes.md)**: Complete reference of all authorization decline codes with hard/soft classification.
- **[API reference guide](./reference/api-reference-guide.md)**: Covers three API endpoints: retrieve available payment options, update payment options, and create an order refund.
- **[Subscription webhook examples](./reference/webhook-subscription-examples.md)**: Covers two subscription webhook examples: 201 Created webhook and full subscription payload.
- **[Webhook definitions](./reference/webhook-definitions.md)**: Covers the terms and concepts you need to set up and manage webhooks.

---

## 🌐 Digital River documentation

- **[Commerce API documentation](https://docs.digitalriver.com/commerce-api)**: Authored and maintained the developer-facing Commerce API documentation, including 73+ REST API endpoints. Digital River wound down operations in 2025; the linked documentation reflects work completed during active operations.

---

## 💼 Professional experience highlights

### Digital River - Senior Technical Writer → Manager, Technical Writer (2016-2025)

I led API documentation for Digital River’s Commerce-as-a-Service platform, documenting 73+ REST API endpoints.

#### Key contributions

- Developed comprehensive API reference documentation for Commerce APIs.
- Created integration guides for electronic funds transfer systems.
- Led a two-phase documentation platform migration: first from Drupal to ReadMe to address a disorganized legacy content structure, then from ReadMe to GitBook after identifying that ReadMe couldn't support our JSON file requirements. Evaluated alternative docs-as-code platforms against our content requirements and made the final GitBook recommendation.
- Established documentation standards and created style guides.
- Implemented docs-as-code workflows using Git for version control.

**Repository:** [DigitalRiver/GitBook](https://github.com/DigitalRiver/GitBook) *(company repository, proprietary content)*

---

## 🛠 Technical skills

**Documentation platforms**: GitBook, ReadMe, MadCap Flare, Swagger/OpenAPI Editors
**GitHub actions**: Automated link checking and Markdown linting
**API specifications**: OpenAPI 3.0, REST API documentation, Postman
**Markup languages**: Markdown, YAML, JSON, XML, DITA
**Version control**: Git, GitHub, GitHub Actions, SVN
**Programming languages**: JavaScript, Python (for code examples and documentation)
**Documentation frameworks**: Diátaxis, docs-as-code
**AI tools**: Claude, Gemini, and GitHub Copilot. Used AI in my workflow for editorial control, prompt iteration, and review.

---

## 📫 Contact

**LinkedIn:** [linkedin.com/in/gale-james](https://linkedin.com/in/gale-james)  
**Email:** [Available on LinkedIn]

---

*This portfolio was last updated in April 2026.*
