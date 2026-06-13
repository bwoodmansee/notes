## Text-to-Quote Implementation Plan

### Goal

Launch the Text-to-Quote idea with the lowest possible cost and complexity, while testing whether low-tech solo service providers will actually use and pay for professional quote artifacts generated from a text message.

The goal is not to build a polished SaaS product. The goal is to answer one market question:

> Will low-tech solo operators repeatedly pay for professional quote links/PDFs generated from simple text messages?

### Initial Positioning

Use simple buyer-facing language:

> Text us the job details. We send back a professional quote link/PDF you can forward to your customer. No app. No login. No subscription.

Avoid saying:

- AI quote platform.
- SaaS for contractors.
- CRM.
- Field-service software.
- Business operating system.

The positioning should emphasize:

- No app.
- No subscription.
- Works by text.
- Looks professional.
- English/Spanish available.
- Pay only when you need it.

### Initial Target Market

Start with one or two low-tech local service segments.

Recommended first segments:

1. Solo gardeners / small landscaping crews.
2. Handymen.
3. Mobile detailers.
4. Junk removal / hauling operators.

The best first segment is probably solo gardeners and small landscaping crews because many work locally, quote custom residential jobs, may use SMS heavily, and may benefit from a more professional-looking quote without wanting software.

### What Needs To Be Built First

Build only the smallest capability that can generate a real quote artifact.

#### Manual MVP

Before writing much code, launch a concierge-style version:

1. Buy or set up a dedicated phone number.
2. Create 3 reusable quote templates.
3. Create a simple intake format.
4. Manually turn texted job details into a quote.
5. Generate a PDF or hosted quote page.
6. Text the quote link back to the provider.
7. Track usage and repeat requests in a spreadsheet.

This can be done with:

- Google Voice, OpenPhone, or Twilio.
- Google Docs, Canva, or a simple HTML template.
- Google Drive, Cloudflare R2, S3, or static hosting for PDFs/pages.
- Stripe Payment Links, Venmo, Zelle, or cash for early payment tests.
- Google Sheets or Airtable for operator/customer/job tracking.

The manual MVP should be ugly internally but polished externally.

### First Quote Artifact

The generated quote should include:

- Provider name.
- Provider phone number.
- Optional logo/photo/business card styling.
- Customer name.
- Customer address or neighborhood.
- Scope of work.
- Included services.
- Exclusions / assumptions.
- Price.
- Deposit terms if any.
- Quote expiration date.
- Photos if provided.
- English/Spanish version if requested.
- Approve button or reply instructions.
- PDF download.

The quote should look much more professional than a plain text message, but it should not require a full account system.

### Low-Cost Technical Architecture

After manual validation, build a lightweight version.

#### V1 Components

- SMS intake: Twilio.
- App/webhook: Cloudflare Workers, Vercel, or a small Node/Rails/Python app.
- Parsing: LLM extracts customer, scope, price, terms, photos, language, missing fields.
- Storage: Cloudflare R2 or S3 for quote HTML/PDF artifacts.
- Database: Supabase, SQLite, Postgres, or Airtable initially.
- PDF generation: Playwright/Puppeteer, browserless service, or a hosted HTML-to-PDF tool.
- Payments: Stripe Payment Links or prepaid bundles.
- Hosting: Cloudflare Pages, Vercel, or static S3/R2 pages.

#### Avoid In V1

Do not build:

- Mobile app.
- Provider dashboard.
- Full CRM.
- Full invoice system.
- Payment processing inside quotes.
- Scheduling.
- Customer accounts.
- Complex estimate line-item catalogs.
- Real-time approval workflows beyond a simple approve button.

### Product Flow

#### Provider Flow

1. Provider texts job details to the Text-to-Quote number.
2. System detects missing fields.
3. System asks one concise follow-up question if needed.
4. Provider replies.
5. System generates quote artifact.
6. Provider receives link and optional PDF.
7. Provider forwards link to customer.

#### Customer Flow

1. Customer opens quote link.
2. Customer sees professional quote.
3. Customer can download PDF.
4. Customer can approve by tapping a button or replying to provider.

#### Admin Flow

1. Review generated quotes before sending during early stage.
2. Correct bad parsing manually.
3. Track each quote request, generated quote, user, and repeat usage.
4. Record whether the operator forwarded the quote and whether the customer approved.

### Pricing Test

Start simple.

#### Suggested Beta Pricing

- First quote free.
- $5 per quote after that.
- $20 for 5 prepaid quotes.
- $49 for 15 prepaid quotes.

The preferred test is the prepaid bundle because it reduces micro-payment friction and tests real buying intent.

#### What Not To Do Initially

Avoid monthly subscription pricing at first. The wedge is no subscription. If repeat use is proven, subscription can be tested later.

### Marketing Plan

### Core Marketing Message

Use direct, non-technical language:

> Stop texting messy prices. Text us the job details and get back a professional quote you can send to your customer.

Alternative messages:

- Professional quotes by text message.
- No app. No login. No monthly fee.
- Text the job. Get the quote.
- Look professional without learning software.
- English/Spanish quotes from a simple text.

### First Customer Acquisition Channels

#### 1. Direct local outreach

Build a list of local solo operators and small crews:

- Gardeners.
- Handymen.
- Mobile detailers.
- Junk haulers.
- Pressure washers.
- House cleaners.

Sources:

- Google Maps.
- Yelp.
- Nextdoor.
- Facebook local groups.
- Craigslist service listings.
- Business cards / flyers on trucks.
- Local supply stores and nurseries.
- Referrals from homeowners.

#### 2. Field marketing

This product is for low-tech users, so purely online marketing may miss them.

Test:

- Simple business cards.
- Flyers at local nurseries, hardware stores, paint stores, and supply shops.
- QR code plus phone number.
- Spanish/English copy.
- Example before/after quote visuals.

Example flyer headline:

> Still texting prices to customers? Send professional quotes by text.

Call to action:

> Text QUOTE to [number] and get your first quote free.

#### 3. Demonstration outreach

Create a sample quote for a hypothetical landscaping or handyman job and show it directly.

Message:

> I made an example of what your quotes could look like. You text job details, and we send back a clean quote link you can forward to the customer. First quote is free.

#### 4. Spanish-language angle

Test bilingual positioning if relevant locally:

> Cotizaciones profesionales por mensaje de texto. Sin aplicación. Sin mensualidad. Primera cotización gratis.

This may be a meaningful wedge if many operators are comfortable doing work by SMS/WhatsApp but not with English-first business software.

### First Outreach Script

#### Text / DM

> Hi — I’m testing a simple service for gardeners/handymen. You text the job details, and we send back a professional quote link/PDF you can send to your customer. No app or monthly fee. First quote is free. Want to try it on your next job?

#### In-person

> When you quote jobs, do you usually text the customer a price? I’m testing a service where you text us the job details and we send back a professional quote you can forward. First one is free.

### What To Measure

Do not measure vanity signups. Measure actual quote behavior.

#### Activation Metrics

- Number of operators contacted.
- Number who respond.
- Number who send a real quote request.
- Number who forward the quote to a real customer.
- Number who ask for changes.
- Number who request a second quote.
- Number who prepay for a bundle.

#### Business Metrics

- Cost to acquire one active operator.
- Time to generate one quote manually.
- Percentage of first-time users who request a second quote.
- Percentage willing to pay $5 per quote.
- Percentage willing to prepay $20 for 5 quotes.
- Average quote value.
- User feedback on quote professionalism.

### Validation Thresholds

#### Continue If

Continue if, after contacting 50-100 operators:

- 10+ operators try a first quote.
- 5+ use the quote with a real customer.
- 2+ request a second quote without being chased.
- 2+ prepay for a bundle or pay after the free quote.
- Manual quote generation takes under 10 minutes per quote.

#### Kill Or Pivot If

Stop or pivot if:

- Operators already use Square/Joist/etc. and are satisfied.
- Operators only want it if free.
- Nobody requests a second quote.
- Quote generation requires too much custom editing.
- Customer acquisition is too expensive relative to $5-$10 quotes.
- Users actually want invoicing, scheduling, CRM, or payments rather than quote artifacts.

### Launch Sequence

#### Phase 1: One-Weekend Manual MVP

Build:

- Dedicated phone number.
- 3 quote templates.
- Simple landing page.
- Example quote link.
- Stripe/Venmo/Zelle payment option.
- Tracking spreadsheet.
- Flyer/business card.

Outcome:

- Ready to generate quotes manually.

#### Phase 2: 50-Operator Outreach

Reach out to 50 local operators.

Targets:

- 20 gardeners/landscapers.
- 15 handymen.
- 10 mobile detailers.
- 5 junk haulers or pressure washers.

Goal:

- Get 10 first quote attempts.

#### Phase 3: Manual Fulfillment

For every quote request:

1. Parse the job details manually.
2. Ask missing questions by SMS.
3. Generate quote using template.
4. Send back link/PDF.
5. Ask whether they sent it to the customer.
6. Ask whether they want another.

Goal:

- Learn whether the job-to-quote transformation is valuable enough to repeat.

#### Phase 4: Automate Repeated Steps

Only automate after repeated usage.

Automate:

- SMS intake.
- Field extraction.
- Missing question detection.
- HTML quote generation.
- PDF generation.
- Hosted quote link.
- Payment bundle tracking.

Do not automate everything at once.

### Minimal Landing Page

The landing page should be very simple.

Sections:

1. Headline.
2. Example quote screenshot.
3. How it works.
4. Pricing.
5. Text number call-to-action.
6. FAQ.

Suggested copy:

> Professional quotes by text message
>
> Send us the job details. We send back a clean quote link/PDF you can text to your customer. No app. No login. No monthly fee. First quote free.

### FAQ

#### Do I need an app?

No. Just text the job details.

#### Do I need a subscription?

No. Pay per quote or buy a small prepaid bundle.

#### Can I send photos?

Yes. Send photos with the job details and they can be included in the quote.

#### Can quotes be in Spanish?

Yes. Quotes can be English, Spanish, or both.

#### Can the customer approve the quote?

Yes. The quote page can include a simple approve button or instructions to reply.

### Budget

Possible launch budget:

- Domain: $10-$20.
- Landing page: free to $20/month.
- Phone/SMS: $10-$50/month initially depending on provider and usage.
- Hosting/storage: free to $20/month initially.
- Design/templates: free to $20.
- Flyers/business cards: $50-$150.
- Total initial budget: roughly $100-$300.

### Key Product Principle

Do not compete with Square, Joist, LawnPro, Housecall Pro, or Jobber on features.

Compete on friction:

> The user should be able to get a professional quote without learning software.

### Next Concrete Step

Build the manual version and get 10 real quote attempts from real operators before writing a full product.
