---
name: daily-customer-insights
description: Compile and send daily customer insights from key Agentforce Slack channels. Aggregates customer feedback, learnings, and open questions from the last 48 hours into a categorized digest sent to Slack. Use for daily customer intelligence briefings.
argument-hint: "[optional: date override or focus area]"
---

# Daily Customer Insights

> Compile customer insights from key Agentforce Slack channels and send as a daily digest to Slack.

## Usage

```
/daily-customer-insights
/daily-customer-insights 2026-05-14
/daily-customer-insights focus: multi-agent orchestration
```

## Workflow

### 1. Gather Insights from Slack Channels

Search the following channels for messages from the last 48 hours:
- `#your-customer-channel-1`
- `#your-customer-channel-2`
- `#your-customer-channel-3`
- `#your-customer-channel-4`

**Search strategy:**
1. Use `slack_search_public_and_private` to search each channel for messages from the last 48 hours
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
- If a thread has new replies in the last 48 hours, check if the new replies add net new information (answers, resolutions, workarounds, additional context)
- Highlight threads where the latest update materially changes the insight or provides resolution

### 2. Categorize Insights

Sort insights into two categories:

#### Customer Feedback / Learning (max 10 items)
Insights that teach us something new about:
- How customers are using Agentforce
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

#### Open Questions (max 10 items)
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
- **Thread update flag**: If new replies in last 48h added material information, note it: _(updated: resolution found)_ or _(updated: workaround posted)_

**Example format:**
```
• 🔴 [#af-multi-agent-orchestration] <https://salesforce.slack.com/...|Customer blocked on agent handoff timeout> — Acme Corp hitting 30s timeout when orchestrating 5+ agents; no clear workaround _(updated: eng team investigating)_
```

### 4. Compose the Slack Message

Structure the Slack DM as follows. Use Slack markdown (*bold*, _italic_, bullet points). Keep it scannable — focus on actionable insights.

---

```
📊 *Daily Customer Insights — [WEEKDAY], [DATE]*

━━━━━━━━━━━━━━━━━━━━━━━━

💡 *Customer Feedback & Learning* ([N] insights)

• 🔴 [#channel] <[SLACK_PERMALINK]|[Brief title/summary]> — [Detail with customer context]
• 🟡 [#channel] <[SLACK_PERMALINK]|[Brief title/summary]> — [Detail with customer context]
• 🟢 [#channel] <[SLACK_PERMALINK]|[Brief title/summary]> — [Detail with customer context] _(updated: [new info])_
• [continue up to 10 items]

> [N] total insights from [N] channels · Covering last 48 hours

━━━━━━━━━━━━━━━━━━━━━━━━

❓ *Open Questions* ([N] questions)

• [#channel] <[SLACK_PERMALINK]|[Question summary]> — [Context about who's asking and why] _(awaiting response)_
• [#channel] <[SLACK_PERMALINK]|[Question summary]> — [Context about who's asking and why]
• [continue up to 10 items]

> [N] questions need answers · [N] customer-blocking

━━━━━━━━━━━━━━━━━━━━━━━━

_Insights compiled from:_
_• #your-customer-channel-1_
_• #your-customer-channel-2_
_• #your-customer-channel-3_
_• #your-customer-channel-4_
```

---

### 5. Send the DM

Send the composed insights digest as a Slack DM to U_REDACTED.

### 6. Confirm

After sending, confirm to the user in the Claude Code terminal:
"Daily customer insights sent to Slack ✓ — [N] feedback/learnings, [N] open questions from [N] channels."

## Configuration

These defaults can be overridden by the user at invocation time:

| Setting | Default |
|---|---|
| Slack recipient | U_REDACTED (Shiv Ramanna) |
| Channels | #your-customer-channel-1, #your-customer-channel-2, #your-customer-channel-3, #your-customer-channel-4 |
| Lookback window | 48 hours |
| Max insights per category | 10 |
| Categories | Customer Feedback/Learning, Open Questions |

## Scheduling

To run this automatically every morning, ask Claude Code:
> "Schedule daily-customer-insights daily at 7:30 AM"

This will create a durable cron job that fires the skill each weekday morning.

## Tips

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
