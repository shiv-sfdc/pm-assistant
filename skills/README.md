# Daily Automation Skills for Product Managers & Engineers

A collection of Claude Code skills that automate your daily information gathering and reporting workflows. Get personalized briefs, customer insights, and competitive intelligence delivered to Slack automatically.

---

## 🚀 Quick Setup (5 minutes)

### Step 1 — Create your `config.json`

This skill bundle ships with a single configuration file that holds your Slack ID, email, channel list, news topics, baseline paths, etc. Your real `config.json` is **gitignored** — only the sanitized example file is committed.

```bash
# From the skills directory:
cp config.example.json config.json
```

Then open `config.json` and replace every fictional value (the example uses a fictional "Jane Doe" at "Acme AI" working on a product called "Pulse") with your own.

**Required fields:**

| Field | Description | How to find it |
|---|---|---|
| `user.email` | Your work email | — |
| `user.slack_user_id` | Your Slack member ID | Slack → profile → ⋯ → Copy member ID. Format: `U` + 10 alphanumeric chars |
| `user.slack_handle` | Your Slack handle (no `@`) | Used to find @-mentions of you |
| `user.display_name` | Your first name (used in greetings) | — |
| `paths.competitive_baseline_dir` | Where competitive analysis files live | Folder where you keep `competitive-*.md` baselines |
| `paths.customer_analysis_output_dir` | Where customer-analysis reports get saved | Folder for per-account deep-dives |
| `daily_customer_insights.channels` | Slack channels to monitor for customer feedback | Comma-separated channel names with `#` |
| `daily_customer_insights.domain` | Your product/business domain (used in skill phrasing) | e.g., "YourProduct" |
| `daily_customer_insights.product_areas` | Tags used to categorize insights | List of feature areas |
| `daily_morning_brief.news_topics` | Topics to filter news by | Your products + competitors |
| `daily_morning_brief.news_search_queries` | Google News query strings | Phrases to actually pass to news search |
| `daily_competitive_insights.default_baseline_path` | Baseline used when no path argument is passed | Full path to your main competitive analysis MD |

For the full schema and what each setting controls, see [`CONFIGURATION_GUIDE.md`](CONFIGURATION_GUIDE.md).

> ⚠️ **Do not commit `config.json`.** It's already in `.gitignore`, but if you fork this repo, double-check by running `git status` and confirming `config.json` does not appear. Only `config.example.json` should be tracked.

### Step 2 — Grant Permissions

The first time you run each skill, you'll be prompted to allow:
- ✅ Google Calendar access
- ✅ Google Tasks access
- ✅ Gmail search (for email highlights)
- ✅ Slack search and messaging
- ✅ Web fetching (for news)

Approve once and they're saved automatically.

**Optional: Auto-approve all** (add to `~/.claude/settings.json`):

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

### Step 3 — Schedule Your Daily Briefs

```bash
# Morning brief at 7:30 AM
"Schedule daily-morning-brief at 7:30 AM"

# Customer insights at 7:30 AM  
"Schedule daily-customer-insights at 7:30 AM"

# Competitive intel at 8:30 AM
"Schedule daily-competitive-insights at 8:30 AM"
```

Done! You'll now receive daily automation without lifting a finger.

> Cron jobs auto-expire after 7 days. Each scheduled skill will also schedule a one-shot reminder to renew it.

---

## 📋 Available Skills

### 🌅 daily-morning-brief

**Your personalized daily standup in one message.**

Automatically compiles and sends a morning brief with:
- **Today's calendar** — All meetings with attendee counts and back-to-back warnings
- **Tasks due today** — From Google Tasks, including overdue items
- **Email highlights** — Unreplied emails needing your attention (configurable lookback)
- **Slack highlights** — @mentions, threads you're in with new replies
- **News** — Filtered by topics from `config.daily_morning_brief.news_topics`

**Use case**: Start your day knowing exactly what's ahead without checking 5 different apps.

```bash
/daily-morning-brief
"Schedule daily-morning-brief at 7:30 AM"
```

---

### 📊 daily-customer-insights

**Stay on top of customer feedback without drowning in Slack.**

Monitors the channels listed in `config.daily_customer_insights.channels` and extracts:
- **🔴 Critical/Blocker feedback** — Issues needing immediate attention
- **🟡 Important feedback** — Feature requests and pain points
- **🟢 Learnings/Success** — What's working well for customers
- **❓ Open questions** — Customer questions awaiting answers

Organized by product area (configurable in `config.daily_customer_insights.product_areas`).

**Use case**: Product managers can review a day's worth of customer feedback in 2 minutes instead of scrolling through dozens of Slack channels.

```bash
/daily-customer-insights
"Schedule daily-customer-insights at 7:30 AM"
```

---

### 🔍 daily-competitive-insights

**Automated competitive intelligence monitoring.**

Monitors a baseline competitive analysis for changes:
- Searches web for competitor announcements (configurable lookback)
- Classifies updates: 🔴 High-signal / 🟡 Notable / ⚪ No change
- Generates dated delta reports
- Updates baseline artifacts (MD/PDF/PPTX) when changes occur
- Sends Slack DM with summary

**Use case**: Keep your competitive analysis current without manual research. Know immediately when competitors launch features, change pricing, or announce partnerships.

```bash
# First: Create baseline with /competitive-analysis skill
/competitive-analysis competitors: CompetitorA, CompetitorB

# Then: Monitor for changes (uses default_baseline_path from config.json)
/daily-competitive-insights

# Or override the baseline
/daily-competitive-insights ~/path/to/another-baseline.md

# Schedule daily monitoring
"Schedule daily-competitive-insights at 8:30 AM"
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

Output saved to `config.paths.customer_analysis_output_dir`.

**Use case**: Before a customer meeting, get a complete picture of the relationship. For account reviews, have all context in one place. For escalations, understand the full history instantly.

```bash
/customer-analysis "Acme Corporation"
# Claude will prompt you for product focus area
```

---

### 📈 competitive-analysis

**One-time competitive landscape research.** Use to create the initial baseline that `daily-competitive-insights` then monitors. Parameter-driven; does not currently read `config.json`.

```bash
/competitive-analysis competitors: CompetitorA, CompetitorB
/competitive-analysis feature-area: agent-orchestration
```

---

## 🔒 Sharing These Skills (Git Setup)

This repo already has the right `.gitignore` rules. To re-verify:

```bash
# config.json should be ignored
git check-ignore -v config.json
# → .gitignore:2:config.json    config.json

# config.example.json should NOT be ignored
git check-ignore config.example.json
# → (no output = tracked)
```

### What gets committed

✅ **DO commit:**
- `README.md`, `CONFIGURATION_GUIDE.md`
- `config.example.json` (fictional values only)
- All `SKILL.md` files (now config-driven, no hardcoded personal data)
- `.gitignore`

❌ **DON'T commit:**
- `config.json` (your real values — gitignored)
- Any `*personal*.md` notes
- Output files: `competitive-delta-*.md`, `customer-analysis-*.md` (also gitignored)

### Forking workflow

When someone forks this repo:
1. They clone it.
2. They `cp config.example.json config.json`.
3. They edit `config.json` with their own values.
4. Their `config.json` stays local — never pushed.

If you ever discover that real values leaked into `config.example.json`, treat it as a credentials incident: scrub the file, rotate any exposed identifiers, and force-push the corrected history.

---

## 📚 Advanced Usage

### Customizing channels, news topics, lookback windows

Edit `config.json` — that's the single source of truth. No `SKILL.md` edits required for normal customization.

### Multiple competitive baselines

`config.json` only holds one default baseline path, but you can pass any baseline as an argument:

```bash
# Default (from config.json)
/daily-competitive-insights

# Override per run
/daily-competitive-insights ~/docs/data-platforms-competitive.md
/daily-competitive-insights ~/docs/dev-tools-competitive.md
```

If you need multiple defaults, schedule each as its own cron job with the path baked into the prompt.

### Disabled email path

Email delivery is currently disabled in all skills (SMTP path is commented out). The values (`user.email`, `paths.send_email_script`) are still loaded so re-enabling later requires only uncommenting the relevant block in each `SKILL.md`.

---

## 🐛 Troubleshooting

| Issue | Solution |
|---|---|
| **"Missing ~/.claude/skills/config.json"** | Run `cp config.example.json config.json` and fill in your values |
| **"Slack DM failed"** | Verify `user.slack_user_id` in `config.json` matches your Slack member ID exactly |
| **"Email unavailable"** | Connect Gmail in AI Expert Suite settings (requires Google Workspace integration) |
| **"Permission prompt every run"** | Add the MCP tool to `permissions.allow` in `settings.json` (see Setup Step 2) |
| **"No calendar events"** | Ensure Google Calendar is connected via AI Expert Suite |
| **"Channel not found"** | Check channel names in `config.daily_customer_insights.channels` (must include `#`) |
| **Scheduled skills not running** | Check cron jobs with `claude crons` — they auto-expire after 7 days |
| **Skill picks up wrong baseline** | Update `daily_competitive_insights.default_baseline_path` in `config.json` or pass an explicit path argument |

---

## 📝 Contributing

If you improve these skills or add new ones:

1. **Use config-driven values** — never hardcode personal data in `SKILL.md` files
2. **Update `config.example.json`** with sanitized fictional values for any new settings
3. **Update `CONFIGURATION_GUIDE.md`** when adding new config keys
4. **Update README.md** with clear descriptions and examples
5. **Test with a fresh `config.json`** to ensure new users can set it up

---

## 📄 License

These skills are provided as-is for personal and commercial use. Customize freely for your workflows.

---

## 🙏 Credits

Built with [Claude Code](https://claude.ai/code) using:
- MCP (Model Context Protocol) for Google Calendar, Gmail, Slack integration
- Web scraping for competitive intelligence
- Automated scheduling with cron jobs
