---
name: customer-analysis
description: Deep-dive analysis of a specific customer account. Aggregates health indicators, product feedback, recent issues, contact mapping, meeting history, product usage, and external intelligence. Use for pre-meeting prep, account reviews, escalation context, or strategic account planning.
argument-hint: "<customer-name>"
---

# Customer Analysis

> Comprehensive deep-dive on a specific customer account, centered around a product focus with full context.

## Usage

```
/customer-analysis <customer-name>
```

**Argument**:
- `customer-name`: The customer/account name to analyze (e.g., "Acme Corp", "Tesla", "United Airlines")

## When to Use This Skill

- **Pre-meeting preparation**: Before executive meetings, QBRs, or escalation calls
- **Account reviews**: Understanding account health and product adoption
- **Escalation context**: Getting full background when issues arise
- **Strategic planning**: Identifying expansion opportunities or risks
- **Onboarding new team members**: Bringing them up to speed on key accounts

## Workflow

### Step 0: Clarify Product Focus

**Ask the user**:
```
What product or feature area should I focus this analysis on?

Examples:
- Agentforce / Einstein AI
- Data Cloud
- Sales Cloud / Service Cloud
- Slack
- Tableau / Analytics
- Platform / AppExchange

While I'll center the insights around [PRIMARY PRODUCT], I'll also surface any 
relevant information about other products for full context.
```

Wait for the user to specify the primary product focus before proceeding.

### Step 1: Search Internal Sources

Search broadly across all available Salesforce internal data sources. Do NOT limit to pre-configured channels.

#### 1a. Slack Search (All Channels)

Use `mcp__plugin_slack_slack__slack_search_public_and_private` to search across **all channels** (not just specific ones):

**Search queries to run**:
- `"[customer-name]"` (exact company name)
- `"[customer-name]" AND ([primary-product] OR [related-keywords])`
- `"[customer-name]" AND (issue OR bug OR escalation)`
- `"[customer-name]" AND (feedback OR feature request)`
- `"[customer-name]" AND (meeting OR call OR sync)`
- `"[customer-name]" AND (win OR success OR adoption)`
- `"[customer-name]" AND (churn OR risk OR concern)`

**Time range**: Last 180 days (6 months) for comprehensive history

**What to extract**:
- Customer health signals (sentiment, satisfaction indicators)
- Product feedback and feature requests
- Issues, bugs, escalations (severity, resolution status)
- Adoption milestones and wins
- Meeting summaries and action items
- Risk indicators (churn risk, stalled deals, blockers)
- Team members mentioned (AE, CSM, FDE, ProServe, executives)

**Organize chronologically** within each category.

#### 1b. Gmail Search

Use `mcp__plugin_gmail_gmail__gmail_search` to find emails involving this customer:

**Search queries**:
- `from:[customer-domain] OR to:[customer-domain]` (if domain known)
- `subject:"[customer-name]"` 
- `"[customer-name]" AND (meeting notes OR action items OR recap)`
- `"[customer-name]" AND [primary-product]`
- `"[customer-name]" AND (escalation OR critical OR urgent)`

**Time range**: Last 180 days

**What to extract**:
- Meeting notes and summaries
- Email threads about issues or feedback
- Executive correspondence
- Contract/renewal discussions
- Product usage questions

Use `mcp__plugin_gmail_gmail__gmail_get` to retrieve full content of relevant emails.

#### 1c. Google Docs & Meeting Notes

Use `mcp__plugin_google_google__docs_search` and `mcp__plugin_google_google__meeting_notes_search`:

**Search for**:
- Meeting notes with customer name
- Account plans or strategy docs
- QBR presentations
- Success plans
- Escalation documents

**Search queries**:
- `"[customer-name]" meeting notes`
- `"[customer-name]" QBR`
- `"[customer-name]" account plan`
- `"[customer-name]" success plan`
- `"[customer-name]" escalation`

Use `mcp__plugin_google_google__docs_get` to retrieve full content.

#### 1d. Calendar Events

Use `mcp__plugin_google_google__calendar_events`:

**Search for**:
- Meetings with customer name in title or description
- Recent meetings (last 90 days)
- Upcoming meetings (next 30 days)

**Extract**:
- Meeting frequency and cadence
- Who attends from Salesforce side (reveals team structure)
- Meeting types (1:1, QBR, technical review, escalation)

#### 1e. Internal Search (Confluence, Quip, etc.)

Use `mcp__plugin_search_search__search` across all available sources:

**Sources to search**: `confluence`, `quip`, `stackoverflow`, `webpages`

**Queries**:
- `"[customer-name]"`
- `"[customer-name]" [primary-product]`
- `"[customer-name]" architecture`
- `"[customer-name]" implementation`

**What to look for**:
- Technical architecture documents
- Implementation guides
- Support case summaries
- Internal wiki pages about the account

### Step 2: Search External Sources

Research public information about the customer to provide business context.

#### 2a. Company News & Updates

Use `WebFetch` to check:
- Company website (About page, News/Press section)
- LinkedIn company page (recent posts, company size, growth)
- Recent news articles (last 90 days)
- Earnings reports or financial filings (if public)
- Glassdoor (company health, hiring trends)

**What to look for**:
- Recent company news (layoffs, expansions, funding, acquisitions)
- Strategic initiatives or transformations
- Executive changes (new CEO, CTO, etc.)
- Financial health indicators
- Technology investments or modernization efforts

#### 2b. Salesforce-Specific External Intelligence

Search for:
- Customer testimonials or case studies on Salesforce websites
- Trailblazer Community posts by customer employees
- Dreamforce or event appearances
- Social media mentions of Salesforce by customer
- Public statements about Salesforce usage

### Step 3: Synthesize Contact Mapping

From all sources gathered, identify:

**Customer Contacts**:
- Names, titles, roles
- Frequency of interaction
- Areas of responsibility (technical, business, executive)
- Key decision makers vs. day-to-day users

**Salesforce Team**:
- **Account Executive (AE)**: Sales owner
- **Customer Success Manager (CSM)**: Post-sale relationship
- **Field Data Engineer (FDE)**: Technical implementation
- **ProServe / Professional Services**: Implementation consultants
- **Solution Engineers (SE)**: Pre-sale technical
- **Product team contacts**: PMs or engineers engaged
- **Executive sponsors**: VPs or C-level engaged

Present as:
```
### Contact Mapping

**Customer Contacts**
- [Name], [Title] — [Role/Responsibility] (last interaction: [date])
- [Name], [Title] — [Role/Responsibility] (last interaction: [date])

**Salesforce Team**
- **AE**: [Name] (last mention: [date])
- **CSM**: [Name] (last mention: [date])  
- **FDE**: [Name] (last mention: [date])
- **ProServe**: [Names/Team] (last engagement: [date])
- **Executive Sponsor**: [Name, Title] (if applicable)
```

### Step 4: Analyze Product Usage & Adoption

Based on information gathered, infer:

**Primary Product Focus ([Product Name])**:
- What features/modules are they using?
- Adoption stage (pilot, rollout, fully deployed, expanding)
- Usage patterns mentioned (volume, scale, use cases)
- Integration with other systems
- Customizations or extensions built
- Training or enablement activities

**Secondary Products/Features Detected**:
- Other Salesforce products mentioned
- Level of usage or adoption
- Integration points with primary product
- Future expansion opportunities

Present chronologically where possible (when they started, how usage evolved).

### Step 5: Generate the Customer Analysis Report

Create a comprehensive markdown report with the following structure:

```markdown
# Customer Analysis: [Customer Name]
*Primary Focus: [Product Name] | Generated: [Date]*

---

## Executive Summary

[3-5 bullet points]
- Overall account health and sentiment
- Key opportunities or risks
- Most critical recent issues or blockers
- Strategic importance to Salesforce
- Recommended actions before next interaction

---

## Company Overview

**Industry**: [Industry]
**Size**: [Employees/Revenue if available]
**Headquarters**: [Location]

**Recent Company News** (Last 90 Days):
- [Date] — [Headline or update]
- [Date] — [Headline or update]

**Strategic Context**:
[What's happening at this company that's relevant to your engagement]

---

## Account Health

**Overall Sentiment**: 🟢 Healthy / 🟡 At Risk / 🔴 Critical
[Based on tone of recent interactions, issue volume, renewal signals]

**Health Indicators**:
- **Engagement Level**: [High/Medium/Low] — [Evidence]
- **Product Satisfaction**: [Positive/Mixed/Negative] — [Evidence]
- **Adoption Trajectory**: [Growing/Stable/Declining] — [Evidence]
- **Risk Factors**: [List any churn risk, stalled initiatives, competitive threats]

---

## Product Usage & Adoption

### Primary Product: [Product Name]

**Adoption Stage**: [Pilot / Rollout / Production / Expanding]

**Features/Modules in Use**:
- [Feature 1] — [Usage context]
- [Feature 2] — [Usage context]

**Use Cases**:
- [Use case 1]
- [Use case 2]

**Scale/Volume**:
[Any metrics mentioned: users, records, transactions, etc.]

**Timeline** (Chronological):
- [Date] — [Adoption milestone or change]
- [Date] — [Adoption milestone or change]

### Secondary Products Detected

**[Product Name 2]**:
- Usage: [Context]
- Integration with primary product: [Yes/No, how]

**[Product Name 3]**:
- Usage: [Context]
- Opportunity: [Expansion potential]

---

## Product Feedback & Feature Requests

*Centered on [Primary Product], with other products noted*

### [Primary Product] Feedback

**Positive Feedback**:
- [Date] — [Channel/Source] — [Feedback quote or summary]
- [Date] — [Channel/Source] — [Feedback quote or summary]

**Constructive Feedback / Pain Points**:
- [Date] — [Channel/Source] — [Issue or concern]
- [Date] — [Channel/Source] — [Issue or concern]

**Feature Requests**:
- [Date] — [Request] — [Priority: High/Medium/Low] — [Status if known]
- [Date] — [Request] — [Priority: High/Medium/Low] — [Status if known]

### Other Product Feedback

[If feedback on secondary products was found, list here for context]

---

## Recent Issues & Escalations

*Chronological order (most recent first)*

### Critical/High Priority

- **[Date]** — [Issue Title/Summary]
  - **Severity**: Critical / High / Medium / Low
  - **Status**: Open / In Progress / Resolved
  - **Impact**: [Business impact described]
  - **Owner**: [Salesforce team member handling]
  - **Resolution**: [If resolved, how and when]

### Medium/Low Priority

[Same format as above]

---

## Meeting History & Key Interactions

*Chronological order (most recent first)*

### Recent Meetings (Last 90 Days)

- **[Date]** — [Meeting Type] — [Attendees]
  - **Topics**: [Key discussion points]
  - **Action Items**: [Any follow-ups identified]
  - **Sentiment**: [Positive/Neutral/Concerned]

### Upcoming Meetings (Next 30 Days)

- **[Date]** — [Meeting Type] — [Attendees]
  - **Purpose**: [If known]

---

## Contact Mapping

### Customer Contacts

| Name | Title | Role/Responsibility | Last Interaction |
|------|-------|---------------------|------------------|
| [Name] | [Title] | [Role] | [Date] |
| [Name] | [Title] | [Role] | [Date] |

**Key Decision Makers**: [Highlight executives or primary decision makers]

### Salesforce Team

| Role | Name | Last Activity/Mention |
|------|------|----------------------|
| Account Executive (AE) | [Name] | [Date] |
| Customer Success Manager (CSM) | [Name] | [Date] |
| Field Data Engineer (FDE) | [Name] | [Date] |
| ProServe | [Name/Team] | [Date] |
| Solution Engineer (SE) | [Name] | [Date] |
| Executive Sponsor | [Name, Title] | [Date] |

---

## Strategic Insights & Recommendations

### Opportunities
- [Expansion opportunity 1]
- [Upsell or cross-sell opportunity 2]
- [Strategic partnership opportunity 3]

### Risks
- [Risk factor 1]
- [Risk factor 2]

### Recommended Actions
1. [Action before next meeting]
2. [Follow-up needed]
3. [Strategic initiative to propose]

### Open Questions
- [Question to clarify with customer]
- [Information gap to investigate]

---

## Timeline: Key Events (Chronological)

*Full chronological view of major events, issues, wins, meetings*

- **[Date]** — [Event/Milestone]
- **[Date]** — [Event/Milestone]
- **[Date]** — [Event/Milestone]
[Continue through 180-day lookback period]

---

## Sources Referenced

**Internal Sources**:
- Slack: [# channels searched, # messages found]
- Gmail: [# emails reviewed]
- Google Docs: [# documents reviewed]
- Calendar: [# meetings found]
- Internal search: [Sources and # results]

**External Sources**:
- Company website
- News articles: [List key sources]
- LinkedIn
- [Other sources]

**Time Range**: [Start date] to [End date]

---

*Report generated: [Timestamp]*
*Primary product focus: [Product Name]*
*For questions or updates, re-run `/customer-analysis "[Customer Name]"`*
```

### Step 6: Save the Report

Save the report to a logical location:

**Default path**: `~/Documents/Work/CompetitiveAnalysis/customer-analysis-[customer-name-slug]-[YYYY-MM-DD].md`

**Naming convention**: 
- Lowercase, hyphen-separated
- Include date for versioning
- Example: `customer-analysis-tesla-2026-05-18.md`

After saving, inform the user of the file location.

### Step 7: Offer Follow-Up Actions

After generating the report, ask:

```
I've generated a comprehensive customer analysis for [Customer Name] focused on [Product].

Would you like me to:
1. Create a one-page executive summary (for quick prep before a meeting)
2. Generate a pre-meeting brief email you can send to attendees
3. Create a list of open action items or follow-ups needed
4. Draft talking points for an upcoming meeting
5. Export this report to PDF or PPTX format
6. Set up a recurring customer health monitor (similar to daily-competitive-insights)
```

## Special Handling

### Missing Information

If certain data is not available:
- **Explicitly note gaps**: "No recent Gmail correspondence found in the last 180 days"
- **Suggest alternatives**: "Consider searching [customer-name].com for their support portal or community"
- **Avoid speculation**: Don't invent data; mark sections as "No information available"

### Sensitive Information

- Be mindful of confidential customer data
- Do not expose internal Salesforce strategy or competitive details in outputs
- If the report will be shared with the customer, mark sections as "Internal Only"

### Large Result Sets

If searches return hundreds of results:
- **Prioritize by recency**: Focus on last 90 days for detailed analysis, older data for trends
- **Prioritize by relevance**: Issues/escalations > feedback > general mentions
- **Sample representative items**: Don't list every single Slack message; extract key themes

### Customer Name Variations

Search for variations:
- Full company name ("Tesla, Inc.")
- Short name ("Tesla")
- Common abbreviations or nicknames
- Domain name ("tesla.com")
- Stock ticker (if public)

### Data Freshness

- Always note the **time range** for each data source
- Flag if data appears stale (e.g., last Slack mention was 6+ months ago)
- Indicate if customer might be inactive or off-platform

## Integration with Other Skills

**Relationship to `/daily-customer-insights`**:
- `daily-customer-insights`: Aggregates signals across many customers, broad monitoring
- `customer-analysis`: Deep-dive on a single customer, pre-meeting prep

**Complementary workflow**:
1. `daily-customer-insights` surfaces a customer with issues → alerts you
2. `customer-analysis` provides deep context on that customer
3. Armed with full context, have effective customer conversation
4. Document outcomes back in Slack/email for future analysis

## Tips for Effective Analysis

### Search Strategy
- **Cast a wide net**: Don't limit to known channels; search all available sources
- **Use multiple query variations**: Company name alone, product-specific, issue-specific
- **Go back far enough**: 180 days captures quarterly context and trends
- **Cross-reference**: Corroborate findings across multiple sources

### Signal vs Noise
- **Strong signals**: Direct customer quotes, escalations, executive involvement, renewal discussions
- **Weak signals**: Passing mentions, old information, secondhand reports
- **Context matters**: A single negative comment vs. pattern of concerns

### Chronological Organization
- Within each section, order items by date (most recent first for Issues, chronological for Timeline)
- This shows trajectory: Is the situation improving or worsening?
- Helps identify patterns (e.g., issues spike around releases)

### Contact Mapping
- Focus on who you actually find evidence of, not theoretical org charts
- Last interaction date shows who's active vs. who's gone quiet
- Multiple contacts from one area (e.g., 3 engineers) signals focus area

### External Intelligence
- Public news can reveal context (layoffs → budget concerns, new CTO → strategy shift)
- Don't over-index on marketing materials (case studies, testimonials)
- Look for authentic signals: job postings, earnings calls, analyst reports

## Example Invocations

```bash
# Basic usage
/customer-analysis "Tesla"

# With product focus clarification
/customer-analysis "United Airlines"
→ "What product should I focus on?" 
→ "Service Cloud"
→ [Generates report centered on Service Cloud]

# After daily-customer-insights alert
# daily-customer-insights found: "Acme Corp mentioned 'critical bug' in #agentforce"
/customer-analysis "Acme Corp"
→ Focus on: Agentforce
→ [Deep-dive reveals full context of the issue]
```

## Output Example Structure

```
customer-analysis-tesla-2026-05-18.md
├─ Executive Summary (3-5 bullets)
├─ Company Overview (industry, size, recent news)
├─ Account Health (sentiment, indicators, risks)
├─ Product Usage & Adoption
│  ├─ Primary Product: [Focus Product]
│  │  ├─ Adoption stage
│  │  ├─ Features in use
│  │  ├─ Use cases
│  │  └─ Timeline (chronological)
│  └─ Secondary Products Detected
├─ Product Feedback & Feature Requests (chronological within categories)
├─ Recent Issues & Escalations (chronological)
├─ Meeting History (chronological)
├─ Contact Mapping (Customer + Salesforce team)
├─ Strategic Insights & Recommendations
├─ Timeline: Key Events (full chronological)
└─ Sources Referenced
```

---

*This skill aggregates multi-source intelligence into a single comprehensive customer view, organized chronologically for easy consumption and decision-making.*
