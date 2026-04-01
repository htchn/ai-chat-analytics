[📖 中文版文档](README_CN.md)

# 🤖 AI Chat Analytics

> A complete user behavior analytics pipeline for conversational AI products, from raw event data to actionable product insights.

## 🎯 What is This?

If you build a ChatGPT/Claude-like product, your users will chat, like 👍, dislike 👎, retry 🔄, and follow up 💬 — or just leave silently 🤐. **How do you turn these signals into product decisions?**

This project demonstrates a full analytics framework that answers:

- 📊 **How is overall quality?** — Behavior distributions, satisfaction rates
- 🔍 **Where are the problems?** — Per-scenario quality breakdown
- ❓ **Why are users unhappy?** — Semantic root-cause attribution
- 💬 **What do follow-ups really mean?** — Exploratory vs. corrective signal decomposition
- 🎯 **What should we fix first?** — Prioritized optimization recommendations

---

## 🧠 Thinking Chain & Tree

### Overview

```mermaid
graph TD
    Q0{"🤔 Is the product<br/>working well?"}
    Q0 --> M["📊 02 Behavior Metrics<br/>How to measure?"]
    M --> Q1{"Where are<br/>the problems?"}
    Q1 --> S["🔍 02 Scenario Breakdown<br/>Which intents hurt most?"]
    S --> Q2{"Why are users<br/>unhappy?"}
    Q2 --> A["🔬 03 Semantic Attribution<br/>Root cause analysis"]
    Q2 --> F["💬 04 Follow-up Decomposition<br/>Hidden signal mining"]
    A --> Q3{"What should<br/>we fix first?"}
    F --> Q3
    Q3 --> R["🎯 05 Priority Matrix<br/>+ Monitoring System"]

    style Q0 fill:#6366f1,stroke:#4f46e5,color:#fff
    style Q1 fill:#f97316,stroke:#ea580c,color:#fff
    style Q2 fill:#e74c3c,stroke:#dc2626,color:#fff
    style Q3 fill:#2ecc71,stroke:#16a34a,color:#fff
    style M fill:#ede9fe,stroke:#6366f1,color:#333
    style S fill:#fff7ed,stroke:#f97316,color:#333
    style A fill:#fee2e2,stroke:#e74c3c,color:#333
    style F fill:#dbeafe,stroke:#3498db,color:#333
    style R fill:#dcfce7,stroke:#2ecc71,color:#333
```

### Branch 1 — "Is the product working well?" → Behavior Metrics

```mermaid
graph TD
    ROOT["📊 How to measure<br/>product quality?"]
    ROOT --> EXPLICIT["Explicit Signals"]
    ROOT --> IMPLICIT["Implicit Signals"]
    ROOT --> TIME["Time Dimension"]
    ROOT --> USER["User Dimension"]

    EXPLICIT --> LIKE["👍 Like rate<br/>frequency count / total convs"]
    EXPLICIT --> DISLIKE["👎 Dislike rate<br/>frequency count / total convs"]
    EXPLICIT --> RETRY["🔄 Retry rate<br/>implicit negative — user unsatisfied<br/>but didn't explicitly dislike"]
    EXPLICIT --> FOLLOWUP["💬 Follow-up rate<br/>ambiguous signal — good or bad?<br/>→ decomposed in notebook 04"]

    IMPLICIT --> SILENT["🤫 Silent rate<br/>method: total convs − convs with events<br/>why: silent users absent from event table<br/>ignoring them inflates satisfaction metrics"]

    TIME --> DAILY["Daily trends<br/>daily aggregation + multi-metric overlay<br/>purpose: detect spikes from model updates"]

    USER --> ACTIVE["Active days<br/>group by user_type + box plot<br/>heavy / casual / churned"]
    USER --> RETENTION["Retention curve<br/>day-level retention with eligible<br/>user filtering to avoid survivorship bias"]

    ROOT --> FINDING["📋 Findings:<br/>24% silent, 14% dislike, 19% retry<br/>like rate ≈ silent rate → even satisfied<br/>users don't always give feedback"]

    style ROOT fill:#6366f1,stroke:#4f46e5,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### Branch 2 — "Where are the problems?" → Scenario Breakdown

```mermaid
graph TD
    ROOT["🔍 Which scenarios<br/>have the most issues?"]

    ROOT --> PREREQ["Prerequisite: Intent Classification"]
    PREREQ --> IC1["This project: pre-classified field<br/>simulates classification model output"]
    PREREQ --> IC2["Production: 4 methods<br/>keyword rules / LLM / BERT fine-tune / manual"]

    ROOT --> METHOD["How to break down?"]
    METHOD --> CROSS["Cross-tabulation<br/>pd.crosstab intent × event_type<br/>normalize='index' because conv volume<br/>differs per scenario — use ratios not counts"]
    METHOD --> RANK["How to rank?<br/>dissatisfaction = dislike + retry<br/>why merge? retry is implicit negative"]

    ROOT --> FINDING["📋 Findings"]
    FINDING --> F1["🔴 Code Gen: 45.1%<br/>retry rate 30% drives dissatisfaction"]
    FINDING --> F2["🔴 Knowledge QA: 39.3%<br/>dislike rate 24% is highest"]
    FINDING --> F3["🟡 Data Analysis: 39.4%"]
    FINDING --> F4["🟡 Creative Writing: 25.7%"]
    FINDING --> F5["🟢 Translation: 19.8%"]
    FINDING --> F6["🟢 Chitchat: 9.7%<br/>65% silent but expected for scenario"]

    style ROOT fill:#f97316,stroke:#ea580c,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### Branch 3 — "Why are users unhappy?" → Semantic Attribution

```mermaid
graph TD
    ROOT["🔬 Root cause analysis<br/>for 7,363 bad cases"]

    ROOT --> TAXONOMY["Attribution Taxonomy — 5 categories"]
    TAXONOMY --> T1["🟥 Hallucination<br/>fabricated facts / people / papers"]
    TAXONOMY --> T2["🟧 Off-topic<br/>misunderstood user intent"]
    TAXONOMY --> T3["🟪 Refusal<br/>over-safety, refused answerable questions"]
    TAXONOMY --> T4["🟩 Format mismatch<br/>wanted code, got text explanation"]
    TAXONOMY --> T5["📘 Quality insufficient<br/>right direction but too vague — catch-all"]

    ROOT --> PATHA["Path A: Keyword Rules<br/>+ Manual Review"]
    PATHA --> PA1["Step 1: Priority rule chain<br/>refusal keywords → hallucination signals<br/>→ format check → off-topic signals<br/>→ fallback: quality insufficient"]
    PATHA --> PA2["Step 2: Manual review<br/>export CSV → annotator checks rule output<br/>focus: reclassify catch-all cases<br/>QC: dual annotation + Cohen's Kappa"]
    PATHA --> PA3["Limitation: 71% falls to catch-all<br/>translation and chitchat: 100% catch-all"]

    ROOT --> PATHB["Path B: LLM Auto-labeling"]
    PATHB --> PB1["Method: zero-shot prompt classification<br/>input: query + response → prompt template<br/>→ returns attribution category"]
    PATHB --> PB2["Model: gpt-4o-mini<br/>temperature=0 for reproducibility"]
    PATHB --> PB3["Validation: use Path A manual<br/>labels as golden set"]

    ROOT --> FINDING["📋 Findings"]
    FINDING --> F1["Knowledge QA → hallucination 22%"]
    FINDING --> F2["Data Analysis → format mismatch 65%"]
    FINDING --> F3["Creative Writing → refusal 23%"]
    FINDING --> F4["Code Gen → refusal 12% + format 13%"]

    style ROOT fill:#e74c3c,stroke:#dc2626,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### Branch 4 — "Are follow-ups good or bad?" → Signal Decomposition

```mermaid
graph TD
    ROOT["💬 Follow-up rate 19.4%<br/>is this good or bad?"]

    ROOT --> WHY["Why decompose?<br/>follow-up is ambiguous — could be<br/>curiosity OR frustration"]

    ROOT --> HOW["How to classify?"]
    HOW --> CORRECTIVE["🔴 Corrective<br/>keyword detection: negation words,<br/>contradiction signals, correction phrases"]
    HOW --> EXPLORATORY["🟢 Exploratory<br/>default: no corrective keywords found"]
    HOW --> PROD["Production improvement:<br/>replace keywords with LLM<br/>sentiment/intent classification"]

    ROOT --> SCENE["Scene correlation<br/>cross-analysis: intent × followup_type"]
    ROOT --> BEHAVIOR["Behavior correlation<br/>temporal event chain:<br/>sort by timestamp → get first event<br/>after follow-up"]

    ROOT --> FINDING["📋 Findings"]
    FINDING --> F1["Knowledge QA: 64% corrective 🚨<br/>confirms hallucination problem from 03"]
    FINDING --> F2["Creative Writing: only 10% corrective<br/>follow-ups are mostly positive exploration"]
    FINDING --> F3["Insight: corrective follow-ups can serve<br/>as automated quality monitoring signal"]

    style ROOT fill:#3498db,stroke:#2563eb,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### Branch 5 — "What should we fix first?" → Priority Matrix

```mermaid
graph TD
    ROOT["🎯 Prioritization"]
    ROOT --> LOGIC["Ranking logic:<br/>dissatisfaction rate × corrective rate<br/>× attribution severity"]

    ROOT --> P0["🔴 P0 — Fix immediately"]
    P0 --> P0A["Knowledge QA<br/>root cause: hallucination<br/>action: integrate RAG + confidence detection<br/>when low confidence → flag as uncertain"]
    P0 --> P0B["Code Generation<br/>root cause: refusal + bad output<br/>action: code sandbox + prompt optimization<br/>give code directly, not just ideas"]

    ROOT --> P1["🟡 P1 — Short-term"]
    P1 --> P1A["Data Analysis<br/>root cause: format mismatch<br/>action: default to code-block output<br/>give solution first, ask questions later"]
    P1 --> P1B["Creative Writing<br/>root cause: over-safety<br/>action: relax safety for creative tasks<br/>style imitation ≠ copyright infringement"]

    ROOT --> P2["🟢 P2 — Monitor"]
    P2 --> P2A["Translation — stable, keep watching"]
    P2 --> P2B["Chitchat — high silence is expected"]

    ROOT --> MONITOR["📡 Monitoring System"]
    MONITOR --> MON1["Daily: DAU, retention,<br/>dislike rate, retry rate"]
    MONITOR --> MON2["Weekly: per-scenario dissatisfaction,<br/>attribution distribution, corrective rate"]
    MONITOR --> MON3["Alerts: dislike rate daily +5%<br/>hallucination share >15%<br/>any scenario dissatisfaction >50%"]

    style ROOT fill:#2ecc71,stroke:#16a34a,color:#fff
    style P0 fill:#fee2e2,stroke:#e74c3c,color:#333
    style P1 fill:#fef3c7,stroke:#f39c12,color:#333
    style P2 fill:#dcfce7,stroke:#2ecc71,color:#333
```

---

## 📁 Project Structure

```
ai-chat-analytics/
├── README.md                          # English (default)
├── README_CN.md                       # 中文版
├── requirements.txt
├── data/
│   ├── users.csv                      # 500 simulated users
│   ├── conversations.csv              # ~22K conversations
│   └── events.csv                     # ~20K behavioral events
├── notebooks/
│   ├── 01_data_generation.ipynb       # Synthetic data with realistic patterns
│   ├── 02_behavior_analysis.ipynb     # Multi-dimensional behavior metrics
│   ├── 03_semantic_attribution.ipynb  # Bad case root-cause analysis
│   ├── 04_followup_signal.ipynb       # Follow-up type decomposition
│   └── 05_insights_summary.ipynb      # Actionable recommendations
├── src/
│   └── utils.py                       # Shared utilities
└── output/
    └── bad_cases_for_labeling.csv     # Exported for manual annotation
```

## ⚡ Quick Start

```bash
# Clone
git clone https://github.com/gh59/ai-chat-analytics.git
cd ai-chat-analytics

# Install dependencies
pip install -r requirements.txt

# Run notebooks in order
cd notebooks
jupyter notebook 01_data_generation.ipynb
```

## 📦 Requirements

```
pandas
numpy
plotly
jupyter
```

## 🧩 Design Philosophy

1. **No pre-labeled data** — Intent classification and attribution labels are NOT baked into synthetic data. The pipeline demonstrates how to derive them, matching real-world conditions.

2. **Event stream, not single labels** — One conversation can trigger multiple events (retry → like). This mirrors real product instrumentation.

3. **Silence is data** — 24% of conversations have zero feedback. Ignoring them biases your quality metrics upward.

4. **Follow-ups are ambiguous** — A high follow-up rate could mean high engagement OR high frustration. You must decompose before interpreting.

5. **Attribution needs multiple paths** — Keyword rules are fast but shallow; LLM labeling is smart but costly; manual labeling is accurate but unscalable. A production system needs all three.
