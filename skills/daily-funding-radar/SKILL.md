# Daily Funding Radar

> Track startups (default: AI/ML companies) that recently raised funding or got backed by VCs/angels, compiled from startup news + funding databases, categorized by sector and round, and delivered as a Slack DM with a dated archive.

## Usage

```
/daily-funding-radar
/daily-funding-radar 2026-07-02
/daily-funding-radar focus: AI agents infrastructure
```

## What this does

Every run looks back over the last `{{config.daily_funding_radar.lookback_hours}}` hours (default 48) for **newly announced startup funding** — seed, Series A/B/C+, pre-seed, angel/grant rounds, and stealth exits with capital — with an emphasis on **AI and AI-adjacent companies**. For each startup it surfaces a self-contained summary (what the company does, round + amount, who led/backed it, and why it's worth noting) **so you don't have to click into the source**, plus a link to the source for each find. Results are categorized by sector and round, sent to Slack, and archived to a dated markdown file.

## Workflow

### 0. Load Configuration

Before doing any work, read `~/.claude/skills/config.json` and extract:
- `user.slack_user_id` — Slack DM recipient
- `user.display_name` — used in greeting
- `daily_funding_radar.lookback_hours` — default 48
- `daily_funding_radar.max_deals_per_category` — default 12
- `daily_funding_radar.archive_dir` — where dated digests are saved
- `daily_funding_radar.funding_search_queries` — Google News query strings
- `daily_funding_radar.focus_sectors` — sector buckets for categorization
- `daily_funding_radar.min_round_stages` — the round labels to normalize to

If `config.json` is missing, error out: "Missing ~/.claude/skills/config.json. Copy config.example.json and fill in your values."

References to `{{config.X}}` mean: substitute the value loaded from config.json.

If the user passed `focus: <text>`, treat it as an additional emphasis filter — prioritize matching startups and you may add 1–2 ad-hoc queries reflecting that focus. If the user passed a date (`YYYY-MM-DD`), use it as the reference "today" for the lookback window and the archive filename.

### 1. Gather Funding News (run searches in parallel where possible)

**Primary source — Google News RSS via the helper script, with `--resolve`.** This is the reliable, allowlisted spine. Fetch all queries in ONE call (no ad-hoc `for`/`curl` loops — those aren't preapproved and will prompt):

```bash
python3 ~/.claude/scripts/fetch_news.py --when 2d --max 8 --resolve \
  "AI startup raises seed funding" "AI startup Series A funding" "AI agent startup funding round" \
  "generative AI startup raises" "AI infrastructure startup funding" "enterprise AI startup venture funding" \
  "AI startup backed by a16z Sequoia" "AI startup emerges from stealth funding" \
  "AI coding agent startup raises" "vertical AI startup funding healthcare fintech legal" \
  "AI data / observability startup funding"
```

- Pass the queries from `{{config.daily_funding_radar.funding_search_queries}}` (one quoted arg each), plus any `focus:` ad-hoc queries.
- Set `--when` from `lookback_hours` (48h → `2d`). Use `--max 8` to widen the funnel since many hits will be dupes or off-topic.
- **`--resolve` is important here:** it adds a `RESOLVED:` line per item carrying the **real publisher URL** (e.g. `techcrunch.com`, `siliconangle.com`, `news.crunchbase.com`). The plain `LINK:` value is a `news.google.com` redirect token that renders **empty** under WebFetch — always deep-read the `RESOLVED:` URL, not `LINK:`. `--resolve` costs one extra request per item, which is fine at this scale.
- The script prints `TITLE: / LINK: / RESOLVED: / DATE: / SOURCE:` blocks grouped under `=== QUERY: ... ===`. Parse those directly. If a `RESOLVED:` line is missing or equals `LINK:` (resolution failed), fall back to the title + any other source covering the same raise.

**Secondary source — deep-read the strongest hits with WebFetch (on the `RESOLVED:` URL).** For headlines that look like real, in-window funding events but whose amount/investors/description aren't fully clear from the title, fetch the resolved article and extract the details. Batch these WebFetch calls in parallel. Prioritize public, non-paywalled outlets:
- Startup/tech news: **TechCrunch, VentureBeat, TechFundingNews, EU-Startups, Tech.eu, FinSMEs, SiliconANGLE, Axios Pro Rata, Business Insider, PYMNTS**. (Bloomberg / The Information may be partially paywalled — take what renders.)
- **Funding databases:** Crunchbase and PitchBook **news/blog posts** are public and fetchable (e.g. `news.crunchbase.com`); the **database record pages and AngelList/Wellfound profiles are auth-gated** — `WebFetch` fails on those, so do NOT rely on them. When a DB page is inaccessible, fall back to the news article covering the same raise.

```
WebFetch(url="<RESOLVED url>", prompt="Extract: startup name, one-line description of what it does,
  funding round/stage, amount raised, lead investor(s) and other backers named, HQ/location,
  announcement date, and any notable detail (traction, founders' pedigree, valuation). If this is
  NOT a funding announcement from the last 2 days, say so.")
```

For the Slack/archive **source link**, you may cite either the `RESOLVED:` publisher URL (preferred — direct, clean) or the original `LINK:` (also works in a browser). Prefer `RESOLVED:` when present.

**Research discipline:**
- **Recency:** only include rounds **announced within the lookback window**. Many articles resurface older raises — verify the announcement date. If a date can't be confirmed as in-window, drop it (or mark `_(date unconfirmed)_` only if the raise is otherwise high-signal).
- **Startup funding only:** include seed → growth rounds, angel/grant, and funded stealth launches. **Exclude:** public-company earnings, M&A/acquisitions (unless an acqui-hire that reads as a funding-equivalent signal — mark it clearly), PE buyouts of mature firms, and pure product launches with no capital event.
- **AI emphasis:** default to AI/ML and AI-adjacent companies. Non-AI rounds should be dropped unless they're unusually notable (then bucket under "Other AI" only if there's an AI angle; otherwise skip).
- **Corroborate amounts:** if two sources disagree on the amount/round, prefer the primary outlet and note the discrepancy.

### 2. Deduplicate & Normalize

- The same raise will appear across multiple queries/outlets — **collapse to one entry per startup**, keeping the most detailed/authoritative source as the primary link (optionally add a second link if it adds material info).
- Normalize the round to one of `{{config.daily_funding_radar.min_round_stages}}` (Pre-seed, Seed, Series A, Series B, Series C+, Grant / Angel, Stealth / Undisclosed).
- Normalize amounts to USD (`$12M`, `$1.2B`); if undisclosed, write `undisclosed`.

### 3. Categorize

Assign each startup to exactly one sector from `{{config.daily_funding_radar.focus_sectors}}` (best fit; use "Other AI" if none fits). Within each sector, order by signal:
1. Largest / most notable rounds and marquee investors first
2. Companies most relevant to the user's domain (agentic AI, enterprise AI, orchestration, observability, dev tools) get a boost
3. Earlier-stage but strategically interesting (stealth exits, notable founders) next

Cap each sector at `{{config.daily_funding_radar.max_deals_per_category}}` items; if more, keep the top N and note "…and N more" for that sector.

### 4. Format Each Find

One bullet per startup, self-contained so the source rarely needs opening:

```
• *[Startup Name]* — [Round] · [Amount] <[SOURCE_URL]|([Source])>
  [1–2 sentences: what the company does + the standout detail — lead investor, valuation, traction, notable founders, or why it matters]. _[optional: 🔥 relevance flag]_
```

Guidelines:
- **Lead with the company, round, and amount** — the scannable facts.
- Always name the **lead investor(s)** and notable backers when known (a16z, Sequoia, Lightspeed, Khosla, angels, etc.).
- Add a 🔥 flag + short note when a startup is **directly relevant to the user's space** (multi-agent orchestration, agent observability/eval, enterprise AI agents, AI dev tools, agent governance) — these are competitive/landscape signals worth extra attention.
- Include HQ/geography when notable (e.g., non-US hubs).
- Keep the source link on the company line; each find links to its source.

### 5. Compose the Slack Message

Use Slack markdown (`*bold*`, `_italic_`, bullets). Group by sector; omit empty sectors. Keep it scannable.

```
💰 *Daily Funding Radar — [WEEKDAY], [DATE]*
_Startups funded in the last [lookback_hours]h · [N] deals · [total $ tracked]_

━━━━━━━━━━━━━━━━━━━━━━━━

🤖 *AI Agents / Agentic* ([N])
• *[Startup]* — [Round] · [Amount] <[url]|([Source])>
  [what it does + standout detail] _🔥 [relevance note]_
• …

🏗️ *AI Infrastructure & Compute* ([N])
• …

🧠 *Foundation Models / LLMs* ([N])
• …

🛠️ *Developer Tools / Coding Agents* ([N])
• …

🏢 *Enterprise / Vertical SaaS AI* ([N])
• …

📊 *Data / Retrieval / Observability* ([N])
• …

🔐 *AI Security & Governance* ([N])
• …

🤖 *Robotics / Physical AI* ([N])
• …

🗣️ *Voice / Multimodal* ([N])
• …

✨ *Other AI* ([N])
• …

━━━━━━━━━━━━━━━━━━━━━━━━

📌 *Worth watching:* [1–2 line synthesis — the biggest round, a theme across the day's raises, or a startup directly adjacent to your space]

_Sources: Google News (startup/VC coverage), TechCrunch, VentureBeat, Crunchbase News, et al. Paywalled DB records (PitchBook/AngelList) not directly accessible._
```

Sector emoji mapping (use these): AI Agents 🤖 · Infrastructure 🏗️ · Foundation Models 🧠 · Dev Tools 🛠️ · Enterprise/Vertical 🏢 · Data/Observability 📊 · Security/Governance 🔐 · Robotics 🤖 · Voice/Multimodal 🗣️ · Other ✨.

If **no** qualifying rounds were found in the window, send a brief note:
```
💰 *Daily Funding Radar — [WEEKDAY], [DATE]*
No new startup funding rounds matched in the last [lookback_hours]h. (Quiet window or holiday.)
```

### 6. Send the DM

Send the composed digest as a Slack DM to `{{config.user.slack_user_id}}`.

### 7. Archive to Dated File

Write the digest (in markdown, links as `[text](url)` rather than Slack `<url|text>`) to:
```
{{config.daily_funding_radar.archive_dir}}/funding-radar-[YYYY-MM-DD].md
```
Create `archive_dir` if it doesn't exist. Include a short YAML-free header line with the date, deal count, and total capital tracked, then the categorized entries. This builds a searchable history over time.

### 8. Confirm

After sending, confirm in the terminal:
"Funding radar sent ✓ — Slack DM — [N] startups across [M] sectors, [total $] tracked. Archived to funding-radar-[DATE].md."

## Tips

- **Signal over volume:** a well-summarized set of 10–20 real, in-window raises beats 40 half-verified hits. Drop anything you can't confirm is a recent funding event.
- **Watch for stale resurfacing:** funding "listicles" and weekly roundups repackage old deals — always verify the individual announcement date.
- **Name the money:** the lead investor is often the most useful signal — always try to capture it.
- **Flag adjacency:** anything in multi-agent orchestration, agent eval/observability, enterprise AI agents, AI dev tooling, or agent governance is a competitive/landscape signal — mark it 🔥.
- **Fetchable links:** always deep-read the `--resolve` `RESOLVED:` URL, never the raw `news.google.com` `LINK:` (the latter renders empty under WebFetch). Resolution occasionally fails — when it does, fall back to the headline + another outlet.
- **Paywalls:** Crunchbase/PitchBook/AngelList *records* are auth-gated; `WebFetch` can't read them. Use their public *news/blog* posts and the covering news article instead — never block the run on an inaccessible DB page.
- **Cross-check amounts:** the resolved article is authoritative over the headline — e.g. a title may say "$90M" while the body says "$95M". Trust the article body.
- **Geography:** include non-US raises (Europe, India, MENA, APAC) — Tech.eu, EU-Startups, Inc42, and FinSMEs surface these well; consider adding region-specific queries via `funding_search_queries` if you want deeper coverage.

## Configuration

All defaults live in `~/.claude/skills/config.json` under `daily_funding_radar`. Edit that file to change lookback window, queries, sectors, per-sector caps, or archive location.

| Setting | Config key |
|---|---|
| Slack recipient | `user.slack_user_id` |
| Lookback window | `daily_funding_radar.lookback_hours` |
| Max deals per sector | `daily_funding_radar.max_deals_per_category` |
| Archive directory | `daily_funding_radar.archive_dir` |
| News search queries | `daily_funding_radar.funding_search_queries` |
| Sector buckets | `daily_funding_radar.focus_sectors` |
| Round-stage labels | `daily_funding_radar.min_round_stages` |

## Scheduling

This skill is wired into the daily morning catch-up (SessionStart hook) and runs automatically once per day after the other three daily skills. To run it manually any time, use `/daily-funding-radar`.
