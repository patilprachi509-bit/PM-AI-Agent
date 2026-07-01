# VETO — PM Decision Intelligence Agent

**The last line of defense between a PM's untested instincts and an engineering team's time.**

VETO sits between the PM's brain and the engineering team. Nothing gets built without surviving it first.

---

## What VETO Does

Every PM has stakeholders who nod, engineers who build what's asked, and designers who polish the brief. Nobody in that room is professionally obligated to tell the PM they are wrong.

VETO is.

VETO operates in four layers covering the entire pre-build decision lifecycle:

```
LAYER 1: SIGNAL           What is happening outside?
         ↓
LAYER 2: DEVIL'S ADVOCATE  Is your response to it wrong?
         ↓
LAYER 3: PRE-REGRET        What will you wish you had done differently?
         ↓
LAYER 4: ANTI-ROADMAP      What should you stop building entirely?
```

---

## How It Works

### Layer 1: Market Signal
Monitors competitors every week and delivers a radar to your Gmail every Monday at 9 AM. Watches competitor websites, G2 and Capterra reviews, LinkedIn hiring patterns, app store ratings, funding news, and pricing page changes. Interprets every move strategically — not just what happened, but what it signals and what you should do about it.

### Layer 2: Devil's Advocate
Bring any decision to VETO and it builds the strongest possible case against it. Identifies every hidden assumption, classifies each as validated or unvalidated, and forces you to defend your reasoning before a line of code is written. Confrontational about decisions. Never about people.

### Layer 3: Pre-Regret Analysis
After a decision clears Layer 2, VETO runs three scenarios (optimistic, realistic, worst case) and identifies what you will most regret 6 months from now. Outputs one specific action to take before you build.

### Layer 4: Anti-Roadmap
Every 90 days VETO audits everything on your roadmap and produces a Kill List. Applies three filters to every item: sunk cost, politics, and decay. Tells you what to stop building, what is a zombie, and what capacity you recover by killing the right things.

---

## Tech Stack

```
Agent framework:  OpenClaw v2026.6.9
AI model:         Google Gemini 2.5 Flash (free tier)
Hosting:          Hostinger VPS (Docker container)
Web search:       DuckDuckGo via web-search skill
Email delivery:   Gmail MCP
Interface:        OpenClaw WebChat
```

---

## File Structure

```
/data/.openclaw/workspace/
├── SOUL.md              Agent brain — identity, all 4 layers, frameworks
├── IDENTITY.md          Agent name and persona override
├── BOOTSTRAP.md         First run config (cleared after setup)
├── TOOLS.md             Tool configuration and search patterns
├── AGENTS.md            OpenClaw workspace config (default)
├── MEMORY.md            Long term memory (auto-updated by agent)
├── USER.md              PM context (populated during onboarding)
└── skills/
    ├── web-search/      DuckDuckGo search (installed from ClawHub)
    ├── pricing-monitor/ Competitor pricing page diff detection
    ├── competitor-profile/ Living competitor intelligence profiles
    └── decision-log/    PM decision history and pattern detection
```

---

## Setup

### Prerequisites
- Hostinger VPS with Docker
- Google Gemini API key (free at aistudio.google.com)
- Gmail account for report delivery

### Installation Checklist

```
[ ] 1. Deploy OpenClaw on Hostinger via Docker Compose
[ ] 2. Get Gemini API key from aistudio.google.com
[ ] 3. Configure Gemini as AI provider in OpenClaw settings
[ ] 4. Open container terminal (Hostinger Docker Manager → Terminal)
[ ] 5. Create workspace: mkdir -p /data/.openclaw/workspace
[ ] 6. Upload SOUL.md to workspace
[ ] 7. Update IDENTITY.md with VETO identity
[ ] 8. Clear BOOTSTRAP.md
[ ] 9. Update TOOLS.md with VETO config
[ ] 10. Install web-search skill: npx openclaw skills install web-search
[ ] 11. Fix /tmp permissions: mkdir -p /tmp/openclaw-1000 && chmod 777 /tmp/openclaw-1000
[ ] 12. Update openclaw.json: set skipBootstrap false, thinkingDefault medium
[ ] 13. Disable memorySearch: npx openclaw config set agents.defaults.memorySearch.enabled false
[ ] 14. Restart container from Hostinger Docker Manager
[ ] 15. Open WebChat and complete 5-question onboarding
[ ] 16. Install custom skills (pricing-monitor, competitor-profile, decision-log)
[ ] 17. Connect Gmail MCP for automated delivery
[ ] 18. Enable cron scheduling for Monday radar
```

---

## Daily Usage

### Monday Morning
VETO automatically emails your Weekly Radar at 9 AM IST. Open it, scan Critical signals, decide if any require running through Layer 2.

### Bringing a Decision to VETO
Open WebChat. Type your decision naturally:
```
I want to build a loyalty rewards feature for Zepto
```
VETO runs Layer 2 automatically. Respond to each unvalidated assumption or say "VETO override — proceeding anyway" with your reason.

### Running Pre-Regret Analysis
After a decision clears Layer 2:
```
Run pre-regret on [your decision]
```

### Asking About Competitors
```
What is Blinkit doing in quick commerce right now?
What if Swiggy Instamart cuts delivery fees to zero?
Has anyone in our space changed pricing recently?
```

### Running Anti-Roadmap
```
Run Anti-Roadmap
```
VETO will ask whether to use paste mode or structured question mode.

---

## Onboarding Questions

On first conversation VETO asks:

1. What does your product do and who is your primary user?
2. What market are you in?
3. What is your biggest strategic concern right now?
4. Who are your top 3 competitors?
5. What geography is your primary focus?

Answers build the Product Context Card used to interpret all signals and decisions.

---

## Automated Schedule

```
Every Monday 9:00 AM IST    Weekly Radar email
Every 14 days               Pricing page monitoring per competitor
Every 30 days               Competitor discovery refresh
Every 90 days               Anti-Roadmap audit prompt
On critical signal          Immediate email alert (max 3 per week)
```

---

## Key Design Decisions

**Confrontational by design.** VETO attacks decisions, never people. If the agent is too polite, PMs brush it off. Productive friction is the point.

**Override is always honored.** PM has final call. VETO logs every override but never argues after one is invoked.

**Interpretation over information.** Raw competitor data never reaches the PM. Every piece of information passes through the Interpretation Framework before output.

**Product Context Card as the lens.** Every signal, every decision, every audit is interpreted specifically against this PM's product context. Generic advice is useless.

**Decision Memory compounds over time.** The longer VETO is used, the more valuable it becomes. After 3 months it starts surfacing patterns in how this specific PM makes decisions.

---

## Known Limitations

**Gemini free tier rate limits.** Gemini 2.5 Flash free tier has low rate limits. On heavy usage days responses may be delayed. Quota resets every 24 hours.

**Web search via DuckDuckGo.** Results are good but not as deep as paid scraping. G2 and Capterra reviews require manual paste or Firecrawl integration for bulk extraction.

**No LinkedIn automation.** LinkedIn blocks automated scraping. Hiring signals require manual monitoring or Proxycurl integration (~$10/month).

**Single PM only.** Current setup supports one PM's context. Multi-PM support requires separate agent instances or a database-backed memory system.

---

## Roadmap

**Phase 1 (Current)**
Single PM, WebChat interface, web search for Layer 1, Gmail for delivery.

**Phase 2**
Firecrawl integration for automated competitor page monitoring. Pricing page diff automation. G2 and Capterra bulk review extraction.

**Phase 3**
Multi-PM support. Team-level decision quality dashboard. Slack integration for radar delivery. Jira/Linear integration for Layer 4 roadmap pull.

**Phase 4**
White-label for PM teams at product companies. API for integration with existing PM tools.

---

## Agent Philosophy

VETO is not a PM assistant. It is a PM challenger.

The goal is not to make PM work easier. The goal is to make PM decisions better. Sometimes those are the same thing. Often they are not.

A PM who uses VETO for 90 days should be making fewer bad decisions, killing the right things faster, and spending more time on work that actually compounds. Not because VETO made them smarter, but because VETO forced them to think before they built.

---

## Built By

Prachi Sharma — Technical AI PM
Built on OpenClaw framework
Hosted on Hostinger VPS
June 2026
