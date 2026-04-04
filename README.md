# build-evenhub-app

A Claude Code skill for building production-quality Even Realities G2 apps.

---

## What is this?

A comprehensive Claude Code slash command that guides you through building EvenHub plugins for the [Even Realities G2](https://evenrealities.com) smart glasses. It encodes the full design system, SDK API, UX patterns, performance rules, and QA checklist so any developer can produce apps that pass quality review on first submission.

No guesswork. No back-and-forth with docs. One command, complete app.

## What's inside

- **SDK API reference** -- container types, device APIs, event system, lifecycle hooks
- **Design system** -- even-toolkit colors, typography, spacing tokens from Design Library 3.0
- **OS display constraints** -- 576x288 monochrome canvas, container model, max 4 containers per type per page
- **UX patterns** -- mandatory exit mechanism, glasses-first navigation, R1 ring input handling
- **Performance optimization** -- render queue batching, keep-alive, memory management
- **File-based planning** -- 7 EvenHub-specific phases with persistent task tracking
- **QA self-check** -- built from the official Even Realities review criteria

## Installation

### User-level (all projects)

```bash
cp build-evenhub-app.md ~/.claude/commands/
```

### Project-level (one repo)

```bash
mkdir -p .claude/commands && cp build-evenhub-app.md .claude/commands/
```

## Usage

```
/build-evenhub-app <describe your app idea>
```

**Examples:**

```
/build-evenhub-app a pomodoro timer with custom intervals
/build-evenhub-app an ebook reader with bookmarks
/build-evenhub-app a real-time stock ticker for my watchlist
/build-evenhub-app a cycling HUD with speed, cadence, and heart rate
```

## What happens when you run it

The skill creates persistent planning files (`task_plan.md`, `findings.md`, `progress.md`) in your project root, then works through 7 phases:

| Phase | What it does |
|-------|-------------|
| 1. Requirements & Architecture | Clarifies features, maps pages, identifies device APIs needed |
| 2. Project Scaffold | Creates project structure, configures Vite + TypeScript, sets up app.json |
| 3. SDK Integration | Wires up container rendering, event listeners, device APIs |
| 4. OS Display Layer | Implements 576x288 canvas layout, applies design tokens, handles text fitting |
| 5. App-side UI | Builds the companion phone UI (settings, state sync, onboarding) |
| 6. Performance | Applies render queue batching, optimizes keep-alive, audits memory |
| 7. QA & Submission | Runs the full review checklist, fixes issues, produces a package ready for `evenhub pack` |

Output: a complete app ready to pack and submit to the EvenHub developer portal.

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Node.js >= 20
- [@evenrealities/even_hub_sdk](https://www.npmjs.com/package/@evenrealities/even_hub_sdk)

## Links

- **Dev Portal**: [hub.evenrealities.com](https://hub.evenrealities.com)
- **SDK on npm**: [@evenrealities/even_hub_sdk](https://www.npmjs.com/package/@evenrealities/even_hub_sdk)
- **Even Realities GitHub**: [github.com/even-realities](https://github.com/even-realities)

## License

[MIT](LICENSE)

---

**Built for the EvenHub developer community by [Even Realities](https://evenrealities.com).**
