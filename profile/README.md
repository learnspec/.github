# LearnSpec

**Open standards for structured learning content in the AI era.**

LearnSpec is a suite of complementary, Markdown-based formats that together cover the full lifecycle of learning content — instruction, assessment, review, paths, credentials — with no proprietary tools and no lock-in.

## The Suite

| Format | Extension | Role |
|---|---|---|
| **[LearnMD](https://github.com/learnspec/learnmd)** | `.learn.md` | Instructional content (lessons, examples, summaries) |
| **[QuizMD](https://github.com/learnspec/quizmd)** | `.quiz.md` | Assessments and quizzes |
| **[FlashMD](https://github.com/learnspec/flashmd)** | `.flash.md` | Flashcards for spaced repetition |
| **[NuggetMD](https://github.com/learnspec/nuggetmd)** | `.nugget.md` | Micro-learning concepts for spaced repetition |
| **[ExerciseMD](https://github.com/learnspec/exercisemd)** | `.exercise.md` | Exercises with solutions and grading rubrics |
| **[TrackMD](https://github.com/learnspec/trackmd)** | `.track.md` | Sequenced learning paths |
| **[DiagramMD](https://github.com/learnspec/diagrammd)** | `.diagram.md` | Diagram syntax (Mermaid, TikZ, Graphviz, abc, chess, vega-lite…) |
| **[AnimMD](https://github.com/learnspec/animmd)** | `.anim.md` | Step-reveal animations over vector scenes |
| **[MediaMD](https://github.com/learnspec/mediamd)** | `.media.md` | Sourced, licence-checked media catalogues |
| **[GlossaryMD](https://github.com/learnspec/glossarymd)** | `.glossary.md` | Definitions and key terms |
| **[CurriculumMD](https://github.com/learnspec/curriculummd)** | `.curriculum.md` | Reference frameworks and syllabi |
| **[BadgeMD](https://github.com/learnspec/badgemd)** | `.badge.md` | Micro-credentials (Open Badges 3.0 compatible) |
| **[CertMD](https://github.com/learnspec/certmd)** | `.cert.md` | Macro-credentials for completed tracks |
| **[ListenMD](https://github.com/learnspec/listenmd)** | `.listen.md` | Speech-only audio episode scripts (rendition format) |

## Design Principles

- **Markdown-first** — every file is valid Markdown, readable in any editor
- **File-native** — content lives in flat files, no database required
- **Git-native** — versionable, diffable, and mergeable like code
- **AI-native** — generatable and consumable by LLMs without special tooling
- **Gracefully degradable** — a LearnSpec file renders as readably as possible in any standard Markdown reader, with no prior knowledge of the spec
- **Progressively enriched** — Level 0 (plain Markdown) → Level 1 (YAML frontmatter) → Level 2 (special fenced blocks and directives), each a strict superset of the previous

## Cross-Format Interoperability

All formats share universal frontmatter fields (`lang`, `license`, `spec_version`, `created`, `updated`…) and two cross-format directives:

- **`!import ./file.learn.md`** — composition: inline another LearnSpec file
- **`!ref ./file.media.md`** — context: declare a dependency without inline rendering (media resolution, glossary highlighting…)

A learning corpus can be assembled, versioned, and published as a folder of plain `.md` files.

## Get Started

- 🌐 **[learnspec.org](https://learnspec.org)** — documentation site
- 📦 **[samples-learnspec](https://github.com/learnspec/samples-learnspec)** — example content across all formats
- 🐍 **pylearnspec** — Python parser & validator (private, coming soon)

## MCP Integration

LearnSpec formats are natively supported by the [Neuroneo MCP server](https://neuroneo.md), giving AI assistants tools to parse, validate, generate, and publish content across the suite.

## Contributing

LearnSpec is open and community-driven. Each format has its own repository — open an issue or pull request in the relevant one, or start a discussion in [learnspec.org](https://github.com/learnspec/learnspec.org) for cross-cutting topics.
