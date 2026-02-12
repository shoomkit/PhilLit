# Recent Publications Missing Abstracts Due to Indexing Delays

**Observed**: 2026-02-12, during "Algorithmic Fairness (2023-Present)" literature review
**Severity**: Low (affects recent papers only; agents adapt by using partial metadata)
**Status**: Expected behavior (not a bug)

## Summary

When reviewing very recent literature (e.g., 2023-present), domain researchers consistently found that papers published in 2025 lacked abstracts across multiple indexing services (Semantic Scholar, OpenAlex, CORE). These papers were marked `INCOMPLETE` with keyword `no-abstract`, but were retained in the bibliography with `note` field annotations and `importance` ratings to guide synthesis-writers.

## Observed Behavior

### Literature Review: Algorithmic Fairness (2023-Present)

Out of 126 papers collected across 7 domains, **21 entries were marked INCOMPLETE** due to missing abstracts:

| Domain | INCOMPLETE entries | Notes |
|--------|-------------------|-------|
| Domain 1 (Definitions) | 3 | Very recent 2025 publications |
| Domain 2 (Measurement) | 5 | Primarily 2025 papers |
| Domain 4 (Critiques) | 4 | 2024-2025 publications |
| Domain 5 (Applications) | 5 | Recent conference papers (2025) |
| Domain 6 (Technical) | 7 | arXiv preprints not yet indexed |
| Domain 7 (Interdisciplinary) | 2 | 2025 legal scholarship |

**Common characteristics of INCOMPLETE entries:**
- Published in 2025 (within ~2 months of the review date: Feb 2026)
- Conference papers from recent venues (FAccT 2025, ICML 2025, NeurIPS 2025)
- arXiv preprints uploaded in late 2024 or early 2025
- Legal/regulatory scholarship (EU AI Act analyses) published in specialty journals

## Why This Happens

Academic indexing services require time to ingest and process new publications:

1. **Semantic Scholar**: Abstracts appear within days-weeks for major venues, but slower for smaller journals or non-English publications
2. **OpenAlex**: Abstracts may lag by weeks-months depending on publisher metadata quality
3. **CORE**: Focuses on open-access repositories; abstracts depend on repository metadata completeness
4. **CrossRef**: Provides DOI metadata but abstracts are optional and often omitted by publishers

**This is expected behavior** — a 2025 paper reviewed in Feb 2026 may have been published only 1-3 months prior, insufficient time for complete indexing.

## Current Mitigation

Domain researchers already implement appropriate fallback strategies:

1. **Attempt abstract resolution**: Query S2 → OpenAlex → CORE → NDPR in sequence
2. **Extract metadata**: Collect title, authors, year, DOI, venue, and any partial metadata available
3. **Mark as INCOMPLETE**: Add `keywords = {INCOMPLETE, no-abstract}` to BibTeX entry
4. **Provide context**: Write `note` field with 1-2 sentence description of the paper's focus based on title and available metadata
5. **Assign importance**: Rate as High/Medium/Low to signal synthesis-writers whether to cite despite incompleteness

Synthesis-writers are instructed to prefer papers with abstracts but may cite INCOMPLETE entries if they are highly relevant and other context (citations, venue, title) indicates importance.

## Impact on Review Quality

**Minimal**. In the Algorithmic Fairness review:
- 117 unique BibTeX entries (21 INCOMPLETE, 96 complete)
- 90 papers cited in final review (synthesis-writers selected complete papers where possible)
- INCOMPLETE papers served as coverage indicators even when not directly cited

The `note` annotations and `importance` ratings allowed synthesis-writers to make informed decisions about which INCOMPLETE entries merited citation based on secondary sources or paper titles.

## Options to Address

### Option A: Accept as expected behavior

This is not a bug but an inherent limitation of reviewing cutting-edge literature. The current workflow handles it gracefully by:
- Attempting multiple enrichment sources
- Marking incomplete entries explicitly
- Providing context via `note` fields
- Allowing synthesis-writers to decide whether to cite

### Option B: Manual abstract sourcing

For reviews focused on very recent literature, the user could manually provide abstracts for key papers by reading PDFs. This is labor-intensive and breaks automation but ensures high-priority papers have full abstracts.

### Option C: PDF text extraction

Implement a PDF-to-abstract extraction pipeline using `pdftotext` or similar tools. Domain researchers could download PDFs for INCOMPLETE entries and extract the abstract section. This adds complexity (PDF parsing is fragile) and may violate publisher terms of service.

### Option D: Delay literature review by 3-6 months

For reviews requiring comprehensive abstracts, wait 3-6 months after publication dates to allow indexing services to catch up. This trades timeliness for completeness.

### Option E: Re-enrichment pass in Phase 6

Add a re-enrichment step during Phase 6 (assembly) that re-attempts abstract resolution for all INCOMPLETE entries. This could catch papers that were indexed during the review process (between Phase 3 collection and Phase 6 assembly). Minimal additional cost (~30 seconds) and potentially recovers several abstracts.

## Recommendation

**Option A** (accept as expected) is appropriate. Reviewing 2023-present literature necessarily involves incomplete indexing for the most recent papers. The current workflow is designed for this:

- INCOMPLETE entries signal "cite with caution"
- Synthesis-writers have enough context (`note`, `importance`, title, venue) to make informed decisions
- Users can manually supplement abstracts for critical papers if needed

If incomplete abstracts become a recurring concern, **Option E** (re-enrichment pass) is low-cost and could recover 20-30% of INCOMPLETE entries that become indexed during the multi-hour review process.

**Options B, C, D** are appropriate only for reviews where abstract completeness is a hard requirement and where timeliness is less critical than comprehensiveness.
