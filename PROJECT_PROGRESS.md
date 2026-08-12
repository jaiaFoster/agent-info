# LUX — Executive Progress Dashboard

*Diagrams first. Full narrative: [`CEO_BRIEF.md`](CEO_BRIEF.md). Authoritative detail: [`AGENTS.md`](AGENTS.md). Reconciled by STATE-RECONCILE-001, 2026-08-12.*

## Roadmap position

```mermaid
flowchart LR
    A[Evidence] --> B[Canonical Output]
    B --> C[Scores]
    C --> D[Matches]
    D --> E[Revenue Qualification]
    E --> F[Paid Identification]
    F --> G[Human Outreach]
    G --> H[First Dollar]

    style A fill:#2d6a4f,color:#fff
    style B fill:#2d6a4f,color:#fff
    style C fill:#2d6a4f,color:#fff
    style D fill:#2d6a4f,color:#fff
    style E fill:#40916c,color:#fff
    style F fill:#f4a261,color:#000
    style G fill:#adb5bd,color:#000
    style H fill:#adb5bd,color:#000
```

Green = live. Orange = blocked/gated. Gray = not authorized.

Current gap: compliance/provider gate before paid identification. No paid
skip-trace or outreach is authorized.

## MVP-001

```mermaid
flowchart LR
    S[Motivated seller evidence] --> M[Explainable buyer/seller match]
    M --> P[Founder-approved paid identification]
    P --> O[Human-reviewed outreach]
    O --> C[Closed transaction]
```

Current production has seller evidence, buyer intelligence, and matches. It
does not yet have founder-authorized paid identification, callable contacts, or
outreach sends.

## Verified production metrics

| Metric | 2026-08-12 production |
|---|---:|
| Assets | 532,451 |
| Transactions | 159 |
| Transactions linked to Assets | 34 |
| Transaction→Asset coverage | 21.38% |
| SourceRecords | 560,102 |
| Observations | 11,255,764 |
| RawEvidence | 53 |
| PipelineRuns | 82 |
| Scored buyers | 57 |
| Persisted/quality-analyzed matches | 957 |
| Financial-opportunity observations | 821 |
| Asset-linked financial observations | 28 |
| Canonical facts | 0 |

## Capability status

| Capability | Status |
|---|---|
| Buyer intelligence | Live |
| Asset-scoped seller opportunities | Live |
| Explainable matching | Live |
| Match-quality analysis | Live |
| Candidate ranking | Live |
| Revenue qualification packet | Live |
| Skip-trace/DNC shadow validation | Live, read-only |
| Paid provider call | Not authorized |
| Outreach send | Not authorized |

## Current blockers

| Blocker | State |
|---|---|
| PROVIDER-DNC-001 | Hard compliance blocker before paid pilot/live callable contacts |
| BatchData retry/backoff | Open resilience gap before unattended provider reliance |
| Scoring-readiness policy | Founder decision; current graph coverage is below existing threshold |
| SKIP-PILOT-002 | Founder authorization required |
| FACT-LIVE-001/002 | Canonical facts absent in Railway production; audit follow-up |

## Next priorities

```mermaid
flowchart TD
    p1[PROVIDER-DNC-001] --> p2[BatchData retry/backoff]
    p2 --> p3[Scoring-readiness policy decision]
    p3 --> p4[SKIP-PILOT-002 founder authorization]
    p4 --> p5[Human outreach preparation]
    p5 --> p6[MVP-001 First Dollar]
```

Nothing in this file is a second source of truth. `AGENTS.md` remains
authoritative; this is the compact executive/mirror view.
