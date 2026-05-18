---
name: morning-brief
description: Compile and send a personalized morning brief to Slack. Aggregates calendar, tasks, email highlights, Slack highlights and news into a single daily digest. Use when setting up a daily standup summary, briefing for the day ahead, or a scheduled morning update.
argument-hint: "[optional: focus areas or date override]"
---

# Morning Brief

> Compile a personalized morning brief and send it to Slack as a DM.

## Usage

```
/morning-brief
/morning-brief focus: product strategy
/morning-brief 2026-05-14
```

## Workflow

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
1. Search for unread messages in inbox from last 96 hours: `query="is:unread in:inbox newer_than:4d" max_results=20`
2. Search for recent inbox messages (read or unread) from last 96 hours: `query="in:inbox newer_than:4d" max_results=30`
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
1. Unread unreplied messages (oldest first within last 96 hours)
2. Read unreplied messages that need action (oldest first)
3. Show age of message (e.g., "3 days old") for items > 24 hours

**Email links:**
- Use Gmail message IDs from search results to create clickable links
- Format: `<https://mail.google.com/mail/u/0/#inbox/MESSAGE_ID|"Subject">` in Slack markdown
- This creates a clickable subject line that opens the email directly in Gmail

Summarize each email in 1 line max — never quote full body. If Gmail is not connected, note "Email unavailable — Gmail not connected in AI Expert Suite" and skip the section.

#### Slack Highlights
- Search for unread mentions (@your.handle) from the past 48 hours
- Search for threads where you've participated and there have been new replies in the last 48 hours (use `from:me` to find your messages, then check for newer replies in those threads)
- Search for messages in key channels that need a response
- Search for saved messages from the past 48 hours
- Surface any urgent threads or decisions pending
- Prioritize: direct @mentions > active threads with new replies > saved items > general discussions
- **Include permalink URLs** from search results to make each Slack highlight clickable


#### News / Web
- Search for top AI agent / enterprise software news from external sources (last 48 hours)
- Filter for relevance to: Agentforce, Claude, Gemini, OpenAI, AI Agents, ServiceNow Agent Assist, Anthropic
- Max 5 items — signal only, no filler
- **Search strategy:**
  - Use Google News RSS feeds via curl: `https://news.google.com/rss/search?q=QUERY+when:2d&hl=en-US&gl=US&ceid=US:en`
  - Search queries: "Claude AI", "Anthropic", "OpenAI", "Google Gemini", "AI agents", "Agentforce Salesforce", "ServiceNow Agent"
  - Parse RSS XML to extract: title, link, pubDate, source
  - Filter by relevance and deduplicate across queries
  - Prioritize: product launches, major features, partnerships, research papers, competitive moves
  - Skip: opinion pieces, minor updates, marketing fluff
  - For each item: `<[URL]|[Source] — [Headline]>: [1-line summary]`
  - If no relevant news found, note "No significant news in the last 48 hours"

### 2. Compose the Brief

Structure the Slack DM as follows. Use Slack markdown (*bold*, _italic_, bullet points). Keep each section tight — the brief should be readable in under 2 minutes.

---

```
🌅 *Good morning, Shiv — [WEEKDAY], [DATE]*

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

Send the composed brief as a Slack DM to U_REDACTED.

### 4. Confirm

After sending, confirm to the user in the Claude Code terminal:
"Morning brief sent to Slack ✓ — [N] meetings, [N] tasks, [N] emails ([N] need response), [N] Slack highlights, [N] news items."

## Configuration

These defaults can be overridden by the user at invocation time:

| Setting | Default |
|---|---|
| Slack recipient | U_REDACTED (Shiv Ramanna) |
| Calendar lookahead | Today only |
| Slack lookback | 48 hours |
| Slack criteria | Direct @mentions, threads you've participated in with new replies, saved messages, key channel discussions |
| News topics | Agentforce, Claude, Gemini, OpenAI, AI Agents, ServiceNow Agent Assist, Anthropic |
| News timeframe | 48 hours |
| News sources | External web search (news sites, blogs, official announcements) |
| Max news items | 5 |
| Email lookback | 96 hours |
| Email source | Gmail via MCP plugin (requires Google connected in AI Expert Suite) |
| Email criteria | Unreplied emails from last 96 hours (unread prioritized first) |
| Max email highlights | 10 (🔴 must-respond first, then ℹ️ FYI) |

## Scheduling

To run this automatically every morning, ask Claude Code:
> "Schedule morning-brief daily at 7:30 AM"

This will create a durable cron job that fires the skill each weekday morning.

## Tips

- If Google Calendar or Tasks are not connected, skip those sections gracefully and note they are unavailable
- If Slack search returns too many results, prioritize: direct @mentions > DMs > channels the user owns or manages > all others
- Never include raw message content that looks sensitive — summarize rather than quote verbatim for long messages
- The brief should surface decisions and prep needed, not just list events — add context where useful
# If it's a weekend and there are no meetings or tasks, keep the brief short: news
- For email: prioritize 🔴 "must respond" (unreplied emails) over ℹ️ FYI — show unread unreplied first, then read unreplied, ordered by age (oldest first) — if there are more than 10, show the top 10 and truncate the rest with "…and [N] more"
- To enable email: connect Google account in AI Expert Suite settings (must have Gmail API access)
