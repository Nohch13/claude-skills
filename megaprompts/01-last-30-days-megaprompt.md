# Mega Prompt: Last-30-Days Research Skill

## Role

You are a **Skill Architect** specializing in research workflows. Generate a production-grade, distributable Claude skill that performs multi-source research on any topic within a configurable recent window (default: 30 days).

## Output Target

Single file: `${SKILLS_DIR}/last-30-days/SKILL.md`

Word budget: 1,800–2,200 words. Hard ceiling: 2,500.

## Skill Purpose

Synthesize what people are saying about a topic across Reddit, Hacker News, the open web, and (optionally) X/Twitter, within a configurable time window. Output a single coherent research briefing with citations, engagement signals, and cross-platform pattern analysis.

## Required Capabilities

The skill must specify how to:

1. **Accept topic input** — From explicit invocation, conversational reference, or attached brief.
1. **Run Reddit search** — Use Reddit’s public JSON API (`reddit.com/search.json`) with `sort=top&t=month` and `sort=new&t=month`. Fetch top thread comments for the top 3–5 posts by score.
1. **Run Hacker News search** — Use Algolia HN search API with computed Unix timestamp filter. Search both stories and comments.
1. **Run web search** — Use available web search + fetch tools. Issue 2–3 targeted queries: trusted-publisher news, recent reviews, honest-opinion sources (problems/complaints/worth-it).
1. **Run X/Twitter (optional)** — Use Grok or similar accessible interface if browser automation is available. Otherwise skip with a documented note.
1. **Synthesize** — Cross-platform pattern detection: consensus, controversy, pain points, excitement, emerging trends, gaps.

## Workflow Structure

The generated skill must follow this exact structure:

```
1. Invocation (how triggers route to this skill)
2. Pre-flight (validate topic, set time window, plan phases)
3. Phase 1: Reddit (run in parallel with HN + Web)
4. Phase 2: Hacker News (parallel)
5. Phase 3: Web Search (parallel)
6. Phase 4: X/Twitter (sequential, optional)
7. Synthesis (cross-platform analysis)
8. Output (file + chat delivery)
9. Troubleshooting (documented failure modes)
```

## Critical Improvements Over Naive Implementation

The skill MUST address these production concerns:

1. **Configurable time window** — Default 30 days, but accept `7d`, `14d`, `60d`, `90d`. Compute Unix timestamps dynamically using the current date in context.
1. **Parallel execution** — Phases 1, 2, 3 are independent and must run concurrently. Document this explicitly.
1. **Graceful degradation** — If any single source fails (rate limit, 404, timeout, login wall), note it in the output and continue with remaining sources. Never fail the entire run on one source failure.
1. **Source-agnostic X handling** — Don’t hardcode “Grok”. Specify: “Use whatever X/Twitter-accessible interface is available (Grok, X API if authenticated, or skip with note).”
1. **Citation discipline** — Every claim in synthesis must trace back to a specific source with URL.
1. **Output saved AND displayed** — File to `${RESEARCH_DIR}/last-30-days/<topic-slug>-<YYYY-MM-DD>.md` AND full briefing pasted in chat.

## Output Format Specification

The skill must produce markdown with this structure:

```markdown
# [TOPIC] — Last [N] Days Research
*Generated: [DATE]*

## TL;DR
[2-3 sentences max]

## Reddit
### Top Posts
- **[Title]** (r/sub) — [score, comments] — [summary] — [URL]
### What Reddit Is Saying
[Narrative paragraph]

## Hacker News
### Notable Stories
- **[Title]** — [points, comments] — [summary] — [URL]
### What HN Is Saying
[Narrative paragraph; note HN's technical/builder bias]

## Web
### Key Sources
- **[Title]** ([Publication]) — [takeaway] — [URL]
### What the Web Is Saying
[Narrative paragraph]

## X/Twitter (if available)
[Cleaned response, with handles/references preserved]
[Or: "Skipped — [reason]"]

## Cross-Platform Patterns
[Highest-confidence signals across sources]

## Key Takeaways
- [3-5 bullets]

## Content Angles (if applicable)
[2-3 specific angles supported by the data]
```

## Trigger Phrases (for frontmatter description)

Include these patterns:

- “research [topic]”
- “last-30-days on [topic]”
- “what’s happening with [topic]”
- “what are people saying about [topic]”
- “find me info on [topic]”
- Plus: competitor research, trend discovery, tool comparisons, audience sentiment

## Error Handling Requirements

Document explicit handling for:

|Failure                          |Behavior                                                     |
|---------------------------------|-------------------------------------------------------------|
|Reddit blocks/rate-limits        |Try `?raw_json=1` or fall back to subreddit-restricted search|
|HN returns empty                 |Broaden query, drop timestamp filter as last resort          |
|Web search returns nothing useful|Note in output; don’t fabricate sources                      |
|Browser automation unavailable   |Skip X phase with documented note                            |
|WebFetch times out               |Use what loaded, mark as truncated                           |
|All sources fail                 |Return error with diagnostic info, don’t deliver empty file  |

## Portability Requirements

- **Claude Code CLI**: Native — uses WebFetch, WebSearch, file write tools.
- **Claude.ai web**: Works for Reddit/HN/Web phases via available web tools. Document that X phase requires browser automation (CLI-only) and will be skipped in web context.

Add this notice at the top of the generated skill:

> **Portability:** Works in both Claude Code CLI and Claude.ai. The optional X/Twitter phase requires browser automation and is skipped automatically if unavailable.

## Frontmatter Spec

```yaml
---
name: last-30-days
description: "Multi-source research skill that investigates any topic across Reddit, Hacker News, the open web, and optionally X/Twitter within a configurable recent window (default 30 days). Returns a synthesized briefing with citations, engagement metrics, and cross-platform pattern analysis. Triggers: 'research [topic]', 'last-30-days on [topic]', 'what's happening with [topic]', 'what are people saying about [topic]', 'find me info on [topic]', or any variation requesting multi-source intelligence on a topic. Also use for competitor research, trend discovery, tool comparisons, and audience sentiment analysis."
---
```

## Anti-Patterns To Reject

- Hardcoded URLs that won’t survive API changes (note the format but explain it may evolve)
- Specific person/brand references
- Tight coupling to one X/Twitter interface
- Missing fallback behavior
- “Just use [specific tool]” without explaining what the tool does

## Validation Checklist (Run Before Delivery)

- [ ] Frontmatter parses as YAML
- [ ] Word count 1,800–2,500
- [ ] All 4 phases documented with concrete API patterns
- [ ] At least 6 failure modes documented
- [ ] Parallel execution explicitly stated
- [ ] Time window is configurable, not hardcoded
- [ ] Output paths use variables, not absolute paths
- [ ] No personal/brand references
- [ ] Portability notice present
