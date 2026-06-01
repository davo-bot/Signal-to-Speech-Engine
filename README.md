# Signal-to-Speech-Engine

A seven-layer multi-agent AI pipeline that replaces generic cold outreach with intelligence-backed, personalised conversation openers for management consulting recruitment.

---

## The Problem

Management consulting recruiters rely on templated cold outreach that fails immediately with sophisticated senior Partners. A generic "are you hiring?" email does not just underperform; it actively damages credibility with the exact audience it targets. A Senior Partner at McKinsey, Bain, or Oliver Wyman who receives one of these emails forms a lasting negative impression.

Manually researching each contact to the depth required for peer-level outreach takes 45 to 90 minutes per person. At that rate, personalised outreach at scale is not a realistic option.

The Signal to Speech Engine solves this by combining live macro-level market intelligence with person-specific research, evaluating commercial fit through a structured three-filter synthesis, and generating a personalised message framework that the consultant completes and sends.

The result is an email the recipient reads and thinks: *how did they know?*

---

## Pipeline Architecture

The pipeline operates across seven layers, each with a distinct role:

| Layer | Tool | Role |
|---|---|---|
| 1 — Ingestion | PhantomBuster | Identity resolution and dynamic list generation |
| 2 — Enrichment | Clay | Baseline profile, domain scoping, search execution |
| 3 — Search Intelligence | Perplexity API + Exa.ai | Macro signals (radar) + person-specific content (scalpel) |
| 4 — Orchestration | Make.com | Flow control, routing, module sequencing |
| 5 — Synthesis | Claude Agent 1 | Three-filter evaluation, fit scoring, structured output |
| 6 — Generation | Claude Agent 2 | Personalised email framework generation |
| 7 — Review | Google Sheets | Draft Library (HIGH) + Manual Review Queue (LOW) |

---

## How It Works

### Search Intelligence — Two Tools, Two Jobs

Perplexity and Exa.ai operate in parallel with deliberately separate roles. Using a single tool for both tasks yields mediocre results in both dimensions.

**Perplexity API — The Radar**
Scans broadly for macro-level commercial signals affecting the target's sector and region. Regulatory changes, market shifts, competitive pressures, and growth events. Feeds the Timing and Relevance filters in Agent 1. Queries are structured using the RTCC framework (Role, Task, Context, Constraint) to constrain retrieval to commercially relevant sources before synthesis runs.

**Exa.ai — The Scalpel**
Retrieves person-specific content attributed to the named individual using neural semantic search. Podcast transcripts, conference talks, published essays, LinkedIn posts. Finds content by meaning rather than keywords, enabling retrieval of a transcript where an executive mentions a challenge without those exact words appearing in the title. Feeds the Credibility filter in Agent 1.

---

### Agent 1 — Intelligence Evaluation

Agent 1 is the quality gate. It receives structured Perplexity and Exa data and applies three filters before making any routing decision.

**Filter 1 — Timing** *(Source: Perplexity)*
PASS if at least one finding carries a publication date within 365 days AND contains a genuine operational urgency signal. Timing FAIL produces a LOW fit score regardless of other filter outcomes. Recency must be confirmed, not assumed.

**Filter 2 — Relevance** *(Source: Perplexity)*
PASS if there is direct or strongly implied evidence of a talent gap, headcount pressure, restructuring, or operational challenge that Consulting Point can address. General market commentary without a specific operational mechanism fails this filter.

**Filter 3 — Credibility** *(Source: Exa.ai)*
PASS if a specific, non-obvious detail about the named individual is present in an Exa result that explicitly names both their full name and their current firm. False positive prevention: results about adjacent individuals in similar domains are rejected and logged in `data_gaps`.

**Scoring Gate**

| Timing | Relevance | Credibility | Fit Score | Routing |
|---|---|---|---|---|
| FAIL | Any | Any | LOW | Manual Review Queue |
| PASS | PASS | Any | HIGH | Agent 2 Generation |
| PASS | FAIL | PASS | HIGH | Agent 2 Generation |
| PASS | FAIL | FAIL | LOW | Manual Review Queue |

Agent 1 outputs a structured JSON object containing: `trigger_event`, `motivation_match`, `resonance_profile`, `timing_rationale`, `scoring_rationale`, `fit_score`, and `data_gaps`. All fields are produced regardless of fit score.

---

### Agent 2 — Message Generation

Agent 2 receives Agent 1's structured synthesis output and generates a four-component email framework:

1. **Executive Hook** — leads with the recipient's world, anchored to `resonance_profile`
2. **[INSERT YOUR NETWORK CONNECTION HERE]** — deliberate gap, never filled by the pipeline
3. **Bridge** — connects their commercial situation to **[Consulting Company's] **intelligence, drawn from `trigger_event` and `motivation_match`
4. **Soft Ask** — one sentence, no recruitment language

Agent 2 does not re-evaluate the data. It executes from Agent 1's brief. The network connection gap is by intentional design — In the digital age, the hinderance between automation and quality isn't how well you prompt or design the pipeline, it is ensuring the humans with the built and expertise are constantly positioned to supervise and oversee the final preparations of any automated work, this last hurdle is a learnt trait that only the consultant can provide, and making it a labelled placeholder ensures it is never skipped.

Output includes: `subject_line`, `email_draft`, `quality_flag` (FULL or REDUCED), `word_count`, `edit_notes`, and `personalisation_guidance`.

---

### Google Sheets Outputs

**HIGH Signal — Draft Library tab**
Agent 2's complete output is written to the Draft Library, including subject line, email draft, quality flag, word count, edit notes, and personalisation guidance. Consultant Status dropdown: Pending / Approved / Sent / Revised.

**LOW Signal — Manual Review Queue tab**
Agent 1's full evaluation is written, including all three filter verdicts, evidence for each, trigger event, resonance profile, scoring rationale, and data gaps. Consultant Decision dropdown: Pending / Pursue Manually / Archive / Escalate.

---

## Design Philosophy

**Conservative by default**
The pipeline treats ambiguity as grounds for not proceeding. A missed outreach opportunity costs nothing. A credibility-damaging email to a Senior Partner can permanently damage the relationship. Every Agent 1 default is conservative — unclear recency fails Timing, ambiguous Exa attribution fails Credibility.

**Human layer intentionally preserved**
The pipeline does not produce a complete email. It produces a framework that requires a human to insert the network connection. This is not a limitation — it is a design decision. The relationship layer cannot be automated and should not be.

**Failure states first**
The hardest problem in building this was not the AI. It was the behavioural and intelligence decisions that determined whether output would resonate with clients or end up in the bin. Building the system primarily around its failure states — conservative defaults, false-positive controls, data-gap logging, transparent limitations — was more important than building the success states.

**Transparent limitations**
Agent 1's `data_gaps` field explicitly records what was absent, uncertain, or rejected. The consultant always knows where the intelligence is strong and where it is inferred. Honesty about limitations makes the system more trustworthy, not less.

---

## Repository Structure

```
Signal-to-Speech-Engine/
├── README.md
├── docs/
│   ├── 01_pipeline_diagram.pdf
│   ├── 02_system_overview.pdf
│   ├── 03_search_intelligence_methodology.pdf
│   ├── 04_synthesis_layer_architecture.pdf
│   └── 05_prompt_architecture.pdf
├── prompts/
│   ├── agent_1_system_prompt.md
│   └── agent_2_system_prompt.md
└── examples/
    ├── agent_1_output_example.json
    └── agent_2_output_example.json
```

---

## Built With

- **Clay** — Contact enrichment and search query execution
- **Perplexity API** — Live market intelligence via RTCC-structured queries
- **Exa.ai** — Neural semantic search for person-specific content retrieval
- **Make.com** — Orchestration, routing, and module sequencing
- **Claude Sonnet 4.6** — Agent 1 (synthesis) and Agent 2 (generation)
- **Google Sheets** — Draft Library and Manual Review Queue output storage
- **PhantomBuster** — Identity resolution and list generation (production layer, manually substituted in prototype)

---

## Prototype vs Production

This repository documents a functional end-to-end prototype. Two components are documented but not implemented in the prototype:

**PhantomBuster** handles automated list generation from LinkedIn searches in production. In the prototype, contacts are manually entered in Clay. The downstream pipeline is identical either way.

**Crunchbase Pro** would be added as a dedicated Clay enrichment column for financial trigger identification in production. In the prototype, Perplexity's web search surfaces funding events as part of its broader market signal retrieval.

---

## Documentation

Full technical documentation is available in the `/docs` folder:

- [System Overview](docs/02_system_overview.pdf) — end-to-end pipeline explanation and design philosophy
- [Search Intelligence Methodology](docs/03_search_intelligence_methodology.pdf) — Perplexity and Exa.ai design decisions and query architecture
- [Synthesis Layer Architecture](docs/04_synthesis_layer_architecture.pdf) — Agent 1 filter design, scoring gate, and output structure
- [Prompt Architecture](docs/05_prompt_architecture.pdf) — annotated Agent 1 and Agent 2 system prompt design decisions

---

*Built by David Ajayi — [LinkedIn](https://linkedin.com/in/david-ajayi28)*
