# News

A multi-agent editorial pipeline that turns the daily news flow of a niche topic into
publication-ready materials: a short Telegram-style digest and a deep analytical longread.

The pipeline is defined with [goga](https://github.com/qarium) and parameterized by topic
profiles — the same machinery serves AI, dev, DevOps, QA, design and product topics.

## Quick start

```bash
pipx install goga

# run with a specific profile (default: ai)
goga pipeline generate -p <topic-name>

# examples
goga pipeline generate -p ai
goga pipeline generate -p qa
```

Note: a full run (digest + article) involves many LLM calls, including web-search
assistants — it consumes a noticeable amount of tokens.

## How it works

The `generate` pipeline (`NewsGenerator`) runs a sequence of stages:

```
search ──► merge ──► draft ──► digest   (short post, ~1300–2300 chars)
                          └─► article   (longread, ~7000–11000 chars)
```

| Stage | What happens |
|-------|--------------|
| `search` | An analyst scans the news flow, deduplicates stories, scores each one (impact / practical value / industry resonance, with a penalty for advertorials) and picks the TOP-10. |
| `merge` | Stories are clustered by technological and market trends (not by company) into a coherent analytical reference. |
| `draft` | A ready-to-publish draft written for the topic's target audience. |
| `digest` | Interactive stage: pick a delivery style (strict `punchup-writer` or humorous `standup-writer`), then a `managing-editor` gate runs an anti-slop audit. |
| `article` | Interactive stage: the longread team designs the architecture, researches the second layer of facts, grounds it in practice, adds tables/diagrams and assembles the final text. |

Each stage is backed by a cast of assistants (fact-checker, hype-filter, narrative-designer,
angle-generator, quality-gatekeeper, managing editor and others) defined in
`.goga/profiles/ai.yml`.

## Topic profiles

Profiles live in `.goga/profiles/`:

| Profile | Topic |
|---------|-------|
| `ai` *(default)* | artificial intelligence |
| `dev` | software development |
| `devops` | DevOps, administration and infrastructure operations |
| `qa` | QA testing and software quality assurance |
| `design` | digital product design (UI/UX) |
| `product` | product development and product management |

A profile sets the content/communication languages, the topic name, scoring hints
(key players, landmark events, metrics) and the target audience.

### Adding a new topic

1. Copy `.goga/profiles/_template.yml` to `.goga/profiles/<topic-name>.yml`
   (ASCII letters, digits, dots, hyphens).
2. Fill in at least `topic.name` — everything else falls back to neutral defaults.
3. Run the pipeline with the new profile (see below).

## Output artifacts

Results are written to (git-ignored):

```
.artifacts/<slug>/<issue date YYYY-MM-DD>/digest.md
.artifacts/<slug>/<issue date YYYY-MM-DD>/article.md
```

## Repository layout

```
.goga/
├── config.yml              # language, image, agent and pipeline settings
├── pipelines/generate.yml  # pipeline stages and prompts
├── workflows/generate.yml  # workflow definition
└── profiles/               # topic profiles (ai, dev, devops, qa, design, product)
```
