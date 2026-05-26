---
name: daily-morning-brief
description: Compile and send a personalized morning brief to Slack. Aggregates calendar, tasks, email highlights, Slack highlights and news into a single daily digest. Use when setting up a daily standup summary, briefing for the day ahead, or a scheduled morning update.
argument-hint: "[optional: focus areas or date override]"
---

# Daily Morning Brief

> Compile a personalized morning brief and send it to Slack as a DM.

## Usage

```
/daily-morning-brief
/daily-morning-brief focus: product strategy
/daily-morning-brief 2026-05-14
```

## Workflow

### 0. Load Configuration

Before doing any work, read `~/.claude/skills/config.json` and extract:
- `user.slack_user_id` — Slack DM recipient
- `user.slack_handle` — used to find @-mentions
- `user.display_name` — used in greeting line
- `user.email` / `paths.send_email_script` — for email delivery (when SMTP enabled)
- `daily_morning_brief.news_topics` — relevance filter for news
- `daily_morning_brief.news_search_queries` — Google News query strings
- `daily_morning_brief.news_lookback_hours` — default 48
- `daily_morning_brief.max_news_items` — default 5
- `daily_morning_brief.email_lookback_hours` — default 96
- `daily_morning_brief.max_email_highlights` — default 10
- `daily_morning_brief.slack_lookback_hours` — default 48

If `config.json` is missing, error out: "Missing ~/.claude/skills/config.json. Copy config.example.json and fill in your values."

References to `{{config.X}}` in this document mean: substitute the value loaded from config.json.

### 1. Gather Inputs (run in parallel)

Collect from all available sources simultaneously:

#### Calendar
- Fetch today's calendar events from Google Calendar
- Note: meeting titles, times, attendees, and any attached docs/agendas
- Flag back-to-back blocks and any prep needed

#### Tasks
- Fetch open tasks from Google Tasks
- Surface anything due today or overdue
- Note tasks marked high priority

#### Email Highlights
Use Gmail MCP plugin to fetch emails (requires Google to be connected via AI Expert Suite):

**Search strategy:**
1. Search for unread messages in inbox from last `{{config.daily_morning_brief.email_lookback_hours}}` hours: `query="is:unread in:inbox newer_than:Xd" max_results=20` (X = email_lookback_hours / 24)
2. Search for recent inbox messages (read or unread) from same window: `query="in:inbox newer_than:Xd" max_results=30`
   - Includes emails from yourself (self-sent reminders)
   - Searches across all Gmail categories (primary, social, promotions, personal, etc.)
3. For each message, check if it's likely a thread you've replied to by searching for your replies in that conversation
4. Flag messages that appear unreplied and need response

**Filtering logic:**
- **Must respond (🔴)**: 
  - Self-sent emails (reminders to yourself)
  - Unread messages where you are directly addressed or in To: field
  - Messages with question marks or action-oriented language ("can you", "please review", "need your input")
  - Messages where you are the only/primary recipient
  - Threads where the latest message is not from you and appears to expect a response
- **FYI / no action (ℹ️)**: 
  - Threads where you are CC'd
  - Broadcast updates, announcements
  - Threads where you already replied (excluding self-sent reminders)
- **Skip**: 
  - Automated notifications (GitHub, JIRA, calendar invites, CI/CD)
  - Newsletters and marketing emails
  - Social media notifications
  - Non-self-sent messages you've already replied to

**Priority order:**
1. Unread unreplied messages (oldest first within the email lookback window)
2. Read unreplied messages that need action (oldest first)
3. Show age of message (e.g., "3 days old") for items > 24 hours

**Email links:**
- Use Gmail message IDs from search results to create clickable links
- Format: `<https://mail.google.com/mail/u/0/#inbox/MESSAGE_ID|"Subject">` in Slack markdown
- This creates a clickable subject line that opens the email directly in Gmail

Summarize each email in 1 line max — never quote full body. If Gmail is not connected, note "Email unavailable — Gmail not connected in AI Expert Suite" and skip the section.

#### Slack Highlights
- Search for unread mentions (`@{{config.user.slack_handle}}`) from the past `{{config.daily_morning_brief.slack_lookback_hours}}` hours
- Search for threads where you've participated and there have been new replies in the same window (use `from:me` to find your messages, then check for newer replies in those threads)
- Search for messages in key channels that need a response
- Search for saved messages from the past lookback window
- Surface any urgent threads or decisions pending
- Prioritize: direct @mentions > active threads with new replies > saved items > general discussions
- **Include permalink URLs** from search results to make each Slack highlight clickable


#### News / Web
- Search for top AI agent / enterprise software news from external sources (last `{{config.daily_morning_brief.news_lookback_hours}}` hours)
- Filter for relevance to topics in `{{config.daily_morning_brief.news_topics}}`
- Max `{{config.daily_morning_brief.max_news_items}}` items — signal only, no filler
- **Search strategy:**
  - Use Google News RSS feeds via curl: `https://news.google.com/rss/search?q=QUERY+when:2d&hl=en-US&gl=US&ceid=US:en`
  - Use the search queries from `{{config.daily_morning_brief.news_search_queries}}`
  - Parse RSS XML to extract: title, link, pubDate, source
  - Filter by relevance and deduplicate across queries
  - Prioritize: product launches, major features, partnerships, research papers, competitive moves
  - Skip: opinion pieces, minor updates, marketing fluff
  - For each item: `<[URL]|[Source] — [Headline]>: [1-line summary]`
  - If no relevant news found, note "No significant news in the lookback window"

### 2. Compose the Brief

Structure the Slack DM as follows. Use Slack markdown (*bold*, _italic_, bullet points). Keep each section tight — the brief should be readable in under 2 minutes.

---

```
🌅 *Good morning, [config.user.display_name] — [WEEKDAY], [DATE]*

━━━━━━━━━━━━━━━━━━━━━━━━

📅 *Today's Calendar* ([N] meetings)
• [TIME] — [Meeting title] _([attendee count] people)_
• [TIME] — [Meeting title]
> ⚠️ [Flag if back-to-back, no prep time, or missing agenda]

━━━━━━━━━━━━━━━━━━━━━━━━

✅ *Tasks Due Today* ([N] items)
• [Task name] — [due context]
• [Task name]
> ⚠️ [N] overdue items need attention

━━━━━━━━━━━━━━━━━━━━━━━━

📧 *Email Highlights* ([N] need response)
• 🔴 *[Sender]* — <https://mail.google.com/mail/u/0/#inbox/[MESSAGE_ID]|"[Subject]"> _(unreplied, [X] days old)_
• 🔴 *[Sender]* — <https://mail.google.com/mail/u/0/#inbox/[MESSAGE_ID]|"[Subject]"> _(unreplied, unread)_
• ℹ️ *[Sender]* — <https://mail.google.com/mail/u/0/#inbox/[MESSAGE_ID]|"[Subject]"> _(FYI, no action needed)_
> Showing unreplied emails from last 96 hours · Email unavailable if Gmail not connected

━━━━━━━━━━━━━━━━━━━━━━━━

💬 *Slack Highlights*
• <[SLACK_PERMALINK]|@mentioned in #[channel]> by [person]: "[brief quote]"
• 💬 <[SLACK_PERMALINK]|New reply in thread you're in: #[channel]> — "[context]"
• <[SLACK_PERMALINK]|[Thread needing response in #channel]>
> [N] unread mentions · [N] active threads with updates

━━━━━━━━━━━━━━━━━━━━━━━━

🌐 *AI & Enterprise News* (last 48 hours)
• <[URL]|[Source] — [Headline]>: [1-line summary]
• <[URL]|[Source] — [Headline]>: [1-line summary]
• <[URL]|[Source] — [Headline]>: [1-line summary]
• <[URL]|[Source] — [Headline]>: [1-line summary]
• <[URL]|[Source] — [Headline]>: [1-line summary]

━━━━━━━━━━━━━━━━━━━━━━━━

_Have a great day_ 🚀
```

---

### 3. Send the DM

Send the composed brief as a Slack DM to `{{config.user.slack_user_id}}`.

<!--
### 4. Send Email

TEMPORARILY DISABLED - SMTP configuration blocked by Google Workspace 2FA restrictions.
To re-enable: configure SMTP relay or enable app passwords.

Convert the Slack-formatted message to plain text (remove Slack markdown) and send via email:

```bash
python3 {{config.paths.send_email_script}} \
  --to "{{config.user.email}}" \
  --subject "Morning Brief — [WEEKDAY], [DATE]" \
  --body "[Plain text version of the brief]"
```

**Plain text format** (convert from Slack format):
- Remove Slack markdown (*bold* → plain, _italic_ → plain)
- Convert Slack links `<URL|text>` → `text: URL`
- Keep structure: headers, bullets, sections
- Include all sections: Calendar, Tasks, Email, Slack, News
-->

### 4. Confirm

After sending Slack DM, confirm to the user in the Claude Code terminal:
"Morning brief sent ✓ — Slack DM — [N] meetings, [N] tasks, [N] emails ([N] need response), [N] Slack highlights, [N] news items."

## Configuration

All defaults are loaded from `~/.claude/skills/config.json`. To change recipient, news topics, lookback windows, etc., edit that file.

| Setting | Config key |
|---|---|
| Slack recipient | `user.slack_user_id` |
| Slack handle (for @-mentions) | `user.slack_handle` |
| Display name (greeting) | `user.display_name` |
| Email recipient | `user.email` |
| News topics | `daily_morning_brief.news_topics` |
| News search queries | `daily_morning_brief.news_search_queries` |
| News lookback | `daily_morning_brief.news_lookback_hours` |
| Max news items | `daily_morning_brief.max_news_items` |
| Email lookback | `daily_morning_brief.email_lookback_hours` |
| Max email highlights | `daily_morning_brief.max_email_highlights` |
| Slack lookback | `daily_morning_brief.slack_lookback_hours` |

## Scheduling

To run this automatically every morning, ask Claude Code:
> "Schedule daily-morning-brief daily at 7:30 AM"

This will create a durable cron job that fires the skill each weekday morning.

## Tips

- If Google Calendar or Tasks are not connected, skip those sections gracefully and note they are unavailable
- If Slack search returns too many results, prioritize: direct @mentions > DMs > channels the user owns or manages > all others
- Never include raw message content that looks sensitive — summarize rather than quote verbatim for long messages
- The brief should surface decisions and prep needed, not just list events — add context where useful
# If it's a weekend and there are no meetings or tasks, keep the brief short: news
- For email: prioritize 🔴 "must respond" (unreplied emails) over ℹ️ FYI — show unread unreplied first, then read unreplied, ordered by age (oldest first) — if there are more than 10, show the top 10 and truncate the rest with "…and [N] more"
- To enable email: connect Google account in AI Expert Suite settings (must have Gmail API access)
