---
name: daily-competitive-insights
description: Monitor a competitive landscape report for changes and updates. Searches for news/announcements from tracked competitors, generates delta reports, updates baseline artifacts (MD/PDF/PPTX/Keynote), and sends notifications. Use for ongoing competitive intelligence monitoring after creating an initial baseline with /competitive-analysis.
argument-hint: "[baseline-file-path] [slack-user-id]"
---

# Daily Competitive Insights

> Monitor an existing competitive analysis report for changes and updates over time.

## Usage

```
/daily-competitive-insights
/daily-competitive-insights <baseline-file-path>
/daily-competitive-insights <baseline-file-path> <slack-user-id>
```

## Configuration

Before doing any work, read `~/.claude/skills/config.json` and extract:
- `user.slack_user_id` — default Slack DM recipient
- `user.email` / `paths.send_email_script` — for email delivery (when SMTP enabled)
- `daily_competitive_insights.default_baseline_path` — used when no baseline path is passed as argument
- `daily_competitive_insights.research_lookback_hours` — default 48

If `config.json` is missing, error out: "Missing ~/.claude/skills/config.json. Copy config.example.json and fill in your values."

**Arguments**:
- `baseline-file-path`: Path to the markdown file containing your competitive analysis baseline. If omitted, uses `{{config.daily_competitive_insights.default_baseline_path}}`.
- `slack-user-id`: Slack user ID to send notifications to. If omitted, uses `{{config.user.slack_user_id}}`.

## When to Use This Skill

- **After creating a competitive analysis**: Use `/competitive-analysis` first to create the initial baseline report, then use this skill to monitor for changes
- **For scheduled monitoring**: Set up a recurring cron job to run this daily/weekly
- **To track competitor movements**: Product launches, pricing changes, partnerships, funding, major announcements
- **To keep artifacts current**: Automatically updates executive summaries, PDFs, and presentations when meaningful changes occur

## Workflow

### 1. Read the Baseline

Read the specified baseline markdown file to understand:
- Which competitors are being tracked
- The current competitive snapshot and urgency levels
- Critical gaps, defensible moats, and strategic battles
- Last update date

Extract the list of competitors from the baseline (typically from a "Competitive Snapshot" table or similar section).

### 2. Research Competitor Updates

Search for news from the last `{{config.daily_competitive_insights.research_lookback_hours}}` hours (or since last run) on each tracked competitor:

**Research sources**:
- Competitor company blogs and newsrooms
- Tech news sites (TechCrunch, VentureBeat, The Verge)
- Industry analyst blogs (Gartner, Forrester)
- Developer communities (Hacker News, Reddit)
- Company-specific RSS feeds or press release pages

**What to look for**:
- Product launches and feature announcements
- Pricing or packaging changes
- Major partnerships or integrations
- Funding rounds or acquisitions
- Significant customer wins or case studies
- Developer ecosystem activity (SDK releases, API updates)
- Conference presentations or keynote announcements

**Research tips**:
- Use WebFetch to check competitor blogs directly (more reliable than search engine blocks)
- Check multiple sources to corroborate findings
- Track publication dates carefully (only capture genuinely new information)
- Note when research sources are inaccessible (document in delta report)

### 3. Generate the Delta Report

Create a dated delta file: `competitive-delta-[YYYY-MM-DD].md`

**Delta report structure**:

```markdown
---
## Competitive Intelligence Delta — [TODAY'S DATE]
*Changes since [BASELINE_DATE] baseline*

### 🔴 High-Signal Updates
[Changes that materially shift competitive threat levels or require immediate response]

For each update:
- **Competitor Name — Announcement Title** (Date)
  - What was announced and why it matters
  - **Threat**: How this affects the competitive landscape
  - **Impact**: Specific implications for your product
  - **Recommended action**: What should be done in response

### 🟡 Notable Developments
[New features, partnerships, or announcements worth tracking but not urgent]

For each development:
- **Competitor Name — Development** (Date)
  - What happened
  - **Implications**: Why this matters
  - **Relevance**: How this relates to your strategy

### ⚪ No Meaningful Change
[Competitors with no significant news in this period]

List competitors with no updates.

### Baseline Assumptions That May Need Updating
[Any findings that contradict specific claims in the baseline]

- **Original claim**: [Quote from baseline]
- **What changed**: [New information that updates or contradicts this]
- **Recommendation**: [How the baseline should be updated]

---

*Research conducted [DATE] at [TIME]. Sources: [list sources accessed]. [Note any inaccessible sources]*
```

### 4. Update Artifacts (Conditional)

**Only if there are 🔴 High-Signal Updates or 🟡 Notable Developments**, update the following:

#### 4a. Update the Baseline Markdown

Make **surgical edits** to the baseline file:
- Update competitor urgency levels in the Competitive Snapshot table (🟡 Medium → 🔴 High, etc.)
- Update the Critical Gaps or Defensible Moats sections if new information affects them
- Update strategic battle sections if competitive dynamics have shifted
- Add or update Pricing Risk entries if pricing changes
- Update the footer date to today
- Add reference to the delta file: `See competitive-delta-[DATE].md for latest changes`

**Important**: Do NOT rewrite unchanged sections. Only edit what has materially changed based on the delta findings.

#### 4b. Regenerate the PDF

If the baseline has a companion PDF (same path, `.pdf` extension), regenerate it from the updated markdown.

**PDF generation approach**:
- Use ReportLab or similar to convert markdown to PDF
- Match the original styling (fonts, colors, spacing)
- Preserve any branding (company colors, logos)
- If a PDF generation script exists (check `/tmp/generate_pdf.py` or similar), reuse it
- Otherwise, use a simple markdown-to-PDF converter

#### 4c. Regenerate Presentation Files

If PPTX or Keynote files exist (same base path with `.pptx` or `.key` extensions), regenerate them:

**PPTX regeneration**:
- Check for existing builder script (e.g., `/tmp/build_pptx.py`)
- If found, run it to regenerate from updated markdown
- If not found, reconstruct using python-pptx:
  - Match slide count and structure from original
  - Preserve branding (colors, fonts, layout)
  - 16:9 widescreen format (13.33" x 7.5")

**Keynote conversion** (optional):
- Convert PPTX to Keynote via AppleScript/Keynote automation
- May fail if Keynote not running; document in notification
- Not critical if PPTX is available

### 5. Send Slack Notification

Send a Slack DM to the specified user (or default to `{{config.user.slack_user_id}}`).

**If there were 🔴 or 🟡 updates**:

Send the full delta report formatted for Slack (use *bold* for headers, bullets for lists). End with:

```
✅ *Updated files saved locally:*
• [baseline-filename].md (updated)
• [baseline-filename].pdf (regenerated)
• [baseline-filename].pptx (regenerated)
• [baseline-filename].key (regenerated if successful)
```

**If there were NO 🔴 or 🟡 updates** (only ⚪):

Send a brief summary:

```
📊 *Competitive Intelligence Delta — [TODAY'S DATE]*
⚪ No meaningful changes today across [list competitors]. Baseline report unchanged.
```

<!--
### 6. Send Email

TEMPORARILY DISABLED - SMTP configuration blocked by Google Workspace 2FA restrictions.
To re-enable: configure SMTP relay or enable app passwords.

Send email via Python script:

**If there were 🔴 or 🟡 updates**:

```bash
python3 {{config.paths.send_email_script}} \
  --to "{{config.user.email}}" \
  --subject "Competitive Intelligence Delta — [TODAY'S DATE]" \
  --body "[Plain text version of delta report]" \
  --attach "[path-to]/competitive-delta-[YYYY-MM-DD].md" \
  --attach "[baseline-path].pdf"
```

**If there were NO 🔴 or 🟡 updates**:

```bash
python3 {{config.paths.send_email_script}} \
  --to "{{config.user.email}}" \
  --subject "Competitive Intelligence Delta — [TODAY'S DATE]" \
  --body "No meaningful changes today across [list competitors]. Baseline report unchanged."
```
-->

## Setting Up Recurring Monitoring

After running the skill once, offer to set up automated monitoring:

```
I can set up a daily/weekly cron job to run this competitive intelligence monitor automatically.

Cron jobs auto-expire after 7 days, so I'll also schedule renewal reminders to keep monitoring active.

Would you like to:
1. Daily monitoring at [suggested time]
2. Weekly monitoring (specify day/time)
3. Custom schedule
```

**Use the CronCreate tool** with:
- `durable: true` (persists across sessions)
- `recurring: true` (runs on schedule)
- Avoid :00 and :30 minute marks (spread load)
- Schedule renewal reminders every 6 days to maintain continuous monitoring

**Example cron schedule for daily at 8:23 AM**:
```
cron: "23 8 * * *"
recurring: true
durable: true
prompt: [full skill invocation with parameters]
```

## Tips for Effective Monitoring

**Research strategy**:
- Check competitor blogs/newsrooms first (most reliable, least likely to be blocked)
- Fall back to tech news aggregators if direct sources fail
- Document when sources are inaccessible so patterns can be identified
- Verify dates carefully (some sites show stale "latest news")

**Signal vs noise**:
- 🔴 High-signal: Product launches, pricing changes, major partnerships, funding that materially shifts competitive dynamics
- 🟡 Notable: New features, customer wins, minor partnerships worth tracking
- ⚪ No change: No news, or news that doesn't affect the competitive landscape

**When in doubt, be conservative**: Better to flag something as 🟡 than miss a 🔴. The user can reclassify.

**Baseline maintenance**:
- Keep surgical edits minimal to preserve baseline structure and readability
- Link delta reports in the baseline footer so history is accessible
- Consider archiving deltas older than 90 days

**Notification best practices**:
- Front-load the most important information (🔴 updates first)
- Keep Slack messages concise but include key details
- Link to files for full context
- If no updates, keep notification very brief

## Relationship to /competitive-analysis

**Use /competitive-analysis to**:
- Create the initial comprehensive competitive brief
- Do deep-dive research on a new competitor or feature area
- Produce one-time analysis for strategy decisions or board materials

**Use /daily-competitive-insights to**:
- Monitor an existing competitive landscape for changes over time
- Keep executive summaries and artifacts current with minimal manual effort
- Track competitive movements systematically
- Get alerts when competitors make significant moves

**Workflow**: Run `/competitive-analysis` once to create the baseline → run `/daily-competitive-insights` regularly to keep it current.

## Example Invocation

```bash
# Use default baseline + Slack recipient from config.json
/daily-competitive-insights

# Override the baseline path
/daily-competitive-insights ~/Documents/Work/some-other-baseline.md

# Override both baseline and Slack recipient
/daily-competitive-insights ~/Documents/Work/some-other-baseline.md U1234567890

# Set up daily monitoring (uses defaults from config.json)
"Set up daily competitive insights monitoring, running every morning at 8:30 AM"
```

## Output Files

After running, the skill generates/updates:
- `competitive-delta-[YYYY-MM-DD].md` — Dated delta report (always created)
- `[baseline-filename].md` — Updated baseline (only if 🔴/🟡 updates)
- `[baseline-filename].pdf` — Regenerated PDF (only if 🔴/🟡 updates)
- `[baseline-filename].pptx` — Regenerated presentation (only if 🔴/🟡 updates)
- `[baseline-filename].key` — Regenerated Keynote (optional, only if 🔴/🟡 updates)

All files are saved in the same directory as the baseline file.

## Error Handling

**If baseline file not found**:
- Error: "Baseline file not found at [path]. Please provide a valid path to an existing competitive analysis markdown file, or create one first using /competitive-analysis."

**If web research fails**:
- Document inaccessible sources in delta report
- Continue with available sources
- Note limitations in Slack notification

**If artifact regeneration fails**:
- Log the error
- Note in Slack notification which artifacts failed to regenerate
- Still send delta report and update baseline markdown

**If Slack notification fails**:
- Log the error
- Continue with file updates
- User can read delta file directly
