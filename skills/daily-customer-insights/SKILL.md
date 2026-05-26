---
name: daily-customer-insights
description: Compile and send daily customer insights from configured Slack channels. Aggregates customer feedback, learnings, and open questions from the last 48 hours into a categorized digest sent to Slack. Use for daily customer intelligence briefings.
argument-hint: "[optional: date override or focus area]"
---

# Daily Customer Insights

> Compile customer insights from configured Slack channels and send as a daily digest to Slack.

## Usage

```
/daily-customer-insights
/daily-customer-insights 2026-05-14
/daily-customer-insights focus: multi-agent orchestration
```

## Workflow

### 0. Load Configuration

Before doing any work, read `~/.claude/skills/config.json` and extract:
- `user.slack_user_id` — Slack DM recipient
- `user.email` — email recipient (when SMTP enabled)
- `user.display_name` — used in greetings/messages
- `paths.send_email_script` — path to email script (when SMTP enabled)
- `daily_customer_insights.channels` — list of Slack channels to monitor
- `daily_customer_insights.lookback_hours` — default 48
- `daily_customer_insights.max_insights_per_category` — default 10
- `daily_customer_insights.domain` — the product/business domain (e.g., "Agentforce")
- `daily_customer_insights.product_areas` — list of product-area tags to use

If `config.json` is missing, error out: "Missing ~/.claude/skills/config.json. Copy config.example.json and fill in your values."

For the rest of this document, references to `{{config.user.slack_user_id}}`, `{{config.daily_customer_insights.channels}}`, `{{config.daily_customer_insights.domain}}`, `{{config.daily_customer_insights.product_areas}}`, etc. mean: substitute the value loaded from config.json.

### 1. Gather Insights from Slack Channels

Search every channel in `{{config.daily_customer_insights.channels}}` for messages from the last `{{config.daily_customer_insights.lookback_hours}}` hours.

**Search strategy:**
1. Use `slack_search_public_and_private` to search each channel for messages from the last lookback window
2. For each message, check if there are thread replies with new information
3. Read full thread context using `slack_read_thread` for messages with replies
4. Capture permalink URLs for all insights

**What to look for:**
- Customer feedback (positive or negative)
- New learnings about product usage, pain points, or workflows
- Feature requests or capability gaps
- Bug reports or technical issues
- Success stories or use case examples
- Questions from customers that need answers
- Support escalations or blockers
- Product insights from field teams (FDE, support engineers)
- Thread updates with new information or resolutions

**Thread monitoring:**
- If a thread has new replies in the lookback window, check if the new replies add net new information (answers, resolutions, workarounds, additional context)
- Highlight threads where the latest update materially changes the insight or provides resolution

### 2. Categorize Insights

Sort insights into two categories:

#### Customer Feedback / Learning (max `{{config.daily_customer_insights.max_insights_per_category}}` items)
Insights that teach us something new about:
- How customers are using the product (`{{config.daily_customer_insights.domain}}`)
- What's working well or not working
- Customer pain points, workarounds, or success patterns
- Feature gaps or capabilities customers need
- Technical issues or bugs encountered
- Use cases and workflows
- Product feedback from field teams

**Priority order:**
1. Critical issues or blockers affecting multiple customers
2. New learnings that change our understanding of customer needs
3. Feature requests with clear business impact
4. Success stories or positive feedback with replication potential
5. Technical bugs with workarounds

#### Open Questions (max `{{config.daily_customer_insights.max_insights_per_category}}` items)
Questions that need answers:
- Customer questions awaiting response
- Technical questions from field teams
- Clarifications needed on product capabilities
- Support questions escalated but unresolved
- Questions about roadmap, features, or timelines

**Priority order:**
1. Customer-facing questions blocking deals or adoption
2. Technical questions from field teams needing product/eng input
3. Questions with multiple customers asking similar things
4. Questions about missing documentation or unclear behavior

### 3. Format Each Insight

For each insight, include:
- **Product area**: `*[Product Area]*` at the start in bold. Choose from `{{config.daily_customer_insights.product_areas}}`. If unclear, infer from context or use "General".
- **Channel tag**: `[#channel-name]` to show source
- **Category indicator**: Emoji to signal type
  - 🔴 Critical/Blocker
  - 🟡 Important/High Priority
  - 🟢 Learning/Success
  - ❓ Open Question
  - 💬 New thread update
- **Insight summary**: 1-2 sentences capturing the key information
- **Context**: Customer name, use case, or situation (if available)
- **Link**: Slack permalink to the thread
- **Thread update flag**: If new replies in the lookback window added material information, note it: _(updated: resolution found)_ or _(updated: workaround posted)_

**Example format:**
```
• 🔴 *[Product Area]* [#channel] <https://your-workspace.slack.com/...|Customer blocked on agent handoff timeout> — Acme Corp hitting 30s timeout when orchestrating 5+ agents; no clear workaround _(updated: eng team investigating)_
```

### 4. Compose the Slack Message

Structure the Slack DM as follows. Use Slack markdown (*bold*, _italic_, bullet points). Keep it scannable — focus on actionable insights.

---

```
📊 *Daily Customer Insights — [WEEKDAY], [DATE]*

━━━━━━━━━━━━━━━━━━━━━━━━

💡 *Customer Feedback & Learning* ([N] insights)

• 🔴 *[Product Area]* [#channel] <[SLACK_PERMALINK]|[Brief title/summary]> — [Detail with customer context]
• 🟡 *[Product Area]* [#channel] <[SLACK_PERMALINK]|[Brief title/summary]> — [Detail with customer context]
• 🟢 *[Product Area]* [#channel] <[SLACK_PERMALINK]|[Brief title/summary]> — [Detail with customer context] _(updated: [new info])_
• [continue up to max_insights_per_category items]

> [N] total insights from [N] channels · Covering last [lookback_hours] hours

━━━━━━━━━━━━━━━━━━━━━━━━

❓ *Open Questions* ([N] questions)

• *[Product Area]* [#channel] <[SLACK_PERMALINK]|[Question summary]> — [Context about who's asking and why] _(awaiting response)_
• *[Product Area]* [#channel] <[SLACK_PERMALINK]|[Question summary]> — [Context about who's asking and why]
• [continue up to max_insights_per_category items]

> [N] questions need answers · [N] customer-blocking

━━━━━━━━━━━━━━━━━━━━━━━━

_Insights compiled from:_
_• [list each channel from config.daily_customer_insights.channels]_
```

---

### 5. Send the DM

Send the composed insights digest as a Slack DM to `{{config.user.slack_user_id}}`.

<!--
### 6. Send Email

TEMPORARILY DISABLED - SMTP configuration blocked by Google Workspace 2FA restrictions.
To re-enable: configure SMTP relay or enable app passwords.

Convert the Slack-formatted digest to plain text and send via email:

```bash
python3 {{config.paths.send_email_script}} \
  --to "{{config.user.email}}" \
  --subject "Daily Customer Insights — [WEEKDAY], [DATE]" \
  --body "[Plain text version of insights digest]"
```

**Plain text format**:
- Remove Slack markdown and convert links
- Preserve structure: headers, emoji indicators, bullets
- Include all insights and open questions
- Add channel list at bottom
-->

### 6. Confirm

After sending Slack DM, confirm to the user in the Claude Code terminal:
"Daily customer insights sent ✓ — Slack DM — [N] feedback/learnings, [N] open questions from [N] channels."

## Configuration

All defaults are loaded from `~/.claude/skills/config.json`. To change channels, recipient, lookback window, etc., edit that file.

| Setting | Config key |
|---|---|
| Slack recipient | `user.slack_user_id` |
| Email recipient | `user.email` |
| Channels | `daily_customer_insights.channels` |
| Lookback window | `daily_customer_insights.lookback_hours` |
| Max insights per category | `daily_customer_insights.max_insights_per_category` |
| Domain / product name | `daily_customer_insights.domain` |
| Product-area tags | `daily_customer_insights.product_areas` |

## Scheduling

To run this automatically every morning, ask Claude Code:
> "Schedule daily-customer-insights daily at 7:30 AM"

This will create a durable cron job that fires the skill each weekday morning.

## Tips

- **Product area identification**: Use the values in `{{config.daily_customer_insights.product_areas}}`. If unclear, infer from context or use "General".
- Prioritize insights that are actionable or change our understanding
- Skip routine status updates or process messages unless they contain real customer insight
- When threads have new replies, check if they resolve the issue or add meaningful context
- Look for patterns: if multiple customers mention the same issue, that's a strong signal
- Include customer names when available (helps with follow-up)
- For open questions, note if they're blocking a customer or deal
- If a channel is quiet (no relevant insights), skip it gracefully
- Deduplicate: if the same issue appears in multiple channels, consolidate into one insight
- Focus on signal over volume: 5 high-quality insights > 10 marginal ones
- Tag urgency appropriately: use 🔴 sparingly for truly critical items
- Thread updates are valuable: resolutions, workarounds, and new information deserve highlighting
