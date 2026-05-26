# Configuration Guide

All personal/organization-specific values for these skills live in **one** file: `~/.claude/skills/config.json`. The skills read it at the top of their workflow.

## First-time setup

1. Copy the template:
   ```bash
   cp ~/.claude/skills/config.example.json ~/.claude/skills/config.json
   ```
2. Edit `config.json` and replace every value with your own.
3. Verify `.gitignore` is excluding `config.json` (it is, by default).

## What's in config.json

```jsonc
{
  "user": {
    "email": "your.email@company.com",      // Used for SMTP delivery (when enabled)
    "slack_user_id": "U1234567890",         // Slack DM recipient
    "slack_handle": "your.handle",          // Used to find @-mentions
    "display_name": "Your First Name"       // Used in greeting line
  },
  "paths": {
    "send_email_script": "/Users/YOU/.claude/scripts/send_email.py",
    "competitive_baseline_dir": "/Users/YOU/Documents/Work/CompetitiveAnalysis",
    "customer_analysis_output_dir": "~/Documents/Work/CustomerAnalysis"
  },
  "daily_customer_insights": {
    "channels": ["#channel-1", "#channel-2"],
    "lookback_hours": 48,
    "max_insights_per_category": 10,
    "domain": "YourProduct",
    "product_areas": ["Area 1", "Area 2"]
  },
  "daily_morning_brief": {
    "news_topics": ["YourProduct", "Competitor X"],
    "news_search_queries": ["YourProduct news", "Competitor X"],
    "news_lookback_hours": 48,
    "max_news_items": 5,
    "email_lookback_hours": 96,
    "max_email_highlights": 10,
    "slack_lookback_hours": 48
  },
  "daily_competitive_insights": {
    "default_baseline_path": "/Users/YOU/Documents/Work/competitive-baseline.md",
    "research_lookback_hours": 48
  },
  "customer_analysis": {
    "default_product_areas": ["Your default product area"]
  }
}
```

## Where each value is used

| Config key | Skills that read it |
|---|---|
| `user.email` | daily-morning-brief, daily-customer-insights, daily-competitive-insights (SMTP-disabled) |
| `user.slack_user_id` | daily-customer-insights, daily-morning-brief, daily-competitive-insights |
| `user.slack_handle` | daily-morning-brief (find @-mentions) |
| `user.display_name` | daily-morning-brief (greeting line) |
| `paths.send_email_script` | All three daily skills (SMTP-disabled) |
| `paths.competitive_baseline_dir` | daily-competitive-insights |
| `paths.customer_analysis_output_dir` | customer-analysis (output location) |
| `daily_customer_insights.channels` | daily-customer-insights |
| `daily_customer_insights.domain` | daily-customer-insights |
| `daily_customer_insights.product_areas` | daily-customer-insights |
| `daily_morning_brief.news_topics` | daily-morning-brief |
| `daily_morning_brief.news_search_queries` | daily-morning-brief |
| `daily_morning_brief.*_lookback_hours` | daily-morning-brief |
| `daily_competitive_insights.default_baseline_path` | daily-competitive-insights (default arg) |
| `customer_analysis.default_product_areas` | customer-analysis (suggestions in Step 0) |

## How to find your Slack user ID

1. Open Slack
2. Click your profile picture
3. View profile → ⋯ menu → "Copy member ID"
4. Format: `U` + 10 alphanumeric characters

## Notes

- Email sending is currently disabled in all skills (SMTP path commented out). The email values are still loaded so re-enabling later requires only uncommenting the relevant block.
- The competitive-analysis skill (one-time research) does not currently read config — it's parameter-driven. competitive-analysis-related cron prompts may still reference your local paths inline; those need to be regenerated if your `competitive_baseline_dir` changes.
- After editing `config.json`, run any of the skills once to verify it loads correctly.

## Security

✅ `config.json` is in `.gitignore` — never committed
✅ `config.example.json` IS committed — sanitized template only

❌ Never put real Slack IDs, emails, or paths into `config.example.json`
