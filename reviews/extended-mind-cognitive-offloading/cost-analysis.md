# Cost and Efficiency Analysis: Extended Mind Thesis Literature Review

**Date**: 2026-02-11
**Review topic**: The Extended Mind Thesis and Cognitive Offloading
**Final output**: ~9,578 words, 63 cited references, 121 BibTeX entries

---

## 1. Executive Summary

This literature review consumed approximately **1.9 million tokens** across 17 subagent invocations plus the orchestrator, at an estimated cost of **$10.20--$12.50 USD**. Phase 3 (domain research) dominates at **46% of subagent tokens** and ~44% of total cost. The synthesis-planner running on Opus 4.6 instead of Sonnet 4.5 is an avoidable inefficiency. Several structural optimizations could reduce costs by 25--35% without affecting review quality.

---

## 2. Methodology

### Data sources

- **Subagent token counts**: Extracted from `<usage>` tags in Task tool completion messages. These report `total_tokens` (cumulative input + output across all API calls the agent made), `tool_uses`, and `duration_ms`.
- **Orchestrator tokens**: Estimated based on conversation turns, context growth, and tool result sizes. The orchestrator is the hardest component to measure precisely because its token usage is not separately reported.
- **API pricing**: Claude API pricing as of February 2026 (source: Anthropic pricing documentation).

### Input/output split estimation

The `total_tokens` figure includes both input and output tokens summed across all turns. Because each turn re-sends the full conversation history, input tokens dominate. Estimated splits by agent type:

| Agent type | Input % | Output % | Rationale |
|------------|---------|----------|-----------|
| Orchestrator | 93% | 7% | Mostly re-reading growing context; short coordination outputs |
| Domain researcher | 75% | 25% | Heavy API result ingestion (JSON, BibTeX); moderate output |
| Synthesis planner | 75% | 25% | Reads all BibTeX files; produces structured outline |
| Synthesis writer | 65% | 35% | Reads outline + BibTeX subset; produces section prose |
| Lit review planner | 65% | 35% | Reads research idea; produces domain plan |

### Pricing

| Model | Input ($/MTok) | Output ($/MTok) | Notes |
|-------|---------------|-----------------|-------|
| Opus 4.6 | $5.00 | $25.00 | Used by orchestrator, synthesis-planner |
| Sonnet 4.5 | $3.00 | $15.00 | Used by all other subagents |
| Haiku 4.5 | $1.00 | $5.00 | Not used in this review |

Prompt caching (0.1x input price for cache hits) may reduce actual costs. Estimates below assume **no prompt caching** to represent worst-case costs.

---

## 3. Measured Token Usage by Phase

### Phase 1: Environment Verification (Orchestrator)

Minimal cost. Three Bash tool calls + file checks. Estimated ~5,000 tokens of orchestrator context.

### Phase 2: Literature Review Planning

| Component | Model | Total Tokens | Tool Uses | Duration (s) | Est. Cost |
|-----------|-------|-------------|-----------|-------------|-----------|
| literature-review-planner | Sonnet | 9,123 | 1 | 70 | $0.07 |

This is the cheapest phase by far. The planner reads the research idea and writes a structured plan in a single tool call.

### Phase 3: Domain Literature Research

| Domain | Focus | Total Tokens | Tool Uses | Duration (s) | Est. Cost |
|--------|-------|-------------|-----------|-------------|-----------|
| 1 | Foundational Theory | 94,329 | 45 | 553 | $0.57 |
| 2 | Parity Principle | 119,297 | 40 | 551 | $0.72 |
| 3 | Coupling-Constitution | 98,833 | 52 | 513 | $0.59 |
| 4 | Cognitive Integration | 131,433 | 34 | 473 | $0.79 |
| 5 | Empirical Offloading | 92,280 | 46 | 551 | $0.55 |
| 6 | Theory-Empirical Bridge | 94,243 | 43 | 525 | $0.57 |
| 7 | Digital Technology | 159,407 | 42 | 461 | **$0.96** |
| 8 | Critical Perspectives | 101,662 | 39 | 574 | $0.61 |
| **Phase 3 Total** | | **891,484** | **341** | **574 (wall)** | **$5.35** |

**Observations**:
- Domain 7 (Digital Technology) is the single most expensive agent at 159K tokens, 69% more than the cheapest domain (Domain 5, 92K).
- Tool use counts range from 34 to 52, with no clear correlation to token count (Domain 4 used fewest tools but was third most expensive, suggesting larger API results per call).
- All 8 domains ran in parallel; wall-clock time was ~574s (9.6 minutes), limited by the slowest domain (Domain 8).
- 22 duplicate entries were removed during Phase 6 deduplication, indicating domain overlap.

### Phase 4: Synthesis Planning

| Component | Model | Total Tokens | Tool Uses | Duration (s) | Est. Cost |
|-----------|-------|-------------|-----------|-------------|-----------|
| synthesis-planner | **Opus** | 126,696 | 13 | 189 | **$1.27** |

**Key finding**: The synthesis-planner runs on Opus 4.6 because its agent definition specifies `model: inherit`, inheriting from the Opus orchestrator. This is the **most expensive per-token component** in the pipeline. On Sonnet, the same work would cost ~$0.76 (40% less).

### Phase 5: Synthesis Writing

| Section | Title | Total Tokens | Tool Uses | Duration (s) | Words | Est. Cost |
|---------|-------|-------------|-----------|-------------|-------|-----------|
| 1 | Introduction | 54,189 | 5 | 48 | 689 | $0.39 |
| 2 | Phil. Architecture | 67,603 | 6 | 95 | 1,708 | $0.49 |
| 3 | Empirical Offloading | 42,941 | 4 | 85 | 1,649 | $0.31 |
| 4 | Digital Technologies | 55,345 | 5 | 97 | 1,890 | $0.40 |
| 5 | Critical Perspectives | 61,518 | 5 | 79 | 1,228 | $0.44 |
| 6 | Conclusion | 85,100 | 7 | 60 | 1,000 | **$0.61** |
| **Phase 5 Total** | | **366,696** | **32** | **97 (wall)** | **8,164** | **$2.64** |

**Observations**:
- Section 6 (Conclusion) is the most expensive writer at 85K tokens despite producing only 1,000 words. It was instructed to read BibTeX files from 5 domains (vs. 2 for Section 3), inflating context.
- Section 3 is the most efficient: 42.9K tokens for 1,649 words (26 tokens per output word).
- Section 6 is the least efficient: 85.1K tokens for 1,000 words (85 tokens per output word).
- Token efficiency (tokens per output word) varies 3.3x across sections, driven primarily by the number of BibTeX files read as context.

### Phase 6: Assembly (Orchestrator)

Phase 6 runs entirely within the orchestrator context using local Python scripts. The scripts themselves are computationally trivial (see Section 6 below). The cost is the orchestrator's growing context window re-sent on each tool call (~8 calls in this phase).

### Orchestrator (All Phases)

| Component | Model | Est. Total Tokens | Est. Turns | Est. Cost |
|-----------|-------|-------------------|-----------|-----------|
| Orchestrator | Opus | ~500,000 | ~21 | $2.57 |

The orchestrator cost is estimated, not measured. It consists of:
- System prompt + CLAUDE.md + skill prompt (~8,000 tokens, re-sent every turn)
- Growing conversation context (tool results, agent completion messages)
- Modest output per turn (~300 tokens average)

With prompt caching, the orchestrator cost could drop to ~$0.80--$1.20 (system prompt and early context cached at 0.1x rate).

---

## 4. Cost Breakdown Summary

### By phase

| Phase | Model | Measured Tokens | Est. Cost | % of Total |
|-------|-------|----------------|-----------|------------|
| 1: Environment | Opus (orchestrator) | ~5,000 | ~$0.10 | 1% |
| 2: Planning | Sonnet | 9,123 | $0.07 | 1% |
| 3: Domain Research | Sonnet | 891,484 | $5.35 | **44%** |
| 4: Synthesis Planning | Opus | 126,696 | $1.27 | **10%** |
| 5: Section Writing | Sonnet | 366,696 | $2.64 | **22%** |
| 6: Assembly | Opus (orchestrator) | ~45,000 | ~$0.57 | 5% |
| Orchestrator overhead | Opus | ~450,000 | $2.20 | 18% |
| **Total** | | **~1,894,000** | **$12.20** | **100%** |

### By model

| Model | Tokens | Est. Cost | % of Cost |
|-------|--------|-----------|-----------|
| Opus 4.6 | ~627,000 (orchestrator + synthesis-planner) | $4.14 | 34% |
| Sonnet 4.5 | ~1,267,000 (all other subagents) | $8.06 | 66% |
| **Total** | **~1,894,000** | **$12.20** | **100%** |

### By cost component (input vs. output)

| | Input Tokens | Input Cost | Output Tokens | Output Cost | Total |
|---|---|---|---|---|---|
| Opus components | ~536,000 | $2.68 | ~91,000 | $2.28 | $4.96 |
| Sonnet components | ~886,000 | $2.66 | ~381,000 | $5.72 | $8.38 |
| **Total** | **~1,422,000** | **$5.34** | **~472,000** | **$8.00** | **$13.34** |

Note: Output tokens are more expensive per token (5x for Opus, 5x for Sonnet) and account for 60% of the dollar cost despite being only 25% of token volume.

### Cost per output unit

| Metric | Value |
|--------|-------|
| Total review words | 9,578 (incl. references) |
| Review body words | ~8,164 |
| Cost per 1,000 words (body) | ~$1.50 |
| BibTeX entries collected | 151 |
| BibTeX entries in final bibliography | 121 (after dedup) |
| References cited in review | 63 |
| Cost per cited reference | ~$0.19 |
| Cost per BibTeX entry collected | ~$0.08 |

---

## 5. Efficiency Analysis

### 5.1 Most expensive components (ranked)

1. **Phase 3 domain researchers** ($5.35, 44%): 8 parallel agents searching academic APIs. Each agent runs 34--52 tool calls, processing large JSON responses from Semantic Scholar, OpenAlex, CORE, and arXiv.

2. **Phase 5 synthesis writers** ($2.64, 22%): 6 parallel agents, each reading BibTeX files and the outline, then writing prose. Cost scales with BibTeX context size.

3. **Orchestrator overhead** ($2.30, 19%): The Opus orchestrator re-sends growing context on every turn. This is the "coordination tax" of the multi-agent architecture.

4. **Phase 4 synthesis-planner** ($1.27, 10%): Single agent reading all 8 BibTeX files. Runs on Opus unnecessarily.

### 5.2 Token efficiency by agent type

| Agent Type | Avg Tokens | Avg Tool Uses | Tokens/Tool Use | Output Quality |
|------------|-----------|---------------|-----------------|----------------|
| Domain researcher | 111,436 | 43 | 2,594 | 19 BibTeX entries avg |
| Synthesis writer | 61,116 | 5 | 12,223 | 1,361 words avg |
| Synthesis planner | 126,696 | 13 | 9,746 | 226-line outline |
| Lit review planner | 9,123 | 1 | 9,123 | 259-line plan |

Domain researchers have the lowest tokens-per-tool-use because most of their tool calls are Bash invocations of Python scripts returning structured JSON. The high tool count (avg 43) reflects the multi-source search strategy: ~6 API scripts per domain across Semantic Scholar, OpenAlex, CORE, arXiv, SEP/PhilPapers, and abstract enrichment/verification.

### 5.3 Paper utilization

| Metric | Count | % |
|--------|-------|---|
| Papers collected (raw) | 151 | 100% |
| Papers after dedup | 121 | 80% |
| Papers cited in review | 63 | 42% |
| Papers collected but unused | 58 | 38% |
| Duplicate entries removed | 22 | 15% |
| Entries marked INCOMPLETE (no abstract) | ~30 | 20% |

**58% of collected papers were not cited** in the final review. This is partly by design (researchers cast a wide net, the planner selects the most relevant), but it represents tokens spent on papers that didn't contribute to the output.

### 5.4 Wall-clock timing

| Phase | Duration | Notes |
|-------|----------|-------|
| Phase 1 | ~30s | Environment checks |
| Phase 2 | 70s | Single agent |
| Phase 3 | **574s (9.6 min)** | 8 parallel agents; limited by slowest |
| Phase 4 | 189s (3.2 min) | Single agent reading all BibTeX |
| Phase 5 | 97s (1.6 min) | 6 parallel agents; limited by slowest |
| Phase 6 | ~60s | Local Python scripts + orchestrator |
| **Total wall-clock** | **~17 min** | |

Phase 3 dominates wall-clock time due to API rate limiting and the number of search queries per domain.

---

## 6. Phase 6 Script Costs

Phase 6 uses local Python scripts that are computationally trivial compared to LLM API costs:

| Script | Complexity | Runtime | LLM Cost |
|--------|-----------|---------|----------|
| `assemble_review.py` | O(n) file concat | <1s | $0 |
| `dedupe_bib.py` | O(n) hash-based dedup | <1s | $0 |
| `generate_bibliography.py` | O(n x m) text scanning | ~1s | $0 |
| `lint_md.py` | O(m) markdown linting | <1s | $0 |

These scripts add negligible cost. The Phase 6 cost is entirely orchestrator context overhead (re-sending conversation history for each Bash tool call).

---

## 7. Cost Reduction Opportunities

### 7.1 Switch synthesis-planner to Sonnet (save ~$0.51, -4%)

**Current**: `model: inherit` in `.claude/agents/synthesis-planner.md` causes it to run on Opus 4.6 (the orchestrator's model).
**Proposed**: Change to `model: sonnet`.
**Rationale**: The synthesis planner reads BibTeX files and creates a structured outline. This is a structured planning task that Sonnet handles well. The 13 tool calls are all file reads (Glob, Grep, Read) and one Write.
**Risk**: Minimal. The outline structure and paper selection quality may be marginally affected, but the writers compensate.
**Savings**: ~$0.51 per review (from $1.27 to ~$0.76).

### 7.2 Reduce domains from 8 to 5--6 (save $1.00--$1.80, -8% to -15%)

**Current**: 8 domains with significant overlap (22 duplicates, 38% unused papers).
**Proposed**: Merge related domains:
- Merge Domain 2 (Parity Principle) into Domain 1 (Foundational Theory) -- both cover the original thesis and its direct critiques
- Merge Domain 6 (Theory-Empirical Bridge) into Domains 4+5 -- bridging work can be covered by integration and empirical researchers
- Optionally merge Domain 3 (Coupling-Constitution) into Domain 8 (Critical Perspectives) -- both are critical engagement with the thesis

**Rationale**: 5--6 focused domains would still cover all major positions. With fewer domains, each researcher can search more broadly within its scope, reducing redundant searches.
**Risk**: Some niche papers might be missed. Mitigate by expanding search terms within merged domains.
**Savings**: Eliminating 2--3 domain researchers saves $1.10--$1.80 in Phase 3 tokens.

### 7.3 Cap domain researcher tool uses (save ~$0.50--$1.00, -4% to -8%)

**Current**: No `max_turns` limit. Researchers use 34--52 tool calls.
**Proposed**: Set `max_turns: 25` in the Task tool invocation for domain researchers.
**Rationale**: Diminishing returns on additional searches. Most key papers are found in the first 20--30 tool calls. Late-stage calls often yield marginal additions or INCOMPLETE entries.
**Risk**: May miss some papers in domains with broad scope. Monitor paper counts.
**Savings**: Reduces average tokens from ~111K to ~75K per domain, saving ~$2.20 total across 8 domains. If combined with domain reduction (7.2), savings compound.

### 7.4 Reduce BibTeX context passed to synthesis writers (save ~$0.30--$0.50, -2% to -4%)

**Current**: Writers receive full BibTeX files for their relevant domains. Section 6 (Conclusion) received 5 domain files.
**Proposed**: Pass only the entries cited in the outline for each section, not entire domain files. The synthesis-planner already identifies specific papers per section.
**Rationale**: Section 6 used 85K tokens for 1,000 words because it read 5 full BibTeX files. Filtering to only cited entries could reduce input by 50--70%.
**Risk**: Writers may miss relevant adjacent papers. Mitigate by including "related" entries flagged in the outline.
**Savings**: Most impact on Section 6 (Conclusion) and Section 2 (Philosophical Architecture), which reference the most domains.

### 7.5 Run orchestrator on Sonnet (save ~$1.50--$2.00, -12% to -16%)

**Current**: Orchestrator runs on Opus 4.6. The `model` choice for the main conversation determines orchestrator cost.
**Proposed**: Run the main conversation on Sonnet 4.5 when invoking `/literature-review`.
**Rationale**: The orchestrator performs mechanical coordination: invoking agents, reading outputs, running scripts, managing files. These tasks don't require Opus-level reasoning.
**Risk**: Moderate. Sonnet may handle Phase 6 assembly and error recovery less robustly. The orchestrator also reads and verifies the plan and outline, where Opus's judgment may be valuable. Could be tested empirically.
**Savings**: Reduces orchestrator cost from ~$2.57 to ~$1.05. Note: this also changes the model for the synthesis-planner (which inherits), compounding savings.

### 7.6 Use Haiku for literature-review-planner (save ~$0.04, negligible)

**Current**: Sonnet, 9K tokens.
**Proposed**: Haiku.
**Savings**: Negligible ($0.04). Not worth the quality risk.

### Summary of opportunities

| Optimization | Savings | % of Total | Risk | Effort |
|-------------|---------|-----------|------|--------|
| 7.1 synthesis-planner → Sonnet | $0.51 | 4% | Low | Trivial (1-line change) |
| 7.2 Reduce to 5--6 domains | $1.00--$1.80 | 8--15% | Low-Medium | Modify planner prompt |
| 7.3 Cap researcher tool uses | $0.50--$1.00 | 4--8% | Low | Add max_turns parameter |
| 7.4 Filter BibTeX for writers | $0.30--$0.50 | 2--4% | Low | Modify skill orchestration |
| 7.5 Orchestrator on Sonnet | $1.50--$2.00 | 12--16% | Medium | User chooses model |
| **Combined (conservative)** | **$2.30--$3.80** | **19--31%** | | |
| **Combined (aggressive)** | **$3.80--$5.80** | **31--48%** | | |

---

## 8. What Cannot Be Reduced

The following costs are structural and cannot be reduced without affecting quality:

1. **Multi-source API searching**: Domain researchers must query multiple academic APIs (Semantic Scholar, OpenAlex, CORE, arXiv, SEP, PhilPapers) to ensure comprehensive coverage. This is core to the "accuracy" and "comprehensiveness" priorities.

2. **Citation verification**: Each paper is verified through CrossRef or other APIs. This is essential for the "never fabricate references" constraint.

3. **Abstract enrichment**: Papers without abstracts are enriched via multi-source resolution. This provides the content summaries that writers depend on for accurate citation.

4. **Parallel execution**: Running agents in parallel is a design choice that increases wall-clock efficiency at no additional token cost. Serializing would not save tokens.

5. **Phase 6 scripts**: Already computationally trivial. No optimization needed.

---

## 9. Comparison: Cost Per Quality Metric

| Metric | This Review | Hypothetical Optimized |
|--------|-------------|----------------------|
| Total cost | ~$12.20 | ~$8.40--$9.90 |
| Words (body) | 8,164 | 8,164 |
| References cited | 63 | ~55--60 |
| BibTeX entries | 121 | ~90--100 |
| Domains searched | 8 | 5--6 |
| Cost/1000 words | $1.50 | $1.03--$1.21 |
| Wall-clock time | ~17 min | ~13--15 min |

---

## 10. Appendix: Raw Token Data

### Subagent token usage (measured)

```
Phase 2:
  literature-review-planner  9,123 tokens    1 tools   69.9s  Sonnet

Phase 3:
  Domain 1 (Foundational)   94,329 tokens   45 tools  553.1s  Sonnet
  Domain 2 (Parity)        119,297 tokens   40 tools  551.3s  Sonnet
  Domain 3 (Coupling)       98,833 tokens   52 tools  512.8s  Sonnet
  Domain 4 (Integration)   131,433 tokens   34 tools  472.5s  Sonnet
  Domain 5 (Empirical)      92,280 tokens   46 tools  551.0s  Sonnet
  Domain 6 (Bridge)         94,243 tokens   43 tools  525.1s  Sonnet
  Domain 7 (Digital)       159,407 tokens   42 tools  461.1s  Sonnet
  Domain 8 (Critical)      101,662 tokens   39 tools  573.6s  Sonnet

Phase 4:
  synthesis-planner        126,696 tokens   13 tools  188.5s  Opus

Phase 5:
  Section 1 (Intro)         54,189 tokens    5 tools   48.2s  Sonnet
  Section 2 (Phil. Arch.)   67,603 tokens    6 tools   95.1s  Sonnet
  Section 3 (Empirical)     42,941 tokens    4 tools   84.7s  Sonnet
  Section 4 (Digital)       55,345 tokens    5 tools   96.6s  Sonnet
  Section 5 (Critical)      61,518 tokens    5 tools   78.8s  Sonnet
  Section 6 (Conclusion)    85,100 tokens    7 tools   60.5s  Sonnet

Subagent total:          1,393,999 tokens  392 tools
Orchestrator (est.):      ~500,000 tokens   ~21 turns
Grand total (est.):     ~1,894,000 tokens
```

### Model pricing reference (February 2026)

| Model | Input ($/MTok) | Output ($/MTok) | Cache read ($/MTok) |
|-------|---------------|-----------------|---------------------|
| Opus 4.6 | $5.00 | $25.00 | $0.50 |
| Sonnet 4.5 | $3.00 | $15.00 | $0.30 |
| Haiku 4.5 | $1.00 | $5.00 | $0.10 |
