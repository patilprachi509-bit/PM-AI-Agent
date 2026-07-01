# SKILLS.md — VETO

## Overview

This file documents every skill VETO uses, what it does, when it activates, and how it is configured. Skills extend VETO's capabilities beyond the base SOUL.md intelligence layer.

Two skill categories:
- Installed skills: downloaded from ClawHub, ready to use
- Custom skills: written specifically for VETO's competitive intelligence workflow

---

## Installed Skills

### web-search (ClawHub)

**Status:** Installed at `/data/.openclaw/workspace/skills/web-search`
**Version:** 1.0.0
**Provider:** DuckDuckGo API

**What it does:**
Searches the web in real time and returns results in markdown format. Powers Layer 1 Instant Intel and Move Simulator queries.

**When VETO uses it:**
- Any competitor question asked in WebChat
- Layer 1 Instant Intel mode
- Layer 3 Move Simulator (checking if a hypothetical move is already in progress)
- Onboarding competitor discovery (finding competitors based on product description)
- Any question requiring current market data

**Configuration:**
```yaml
name: web-search
provider: duckduckgo
output_format: markdown
filters:
  time_range: last_90_days
  safe_search: moderate
  region: in-en          # India English results prioritized
```

**Query patterns VETO uses:**

```
Competitor moves:
"[competitor] product launch OR feature OR update 2026"
"[competitor] new feature site:techcrunch.com OR site:inc42.com"

Hiring signals:
"[competitor] hiring product manager OR engineer site:linkedin.com 2026"

Funding:
"[competitor] funding OR raises OR series 2026 India"

Reviews:
"[competitor] reviews site:g2.com"
"[competitor] reviews site:capterra.com"

App store:
"[competitor] app update OR rating 2026"

India market specific:
"quick commerce India [topic] 2026"
"[competitor] India expansion OR launch 2026"
```

**Rate limits:** DuckDuckGo free tier, no API key required. No hard rate limit but avoid more than 10 searches per minute.

---

## Custom Skills

### pricing-monitor

**Status:** To be created
**Location:** `/data/.openclaw/workspace/skills/pricing-monitor/SKILL.md`
**Purpose:** Detect changes on competitor pricing pages between crawls

**What it does:**
Stores a snapshot of each competitor pricing page on first crawl. On every subsequent crawl, diffs the new content against the stored snapshot. Only alerts when structural changes occur (tier changes, price changes, feature gating changes). Ignores cosmetic changes.

**When VETO uses it:**
Every 14 days per competitor. Triggered automatically by cron.

**Create this skill:**
```bash
mkdir -p /data/.openclaw/workspace/skills/pricing-monitor
cat > /data/.openclaw/workspace/skills/pricing-monitor/SKILL.md << 'EOF'
---
name: pricing-monitor
description: Monitor competitor pricing pages for structural changes
read_when:
  - Checking competitor pricing changes
  - Running weekly radar pricing section
  - PM asks about competitor pricing
---

# Pricing Monitor Skill

## Purpose
Detect meaningful changes on competitor pricing pages. Ignore cosmetic changes. Only surface structural shifts that signal strategic moves.

## What Counts as a Meaningful Change
- Tier name changed
- Price point changed (any amount)
- Feature moved between tiers
- New tier added
- Tier removed
- Annual discount percentage changed
- Free tier limits changed
- Enterprise tier added or removed

## What to Ignore
- Color or layout changes
- Copy or marketing language tweaks
- Testimonial or logo changes
- Blog post additions

## How to Monitor
1. Search for competitor pricing page
2. Extract tier names, prices, and feature lists
3. Compare against last known state stored in memory
4. If meaningful change detected, classify the strategic signal
5. Report in Weekly Radar under CRITICAL or HIGH priority

## Signal Classification
Feature moved free → paid: Monetization tightening signal
Feature moved paid → free: Land grab or competitive pressure signal
New enterprise tier: Moving upmarket signal
Price increase: Confidence in retention signal
Price decrease: Competitive pressure or growth mode signal
New tier added: Segment expansion signal
Tier removed: Simplification or consolidation signal

## Output Format
PRICING CHANGE DETECTED
Competitor: [name]
Change: [what changed]
Previous: [old state]
Current: [new state]
Signal: [strategic interpretation]
Priority: [Critical/High/Medium]
Recommended action: [specific PM action]
EOF
```

---

### competitor-profile

**Status:** To be created
**Location:** `/data/.openclaw/workspace/skills/competitor-profile/SKILL.md`
**Purpose:** Maintain a living profile for each competitor that updates with every signal

**What it does:**
Every time a signal is detected about a competitor, this skill updates that competitor's profile with the new information. Over time each competitor profile becomes a comprehensive intelligence dossier the PM can query at any time.

**Create this skill:**
```bash
mkdir -p /data/.openclaw/workspace/skills/competitor-profile
cat > /data/.openclaw/workspace/skills/competitor-profile/SKILL.md << 'EOF'
---
name: competitor-profile
description: Living intelligence profiles for each competitor
read_when:
  - PM asks about a specific competitor
  - Updating competitor intelligence after new signal
  - Running Layer 2 Devil's Advocate (pull competitor evidence)
---

# Competitor Profile Skill

## Profile Structure
For each competitor maintain:

COMPETITOR: [name]
Last updated: [date]

STRATEGIC DIRECTION
[What their recent moves collectively signal about where they are going]

RECENT MOVES (last 90 days)
[Dated list of significant product, pricing, hiring, funding moves]

STRENGTHS (from G2/Capterra review analysis)
[What users consistently praise]

WEAKNESSES (from G2/Capterra review analysis)
[What users consistently complain about = your attack surface]

HIRING SIGNALS
[Current open roles and what they signal]

FUNDING STATUS
[Last round, amount, investors, runway estimate]

THREAT LEVEL TO US
[Critical/High/Medium/Low with reasoning]

OPPORTUNITY IN THEIR WEAKNESS
[Specific gap you can exploit based on their weaknesses]

## How to Update
When new signal arrives about a competitor:
1. Pull existing profile from memory
2. Add new signal to RECENT MOVES with date
3. Update STRATEGIC DIRECTION if pattern has shifted
4. Recalculate THREAT LEVEL if warranted
5. Save updated profile to memory

## How to Query
PM asks: "What is Blinkit doing right now?"
Response: Pull Blinkit profile, synthesize STRATEGIC DIRECTION + RECENT MOVES
Do NOT just dump the raw profile. Interpret it.
EOF
```

---

### decision-log

**Status:** To be created
**Location:** `/data/.openclaw/workspace/skills/decision-log/SKILL.md`
**Purpose:** Log every PM decision that passes through VETO and surface patterns over time

**What it does:**
After every Layer 2 interaction, logs the decision, VETO's verdict, the PM's response, and the key assumption flagged. After 30+ decisions, starts surfacing patterns in how this specific PM makes decisions. Builds a personalized PM decision quality profile over time.

**Create this skill:**
```bash
mkdir -p /data/.openclaw/workspace/skills/decision-log
cat > /data/.openclaw/workspace/skills/decision-log/SKILL.md << 'EOF'
---
name: decision-log
description: Log PM decisions and surface decision quality patterns
read_when:
  - After every Layer 2 Devil's Advocate session
  - PM asks about past decisions
  - PM asks VETO to review their decision patterns
  - Similar decision comes up (surface past precedent)
---

# Decision Log Skill

## Log Entry Format
After every Layer 2 session, create an entry:

DATE: [ISO date]
DECISION: [what the PM wanted to do]
VERDICT: [STRONG / WEAK / NEEDS REWORK]
KEY ASSUMPTION FLAGGED: [the most important unvalidated assumption]
PM RESPONSE: [defended / revised / overrode]
OVERRIDE REASON: [if overrode, what reason given]
OUTCOME: [fill in 30/60/90 days later when data available]

## Pattern Detection
After 10+ decisions, look for:
- Recurring assumption type the PM fails to validate (e.g. always skips feasibility)
- Frequency of overrides (high override rate = not using VETO seriously)
- Decision triggers (stakeholder pressure vs user data vs competitor moves)
- Domains where decisions are consistently weak (pricing, scope, timing)

## Surfacing Precedents
When PM brings a new decision:
1. Search decision log for similar past decisions
2. If found, surface it: "You made a similar call on [date]. Here is what you flagged as the risk then."
3. If outcome is known, include it: "That decision resulted in [outcome]."

## Quarterly Pattern Report
Every 90 days, generate:
VETO DECISION QUALITY REPORT
Total decisions reviewed: [n]
Override rate: [%]
Most common weak assumption type: [category]
Strongest decision domain: [area]
Weakest decision domain: [area]
One pattern to break: [specific observation]
EOF
```

---

## Skill Activation Summary

```
ALWAYS ACTIVE
web-search        → Every competitor query, Layer 1 Instant Intel

ACTIVE AFTER SETUP
pricing-monitor   → Every 14 days per competitor (cron)
competitor-profile→ Updates after every Layer 1 signal
decision-log      → Updates after every Layer 2 session

FUTURE SKILLS (Phase 2)
firecrawl         → Automated page crawling when free tier obtained
gmail-scheduler   → Monday radar delivery (after Gmail MCP connected)
gong-integration  → Sales call transcript analysis (if Gong available)
```

---

## How to Install Custom Skills

For each custom skill above, run the create commands in the container terminal:

```bash
# Install pricing-monitor
mkdir -p /data/.openclaw/workspace/skills/pricing-monitor
# Then paste the SKILL.md content as shown above

# Install competitor-profile
mkdir -p /data/.openclaw/workspace/skills/competitor-profile
# Then paste the SKILL.md content as shown above

# Install decision-log
mkdir -p /data/.openclaw/workspace/skills/decision-log
# Then paste the SKILL.md content as shown above
```

After installing all three, restart the container. VETO will automatically discover and load all skills in the workspace/skills directory.

---

## Skill Health Check

After restart, verify all skills are loaded by asking VETO in WebChat:

```
What skills do you have available?
```

Expected response should list:
- web-search
- pricing-monitor
- competitor-profile
- decision-log

If any are missing, check the skill directory exists and SKILL.md is properly formatted.
