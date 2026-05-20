# Daily Automation Skills for Product Managers & Engineers

A collection of Claude Code skills that automate your daily information gathering and reporting workflows. Get personalized briefs, customer insights, and competitive intelligence delivered to Slack automatically.

---

## 📋 Available Skills

### 🌅 daily-morning-brief

**Your personalized daily standup in one message.**

Automatically compiles and sends a morning brief with:
- **Today's calendar** - All meetings with attendee counts and back-to-back warnings
- **Tasks due today** - From Google Tasks, including overdue items
- **Email highlights** - Unreplied emails needing your attention (from last 96 hours)
- **Slack highlights** - @mentions, threads you're in with new replies
- **AI & Enterprise news** - Latest from Anthropic, OpenAI, Google Gemini, AI agents space

**Use case**: Start your day knowing exactly what's ahead without checking 5 different apps.

**Example output:**
```
🌅 Good morning, Shiv — Monday, May 19, 2026

📅 Today's Calendar (13 meetings)
• 9:00 AM - 9:30 AM — Unified Conversational API SoS (48 people)
• 9:30 AM - 10:00 AM — SOMA Beta - SoS (76 people)
⚠️ Back-to-back meetings throughout the day

💬 Slack Highlights
• @mentioned in #team-channel by Alice: "Need your input on the API design"
• New reply in thread you're in: #eng-standup — "Deployment completed"

🌐 AI & Enterprise News
• KPMG integrates Claude across 276,000+ workforce (Anthropic, May 20)
• OpenAI co-founder Andrej Karpathy joins Anthropic (Axios, May 19)
```

**How to use:**
```bash
# Run once manually
/daily-morning-brief

# Schedule to run daily at 7:30 AM
"Schedule daily-morning-brief at 7:30 AM"
```

---

### 📊 daily-customer-insights

**Stay on top of customer feedback without drowning in Slack.**

Monitors specified Slack channels and extracts:
- **🔴 Critical/Blocker feedback** - Issues needing immediate attention
- **🟡 Important feedback** - Feature requests and pain points
- **🟢 Learnings/Success** - What's working well for customers
- **❓ Open questions** - Customer questions awaiting answers

Organized by product area (Multi-Agent Orchestration, Testing Center, Retrievers, etc.)

**Use case**: Product managers can review a day's worth of customer feedback in 2 minutes instead of scrolling through dozens of Slack channels.

**Example output:**
```
📊 Daily Customer Insights — Monday, May 19, 2026

💡 Customer Feedback & Learning (8 insights)

• 🔴 Multi-Agent Orchestration [#af-multi-agent] 
  Customer blocked on agent handoff timeout — Acme Corp hitting 30s 
  timeout when orchestrating 5+ agents; no clear workaround

• 🟡 Testing Center [#agentforce-fde] 
  Request for bulk test execution — Multiple customers asking for ability 
  to run 100+ test cases in parallel

• 🟢 Retrievers [#ri-insights] 
  Success: Auto-chunking working well — Beta customer reports 90% 
  accuracy improvement after switching to auto-chunking

❓ Open Questions (3 questions)

• Multi-Agent Orchestration [#af-multi-agent] 
  Can we configure retry logic per agent? — Enterprise customer planning 
  migration, needs this clarified before commit
```

**How to use:**
```bash
# Run once manually
/daily-customer-insights

# Schedule to run daily at 7:30 AM
"Schedule daily-customer-insights at 7:30 AM"
```

---

### 🔍 daily-competitive-insights

**Automated competitive intelligence monitoring.**

Monitors a baseline competitive analysis for changes:
- Searches web for competitor announcements (last 24-48 hours)
- Classifies updates: 🔴 High-signal / 🟡 Notable / ⚪ No change
- Generates dated delta reports
- Updates baseline artifacts (MD/PDF/PPTX) when changes occur
- Sends Slack DM with summary

**Use case**: Keep your competitive analysis current without manual research. Know immediately when competitors launch features, change pricing, or announce partnerships.

**Example output:**
```
🔍 Competitive Intelligence Delta — May 19, 2026

🟡 NOTABLE DEVELOPMENTS

LangChain — LangSmith Engine Technical Deep-Dive (May 19)
• Published technical follow-up to May 13 announcement
• Signals sustained engineering investment in automated agent improvement
• Threat: Reinforces production-readiness narrative

OpenAI — Guaranteed Capacity offering (May 19)  
• Enterprise pricing/capacity commitment model for API customers
• Implications: Reduces friction for large enterprise deployments

⚪ NO MEANINGFUL CHANGE
• ServiceNow, Microsoft Copilot Studio (no verified news in 24 hours)

✅ Updated files: baseline.md, baseline.pdf (regenerated)
```

**How to use:**
```bash
# First: Create baseline with /competitive-analysis skill
/competitive-analysis competitors: LangChain, Microsoft Copilot Studio, OpenAI

# Then: Monitor for changes
/daily-competitive-insights ~/path/to/competitive-analysis.md

# Schedule daily monitoring
"Set up daily competitive insights monitoring for ~/path/to/baseline.md at 8:30 AM"
```

---

### 🏢 customer-analysis

**Deep-dive customer account analysis on demand.**

Comprehensive research across all available data sources:
- **Slack**: Mentions, threads, channel activity (last 180 days)
- **Gmail**: Email threads, meeting invites
- **Google Docs**: Meeting notes, shared documents
- **Calendar**: Past and upcoming meetings with this customer
- **External web**: Company news, funding, press releases
- **Product usage**: Based on discussions and support tickets

Outputs: Executive summary, account health, contact mapping, issues, feedback, strategic insights, full chronological timeline.

**Use case**: Before a customer meeting, get a complete picture of the relationship. For account reviews, have all context in one place. For escalations, understand the full history instantly.

**Example output:**
```
# Customer Analysis: Acme Corporation
*Generated: May 19, 2026 | Focus: Agentforce*

## Executive Summary
- **Account Health**: 🟡 At Risk (recent escalation on multi-agent timeout issues)
- **Primary Use Case**: Multi-agent orchestration for customer service automation
- **Key Contacts**: Jane Doe (VP Eng), John Smith (Tech Lead)
- **Recent Activity**: 47 Slack mentions, 12 email threads, 3 meetings in last 90 days
- **Critical Issues**: 1 P0 (30s timeout), 2 P1s (observability gaps)

## Recent Feedback
🔴 May 18: "Hitting 30s timeout with 5+ agents - blocking production rollout"
🟡 May 15: "Need better error messages in agent handoff failures"  
🟢 May 10: "Auto-chunking working great, 90% accuracy improvement"

## Strategic Insights
- Heavy investment in agentic architecture (5 engineers dedicated)
- Considering competitive options if timeout issue not resolved by June
- Strong advocate internally - CTO presented at their all-hands

[... full timeline, contact details, meeting notes ...]
```

**How to use:**
```bash
# Run on demand
/customer-analysis

# Claude will prompt you for:
# - Customer name
# - Product focus area (optional)

# Example:
"Run customer analysis for Acme Corporation, focus on Agentforce"
```

---

## 🚀 Quick Setup (5 minutes)

### 1. Configure Your Settings

Add these to `~/.claude/settings.json` under the `"env"` section:

```json
{
  "env": {
    "SKILLS_USER_EMAIL": "your.email@company.com",
    "SKILLS_SLACK_USER_ID": "YOUR_SLACK_USER_ID",
    "SKILLS_CUSTOMER_INSIGHT_CHANNELS": "#channel1,#channel2,#channel3"
  }
}
```

**Finding your values:**

| Config | How to Find It |
|--------|----------------|
| **SKILLS_USER_EMAIL** | Your work email address |
| **SKILLS_SLACK_USER_ID** | Slack → Your profile → ⋯ → Copy member ID (e.g., `U_REDACTED`) |
| **SKILLS_CUSTOMER_INSIGHT_CHANNELS** | Comma-separated Slack channels to monitor (e.g., `#customer-feedback,#support-escalations`) |

### 2. Grant Permissions

First time you run each skill, you'll be prompted to allow:
- ✅ Google Calendar access
- ✅ Google Tasks access
- ✅ Gmail search (for email highlights)
- ✅ Slack search and messaging
- ✅ Web fetching (for news)

Approve once and they're saved automatically.

**Optional: Auto-approve all** (add to `settings.json`):

```json
{
  "permissions": {
    "allow": [
      "mcp__plugin_google_google__calendar_events",
      "mcp__plugin_google_google__tasks_list",
      "mcp__plugin_slack_slack__slack_search_public_and_private",
      "mcp__plugin_slack_slack__slack_send_message",
      "WebFetch",
      "Bash(curl *)",
      "Write(/tmp/**)"
    ]
  }
}
```

### 3. Schedule Your Daily Briefs

```bash
# Morning brief at 7:30 AM
"Schedule daily-morning-brief at 7:30 AM"

# Customer insights at 7:30 AM  
"Schedule daily-customer-insights at 7:30 AM"

# Competitive intel at 8:30 AM
"Schedule daily-competitive-insights at 8:30 AM"
```

Done! You'll now receive daily automation without lifting a finger.

---

## 🔒 Sharing These Skills (Git Setup)

If you're publishing these skills to a public repository, **remove proprietary information first**.

### Create .gitignore

```gitignore
# Personal configuration - DO NOT COMMIT
**/*personal*.md
**/*config*.md

# Skill files with hardcoded values
# (Users will need to configure these themselves per README)
```

### Before Committing Skills

**Option A**: Replace hardcoded values with placeholders

In each `SKILL.md`, replace:
```markdown
❌ Send the composed brief as a Slack DM to U_REDACTED.
✅ Send the composed brief as a Slack DM to ${SKILLS_SLACK_USER_ID}.

❌ --to "your.email@example.com" \
✅ --to "${SKILLS_USER_EMAIL}" \

❌ - `#your-customer-channel-1`
✅ - Channels from ${SKILLS_CUSTOMER_INSIGHT_CHANNELS}
```

**Option B**: Keep a local copy outside git

```bash
# Keep your configured skills in ~/.claude/skills/ (NOT in git)
~/.claude/skills/daily-morning-brief/SKILL.md

# Commit template versions to git repo
~/projects/my-skills-repo/daily-morning-brief/SKILL.template.md
```

### What to Commit

✅ **DO commit:**
- README.md (this file)
- SKILL.md files with placeholders or environment variable references
- Example configuration snippets

❌ **DON'T commit:**
- Your personal email address
- Your Slack user ID
- Internal Slack channel names
- Company-specific information
- Any PII or proprietary data

---

## 📚 Advanced Usage

### Customizing Channels

Edit the `SKILLS_CUSTOMER_INSIGHT_CHANNELS` list to monitor your specific channels:

```json
"SKILLS_CUSTOMER_INSIGHT_CHANNELS": "#eng-support,#customer-success,#product-feedback,#field-insights"
```

### Customizing News Topics

Edit `daily-morning-brief/SKILL.md` → "News topics" section to track your competitors:

```markdown
| News topics | Your Product, Competitor A, Competitor B, Industry Keywords |
```

### Multiple Competitive Baselines

Monitor different competitive landscapes:

```bash
# Monitor AI agent platforms
/daily-competitive-insights ~/docs/ai-agents-competitive.md

# Monitor data platforms
/daily-competitive-insights ~/docs/data-platforms-competitive.md
```

Schedule each separately for different times.

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| **"Slack DM failed"** | Verify `SKILLS_SLACK_USER_ID` matches your Slack member ID |
| **"Email unavailable"** | Connect Gmail in AI Expert Suite settings (requires Google Workspace integration) |
| **"Permission prompt every run"** | Add the MCP tool to `permissions.allow` in settings.json (see setup section) |
| **"No calendar events"** | Ensure Google Calendar is connected via AI Expert Suite |
| **"Channel not found"** | Check channel names in `SKILLS_CUSTOMER_INSIGHT_CHANNELS` (include #) |
| **Scheduled skills not running** | Check cron jobs with `claude crons` - they auto-expire after 7 days |

---

## 📝 Contributing

If you improve these skills or add new ones:

1. **Remove proprietary data** before committing
2. **Use environment variables** for personal config
3. **Update README.md** with clear descriptions and examples
4. **Test with fresh config** to ensure new users can set it up

---

## 📄 License

These skills are provided as-is for personal and commercial use. Customize freely for your workflows.

---

## 🙏 Credits

Built with [Claude Code](https://claude.ai/code) using:
- MCP (Model Context Protocol) for Google Calendar, Gmail, Slack integration
- Web scraping for competitive intelligence
- Automated scheduling with cron jobs
