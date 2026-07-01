# TOOLS.md — VETO

## Overview

This file defines every tool VETO uses to execute its agentic workflows. Three tool categories: scraping (Firecrawl + Puppeteer), delivery (Gmail MCP), and search (web search for real-time Instant Intel). Each tool maps to specific sources defined in SOURCES.md and specific layers defined in SOUL.md.

Setup instructions are included for each tool. Credentials are placeholders — replace with your own keys during installation.

---

## Tool 1: Firecrawl

**Purpose:** Crawl static web pages. Handles competitor websites, G2, Capterra, Crunchbase, TechCrunch, Product Hunt.

**Install:**
```bash
clawhub install firecrawl
```

**Credential setup:**
```bash
# Get your API key at https://firecrawl.dev
# Add to your environment:
export FIRECRAWL_API_KEY=your_api_key_here
```

**Core configuration:**
```yaml
firecrawl:
  api_key: ${FIRECRAWL_API_KEY}
  default_options:
    formats: ["markdown"]          # Extract as markdown, not raw HTML
    only_main_content: true        # Strip nav, footer, ads
    remove_base64_images: true     # Skip images, text only
    timeout: 30000                 # 30 second timeout per page
    wait_for: 2000                 # Wait 2s for page to settle
```

---

### Firecrawl Job Definitions

Each job maps to a source category from SOURCES.md.

---

**JOB: competitor_core_pages**

Runs every 7 days for blogs and changelogs. Every 14 days for homepages and pricing.

```yaml
job: competitor_core_pages
tool: firecrawl
action: crawl
targets:
  - path: /changelog
    frequency: every_7_days
    extract:
      - new_feature_announcements
      - deprecation_notices
      - version_numbers
    layer: layer1
    priority_if_changed: high

  - path: /blog
    frequency: every_7_days
    extract:
      - post_titles
      - post_dates
      - post_topics
      - strategic_themes
    layer: layer1
    priority_if_changed: medium

  - path: /pricing
    frequency: every_14_days
    diff: true                     # Always diff against previous crawl
    extract:
      - tier_names
      - tier_prices
      - features_per_tier
      - cta_text
    layer: layer1
    priority_if_changed: critical  # Any pricing change = critical signal

  - path: /
    frequency: every_14_days
    diff: true
    extract:
      - hero_headline
      - primary_value_proposition
      - target_customer_language
      - social_proof_logos
    layer: layer1
    priority_if_changed: high

  - path: /features
    frequency: every_14_days
    extract:
      - feature_names
      - feature_descriptions
      - new_sections
    layer: layer1
    priority_if_changed: high

  - path: /customers
    frequency: every_30_days
    extract:
      - customer_logos
      - industry_verticals_mentioned
      - case_study_titles
    layer: layer1
    priority_if_changed: medium
```

---

**JOB: g2_reviews**

```yaml
job: g2_reviews
tool: firecrawl
action: scrape
frequency: every_7_days
url_pattern: "https://www.g2.com/products/{competitor_slug}/reviews"
params:
  sort: most_recent
  per_page: 20                     # Last 20 reviews per crawl
extract:
  - review_date
  - star_rating
  - review_title
  - pros_text
  - cons_text
  - problem_solved_text
  - reviewer_role
  - reviewer_company_size
filters:
  flag_immediately:
    - star_rating: [1, 2]          # Low ratings always flagged
    - contains_text:
        - "switched from"
        - "moving to"
        - "cancelling"
        - "churning"
        - "better than"
        - "worse than"
cross_reference: capterra_reviews  # Cross-validate pain points across platforms
layer: layer1
secondary_layer: layer2            # Used as evidence in Devil's Advocate
framework: SWOT
priority_if_pattern: high
priority_if_single: low
```

---

**JOB: capterra_reviews**

```yaml
job: capterra_reviews
tool: firecrawl
action: scrape
frequency: every_7_days
url_pattern: "https://www.capterra.com/p/{competitor_id}/{competitor_slug}/"
params:
  tab: reviews
  sort: most_recent
  per_page: 20
extract:
  - review_date
  - overall_rating
  - ease_of_use_rating
  - customer_service_rating
  - value_for_money_rating
  - pros_text
  - cons_text
  - reviewer_company_size
cross_reference: g2_reviews
layer: layer1
secondary_layer: layer2
framework: SWOT
priority_if_pattern: high
priority_if_single: low
```

---

**JOB: funding_signals**

```yaml
job: funding_signals
tool: firecrawl
action: scrape
frequency: every_14_days
sources:
  - url_pattern: "https://www.crunchbase.com/organization/{competitor_slug}"
    extract:
      - latest_funding_round
      - funding_amount
      - investors
      - acquisition_activity
      - executive_changes

  - url_pattern: "https://techcrunch.com/?s={competitor_name}"
    extract:
      - article_titles
      - article_dates
      - article_summaries

  - url_pattern: "https://www.producthunt.com/search?q={competitor_name}"
    extract:
      - product_launches
      - upvote_counts
      - launch_dates
      - product_descriptions

layer: layer1
framework: "Porter's Five Forces"
priority_if_new_funding: critical
priority_if_acquisition: critical
priority_if_executive_change: high
```

---

## Tool 2: Puppeteer

**Purpose:** Scrape JavaScript-rendered pages that Firecrawl cannot access. Handles LinkedIn job postings and app store reviews.

**Install:**
```bash
clawhub install puppeteer
```

**No API key required.** Puppeteer runs a headless browser locally. However:
```bash
# Install Chromium dependency
npx puppeteer browsers install chrome
```

**Core configuration:**
```yaml
puppeteer:
  headless: true
  args:
    - "--no-sandbox"
    - "--disable-setuid-sandbox"
    - "--disable-dev-shm-usage"
  timeout: 60000                   # 60s timeout — JS pages are slower
  wait_until: networkidle2         # Wait until network is fully idle
  user_agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
  viewport:
    width: 1280
    height: 800
```

---

### Puppeteer Job Definitions

---

**JOB: linkedin_hiring_signals**

```yaml
job: linkedin_hiring_signals
tool: puppeteer
action: scrape
frequency: every_7_days
url_pattern: "https://www.linkedin.com/company/{competitor_linkedin_slug}/jobs/"
extract:
  - job_title
  - department
  - location_city
  - location_country
  - date_posted
  - seniority_level
  - key_responsibilities_first_200_chars
rate_limit:
  delay_between_requests: 3000     # 3s between requests — respect LinkedIn
  max_pages_per_session: 5
interpretation_rules:
  patterns_to_flag:
    - condition: "3+ enterprise sales roles in 30 days"
      signal: "Moving upmarket"
      priority: high

    - condition: "ML/AI engineering roles (new pattern, not historical)"
      signal: "AI feature incoming 60-90 days"
      priority: critical

    - condition: "Growth/PLG/product-led roles"
      signal: "Shifting from sales-led to product-led"
      priority: high

    - condition: "New geography hires (city not seen before)"
      signal: "Market expansion signal"
      priority: high

    - condition: "Domain-specific PM role (e.g. PM Payments, PM Enterprise)"
      signal: "New product vertical"
      priority: high

    - condition: "No new postings for 60+ days"
      signal: "Financial pressure or strategic pause"
      priority: medium

    - condition: "Legal or compliance roles (new pattern)"
      signal: "Regulatory preparation or enterprise push"
      priority: medium

    - condition: "Data/analytics engineering roles (new pattern)"
      signal: "Instrumentation push or metric-driven pivot"
      priority: medium

layer: layer1
framework: "Porter's Five Forces"
```

---

**JOB: app_store_reviews**

```yaml
job: app_store_reviews
tool: puppeteer
action: scrape
frequency: every_7_days
sources:
  ios:
    url_pattern: "https://apps.apple.com/app/{competitor_app_id}"
    extract:
      - review_date
      - star_rating
      - review_title
      - review_text
      - app_version_reviewed
      - developer_response_text

  android:
    url_pattern: "https://play.google.com/store/apps/details?id={competitor_package_id}"
    extract:
      - review_date
      - star_rating
      - review_text
      - developer_response_text

per_crawl_limit: 30                # Last 30 reviews per platform per competitor
filters:
  flag_immediately:
    - star_rating: [1, 2]
    - rating_drop: 0.3             # Flag if average drops 0.3+ points since last crawl
    - review_spike: true           # Flag if review volume 2x normal in 7 days
  extract_patterns:
    - "crash"
    - "broken"
    - "removed"
    - "used to"                    # "It used to work" = regression signal
    - "switched"
    - "love the new"               # Positive signal on new feature
layer: layer1
framework: SWOT
```

---

## Tool 3: Web Search

**Purpose:** Real-time search for Layer 1 Instant Intel and Move Simulator. Not scheduled — triggered on demand when PM asks a question.

**Install:** Built into OpenClaw. No additional setup required.

**Configuration:**
```yaml
web_search:
  type: web_search_20250305
  name: web_search
  use_cases:
    - layer1_instant_intel          # PM asks real-time competitor question
    - layer3_move_simulator         # PM asks "what if competitor does X"
    - onboarding_competitor_discovery # Auto-discover competitors during setup
  query_construction:
    always_include: [competitor_name]
    context_append: [market_from_product_context_card]
    date_filter: last_90_days       # Prioritize recent results
    exclude_terms: ["press release template", "stock photo"]
  max_results_per_query: 10
  follow_top_result: true           # Fetch full content of top result
```

**Query templates per use case:**

```yaml
query_templates:
  competitor_recent_moves:
    template: "{competitor_name} product update OR launch OR feature {current_year}"

  competitor_pricing_change:
    template: "{competitor_name} pricing change OR price increase OR new plan {current_year}"

  competitor_hiring:
    template: "{competitor_name} hiring {role_type} site:linkedin.com OR site:greenhouse.io"

  competitor_funding:
    template: "{competitor_name} funding OR raises OR acquires {current_year}"

  market_entrant:
    template: "new {market_category} tool OR startup {current_year} site:producthunt.com OR site:techcrunch.com"

  move_simulator:
    template: "{competitor_name} {feature_type} launch OR beta OR announcement"

  onboarding_discovery:
    template: "top {market_category} tools OR software OR platforms {geography} {current_year}"
```

---

## Tool 4: Gmail MCP

**Purpose:** Automated delivery of Layer 1 Weekly Radar and Layer 4 Anti-Roadmap audit. Scheduled reports only — real-time interactions stay in WebChat.

**Already connected:** Gmail MCP is active in your connector setup. No additional installation needed.

**Configuration:**
```yaml
gmail:
  mcp_url: "https://gmailmcp.googleapis.com/mcp/v1"
  sender_name: "VETO"
  delivery_jobs:
    - job: weekly_radar_email
    - job: anti_roadmap_email
    - job: critical_signal_alert
```

---

### Gmail Job Definitions

---

**JOB: weekly_radar_email**

```yaml
job: weekly_radar_email
tool: gmail
trigger: cron
schedule: "0 9 * * 1"             # Every Monday 9:00 AM
recipient: ${PM_EMAIL}
subject_template: "VETO Weekly Radar — {date} — {critical_count} critical signal(s)"
body_source: layer1_weekly_radar_output
body_format: html
html_structure:
  header: "VETO WEEKLY RADAR | {date}"
  sections:
    - title: "CRITICAL SIGNALS"
      content: critical_signals_list
      color_indicator: "#FF4444"

    - title: "HIGH PRIORITY"
      content: high_signals_list
      color_indicator: "#FF8800"

    - title: "MEDIUM / LOW"
      content: medium_low_signals_list
      color_indicator: "#888888"

    - title: "THIS WEEK'S BIGGEST THREAT"
      content: top_threat_summary
      style: bold

    - title: "THIS WEEK'S BIGGEST OPPORTUNITY"
      content: top_opportunity_summary
      style: bold

    - title: "RUN LAYER 2 ON ANY SIGNAL"
      content: "Open VETO in WebChat and paste: 'Run Devil's Advocate on: [signal]'"
      style: cta

  footer: "VETO | Confidence: {confidence_level} | Sources: {source_count} pages crawled"
send_condition: always             # Send even if no signals — "Quiet week" email
```

---

**JOB: anti_roadmap_email**

```yaml
job: anti_roadmap_email
tool: gmail
trigger: cron
schedule: "0 9 1 */3 *"          # First day of every quarter at 9:00 AM
recipient: ${PM_EMAIL}
subject_template: "VETO Anti-Roadmap Audit — {quarter} — {kill_count} item(s) flagged for kill"
body_source: layer4_anti_roadmap_output
body_format: html
html_structure:
  header: "VETO ANTI-ROADMAP AUDIT | {quarter}"
  sections:
    - title: "KILL LIST"
      content: kill_list
      color_indicator: "#FF4444"

    - title: "PAUSE LIST"
      content: pause_list
      color_indicator: "#FF8800"

    - title: "ZOMBIE LIST"
      content: zombie_list
      color_indicator: "#888888"

    - title: "SURVIVORS"
      content: survivors_list
      color_indicator: "#00AA44"

    - title: "CAPACITY RECOVERED"
      content: capacity_recovered_summary
      style: bold

    - title: "HARDEST KILL THIS QUARTER"
      content: hardest_kill
      style: bold_red

  footer: "VETO | Run 'Anti-Roadmap' in WebChat to provide your roadmap before next audit"
```

---

**JOB: critical_signal_alert**

```yaml
job: critical_signal_alert
tool: gmail
trigger: event                     # Not scheduled — fires when critical signal detected mid-week
recipient: ${PM_EMAIL}
subject_template: "VETO ALERT — {competitor_name} just made a critical move"
body_format: html
html_structure:
  header: "CRITICAL SIGNAL DETECTED"
  sections:
    - title: "WHAT HAPPENED"
      content: signal_what

    - title: "WHAT IT SIGNALS"
      content: signal_interpretation

    - title: "WHAT IT MEANS FOR YOU"
      content: signal_implication

    - title: "RECOMMENDED ACTION"
      content: signal_action

    - title: "RUN DEVIL'S ADVOCATE"
      content: "Open VETO in WebChat and paste: 'Run Devil's Advocate on: [your current decision]'"
      style: cta

send_condition: only_if_critical   # Only fires for Critical signals, not High/Medium
max_per_week: 3                    # Cap at 3 mid-week alerts — avoid alert fatigue
```

---

## Tool Execution Pipeline

How all four tools connect end to end:

```
SCHEDULED PIPELINE (Layer 1 Weekly Radar)

Monday 6:00 AM — Crawl starts (3 hours before delivery)
  Firecrawl: competitor_core_pages
  Firecrawl: g2_reviews
  Firecrawl: capterra_reviews
  Firecrawl: funding_signals
  Puppeteer: linkedin_hiring_signals
  Puppeteer: app_store_reviews

Monday 7:00 AM — Interpretation runs
  SOUL.md Interpretation Framework processes all raw content
  Signals classified by type and threat level
  Weekly Radar compiled

Monday 8:00 AM — QA check
  Verify at least 1 source per competitor returned content
  Flag any crawl failures for retry
  Confirm signal count and confidence level

Monday 9:00 AM — Delivery
  Gmail MCP sends Weekly Radar email

---

REAL-TIME PIPELINE (Layers 2, 3, 4)

PM input arrives in WebChat
  → SOUL.md detects layer trigger (decision / what-if / anti-roadmap)
  → Web search fires if additional context needed (Layer 2, 3)
  → No crawl needed — works on PM's input directly
  → Response delivered in WebChat within 30 seconds

---

CRITICAL ALERT PIPELINE (mid-week)

Firecrawl or Puppeteer detects critical signal during scheduled crawl
  → SOUL.md classifies as Critical
  → Gmail MCP fires critical_signal_alert immediately
  → Does not wait for Monday radar
  → Also included in Monday radar for completeness
```

---

## Environment Variables

All credentials stored as environment variables. Never hardcoded.

```bash
# Required
FIRECRAWL_API_KEY=your_firecrawl_api_key
PM_EMAIL=your_email@domain.com

# Auto-configured by OpenClaw during Gmail MCP connection
# No manual setup needed for Gmail
GMAIL_MCP_URL=https://gmailmcp.googleapis.com/mcp/v1

# Optional — set if using Gong integration
GONG_API_KEY=your_gong_api_key
GONG_API_SECRET=your_gong_api_secret
```

---

## Error Handling

```yaml
error_handling:
  crawl_failure:
    action: retry
    retry_attempts: 3
    retry_delay: 300               # 5 minutes between retries
    on_final_failure: log_and_skip # Do not block radar delivery
    notify_pm: false               # Silent failure — do not spam PM with errors

  partial_crawl:
    action: proceed_with_available_data
    note_in_output: true           # Flag in radar which sources were unavailable
    confidence_impact: reduce      # Lower confidence rating if sources missing

  rate_limited:
    action: exponential_backoff
    initial_delay: 60              # Start with 60s wait
    max_delay: 3600                # Max 1 hour wait
    platforms_most_sensitive:
      - linkedin                   # Most aggressive rate limiting
      - app_store                  # Moderate

  gmail_delivery_failure:
    action: retry
    retry_attempts: 2
    on_final_failure: deliver_to_webchat  # Fall back to WebChat if email fails
```

---

## Installation Checklist

Run through this in order before first activation:

```
[ ] 1. Create Firecrawl account at https://firecrawl.dev
[ ] 2. Copy API key and set FIRECRAWL_API_KEY in environment
[ ] 3. Run: clawhub install firecrawl
[ ] 4. Run: clawhub install puppeteer
[ ] 5. Run: npx puppeteer browsers install chrome
[ ] 6. Confirm Gmail MCP is connected in OpenClaw connectors
[ ] 7. Set PM_EMAIL environment variable
[ ] 8. Copy SOUL.md to ~/.openclaw/workspace/
[ ] 9. Copy SOURCES.md to ~/.openclaw/workspace/
[ ] 10. Copy TOOLS.md to ~/.openclaw/workspace/
[ ] 11. Run: openclaw agents add
[ ] 12. Run: openclaw start
[ ] 13. Open WebChat — onboarding flow triggers automatically
[ ] 14. Complete 5 onboarding questions
[ ] 15. Confirm competitor list
[ ] 16. Run first manual Layer 1 crawl to verify: openclaw run job competitor_core_pages
[ ] 17. Check WebChat for crawl output
[ ] 18. Send test email: openclaw test job weekly_radar_email
[ ] 19. Verify email received with correct format
[ ] 20. VETO is live
```
