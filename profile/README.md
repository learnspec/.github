# LearnSpec

**Open standards for structured learning content in the AI era.**

LearnSpec defines two complementary, Markdown-based formats that together form a complete **teach → assess** stack — no proprietary tools, no lock-in.

| | LearnMD | QuizMD |
|---|---|---|
| **Purpose** | Instruction | Assessment |
| **Extension** | `.learn.md` | `.quiz.md` |
| **Core unit** | Module / Lesson | Question |

## Design Principles

- **Markdown-first** — valid Markdown, readable in any editor or viewer
- **Git-native** — versionable, diffable, and mergeable like code
- **AI-native** — generatable and consumable by LLMs without special tooling
- **Progressively enriched** — plain text (Level 0) up to rich metadata and interactive blocks (Level 2)
- **Composable** — `!import` directives to build learning paths from independent modules

## Rich Content Support

Both formats include built-in support for **LaTeX math**, **ABC music notation**, and **Penrose diagrams** — all rendered natively in compatible players, with graceful degradation elsewhere.

## Get Started

- 📖 **[LearnMD Specification](https://github.com/learnspec/learnspec.org/tree/main/learnmd)** — the instruction format
- 📝 **[QuizMD Specification](https://github.com/learnspec/learnspec.org/tree/main/quizmd)** — the assessment format
- 🌐 **[learnspec.org](https://learnspec.org)** — documentation site

## MCP Integration

LearnSpec formats are natively supported by the [Neuroneo MCP server](https://neuroneo.md), giving AI assistants tools to parse, validate, generate, and publish `.learn.md` and `.quiz.md` content directly.

## Contributing

LearnSpec is open and community-driven. Contributions, feedback, and discussion are welcome — open an issue or pull request in the [learnspec.org](https://github.com/learnspec/learnspec.org) repository.
