## Text-to-Quote Product Brief

### Product Summary

**Text-to-Quote** is a no-app, SMS-first quote-generation service for low-tech solo service providers who currently send informal quotes by text, phone, handwritten notes, or memory.

The product lets a provider text job details to a dedicated number and receive a polished quote artifact that can be sent to a customer by link, SMS, email, or PDF. The provider pays before the quote becomes shareable.

The core product bet is not that quote software does not exist. Quote software is common. The bet is that a segment of very small operators does not want to learn quote software, does not want a monthly subscription, and would prefer to generate professional quotes directly through SMS.

### Target Customer

#### Primary Customer

Solo or very small local service providers who are operationally informal and have limited appetite for software.

Likely early verticals:

- Gardeners and small landscaping crews.
- Handymen.
- Mobile detailers.
- Junk removal / hauling operators.
- House cleaners doing one-off or deep-clean jobs.
- Small painters or repair contractors with simple quote needs.

#### Customer Characteristics

Good-fit users likely have these traits:

- They quote jobs manually today.
- They often send prices by text message.
- They do not consistently produce professional PDFs.
- They do not want to learn Jobber, Joist, Square, LawnPro, or similar tools.
- They want to look more professional without changing how they work.
- They may be bilingual or serve bilingual customers.
- They may not want a recurring subscription.

Bad-fit users:

- Already using a quoting/invoicing app successfully.
- Larger teams with office/admin staff.
- Businesses needing CRM, scheduling, dispatch, payments, job costing, or accounting.
- Businesses with complex estimating logic.
- Businesses that need deeply customized proposals.

### Product Positioning

#### Simple Positioning

> Professional quotes by text message. No app. No subscription. No setup.

#### More Specific Positioning

> Text us the job details and we send back a professional quote link/PDF you can forward to your customer.

#### What The Product Is

- SMS-first quote creation.
- Professional quote artifact generation.
- Pay-per-quote or prepaid bundle.
- A simple way for informal operators to look more professional.
- A lightweight layer between raw job notes and a customer-facing quote.

#### What The Product Is Not

- Not a CRM.
- Not full invoicing software.
- Not accounting software.
- Not scheduling/dispatch software.
- Not a replacement for Square, Jobber, Joist, LawnPro, or Housecall Pro.
- Not automatic job pricing.
- Not a marketplace.
- Not a customer portal.
- Not a business operating system.

### Core Customer Job

The provider wants to turn an informal job description into a professional quote quickly.

Current behavior may look like:

> "I can do the yard cleanup for $450 next Tuesday."

Desired output:

- Customer name.
- Provider business name.
- Scope of work.
- Price.
- Deposit terms if any.
- Timing.
- Expiration date.
- Notes / exclusions.
- Approve button.
- Downloadable PDF.
- Shareable link.

### Why This Product Might Make Sense

#### Existing Market Reality

Professional quote/estimate generation is already a crowded market. Many tools exist for quotes, estimates, invoices, payments, and client approvals.

That means the broad idea should be rejected:

> "A simple quote generator for small businesses."

The narrower wedge is different:

> "A no-app, SMS-first quote generator for operators who do not want to use software."

The product should deliberately avoid becoming normal quoting software. Its advantage is low friction, not feature depth.

### Primary Use Case

#### Text-to-Quote Flow

1. Provider texts job details to the Text-to-Quote number.
2. System asks for any missing required information.
3. System generates a clean quote draft.
4. Provider previews the quote.
5. Provider pays to unlock the shareable quote.
6. System sends provider a customer-ready quote link and PDF.
7. Provider forwards the quote to the customer.
8. Customer can view, download, and optionally approve.

### Quote Artifact Requirements

The quote should feel polished but lightweight.

Minimum quote fields:

- Provider name / business name.
- Provider phone.
- Provider email, optional.
- Provider logo, optional later.
- Customer name.
- Customer phone/email, optional.
- Job location, optional.
- Quote title.
- Scope of work.
- Line items, optional.
- Total price.
- Deposit required, optional.
- Estimated service date or timing, optional.
- Quote expiration date.
- Terms / notes / exclusions.
- Approval button.
- Download PDF button.

### Pricing Hypotheses

Because generic quoting tools are often free or low-cost, pricing must emphasize no-app simplicity.

Initial pricing tests:

- $5 per quote.
- $10 per quote.
- $20 for 5 quotes.
- $49 for 15 quotes.

Preferred first test:

> $20 prepaid bundle for 5 quotes.

Reasoning:

- Avoids tiny transaction friction.
- Still avoids subscription positioning.
- Tests whether users will prepay for repeated use.
- Better than one-off $2 transactions.

### MVP Product Scope

#### MVP Includes

- SMS intake.
- Simple provider onboarding by SMS.
- Quote detail collection.
- Quote draft generation.
- Hosted quote HTML page.
- PDF generation.
- Stripe payment before unlock.
- SMS delivery of unlocked quote link.
- Optional email delivery.
- Basic admin review tools.

#### MVP Excludes

- Customer payment collection.
- Provider dashboard.
- Native mobile app.
- Scheduling.
- Invoicing lifecycle.
- CRM.
- Accounting integrations.
- Automatic pricing recommendation.
- Complex proposal templates.
- Multi-user accounts.

### Success Metrics

#### Demand Metrics

- Number of providers who try one quote.
- Number of providers who generate a second quote.
- Number of providers who prepay for a bundle.
- Number of real quotes sent to customers.
- Number of customer quote views.
- Number of customer approvals.

#### Product Metrics

- Time from initial SMS to quote preview.
- Number of clarification questions needed per quote.
- Percent of quotes requiring manual correction.
- Payment conversion from preview to unlock.
- Share rate after payment.

#### Business Metrics

- Revenue per provider.
- Quotes per provider per month.
- Cost per quote generated.
- Gross margin per quote.
- Support time per quote.
- Customer acquisition source.

### Validation Criteria

The idea is worth continuing if:

- At least 10 providers try the service.
- At least 3 providers generate more than one quote.
- At least 2 providers prepay for a quote bundle.
- Providers use real customer/job details.
- Providers say the SMS flow is easier than existing quote tools.

The idea should be killed or changed if:

- Providers say they already use Square, Joist, Jobber, or similar tools.
- Providers like the idea but do not send real quote requests.
- Providers refuse to pay even a small per-quote fee.
- Quote generation requires too much human editing.
- Providers need full invoicing, payments, scheduling, or CRM to care.

### Key Product Risks

#### Risk 1: Existing Tools Are Good Enough

Many competitors already generate quotes and send them by email/text. The product must avoid competing on feature depth and compete only on low-friction SMS usage.

#### Risk 2: Users Are Too Low-Tech To Adopt Even SMS Workflow

Some operators may be informal because they do not value quote artifacts at all. They may prefer verbal quotes or simple text messages.

#### Risk 3: Price Sensitivity

Users may like the product but not pay per quote. Bundled prepaid pricing should be tested early.

#### Risk 4: Quote Complexity

If the first vertical requires complex pricing, measurements, materials, or calculations, the MVP will become too hard. Start with simple service quotes.

#### Risk 5: Manual Support Burden

If every quote requires manual editing, margins will suffer. The MVP should constrain inputs and templates.

### Recommended Initial Vertical

Start with one or two simple, quote-by-text service types:

1. Gardeners / small landscaping crews.
2. Handymen.
3. Mobile detailers.
4. Junk removal / hauling.

Avoid initial verticals requiring precise measurement, complex materials, or formal contracts.

### First Launch Offer

> Text us your next job quote. We turn it into a professional quote link/PDF you can send to your customer. First quote free. Then $20 for 5 quotes. No app. No subscription.

### Open Questions

- Do solo operators want professional quote artifacts enough to pay?
- Is SMS the right interface, or should WhatsApp be supported early?
- Should the product support Spanish/English from day one?
- Do providers want customer approval tracking?
- Do providers want payment/deposit collection, or is that too much for MVP?
- Which vertical produces the most repeat usage?
