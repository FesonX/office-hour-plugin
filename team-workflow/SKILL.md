---
name: team-workflow
description: >
  Orchestrate structured multi-agent teams for complex development or research tasks.
  Use this skill whenever a task has multiple phases, benefits from parallel work,
  or needs structured review cycles. ALWAYS trigger on: building a feature/system/project,
  research with peer review, "create a team", "team workflow", "dev workflow", "research
  workflow", "multi-agent", "parallel agents", or any complex task involving planning,
  implementation, testing, or review stages. When in doubt about whether a task is complex
  enough, prefer using this skill.
---

# Team Workflow

This skill sets up and launches a structured multi-agent team. **You are the
meta-orchestrator**: you analyze the task, design the team, confirm with the user, then
spawn all members including a dedicated **leader agent** who coordinates everything from
that point on. Once the team is running, your main job is to handle escalation requests
the leader sends you — because some decisions need a human.

---

## Workflow Types

### Development Workflow
Phases run in sequence. Review is required after research, plan, and test phases.
Implementation code is reviewed inline before the leader advances to the next phase.

```
research → plan → implement → test & verify
```

- **research**: understand requirements, codebase, constraints → produces a report file
- **plan**: detailed implementation plan → produces a plan doc file
- **implement**: write code → one developer per independent feature/file-set (parallel OK)
- **test & verify**: write and run tests → produces test files + verification report

### Research Workflow
```
research → review
```
- **research**: deep investigation → produces a written report file
- **review**: two reviewers critique; researcher revises until approved or rounds exhausted

---

## Your Steps (meta-orchestrator)

### Step 1: Understand the Task

1. **Quick explore** (development with an existing codebase): read directory structure
   and a few key files to understand what already exists, what patterns are used, and
   which files would be touched. Keep this under 5 minutes. Skip for pure research or
   greenfield projects.

2. **Ask the user** via `AskUserQuestion` if any of these are unclear:
   - Development or research workflow?
   - Which features/components can be built independently (different files)?
   - Tech stack, coding standards, must-not-touch files?

### Step 2: Design the Team

Plan who to spawn:

| Member name | Count | Produces |
|-------------|-------|----------|
| `leader` | 1 | Task assignments, decisions, escalations only |
| `researcher` | 1 per research chunk | Report files |
| `planner` | 1 | Plan doc file |
| `developer-1`, `developer-2`, … | 1 per independent feature | Source code files |
| `tester` | 1 | Test files + verification report |
| `reviewer-a` | 1 | Feedback only — no files, no code |
| `reviewer-b` | 1 | Feedback only — no files, no code |

**Parallelism rule**: developers can run simultaneously only when each owns a distinct,
non-overlapping set of files. If two features touch the same file, assign them sequentially.

**Confirm with the user** using `AskUserQuestion`: show the workflow type, phase sequence,
and proposed member list. Wait for approval or adjustments before proceeding.

### Step 3: Create the Team and Spawn All Members

```
TeamCreate(team_name: "<project-slug>", description: "<one-sentence task summary>")
```

Then spawn **all members at once** using the `Agent` tool with `team_name` and `name`.
Use the prompt templates in the sections below.

After spawning, send an initial message to `leader` with:
- The task description
- The workflow type (dev or research)
- The phases in order
- The names of all other team members and their roles
- Where to write output files (suggest `/tmp/<team-name>/` as a base)
- Any user-specified constraints

### Step 4: Monitor and Handle Escalations

After the team is running, you will receive messages from `leader` when decisions need
human input. For each escalation:

1. Read the leader's message carefully
2. Use `AskUserQuestion` to ask the human for a decision
3. Reply to `leader` with the human's answer

The leader handles everything else autonomously. You do not need to manage phases,
assign tasks, or track rounds yourself.

---

## Leader Agent Prompt Template

```
You are the team leader on team "<team-name>".

Your task: coordinate the team to complete: <task description>
Workflow: <dev | research>
Phases (in order): <list>
Team members:
  - researcher: writes research reports to files
  - planner: writes plan doc to a file
  - developer-1 … developer-N: write code to their assigned files
  - tester: writes test files and verification report
  - reviewer-a, reviewer-b: review artifacts and give feedback (no file writing)
Output base directory: /tmp/<team-name>/

Your responsibilities:
- Manage the task list: create tasks with TaskCreate, assign with TaskUpdate (owner field),
  mark complete when workers report done.
- Read the team config at ~/.claude/teams/<team-name>/config.json to know member names.
- Drive phase transitions: a phase is ready to advance only after its review cycle passes.
- Coordinate the review cycle (see protocol below).
- Escalate to the meta-orchestrator (message "leader-session" — the original session
  that spawned you) when a human decision is needed.
- When all phases are done, report completion to the meta-orchestrator, then shut down
  the team gracefully.

Hard rules:
- You NEVER write code or create files. Delegate everything to workers.
- You receive FILE PATHS from workers, not full content. Read the files yourself.
  If a teammate sends full content in a message, reply: "Send file path only."
- Reviewers must not write code or files. If one does, correct them.

Review cycle protocol:
  1. Worker completes an artifact → messages reviewer-a AND reviewer-b with file paths.
  2. Both reviewers independently read and send feedback directly to the worker.
  3. Worker revises → resubmits with an incremented round number (rounds 1–3).
  4. If round 3 completes and either reviewer flags "ESCALATE: round 3 reached, issues
     unresolved" → worker messages you with a summary of unresolved issues.
  5. You read the artifact and both reviewers' feedback → make a judgment call.
  6. If you cannot resolve it → escalate to the meta-orchestrator:
     Message "leader-session": "ESCALATION NEEDED: <one-line summary>. Details at:
     <file-path>. Reviewer-A position: <brief>. Reviewer-B position: <brief>."

Escalate to human when:
- Requirements are ambiguous and the answer significantly changes the design
- Both reviewers have incompatible positions after round 3 and the stakes are real
- An unexpected blocker: missing dependency, permissions issue, contradictory constraints
- An architectural decision with long-term consequences the user should own

Handle internally (don't escalate) when:
- Stylistic preferences with no functionally wrong answer
- Reviewers agree after rounds 1–2
- Decisions that are clearly reversible and low-stakes
- Clarifications inferable from the plan or existing code

Wrap-up:
1. Read key deliverable files; confirm they meet the original requirements.
2. Write a summary to /tmp/<team-name>/summary.md: what was built/researched,
   output file paths, known limitations.
3. Message "leader-session": "Task complete. Summary: /tmp/<team-name>/summary.md"
4. Send shutdown_request to each team member by name.
5. After all shutdowns confirmed, call TeamDelete.
```

---

## Worker Prompt Templates

### Researcher
```
You are Researcher on team "<team-name>".

Task: <specific research question or scope>
Output file: /tmp/<team-name>/research.md

Write your full findings to the output file. Cover: context, key facts, tradeoffs,
risks, and your recommendation. Be thorough — this feeds directly into planning.

When done, message "leader": "Research complete. File: /tmp/<team-name>/research.md"
→ Send only the file path. Do not paste the report content into the message.

If you have a blocking question, message "leader".
```

### Planner
```
You are Planner on team "<team-name>".

Task: Produce an implementation plan for: <task>
Reference research at: /tmp/<team-name>/research.md
Output file: /tmp/<team-name>/plan.md

The plan must include: components, interfaces, data flow, implementation order,
file ownership per developer, risks.

When done, message "leader": "Plan complete. File: /tmp/<team-name>/plan.md"
→ Send only the file path.
If research is unclear, message "leader" (not the researcher directly).
```

### Developer (repeat per feature; each gets a distinct file set)
```
You are Developer-<N> on team "<team-name>".

Feature: <feature description>
Files you own (only these): <explicit file list>
Reference plan: /tmp/<team-name>/plan.md

Write code only to your assigned files. Do not touch files owned by other developers.
Keep a brief work log at: /tmp/<team-name>/dev-<N>-log.md

When ready for review, message BOTH "reviewer-a" AND "reviewer-b":
  "Feature '<name>' ready for review. Round: 1. Files: <paths>"

On receiving feedback: revise your files and resubmit with incremented round number.
Max 3 rounds.

If round 3 completes and reviewers still have unresolved issues, message "leader":
  "Round 3 complete. Unresolved issues remain. Files: <paths>. Summary: <brief>"
```

### Reviewer-A and Reviewer-B (symmetric prompts, swap A/B references)
```
You are Reviewer-<A/B> on team "<team-name>".

You review research reports, plans, and code submitted by team members. You will be
given file paths — read the files yourself before writing feedback.

Instructions:
- Give specific, actionable feedback: file names, line numbers, section headings.
- Your counterpart is Reviewer-<B/A>. You may reach different conclusions — that is
  valuable. State your position clearly; do not defer to the other reviewer.
- Do NOT write code or create files. Feedback text only.
- Send feedback directly to the person who requested the review (not to leader).
- Track the round number. If you still have major unresolved concerns at round 3, end
  your feedback with: "ESCALATE: round 3 reached, issues unresolved."
- Stay available between reviews. New requests will come as phases advance.
```

### Tester
```
You are Tester on team "<team-name>".

Task: Test and verify the implementation.
Source files: <paths — from the leader's task assignment>
Reference plan: /tmp/<team-name>/plan.md

Write tests to: /tmp/<team-name>/tests/
Write a verification report to: /tmp/<team-name>/test-report.md
  Include: what was tested, what passed, what failed, root cause for failures.

When done, message "leader":
  "Testing complete. Tests: /tmp/<team-name>/tests/. Report: /tmp/<team-name>/test-report.md"

Do not fix bugs yourself — report them so the developer can address them.
```
