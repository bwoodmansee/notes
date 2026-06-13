## QuoteListo Claude Context

### Product

QuoteListo is a low-cost product for solo service providers who need to create professional-looking quotes without learning business software.

The target customer is a low-tech solo or very small service provider in the United States, including gardeners, handymen, cleaners, junk haulers, mobile detailers, painters, and similar local operators.

The product should support English and Spanish speakers.

### Core positioning

QuoteListo helps a service provider turn simple job details into a polished quote page they can send to a customer.

Working tagline:

> Professional quotes by text.

Phase 1 should be web-first. SMS is Phase 2.

### What this product is

- A simple way to create professional quotes from a phone or browser.
- A hosted quote artifact generator.
- A lightweight quote intake and quote-sharing flow.
- A payment-gated quote unlock system.
- A bilingual-friendly product for English and Spanish speakers.
- A low-cost MVP designed to test whether providers will pay to generate and share quotes.

### What this product is not

- Not a CRM.
- Not Jobber, Joist, Square, LawnPro, or Housecall Pro.
- Not a field-service management system.
- Not scheduling software.
- Not invoicing software in Phase 1.
- Not payment collection for the provider’s customer in Phase 1.
- Not a full SMS conversation engine in Phase 1.
- Not an AI-first product in the marketing. AI may help internally, but the customer cares about professional quotes and simplicity.

### Phases

#### Phase 1: Web-first MVP

Build the minimum loop needed to test demand:

1. Static bilingual landing page.
2. Simple web quote intake flow.
3. Optional provider/business profile.
4. Image upload support.
5. Quote preview.
6. Stripe payment gate.
7. Public quote URL after payment.
8. Print-friendly HTML quote page.

SMS integration should not be built in Phase 1.

#### Phase 2: SMS convenience layer

Add SMS after the web-based quote/payment/share flow works:

1. SMS phone verification.
2. Send quote links by SMS.
3. Text provider a link to continue quote intake on the web.
4. Optional customer SMS delivery.

Avoid full conversational quote intake at this stage unless Phase 1 has demand.

#### Phase 3: SMS-first automation

Only after validation:

1. Conversational SMS quote intake.
2. AI-assisted parsing and follow-up questions.
3. English/Spanish quote generation.
4. Reusable service templates.
5. PDF generation if needed.
6. Quote approval and deposit/payment support.

### Hosting and infrastructure principles

Prioritize the lowest-cost reasonable AWS setup.

Use boring AWS primitives:

- S3 for static hosting and generated artifacts.
- CloudFront for HTTPS/custom domain/CDN.
- ACM for TLS certificates.
- API Gateway + Lambda later for backend routes.
- DynamoDB later for quote/provider/payment state.
- S3 private assets bucket later for uploaded images using presigned URLs.
- Stripe later for payment gating.
- Twilio later for SMS.

Do not use Vercel, Netlify, Terraform, CDK, Cognito, or a relational database unless explicitly requested.

Use AWS CLI first. Prefer step-by-step commands with output checks.

### Domain and routing target

The custom domain should eventually support:

- `/` landing page
- `/en/` English landing page
- `/es/` Spanish landing page
- `/app/` web quote intake app
- `/quote/{slug}` public quote pages
- `/api/*` backend API routes later

For the first hosting milestone, only deploy the static landing pages.

### Static landing page requirements

The landing page must remain fully static.

Create:

- `public/index.html`
- `public/en/index.html`
- `public/es/index.html`
- `public/styles.css`
- `public/404.html`

English page:

- `<html lang="en">`
- Tagline: `Professional quotes by text.`
- Headline: `Create professional quotes from your phone.`
- Subheadline: `For solo service providers who want polished quotes without learning software.`

Spanish page:

- `<html lang="es">`
- Tagline: `Cotizaciones profesionales por texto.`
- Headline: `Crea cotizaciones profesionales desde tu teléfono.`

The root page can default to English and include a clear language toggle to Spanish.

Add hreflang alternates for English, Spanish, and x-default.

The landing page should be mobile-first, simple, credible, and approachable for non-technical service providers.

### Cost discipline

Be as cost-effective as possible.

Avoid paid platforms unless they clearly save enough time to justify the cost.

The initial static landing page should have near-zero AWS compute cost because it should not use compute.

Avoid standing up API Gateway, Lambda, DynamoDB, Stripe, Twilio, or image storage until the landing page is live and the next phase begins.

For Phase 1 hosting, prefer:

- Private S3 bucket.
- CloudFront distribution.
- CloudFront Origin Access Control.
- ACM certificate.
- DNS provider records pointing to CloudFront.

Do not make the S3 bucket public.

### Claude Code working style

When helping with implementation:

- Start in planning mode.
- Ask for missing domain/DNS/AWS details first.
- Drive AWS CLI commands one step at a time.
- Explain every command before asking to run it.
- Wait for command output before giving dependent next steps.
- Do not invent ARNs, IDs, bucket names, or hosted zone IDs.
- Use shell variables.
- Prefer reversible, simple, low-cost resources.
- Warn before creating CloudFront or DNS records.
- Do not give destructive cleanup commands unless requested.
