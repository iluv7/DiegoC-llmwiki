# LLM Wiki

> A personal knowledge base scaffold built on Karpathy's LLM Wiki concept — let LLMs incrementally build and maintain a persistent, structured, ever-growing wiki.

## Core Idea

**Traditional RAG**: each query assembles answers from raw document chunks — no accumulation across 100 queries on the same topic.  
**LLM Wiki**: the LLM digests, structures, and cross-references during ingest — the wiki grows denser and smarter with every addition.

> "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase." — Andrej Karpathy
>
> "You and the LLM co-evolve this over time." — Karpathy

### Three-Layer Architecture

| Layer | Description |
|---|---|
| **Raw sources** | Curated source documents (articles, papers, images, etc.) — LLM read-only |
| **Wiki** | LLM-generated structured markdown files (summaries, entity pages, concept pages, comparisons, index) — LLM fully owns and maintains this layer |
| **Schema** | CLAUDE.md and command files that define structure, conventions, and workflows |

## Directory Structure

```
vault/
├── CLAUDE.md                  # Schema — rules and conventions for the LLM
├── .claude/commands/          # Slash command definitions
│   ├── ingest.md              # /ingest — compile raw sources into wiki pages
│   ├── query.md               # /query — answer questions from the wiki
│   ├── lint.md                # /lint  — wiki health check
│   └── save.md                # /save  — archive valuable conversations
├── raw/                       # Raw sources — LLM read-only
│   ├── articles/              #   blog posts, papers, docs
│   ├── videos/                #   video/podcast transcripts
│   └── assets/                #   images, attachments
└── wiki/                      # Knowledge base — LLM maintained
    ├── sources/               #   source summary pages (one per raw file)
    ├── entities/              #   entity pages (tools, organizations, people)
    ├── concepts/              #   concept pages (methods, patterns, terms)
    ├── synthesis/             #   cross-source analysis and conclusions
    ├── outputs/               #   archived Q&A
    ├── index.md               #   content index (tldr summaries for retrieval)
    └── log.md                 #   operation log (append-only, grep-friendly)
```

## Getting Started

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup) CLI
- [Obsidian](https://obsidian.md/) (optional, for browsing wiki and Graph View)

### 1. Clone

```sh
git clone <repo-url> my-vault
cd my-vault
```

### 2. Open in Obsidian (Optional)

Open folder as vault → select `my-vault/`

### 3. Drop in Your First Article

Place articles into `raw/articles/` (`.md` files supported)

### 4. Launch Claude Code

```sh
cd my-vault
claude
```

### 5. Run Ingest

Type `/ingest` in Claude Code. The LLM will:

- Read articles in `raw/articles/`
- Create source summary pages in `wiki/sources/`
- Extract entities and concepts, create/update entity & concept pages
- Build cross-references between pages
- Update `index.md` and `log.md`

### 6. Ongoing

- `/query <question>` — answer based on the wiki
- `/lint` — periodic wiki health check
- `/save` — archive valuable conversations into wiki pages

## Commands

| Command | Purpose | Frequency |
|---------|---------|-----------|
| `/ingest` | Compile new raw sources into structured wiki pages | Per source |
| `/query` | Answer questions from the wiki with `[[citations]]` and confidence | Anytime |
| `/lint` | Scan for broken links, orphans, contradictions, stale content | Weekly/monthly |
| `/save` | Archive valuable analysis or Q&A into the wiki | As needed |

## Core Principles

1. **Raw is immutable** — LLM reads raw/ only, never modifies originals
2. **Bidirectional links** — both inline `[[wikilinks]]` and frontmatter `related` fields
3. **kebab-case filenames** — lowercase with hyphens
4. **Surface contradictions** — flag conflicts with callouts, never silently flatten them
5. **One ingest can touch 10-15 pages** — new knowledge must connect to existing knowledge

## Customization

CLAUDE.md and commands are not set in stone. As you use the wiki, refine them:

> "Merge entity and concept into a single topics directory, update CLAUDE.md and all related pages"

## References

- [karpathy/llm-wiki.md](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — original concept
- [Obsidian](https://obsidian.md/) — local markdown knowledge base
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup) — LLM CLI
