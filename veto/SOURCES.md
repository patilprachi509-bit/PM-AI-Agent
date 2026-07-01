# SOURCES.md — VETO

## Overview

This file defines every data source VETO monitors, how frequently, what to extract, and which layer consumes it. Every source maps to a specific agent layer and a specific insight framework.

Firecrawl handles static pages. Puppeteer handles JavaScript-rendered pages. Gmail MCP handles delivery. All sources feed into SOUL.md interpretation logic.

---

## Source Registry

### CATEGORY 1: Competitor Core Pages

**What:** Primary product and company pages for every competitor in the Competitor Registry.

**Tool:** Firecrawl

**Pages per competitor:**
```
Homepage                    → Positioning changes, messaging shifts
/changelog or /updates      → Feature releases, product direction
/blog                       → Strategic narrative, thought leadership signals
/pricing                    → Tier changes, feature gating, price movements
/features or /product       → Capability additions or removals
/customers or /case-studies → New verticals, segment expansion signals
/about or /careers          → Team growth, culture shifts
```

**Frequency:**
```
Homepage:        Every 14 days
Changelog:       Every 7 days
Blog:            Every 7 days
Pricing:         Every 14 days (diff against previous version)
Features:        Every 14 days
Case studies:    Every 30 days
Careers page:    Every 7 days (feeds into LinkedIn hiring signal)
```

**What to extract:**
```
- New feature announcements
- Pricing tier changes (flag any removal or addition of features per tier)
- New customer logos or verticals mentioned
- Blog post topics (reveals strategic focus areas)
- New pages that did not exist in previous crawl
```

**Layer:** Layer 1 Signal
**Framework:** Porter's Five Forces, SWOT

---

### CATEGORY 2: LinkedIn Hiring Signals

**What:** Job postings from competitor company pages on LinkedIn.

**Tool:** Puppeteer (LinkedIn is JavaScript-rendered, Firecrawl cannot access it reliably)

**What to extract per job posting:**
```
- Job title
- Department (engineering, sales, growth, PM, design, legal)
- Location (city and country)
- Date posted
- Key responsibilities (reveals product direction)
- Seniority level (IC vs manager vs director signals org scaling)
```

**Frequency:** Every 7 days per competitor

**Interpretation rules (from SOUL.md):**
```
3+ enterprise sales hires          → Moving upmarket
ML/AI engineers (new pattern)      → AI feature incoming in 60-90 days
Growth/PLG roles                   → Shifting from sales-led to product-led
New geography hires                → Market expansion signal
Domain-specific PM roles           → New product vertical
No new posts for 60+ days          → Financial pressure or strategic pause
Legal/compliance roles             → Regulatory preparation, enterprise push
Data/analytics engineers           → Instrumentation push, metric-driven pivot
```

**Layer:** Layer 1 Signal
**Framework:** Porter's Five Forces (threat of new entrants, competitive rivalry)

---

### CATEGORY 3: G2 Reviews

**What:** Customer reviews for each competitor on G2.

**Tool:** Firecrawl

**URLs to crawl per competitor:**
```
https://www.g2.com/products/[competitor-slug]/reviews
  → Filter: Most Recent (to catch new sentiment shifts)
  → Filter: By star rating (1-2 stars reveal pain points = your opportunities)
```

**What to extract:**
```
- Review date
- Star rating
- Review title
- Pros section (what users love = their strongest retention hooks)
- Cons section (what users hate = your attack surface)
- "What problems are you solving" section (reveals actual JTBD)
- Reviewer role and company size (reveals which segment is most vocal)
```

**Frequency:** Every 7 days

**What to flag immediately:**
```
- Spike in 1-2 star reviews in last 30 days → Quality degradation signal
- Repeated mention of a missing feature → Unmet need you can capture
- Praise for a feature you don't have → Competitive gap to close or counter-position against
- Reviews from your target segment mentioning switching intent → Acquisition opportunity
- "We switched from [your product] to [competitor]" mentions → Churn signal to investigate
```

**Layer:** Layer 1 Signal, Layer 2 Devil's Advocate (used as market evidence)
**Framework:** SWOT (competitor weaknesses from cons = your opportunities)

---

### CATEGORY 4: Capterra Reviews

**What:** Customer reviews for each competitor on Capterra.

**Tool:** Firecrawl

**URLs to crawl per competitor:**
```
https://www.capterra.com/p/[competitor-id]/[competitor-slug]/
  → Reviews tab
  → Sort by Most Recent
```

**What to extract:**
```
- Review date
- Overall rating
- Ease of use rating (proxy for onboarding friction)
- Customer service rating (proxy for support quality)
- Value for money rating (proxy for pricing sensitivity)
- Pros and cons (same extraction as G2)
- Reviewer company size (SMB vs mid-market vs enterprise)
```

**Frequency:** Every 7 days

**Cross-reference with G2:**
```
If same pain point appears on both G2 and Capterra → High confidence signal
If pain point appears only on one platform → Lower confidence, watch for pattern
If ratings diverge significantly between platforms → Investigate audience difference
```

**Layer:** Layer 1 Signal, Layer 2 Devil's Advocate
**Framework:** SWOT, PESTLE (Social factor: shifting user expectations)

---

### CATEGORY 5: App Store Reviews

**What:** iOS App Store and Google Play Store reviews for competitors with mobile apps.

**Tool:** Puppeteer (both stores are JavaScript-rendered)

**URLs:**
```
iOS App Store:
https://apps.apple.com/app/[competitor-app-id]
  → Reviews section, sort by Most Recent

Google Play:
https://play.google.com/store/apps/details?id=[competitor-package-id]
  → Reviews section, sort by Newest
```

**What to extract:**
```
- Review date
- Star rating
- Review text
- Version number reviewed (to correlate complaints with specific releases)
- Developer response (reveals how they handle criticism)
```

**Frequency:** Every 7 days

**What to flag:**
```
- Rating drop after a specific app version → Release caused regression
- Flood of 1-star reviews in short window → Major bug or UX degradation
- Complaints about removed features → They cut something users valued
- Praise for new feature → Validate if this is a gap in your product
- Developer response patterns → How they communicate with users under pressure
```

**Layer:** Layer 1 Signal
**Framework:** SWOT (competitor weaknesses), Porter's Five Forces (substitution risk)

---

### CATEGORY 6: Funding and Business Signals

**What:** Funding rounds, acquisitions, partnerships, and executive changes.

**Tool:** Firecrawl

**Sources:**
```
Crunchbase:
https://www.crunchbase.com/organization/[competitor-slug]
  → Funding rounds tab
  → Team tab (executive changes)

TechCrunch search:
https://techcrunch.com/?s=[competitor-name]

Product Hunt:
https://www.producthunt.com/search?q=[competitor-name]
  → New launches, upvote momentum
```

**What to extract:**
```
- New funding rounds (amount, round type, investors)
- Acquisitions (what capability did they buy?)
- Key executive hires or departures (CEO, CPO, CTO, VP Sales)
- New partnerships announced
- Product Hunt launches (new products or features going to market)
```

**Frequency:** Every 14 days

**Interpretation rules:**
```
Series B+ raised                   → 12-18 months of aggressive product expansion ahead
Acquisition of AI/data company     → Capability gap they couldn't build, now buying
CPO departure                      → Product strategy instability for 6-12 months
VP Sales hire (enterprise focused) → Moving upmarket
Strategic investor (e.g. Salesforce Ventures) → Future integration or acquisition signal
Product Hunt launch                → Feature going GA, gauge market reception by upvotes
```

**Layer:** Layer 1 Signal
**Framework:** Porter's Five Forces (threat of new entrants, bargaining power shifts)

---

### CATEGORY 7: PM's Own Inputs (Layers 2, 3, 4)

**What:** Content the PM brings to VETO for evaluation.

**Input methods:**

```
MANUAL PASTE (default, no integration needed)
  → PM pastes decision description into WebChat
  → PM pastes roadmap items into WebChat
  → PM pastes PRD excerpt for Layer 2 evaluation

GOOGLE DRIVE PULL (if PM connects Drive MCP)
  → Pull PRD documents by sharing link in WebChat
  → Pull roadmap spreadsheet by sharing link
  → VETO reads and evaluates without PM needing to copy-paste

JIRA / LINEAR (future phase, not day one)
  → Pull active epics and stories for Layer 4 Anti-Roadmap
  → Compare spec vs active tickets for reality gap detection
```

**Layer:** Layer 2 Devil's Advocate, Layer 3 Pre-Regret, Layer 4 Anti-Roadmap
**Framework:** Assumption Taxonomy (Layer 2), Scenario Modeling (Layer 3), PESTLE + Sunk Cost (Layer 4)

---

### CATEGORY 8: Gong / Sales Call Transcripts (if integrated)

**What:** Sales call transcripts revealing what prospects say about competitors during evaluation.

**Tool:** Gong MCP or manual paste

**What to extract:**
```
- Competitor names mentioned during calls
- Feature comparisons prospects raise
- Objections tied to competitor capabilities
- Reasons given for choosing competitor over you (lost deals)
- Reasons given for choosing you over competitor (won deals)
```

**Frequency:** Weekly batch pull or real-time if Gong MCP available

**What this unlocks:**
```
Competitor mentions in lost deals    → Direct feature gap to address
Competitor mentions in won deals     → Your actual differentiators (not assumed ones)
Repeated prospect objection          → Layer 2 input: challenge PM assumptions about positioning
Price objections tied to competitor  → Pricing pressure signal for Layer 1
```

**Layer:** Layer 1 Signal, Layer 2 Devil's Advocate (strongest source of market evidence)
**Framework:** SWOT (your real strengths and weaknesses from market's mouth)

**Note:** If Gong is not integrated, PM can paste transcript excerpts directly into WebChat. VETO will extract competitor signals from the text.

---

## Source Priority Matrix

When signals conflict across sources, apply this hierarchy:

```
TIER 1 (Highest confidence — act immediately)
  Gong transcripts: Direct voice of customer and prospect
  G2 + Capterra combined (same signal on both): Cross-validated market sentiment

TIER 2 (High confidence — act within 2 weeks)
  LinkedIn hiring patterns (3+ roles, same function): Strong roadmap signal
  Funding news (Series B+): Validated by investors
  App store rating drops (version-correlated): Objective quality signal

TIER 3 (Medium confidence — watch and verify)
  Changelog entries: Real but may be minor
  Blog posts: Strategic narrative, may be aspirational not actual
  Homepage messaging changes: Repositioning signal, timing unclear

TIER 4 (Low confidence — flag only if pattern emerges)
  Single G2 or Capterra review: Anecdotal
  Product Hunt launch: Gauge reception before acting
  Press releases: Marketing, verify against product reality
```

---

## Pricing Page Diff Protocol

Pricing pages are the highest-signal single page for competitive intelligence. Changes here reveal strategic shifts faster than any other source.

On every crawl of a competitor pricing page:
```
1. Store full page content with timestamp
2. On next crawl, diff against stored version
3. Flag any of the following changes:
   - Feature moved from paid to free tier → Land grab strategy
   - Feature moved from free to paid tier → Monetization tightening
   - New tier added → Segment expansion
   - Tier removed → Simplification or consolidation
   - Price increase → Confidence in retention, enterprise push
   - Price decrease → Competitive pressure, growth mode
   - New annual discount added → Improving cash flow, retention focus
   - Enterprise tier added → Moving upmarket definitively
4. Never alert on cosmetic changes (color, layout, copy tweaks)
5. Only alert on structural or price changes
```

---

## Data Storage Convention

All scraped content stored with the following structure before passing to SOUL.md:

```
{
  source: "g2",
  competitor: "[name]",
  url: "[url]",
  crawl_date: "[ISO date]",
  content_type: "review | changelog | job_posting | pricing | blog | funding",
  raw_content: "[extracted text]",
  previous_crawl_date: "[ISO date]",
  diff_from_previous: "[what changed]",
  layer_destination: "layer1 | layer2 | layer3 | layer4",
  priority: "critical | high | medium | low"
}
```

---

## Source Activation Schedule

```
DAILY
  Nothing. Daily crawls create noise. Weekly is the minimum signal window.

EVERY 7 DAYS
  Competitor blogs
  Competitor changelogs
  G2 reviews (all competitors)
  Capterra reviews (all competitors)
  App store reviews (all competitors)
  LinkedIn job postings (all competitors)
  Careers pages

EVERY 14 DAYS
  Competitor homepages (messaging diff)
  Competitor pricing pages (structural diff)
  Competitor feature pages
  Crunchbase profiles

EVERY 30 DAYS
  Competitor case studies and customer pages
  Competitor discovery refresh (find new emerging competitors)
  Gong batch pull (if not real-time)

EVERY 90 DAYS
  Layer 4 Anti-Roadmap trigger
  Full source audit (are all URLs still valid?)
  Add or remove sources based on PM feedback
```

---

## What VETO Does With All Of This

Every piece of content extracted from every source above gets passed through the Interpretation Framework in SOUL.md before any output is generated.

Raw content never reaches the PM. Only interpreted signal does.

The pipeline is:

```
Source crawled by Firecrawl or Puppeteer
        ↓
Raw content stored with metadata
        ↓
Passed to SOUL.md Interpretation Framework
        ↓
Signal classified by type, threat level, and layer destination
        ↓
Aggregated into Weekly Radar (Layer 1) or triggered in real-time (Layers 2-4)
        ↓
Delivered via Gmail (scheduled) or WebChat (real-time)
```

Nothing skips the interpretation step. Ever.
