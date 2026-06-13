## Text-to-Quote Technical Implementation Plan

### Purpose

This note describes the technical implementation plan for **Text-to-Quote**, a no-app, SMS-first quote-generation service for low-tech solo service providers.

The goal is to launch a low-cost MVP that can test real demand before building a full SaaS product. The product should allow a provider to text job details, complete missing information through SMS or a lightweight web flow, preview a quote, pay to unlock it, and then share a professional HTML/PDF quote with a customer.

### Product Behavior Summary

The basic system flow:

1. Provider texts a dedicated SMS number.
2. System identifies provider by phone number.
3. System collects or confirms provider profile.
4. System collects quote details.
5. System asks clarification questions if required fields are missing.
6. System generates a quote draft.
7. Provider previews the quote.
8. Provider pays to unlock the quote.
9. System generates/stores a shareable quote URL and PDF.
10. Provider can SMS or email the quote to the customer.
11. Customer can view, download, and optionally approve the quote.

### MVP Principles

- Do not build a full CRM.
- Do not build a provider dashboard unless absolutely necessary.
- Do not support every vertical at launch.
- Do not automate payments from the provider's customer at launch.
- Do not integrate with QuickBooks, Square, Jobber, or other systems at launch.
- Keep the interaction SMS-first.
- Use human review where needed to avoid overbuilding.
- Charge before unlocking the shareable quote artifact.

### Recommended MVP Architecture

#### Core Components

| Component | Responsibility | Suggested Tools |
|---|---|---|
| SMS Gateway | Send/receive provider SMS messages | Twilio or Telnyx |
| Web App | Quote preview, customer quote page, payment success page | Next.js |
| API Backend | Conversation state, quote generation, payment webhooks | Next.js API routes, Node/Express, or FastAPI |
| Database | Providers, quotes, messages, payments, quote status | Supabase Postgres, Neon, or Turso |
| Storage | Store PDFs, quote snapshots, uploaded images/logos | S3, Cloudflare R2, or Supabase Storage |
| Payments | Charge provider before unlock | Stripe Checkout or Payment Links |
| PDF Generation | Render quote artifact to PDF | Playwright or Puppeteer HTML-to-PDF |
| AI Extraction | Turn raw SMS into structured quote data | OpenAI or Anthropic |
| Admin Tool | Review/edit quotes and manage edge cases | Simple internal Next.js page, Airtable, or Retool |

### Data Model

#### Provider

Fields:

- `id`
- `phone_number`
- `business_name`
- `provider_name`
- `service_type`
- `email`
- `preferred_language`
- `logo_url`
- `default_terms`
- `default_deposit_terms`
- `created_at`
- `updated_at`

The provider is primarily authenticated by phone number. No password is required for MVP.

#### Quote

Fields:

- `id`
- `provider_id`
- `status`
  - `drafting`
  - `needs_info`
  - `ready_for_preview`
  - `awaiting_payment`
  - `paid_unlocked`
  - `sent`
  - `approved`
  - `expired`
- `customer_name`
- `customer_phone`
- `customer_email`
- `job_location`
- `quote_title`
- `scope_of_work`
- `line_items_json`
- `total_amount`
- `deposit_amount`
- `deposit_terms`
- `estimated_start_date`
- `estimated_duration`
- `expiration_date`
- `terms_and_notes`
- `language`
- `html_url`
- `pdf_url`
- `public_share_token`
- `created_at`
- `updated_at`

#### Message

Fields:

- `id`
- `provider_id`
- `quote_id`
- `direction`
  - `inbound`
  - `outbound`
- `channel`
  - `sms`
  - `web`
  - `email`
- `body`
- `media_urls_json`
- `created_at`

#### Payment

Fields:

- `id`
- `provider_id`
- `quote_id`
- `stripe_checkout_session_id`
- `amount`
- `currency`
- `status`
  - `pending`
  - `paid`
  - `failed`
  - `refunded`
- `created_at`
- `paid_at`

#### QuoteView / CustomerAction

Fields:

- `id`
- `quote_id`
- `action_type`
  - `viewed`
  - `downloaded_pdf`
  - `approved`
  - `declined`
- `customer_ip_hash`
- `user_agent`
- `created_at`

### Provider Authentication

#### MVP Auth Model

Use SMS-based identity.

- Provider's phone number is the primary identity.
- Inbound SMS from a known number maps to a provider.
- If a provider opens a web preview link, use a signed token in the URL.
- For sensitive actions, send a one-time SMS code.

#### Provider Preview Link

Provider preview URLs should be private/unlisted:

`/provider/quotes/{quoteId}/preview?token={signedToken}`

This lets the provider preview before payment but not share publicly.

#### Public Quote Link

Customer-facing quote URLs should use a public share token:

`/q/{publicShareToken}`

The public quote URL should only become active after payment.

Before payment, public quote URL should show:

> This quote is not available yet.

### SMS Conversation Flow

#### First-Time Provider Flow

1. Provider texts: `quote` or any job details.
2. System checks if phone number exists.
3. If new, system asks:
   - "What's your business name?"
   - "What type of work do you do?"
   - "What name should appear on quotes?"
4. System stores provider profile.
5. System asks provider to send job details.

Example:

> Great. Text me the job details like this: customer name, work to be done, price, timing, and any deposit/notes. You can write naturally.

#### Quote Intake Flow

Provider sends:

> Maria Garcia, front yard cleanup, trim hedges, haul away, Tuesday, $450, 50% deposit, valid 7 days.

System extracts structured fields:

- customer_name: Maria Garcia
- scope: front yard cleanup, trim hedges, haul away
- timing: Tuesday
- total_amount: 450
- deposit_terms: 50% deposit
- expiration_date: 7 days

If required fields are missing, system asks targeted questions.

#### Required MVP Fields

Required:

- customer name or placeholder.
- scope of work.
- total price.
- provider business name.

Optional:

- customer phone/email.
- job location.
- start date.
- deposit.
- expiration.
- photos.
- terms.

#### Clarification Logic

If total price is missing:

> What price should I put on this quote?

If scope is vague:

> What work should be included in the quote?

If customer name is missing:

> Who is this quote for? You can send just their first name.

If deposit terms are unclear:

> Should I include a deposit requirement, or no deposit?

### Web Flow Option

SMS should be the main flow, but a web flow can reduce SMS complexity.

#### When To Use Web Flow

Send provider a lightweight URL when:

- SMS parsing fails.
- Quote has many fields.
- Provider wants to edit the draft.
- Provider wants to upload logo/photos.
- Provider wants to add line items.

#### Web Flow Pages

1. `/start/{token}`
   - lightweight quote form.
2. `/provider/quotes/{quoteId}/edit?token=...`
   - edit draft fields.
3. `/provider/quotes/{quoteId}/preview?token=...`
   - preview before payment.
4. `/provider/quotes/{quoteId}/checkout?token=...`
   - payment page.
5. `/provider/quotes/{quoteId}/share?token=...`
   - unlocked share page after payment.

### Quote Generation

#### Step 1: Normalize Inputs

Use an LLM or deterministic parser to transform raw SMS into structured quote JSON.

Example output:

```json
{
  "customer_name": "Maria Garcia",
  "quote_title": "Front Yard Cleanup",
  "scope_of_work": "Front yard cleanup, hedge trimming, and haul-away of green waste.",
  "line_items": [
    { "description": "Front yard cleanup, hedge trimming, and haul-away", "amount": 450 }
  ],
  "total_amount": 450,
  "deposit_terms": "50% deposit required to schedule work.",
  "expiration_date": "7 days from issue date"
}
```

#### Step 2: Generate Professional Copy

Transform rough notes into customer-facing language.

Raw:

> trim hedges, cleanup, haul away, $450

Customer-facing:

> This quote includes front yard cleanup, hedge trimming, and removal of green waste from the property.

#### Step 3: Render HTML Quote

Quote page should be simple, mobile-friendly, and professional.

Sections:

- Provider brand/header.
- Quote title.
- Customer info.
- Scope of work.
- Price.
- Deposit/terms.
- Valid-until date.
- Approval button.
- Download PDF.
- Contact provider button.

#### Step 4: Generate PDF

Use the same HTML template to generate PDF so the web and PDF versions match.

PDF can be generated:

- immediately after payment, or
- at quote draft creation but only exposed after payment.

### Payment Gating

#### Requirement

Provider should not receive the shareable customer quote URL until payment is complete.

#### Flow

1. System generates preview.
2. Provider receives preview link.
3. Preview page says:
   - "Looks good? Unlock this quote for $5."
   - or "Use one quote credit."
4. Provider pays via Stripe Checkout.
5. Stripe webhook confirms payment.
6. Backend updates quote status to `paid_unlocked`.
7. Backend creates/activates public quote URL.
8. System sends provider SMS:

> Your quote is ready: https://example.com/q/abc123. Send this to your customer. PDF: [link]

#### Payment Implementation Options

##### Option A: Stripe Checkout Session

Best for MVP.

- Create checkout session linked to quote ID.
- Use success/cancel URLs.
- Use webhook as source of truth.

##### Option B: Stripe Payment Links

Simpler but less flexible.

- Faster setup.
- Harder to associate deeply with quote objects unless metadata is handled carefully.

##### Option C: Prepaid Credits

Useful after MVP.

- Provider buys 5 quote credits.
- Each unlocked quote consumes one credit.
- Reduces payment friction.

Recommended MVP:

1. Start with Stripe Checkout per quote.
2. Add prepaid credits after there is repeat usage.

### Quote Delivery

#### Provider Delivery

After payment, provider receives:

- SMS with shareable quote link.
- SMS with PDF link or attachment if supported.
- Optional email copy.

Example:

> Quote unlocked. Send this to Maria: https://texttoquote.com/q/abc123. PDF: https://texttoquote.com/pdf/abc123.pdf

#### Customer Delivery

MVP can let the provider manually forward the quote link.

Later, provider can request:

> send to customer

Then system sends customer SMS/email directly.

Direct customer sending should be optional because it introduces consent, deliverability, and support questions.

### Customer Quote Page

Customer-facing page:

- Mobile-first.
- No login.
- Clean design.
- Shows provider contact info.
- Shows quote details.
- Download PDF.
- Approve button.
- Optional decline/question button.

#### Customer Approval Flow

When customer clicks approve:

1. Quote status becomes `approved`.
2. Provider receives SMS:

> Maria approved your quote for $450.

3. Customer sees confirmation:

> Quote approved. [Business name] will contact you about next steps.

Do not collect customer payment in MVP unless needed.

### Admin Tooling

MVP should include a simple internal admin interface.

Admin capabilities:

- View inbound messages.
- View quote draft.
- Edit quote fields.
- Regenerate quote copy.
- Mark quote ready.
- Resend preview link.
- Manually unlock/refund quote if needed.
- View payment status.
- View provider usage.

This allows human-in-the-loop quality control without building perfect automation.

### Human-in-the-Loop MVP

For early validation, human review is useful.

Recommended flow:

1. SMS comes in.
2. AI extracts data.
3. Admin reviews quote draft.
4. Admin clicks "send preview."
5. Provider pays.
6. Quote unlocks automatically.

This reduces risk of bad quotes while learning real patterns.

Later, automate preview generation for simple quotes.

### API / Endpoint Sketch

#### SMS Webhook

`POST /api/sms/inbound`

Responsibilities:

- Verify webhook signature.
- Normalize sender phone.
- Create provider if new.
- Store inbound message.
- Route to conversation handler.
- Send SMS response.

#### Quote Parser

`POST /api/quotes/parse`

Responsibilities:

- Parse raw message.
- Extract structured fields.
- Identify missing fields.
- Create or update quote draft.

#### Quote Preview

`GET /provider/quotes/:id/preview`

Responsibilities:

- Verify signed token.
- Render draft quote.
- Show unlock/payment CTA.

#### Checkout

`POST /api/quotes/:id/checkout`

Responsibilities:

- Verify provider token.
- Create Stripe Checkout session.
- Add quote metadata.
- Return checkout URL.

#### Stripe Webhook

`POST /api/stripe/webhook`

Responsibilities:

- Verify Stripe signature.
- Confirm payment.
- Update payment status.
- Unlock quote.
- Generate public quote token if needed.
- Send SMS with share link.

#### Public Quote

`GET /q/:token`

Responsibilities:

- Verify quote is unlocked.
- Render customer-facing HTML.
- Track quote view.

#### PDF

`GET /q/:token.pdf`

Responsibilities:

- Generate or serve stored PDF.
- Track PDF download.

#### Customer Approval

`POST /q/:token/approve`

Responsibilities:

- Mark approved.
- Notify provider by SMS.
- Show confirmation page.

### Conversation State Machine

Suggested states:

- `new_provider`
- `collecting_provider_profile`
- `ready_for_quote`
- `collecting_quote_details`
- `needs_missing_info`
- `draft_ready`
- `awaiting_payment`
- `paid_unlocked`
- `quote_sent`

Each inbound message should be interpreted based on state.

Example:

- If provider is `ready_for_quote`, treat message as new quote request.
- If quote is `needs_missing_info`, treat message as answer to last missing question.
- If quote is `awaiting_payment`, allow commands like `pay`, `edit`, `cancel`.

### SMS Commands

Support simple commands:

- `new quote`
- `edit`
- `preview`
- `pay`
- `send`
- `help`
- `profile`
- `cancel`

Avoid making commands required. Natural language should work, but commands help edge cases.

### Quote Templates

Start with 2-3 templates.

#### Simple Service Quote

Best for gardeners, cleaning, hauling, mobile detailers.

Fields:

- scope.
- price.
- timing.
- terms.

#### Line Item Quote

For handymen or small contractors.

Fields:

- line items.
- subtotal.
- total.
- notes.

#### Bilingual Quote

English + Spanish version.

Useful if early customers need bilingual output.

### Storage Strategy

#### HTML Quote

Render dynamically from database using public token.

#### PDF Quote

Options:

1. Generate on demand and cache in object storage.
2. Generate once after payment and store permanently.

MVP recommendation:

- Generate once after payment.
- Store PDF in S3/R2/Supabase Storage.
- Store PDF URL on quote record.

#### Media Attachments

If SMS includes images:

- Store media URL from Twilio temporarily.
- Download and store in object storage.
- Associate with quote.
- For MVP, photos may be ignored or included as an appendix.

### Payment / Quote Unlock Logic

Important rule:

> Preview is free. Shareable quote is paid.

#### Before Payment

Provider can see preview through private provider token.

Public/customer quote URL is inactive.

PDF is not downloadable, or is watermarked.

#### After Payment

System:

- activates public quote token.
- generates/stores PDF.
- sends provider share link.
- allows customer view/approval.

### Minimal Security Requirements

- Verify Twilio/Telnyx webhook signatures.
- Verify Stripe webhook signatures.
- Use signed tokens for provider preview/edit links.
- Use unguessable public share tokens.
- Avoid exposing draft quotes publicly before payment.
- Rate limit inbound SMS if abused.
- Store only necessary customer information.
- Do not store sensitive payment details; Stripe handles payment.

### Compliance / Messaging Considerations

For MVP, provider initiates SMS by texting the service.

Be careful with direct customer SMS delivery:

- Provider should ideally forward the quote link themselves at first.
- If the system texts customers directly, consider opt-in/consent requirements.
- Avoid marketing texts to customers.
- Keep messages transactional.

MVP recommendation:

> Send quote links to provider only. Provider forwards to customer.

This lowers compliance and support complexity.

### MVP Build Milestones

#### Milestone 1: Manual Concierge MVP

Goal: validate workflow before automation.

Build:

- Twilio number.
- Inbound SMS logging.
- Manual quote creation form.
- HTML quote template.
- PDF export.
- Stripe payment link.
- Manual SMS back to provider.

Effort:

- 1-2 weekends.

This can test whether users send real quotes and pay.

#### Milestone 2: Semi-Automated MVP

Build:

- Provider profile by phone number.
- Quote parser using LLM.
- Conversation state machine.
- Preview link generation.
- Stripe Checkout integration.
- Webhook-based quote unlock.
- Automatic SMS with paid quote link.

Effort:

- 2-4 additional weekends.

#### Milestone 3: Repeat-Use MVP

Build:

- Prepaid quote credits.
- Simple provider profile edit link.
- Saved default terms.
- Quote history by SMS link.
- Customer approval notifications.
- Basic analytics.

Effort:

- 2-4 additional weekends.

#### Milestone 4: Productization

Build only if repeat use is proven:

- Lightweight provider dashboard.
- Business logo upload.
- More templates.
- Bilingual quote generation.
- WhatsApp support.
- Customer email delivery.
- Payment/deposit collection from customer.

### Launch MVP: Lowest-Cost Version

#### Technical Flow

1. Provider texts quote details to Twilio number.
2. Message appears in admin queue.
3. Admin/AI generates quote draft.
4. System sends preview link by SMS.
5. Provider clicks preview.
6. Provider pays via Stripe Checkout.
7. Stripe webhook unlocks quote.
8. System sends shareable quote link/PDF by SMS.
9. Provider forwards link to customer.

#### Why This Is Good For Validation

- Low engineering cost.
- Keeps quality high.
- Avoids overbuilding conversation logic.
- Tests willingness to pay.
- Lets you observe real quote patterns.
- Avoids complex integrations.

### Suggested Initial Implementation Order

1. Buy/configure SMS number.
2. Set up inbound SMS webhook.
3. Create database tables.
4. Build provider lookup by phone number.
5. Build admin quote creation/edit form.
6. Build HTML quote template.
7. Add PDF generation.
8. Add private provider preview link.
9. Add Stripe Checkout payment flow.
10. Add Stripe webhook to unlock quote.
11. Add public quote URL.
12. Add SMS delivery of unlocked quote link.
13. Add customer approve button.
14. Add quote view/approval tracking.
15. Add simple provider profile defaults.

### Cost Estimate For MVP

Expected low monthly cost while testing:

- Domain: ~$10-20/year.
- Hosting: $0-25/month.
- Database/storage: $0-25/month.
- Twilio/Telnyx: usage-based; likely low during test.
- Stripe: transaction fees only.
- LLM: likely low for quote parsing/generation.
- PDF generation: free if using Playwright/Puppeteer on own infra.

Expected MVP cost can likely remain under $50/month before meaningful usage, excluding development time.

### Operational Workflow For First 10 Users

Keep it human-assisted.

1. User texts quote details.
2. System logs request.
3. You review and generate quote.
4. User gets preview.
5. User pays.
6. System unlocks quote.
7. You observe whether they send more quotes.

Do not optimize before repeat usage is observed.

### Product Analytics To Track

Provider-level:

- first quote requested.
- first quote paid.
- second quote requested.
- prepaid bundle purchased.
- average quotes per provider.

Quote-level:

- quote draft created.
- preview opened.
- payment completed.
- share link generated.
- customer viewed quote.
- customer approved quote.

Operational:

- time to generate quote.
- manual edits required.
- failed SMS interactions.
- payment conversion rate.
- support messages per quote.

### Key Technical Risks

#### SMS Parsing Is Ambiguous

Providers may send incomplete or unclear notes.

Mitigation:

- Use targeted follow-up questions.
- Keep required fields minimal.
- Use human review early.

#### Payment Gating Creates Friction

Provider may not want to pay after preview.

Mitigation:

- Offer first quote free.
- Use prepaid bundles after validation.
- Make payment one-click via Stripe.

#### Existing Tools Are Better For Some Users

Users who already use Square/Joist/Jobber do not need this.

Mitigation:

- Target users who quote by text today.
- Keep product no-app/no-subscription.

#### Direct Customer SMS Raises Compliance Issues

Mitigation:

- Send quote to provider first.
- Provider forwards link to customer.

#### Manual Review Does Not Scale

Mitigation:

- Accept manual review during validation.
- Automate only repeated patterns.

### Future Enhancements

Only build after repeat usage:

- WhatsApp support.
- Spanish/English quote toggle.
- Voice note input.
- Photo attachment support.
- Provider logo/business card upload.
- Prepaid credit bundles.
- Customer approval tracking.
- Deposit/payment collection.
- Quote reminders.
- Simple quote history.
- Reusable service menu.
- Partner/referral codes for local communities.

### Recommended Next Step

Build **Milestone 1: Manual Concierge MVP** first.

The first version should prove:

1. providers will text real quote details,
2. generated quotes are useful enough to send to customers,
3. providers will pay per quote or prepay for a bundle,
4. providers will come back for repeat use.

Do not build the full automated SMS conversation until those behaviors are observed.
