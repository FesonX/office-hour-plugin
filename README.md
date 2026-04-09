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

### Quick install (recommended)

Add the marketplace and install the plugin with two commands inside Claude Code:

```
/plugin marketplace add FesonX/office-hour-plugin
/plugin install office-hour-plugin@fesonx
```

Then reload to activate:

```
/reload-plugins
```

### Install with the CLI

Run these outside of a Claude session:

```bash
claude plugin marketplace add FesonX/office-hour-plugin
claude plugin install office-hour-plugin@fesonx
```

### Installation scope

By default the plugin installs to your **user** scope (available across all projects). To install for a specific project instead:

```bash
# Share with your team via .claude/settings.json
claude plugin install office-hour-plugin@fesonx --scope project

# Personal use in this project only (gitignored)
claude plugin install office-hour-plugin@fesonx --scope local
```

### Update

```
/plugin marketplace update fesonx
```

### Uninstall

```
/plugin uninstall office-hour-plugin@fesonx
```

## Roadmap

More skills coming. This plugin will grow into a full office-hours workflow suite.

## License

MIT
