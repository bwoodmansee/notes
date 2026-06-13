## Text-to-Quote Phased Implementation Prompt

### Purpose

Use this note as the working prompt for Claude Code to initialize the QuoteListo MVP in phases.

The product should launch first as a web-based quote generator with payment-gated public quote links. SMS integration should be treated as Phase 2, after the quote creation, payment, artifact storage, and public sharing flow works end-to-end.

### Product Summary

QuoteListo helps low-tech solo service providers create professional quotes from simple job information.

The Phase 1 product is a web app:

- provider visits the website
- provider enters quote information through a simple form
- provider optionally uploads job images
- system generates a professional HTML quote page
- provider previews the quote
- provider pays to unlock the shareable quote URL
- provider copies/sends the public quote link to the customer

The Phase 2 product adds SMS:

- provider can start or continue quote creation by text message
- provider can receive quote links by SMS
- provider can send customer quote links by SMS
- later, provider can create quotes conversationally by SMS

### Core Principle

Do not build SMS-first until the quote artifact and payment-gated sharing system works.

The first validated asset is not the SMS bot. It is the professional quote artifact plus the willingness to pay for unlocking and sharing it.

### Phase 1 — Web MVP

#### Goal

Create the smallest useful product that allows a provider to generate, preview, pay for, and share a professional quote.

#### User Flow

1. Provider lands on the homepage.
2. Provider clicks "Create a quote."
3. Provider enters phone number or email to start.
4. Provider enters optional business information.
5. Provider enters customer/job/quote details.
6. Provider optionally uploads photos.
7. System renders a professional quote preview.
8. Provider clicks "Unlock share link."
9. Provider pays via Stripe Checkout.
10. Stripe webhook marks quote as paid.
11. System creates or enables public quote URL.
12. Provider gets copyable share link.
13. Customer opens quote page.

#### Phase 1 Requirements

##### Frontend

Build a simple static/SPA frontend with:

- landing page
- quote intake form
- optional business profile form
- image upload UI
- quote preview page
- pay-to-unlock screen
- public quote page

The frontend should be simple and mobile-first.

##### Backend

Implement API routes for:

- create provider/session
- create or update provider profile
- create quote draft
- upload image via presigned S3 URL
- fetch quote draft for preview
- create Stripe Checkout session
- receive Stripe webhook
- unlock paid quote
- fetch public paid quote by slug

##### Data Storage

Use DynamoDB tables:

- Providers
- Quotes
- Payments

Quote state should include:

- DRAFT_UNPAID
- CHECKOUT_STARTED
- PAID_UNLOCKED
- ARCHIVED or CANCELLED later

##### Artifact Storage

Use S3 for:

- uploaded images
- generated quote HTML snapshots if needed
- future generated PDFs

In Phase 1, prefer generated HTML quote pages over PDFs.

The public quote page should be printable with CSS.

##### Payment Gate

Do not expose a shareable public quote URL until payment is complete.

Before payment:

- provider can preview the quote privately
- customer cannot access it publicly
- any public slug should not resolve

After payment:

- quote is marked PAID_UNLOCKED
- public slug is created or activated
- public quote page becomes accessible

##### Hosting

Use AWS-first low-cost hosting:

- S3 for static frontend assets
- CloudFront for the website/domain
- API Gateway HTTP API for backend routes
- Lambda for backend logic
- DynamoDB for state
- S3 for images/artifacts
- Stripe for payment

Do not use Vercel.

##### Domain

Assume domain is configured later through Cloudflare DNS or Route 53.

Target structure:

- / — landing page
- /app — quote creation web app
- /quote/{slug} — public paid quote page
- /api/* — backend API
- /webhooks/stripe — Stripe webhook

If custom domain setup is too much in the first commit, document the DNS/CloudFront/ACM steps clearly and deploy using AWS-generated URLs first.

### Phase 1 Non-Goals

Do not build these in Phase 1:

- SMS quote intake
- Twilio inbound message processing
- AI conversational quote builder
- PDF generation
- recurring subscriptions
- CRM features
- customer database beyond quote metadata
- complex admin dashboard
- WYSIWYG quote editor
- multi-user business accounts
- Cognito

### Phase 2 — SMS Integration

#### Goal

Add SMS as a convenience layer after the core web quote/payment/share flow works.

SMS should not become the primary source of product complexity until the user has already proven that people want the quote artifact.

#### Phase 2 User Flows

##### Flow A — SMS Delivery

1. Provider creates quote in web app.
2. Provider pays to unlock quote.
3. Provider clicks "Text quote to me" or "Text quote to customer."
4. System sends public quote URL by SMS.

##### Flow B — SMS Auth

1. Provider enters phone number in web app.
2. System sends SMS verification code.
3. Provider enters code.
4. Provider session is created.

##### Flow C — SMS Quote Start

1. Provider texts QuoteListo number: "new quote."
2. System replies with a web intake link.
3. Provider completes structured quote flow on the web.

##### Flow D — Conversational SMS Quote Intake

Later only:

1. Provider texts job/customer/price details.
2. System asks structured follow-up questions.
3. System creates quote draft.
4. System sends preview/payment link.
5. Provider pays and receives shareable quote link.

#### Phase 2 Requirements

Add Twilio integration for:

- outbound SMS send
- phone verification codes
- inbound SMS webhook
- simple command parsing
- message event logging

Implement routes:

- POST /webhooks/twilio
- POST /api/sms/send-quote-link
- POST /api/auth/request-sms-code
- POST /api/auth/verify-sms-code

SMS should be used initially for:

- verification
- quote link delivery
- starting a quote with a web link

Conversational SMS quote intake should be Phase 2B or Phase 3, not initial Phase 2.

### Phase 3 — Automation and Polish

Possible later work:

- AI-assisted scope cleanup
- English/Spanish quote generation
- reusable provider defaults
- quote templates by service type
- customer approval button
- approval notifications
- PDF generation
- payment/deposit collection
- prepaid quote bundles
- admin review queue
- analytics dashboard
- abandoned quote follow-up
- referral links

### Technical Architecture

#### AWS Services

Use:

- S3 for frontend hosting and artifacts
- CloudFront for CDN and custom domain
- API Gateway HTTP API for API routes
- Lambda for backend handlers
- DynamoDB for data
- CloudWatch Logs for logging
- ACM for TLS certificate
- IAM roles with least practical permissions

Use Stripe for payments.

Use Twilio only in Phase 2.

#### Suggested Repository Structure

```text
/infra
  template.yaml
  deploy.sh
  destroy.sh
  outputs.sh

/backend
  src/handlers
  src/lib
  src/types
  package.json
  tsconfig.json

/frontend
  src
  public
  package.json
  build-and-upload.sh

/docs
  architecture.md
  deployment.md
  env-vars.md
  phases.md

/scripts
  smoke-test.sh
```

#### Minimum API for Phase 1

```http
POST /api/session/start
GET  /api/provider/profile
PUT  /api/provider/profile
POST /api/uploads/presign
POST /api/quotes
GET  /api/quotes/{quoteId}
PUT  /api/quotes/{quoteId}
POST /api/quotes/{quoteId}/checkout
POST /webhooks/stripe
GET  /api/public/quotes/{slug}
```

#### Minimum API for Phase 2

```http
POST /api/auth/request-sms-code
POST /api/auth/verify-sms-code
POST /api/sms/send-quote-link
POST /webhooks/twilio
```

### Claude Code Prompt

Use the following prompt to initialize the project.

```text
You are helping me initialize a low-cost AWS serverless MVP for a product called QuoteListo.

Product:
QuoteListo helps low-tech solo service providers create professional quotes from simple job information. The product should launch in phases.

Phase 1 is web-only:
- landing page
- mobile-first quote intake form
- optional business profile
- image uploads
- professional quote preview
- Stripe payment to unlock the public quote URL
- public quote page after payment

Phase 2 adds SMS:
- SMS verification
- sending quote links by SMS
- inbound SMS commands that send users into the web flow
- later conversational SMS quote intake

Important architectural rule:
Do not build SMS integration in Phase 1. Stub it only if useful. The first milestone is the web quote/payment/share loop.

Goal:
Create the initial repository structure, infrastructure templates, backend Lambda handlers, frontend landing page/web app, and deployment scripts. Optimize for low cost, simple AWS primitives, and AWS CLI-driven deployment. Do not use Vercel.

Use:
- AWS S3 + CloudFront for static hosting
- API Gateway HTTP API + Lambda for backend
- DynamoDB for state
- S3 for quote artifacts and uploaded images
- Stripe Checkout for payment gating
- Twilio only in Phase 2

Use TypeScript wherever practical.

Phase 1 user flow:
1. Provider opens the landing page.
2. Provider clicks Create a quote.
3. Provider starts a lightweight session with phone or email.
4. Provider optionally enters business info.
5. Provider enters customer/job/scope/price/terms.
6. Provider optionally uploads images through presigned S3 URLs.
7. System generates a professional quote preview.
8. Provider pays through Stripe Checkout to unlock sharing.
9. Stripe webhook marks quote paid.
10. System creates or activates a public slug.
11. Provider receives/copies public quote URL.
12. Customer opens /quote/{slug}.

Phase 1 backend handlers:
1. POST /api/session/start
2. GET /api/provider/profile
3. PUT /api/provider/profile
4. POST /api/uploads/presign
5. POST /api/quotes
6. GET /api/quotes/{quoteId}
7. PUT /api/quotes/{quoteId}
8. POST /api/quotes/{quoteId}/checkout
9. POST /webhooks/stripe
10. GET /api/public/quotes/{slug}

Phase 1 frontend:
- Landing page at /
- App at /app
- Quote intake form
- Quote preview
- Pay-to-unlock screen
- Public quote page at /quote/{slug}
- Mobile-first styling
- Print-friendly quote page CSS

Phase 1 data model:
Providers:
- providerId
- phoneNumber optional
- email optional
- businessName optional
- ownerName optional
- defaultTerms optional
- languagePreference optional
- createdAt
- updatedAt

Quotes:
- quoteId
- providerId
- status: DRAFT_UNPAID | CHECKOUT_STARTED | PAID_UNLOCKED | CANCELLED
- customerName
- customerPhone optional
- customerEmail optional
- jobAddress optional
- serviceType optional
- scopeOfWork
- exclusions optional
- price
- depositTerms optional
- expirationDate optional
- language
- imageAssetKeys
- publicSlug optional
- stripeCheckoutSessionId optional
- createdAt
- updatedAt
- paidAt optional

Payments:
- paymentId
- providerId
- quoteId
- stripeCheckoutSessionId
- status
- amount
- createdAt
- completedAt optional

Artifact/image storage:
- Use private S3 bucket for uploaded images.
- Use presigned URLs for browser uploads.
- Do not make images public by default.
- Public quote page can serve selected images through controlled URLs or copied public artifacts only after payment.
- Prefer HTML quote page over PDF in Phase 1.

Payment gating:
- Provider can preview unpaid quote privately.
- Public /quote/{slug} must only work for PAID_UNLOCKED quotes.
- If quote is unpaid, public endpoint must not return quote details.
- Stripe webhook must mark quote as paid and create/activate publicSlug.

Infra:
Use CloudFormation or AWS SAM.
Create:
- S3 static site bucket
- S3 assets bucket
- DynamoDB tables with PAY_PER_REQUEST billing
- Lambda execution roles
- Lambda functions
- API Gateway HTTP API
- API routes
- CloudWatch log groups
- deployment outputs

CloudFront/custom domain:
If possible, include CloudFront distribution for the static site and documented support for custom domain.
If too much for first pass, document exact follow-up steps for:
- ACM certificate in us-east-1
- CloudFront distribution
- DNS CNAME/Alias setup in Cloudflare or Route 53

Local/dev:
- Provide .env.example files.
- Use Stripe test mode.
- Do not require Twilio in Phase 1.
- Include a simple local dev path.

Security:
- Never expose Stripe secret keys in frontend.
- Validate quote ownership before preview/update/checkout.
- Do not expose unpaid quotes publicly.
- Keep image uploads private until included in paid public quote.
- Use CORS for localhost and configured domain only.

Avoid in Phase 1:
- Twilio implementation
- SMS conversational intake
- Cognito
- full CRM
- admin dashboard
- PDF generation
- subscriptions
- WYSIWYG editor
- complex AI parsing

Deliverables:
1. Generate repository structure.
2. Implement minimal Phase 1 backend handlers.
3. Implement minimal Phase 1 frontend.
4. Implement infra template and deployment scripts.
5. Add README with exact AWS CLI deployment instructions.
6. Add docs/architecture.md.
7. Add docs/deployment.md.
8. Add docs/phases.md explaining Phase 1, Phase 2, and Phase 3.
9. Add TODO markers for Phase 2 SMS integration.

Use placeholder domain:
DOMAIN_NAME=quotelisto.com

Assume AWS region:
us-west-2 for app resources.
For CloudFront certificate, document that ACM certificate must be in us-east-1.

Start by implementing the smallest end-to-end Phase 1 flow:
create quote -> preview quote -> start Stripe checkout -> Stripe webhook unlocks quote -> public quote page works.
```

### Phase 1 Success Criteria

Phase 1 is good enough when:

- homepage loads on the domain
- provider can create a quote from mobile browser
- provider can upload at least one image
- provider can preview a professional quote
- unpaid quote is not publicly shareable
- Stripe test checkout works
- Stripe webhook unlocks quote
- public quote URL works after payment
- quote page looks good enough to send to a real customer

### Phase 2 Success Criteria

Phase 2 is good enough when:

- provider can verify phone via SMS
- provider can request quote link by SMS
- provider can send quote link to customer by SMS
- provider can text `new quote` and receive a web intake link

Conversational SMS intake is explicitly not required for initial Phase 2 success.
