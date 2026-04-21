# Changelog

## [0.1.1] - 2026-04-21

### Changed
- Deep dialogue sessions now delegate all research and web fetches to sub-agents, keeping the main context lean across long discussions
- Research notes and summaries are saved to the project directory instead of `/tmp`, so the full session record is always accessible after the conversation ends
- Deep dialogue now uses `AskUserQuestion` to gather missing context before starting, rather than discovering gaps mid-dialogue
- Deep dialogue wraps up each session by listing all saved file paths so users know exactly where to review the process

## [0.1.0] - 2026-04-01

### Added
- Initial release of office-hour-plugin
- `team-workflow` skill for orchestrating structured multi-agent teams (leaders, developers, reviewers, researchers)
- Marketplace configuration and install guide
