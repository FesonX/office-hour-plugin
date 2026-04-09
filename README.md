# office-hour-plugin

A Claude plugin for orchestrating structured multi-agent team workflows. Spin up coordinated teams of researchers, planners, developers, reviewers, and testers — with a dedicated leader agent that drives phases, manages review cycles, and escalates decisions to you when needed.

## Skills

### `team-workflow`

Launches a full multi-agent team for complex development or research tasks.

**Triggers on:** building a feature/system, research with peer review, "create a team", "team workflow", "dev workflow", "research workflow", or any task involving planning, implementation, testing, or review stages.

**Two workflow types:**

- **Development** — `research → plan → implement → test & verify`
  - Multiple developers can run in parallel when features touch different files
  - Every phase goes through a review cycle before advancing
- **Research** — `research → review`
  - One researcher produces a report; two independent reviewers critique it

**Key behaviors:**
- Always confirms team design with you before creating anything
- Spawns a dedicated leader agent (not the main session) who coordinates phases
- Two reviewers work simultaneously and can disagree — their "battle" surfaces better feedback
- Review cycles run up to 3 rounds; unresolved issues escalate to the leader, then to you
- Workers send file paths to reviewers/leader, never full content dumps
- Leader never writes code or files — coordination only

## Installation

Download the `.plugin` file from [Releases](https://github.com/FesonX/office-hour-plugin/releases) and install it in Claude Cowork or Claude Code.

## Roadmap

More skills coming. This plugin will grow into a full office-hours workflow suite.

## License

MIT
