---
name: codeskill
description: |
  AI project intelligence system. Manages .ai/ directory for rules, behaviours,
  sessions, incidents, memory, snapshots, and learning loops.

  Use when: starting a session, switching behaviour, logging an incident,
  saving feedback, reviewing past sessions, checking active hotfixes,
  managing snapshots, creating snippets/prompts.

  Proactively suggest when: user corrects AI behavior ("no", "don't", "wrong",
  "stop", "always", "never"), session ends, a mistake pattern repeats,
  starting work on unfamiliar code, user says "remember this" or "learn this".
argument-hint: "[command] [args] — e.g., init, sync, behaviour careful, session end, feedback, snapshot list"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
---

# codeskill — AI Project Intelligence System

You manage the `.ai/` directory in projects — rules, behaviours, memory, snapshots, sessions, incidents, and learning loops.

## Command Routing

Parse `$ARGUMENTS` to determine which command to execute:

```
ARGS = "$ARGUMENTS"
```

---

### If ARGS starts with "init"

Scaffold a new `.ai/` directory in the current project.

1. Check if `.ai/` already exists. If yes, ask user: "`.ai/` already exists. Overwrite? (This won't delete logs/memory)"
2. Detect project type by reading go.mod, package.json, pyproject.toml, etc.
3. Get the user's GitHub username: `git config user.name` and `git config user.email`
4. Create the directory structure:
   ```bash
   mkdir -p .ai/{behaviours,skills/{backend,frontend,infra},workflows,guides}
   mkdir -p .ai/{snippets,prompts,memory/{feedback,decisions},templates}
   mkdir -p .ai/{logs/{sessions,activity,incidents},plans/features,tasks,snapshots,handoffs}
   ```
5. Create Tier 0 files:
   - `.ai/RULES.md` — project-specific rules (start with generated code table, code style, commits)
   - `.ai/HOTFIXES.md` — empty with header and instructions
   - `.ai/taste.md` — project taste template with @username placeholders
6. Create Tier 1 files:
   - `.ai/ARCHITECTURE.md` — if project type detected, scan main dirs and describe structure
   - `.ai/STACK.md` — extract from dependency files (go.mod, package.json, etc.)
   - `.ai/PATTERNS.md` — empty with header
7. Create domain skill files based on project type:
   - Go project → `.ai/skills/backend/RULES.md`, `PATTERNS.md`, `SNIPPETS.md`
   - Node project → `.ai/skills/frontend/RULES.md`, `PATTERNS.md`, `SNIPPETS.md`
8. Create INDEX.md files for: `snippets/`, `prompts/`, `memory/`, `logs/sessions/`, `logs/activity/`, `logs/incidents/`, `plans/`, `snapshots/`
9. Create `.ai/TASKS.md` from `templates/tasks-index.md` (master checklist)
10. Create `.ai/memory/tasks/` directory and `.ai/memory/TASKS_LOG.md` from `templates/tasks-log.md`
11. Create behaviour files: `default.md`, `careful.md`, `review.md`, `debug.md`, `scaffold.md` with `behaviours/README.md`
12. Copy templates from this skill's `templates/` directory into `.ai/templates/`
12. Create `.ai/.gitignore`:
    ```
    logs/
    CONTINUATION.md
    memory/decisions/
    tasks/
    ```
13. Run the `sync` command to generate CLAUDE.md
14. Report: "`.ai/` initialized. Edit RULES.md and taste.md, then run `/codeskill sync`."

---

### If ARGS starts with "sync"

Regenerate CLAUDE.md from `.ai/` Tier 0 files.

1. Read these source files (skip any that don't exist):
   - `.ai/global/RULES.md` (if global dir exists)
   - `.ai/RULES.md`
   - `.ai/global/taste.md` (if global dir exists)
   - `.ai/taste.md`
   - `.ai/HOTFIXES.md`
   - `.ai/behaviours/default.md` (or current active behaviour)
2. Read `.ai/logs/sessions/INDEX.md` — get the last session entry for "Where We Left Off"
3. Read the CLAUDE.md template from this skill's `templates/claude-md.md`
4. Assemble CLAUDE.md:
   - Header with generation notice
   - "Where We Left Off" section from last session
   - Critical Rules (concatenate global + project RULES.md)
   - Active Hotfixes
   - Taste (global, then project overrides)
   - Active Behaviour
   - Memory Protocol instructions
   - Exploration Protocol instructions
   - Before Starting Work checklist
   - Auto-Behaviour Hints
   - Workflow Index (scan `.ai/workflows/` for files)
   - After Session instructions
   - Reference links
5. Write to project root `CLAUDE.md`
6. Report: "CLAUDE.md regenerated. Tier 0: ~[X] lines."

---

### If ARGS starts with "behaviour"

Switch or list behaviour profiles.

**If ARGS is "behaviour list":**
1. Glob `.ai/behaviours/*.md` (exclude README.md)
2. For each, read the frontmatter `name` and `description`
3. Print table: name, description, active?

**If ARGS is "behaviour [name]":**
1. Check `.ai/behaviours/<name>.md` exists
2. If not, list available behaviours and ask user to pick
3. Read the behaviour file
4. Run `sync` to regenerate CLAUDE.md with new active behaviour
5. Report: "Switched to `<name>` mode."

---

### If ARGS starts with "learn"

Mid-session learning — save AND apply immediately in the current session.

This is the user saying "learn this NOW" — not waiting for session end.

1. Parse what to learn from ARGS after "learn"
2. Classify the learning type:
   - Contains "always"/"never" → rule candidate
   - Contains "I prefer"/"I hate"/"I like" → taste candidate
   - Contains code/pattern → snippet/pattern candidate
   - Contains "remember"/"don't forget" → factual memory
   - Otherwise → correction/feedback
3. Classify scope (project/global_candidate) using standard logic
4. Check for duplicates in memory/feedback/ and RULES.md
   - If similar exists → update it instead of creating new
5. Save to `.ai/memory/feedback/<slug>.md` with `trigger: "user explicit (learn this)"`
6. **APPLY IMMEDIATELY** — add to your working context for the rest of this session:
   - Treat it as highest-priority rule for current conversation
   - Don't wait for session end
7. Check promotion: Is this the 3rd+ time this correction was saved?
   - If yes → propose adding to RULES.md or taste.md right now
   - If no → report occurrence count
8. If this looks like a taste update (style/naming/preference):
   - Ask: "Should I update taste.md with this? (It will apply to all future sessions)"
9. Report: "Learned and applying now: <summary>. Occurrence #<N>."

**Undo**: If user says "undo the last thing I told you to learn" or "I changed my mind about X":
1. Find the most recent feedback/ entry from this session (or matching X)
2. Delete it
3. Remove from in-session working context
4. Report: "Removed: <summary>. No longer applying."

---

### If ARGS starts with "session start"

Load context and report status.

1. Get user info: `git config user.name`, `git config user.email`
2. Check for `CONTINUATION.md` in project root:
   - If exists: read it, report "Resuming from: [task]", delete the file
3. Read `.ai/HOTFIXES.md` — report active hotfixes count
4. Read `.ai/logs/sessions/INDEX.md` — get last session date
5. Session diff: run `git log --oneline --since="<last_session_date>"` to see what changed
6. Check snapshot staleness: for each snapshot in `.ai/snapshots/INDEX.md`, run `git diff --name-only <hash> HEAD -- <globs>`
7. Report:
   ```
   Session started for @<username>
   Last session: <date> — <goal>
   Since then: <N> commits, <files changed>
   Hotfixes active: <N>
   Snapshots: <N> fresh, <N> stale
   ```

---

### If ARGS starts with "session end"

Write session summary and activity log.

1. Get user info from git config
2. Ask user (or infer from conversation): "What was the goal of this session?"
3. Scan the conversation for:
   - Files changed (from tool calls)
   - Decisions made
   - Corrections received
   - Mistakes / incidents
4. Generate timestamp: `date +%Y-%m-%d-%H%M`
5. Write session log to `.ai/logs/sessions/<timestamp>.md` using template:
   ```markdown
   # Session: <timestamp>

   **User**: <name> (<email>)
   **GitHub**: @<username>
   **Goal**: <goal>
   **Outcome**: <completed|partial|failed>
   **Behaviour**: <active behaviour>
   **Files Changed**: <count>

   ## Actions
   - <bullet list of what was done>

   ## Decisions Made
   - <decisions, linked to memory/decisions/ if saved>

   ## Corrections Received
   - <corrections, linked to memory/feedback/ if saved>

   ## Mistakes
   - <any mistakes or incidents>
   ```
6. Write activity log to `.ai/logs/activity/<timestamp>.md` using compact format:
   ```markdown
   # Activity: <timestamp>

   **User**: <name> (<email>)
   **GitHub**: @<username>
   **Behaviour**: <active>
   **Prompt Count**: <N>

   ---

   ## #1 — <time>
   "<user prompt>"
   → Loaded: <workflow> | Files: <files> | Correction: <yes/no>

   ## #2 — <time>
   ...
   ```
7. Update `.ai/logs/sessions/INDEX.md` — append one-line entry
8. Update `.ai/logs/activity/INDEX.md` — append one-line entry
9. Update "Where We Left Off" and run `sync` to regenerate CLAUDE.md
10. Check promotion thresholds:
    - Any feedback item with 3+ occurrences? → suggest RULES.md update
    - Any hotfix 30+ days without recurrence? → suggest graduation/archive
11. Report: "Session logged. <N> corrections saved, <N> decisions logged."

---

### If ARGS starts with "continue"

Load emergency resume file.

1. Check for `CONTINUATION.md` in project root
2. If not found: "No continuation file found. Start a fresh session with `/codeskill session start`."
3. If found: read and display it, then delete it
4. Report: "Loaded resume point. Continuation file deleted."

---

### If ARGS starts with "feedback"

Save a user correction to memory.

1. Get user info from git config
2. Parse the correction from ARGS (after "feedback"), or ask user to describe it
3. Classify:
   - Does it reference project-specific files/paths? → `scope: project`
   - Is it about general coding/AI behaviour? → `scope: global_candidate`
4. Check for duplicates: grep `.ai/memory/feedback/` for similar content
   - If similar exists: update the existing entry instead of creating new
5. Generate filename from correction summary (slugified)
6. Write to `.ai/memory/feedback/<slug>.md`:
   ```markdown
   ---
   date: <today>
   type: correction
   user: "@<username>"
   scope: <project|global_candidate>
   trigger: "<what AI did wrong>"
   ---

   <correction content>
   ```
7. Update `.ai/memory/INDEX.md`
8. Apply immediately in current session (add to working context)
9. If scope is `global_candidate`: "This looks global (no project-specific refs). Consider adding to your global repo."
10. Report: "Saved correction to memory/feedback/<slug>.md. Applying for this session."

---

### If ARGS starts with "decide"

Log an AI decision to memory.

1. Get user info from git config
2. Parse decision title and context from ARGS
3. Ask user if not provided:
   - What was decided?
   - What alternatives were considered?
   - Why this choice?
4. Classify scope (same logic as feedback)
5. Write to `.ai/memory/decisions/<slug>.md`:
   ```markdown
   ---
   date: <today>
   type: decision
   user: "@<username>"
   scope: <project|global_candidate>
   context: "<what prompted this decision>"
   ---

   # <Decision Title>

   **Decision**: <what was decided>
   **Alternatives**: <what else was considered>
   **Why**: <reasoning>
   **Trade-off**: <what we gave up>
   **Outcome**: pending review
   ```
6. Update `.ai/memory/INDEX.md`
7. Report: "Decision logged to memory/decisions/<slug>.md"

---

### If ARGS starts with "incident"

Create an incident report.

1. Get user info from git config
2. Parse description from ARGS, or ask user to describe what went wrong
3. Check `.ai/logs/incidents/INDEX.md` for similar past incidents (same category)
4. Count recurrences
5. Generate timestamp filename
6. Write to `.ai/logs/incidents/<date>-<slug>.md`:
   ```markdown
   # Incident: <short description>

   **Date**: <today>
   **User**: @<username>
   **Severity**: <low|medium|high|critical>
   **Category**: <generated-code|db-access|graphql|config|deployment|testing|other>
   **Rule Violated**: <ref or "none — new rule needed">
   **Recurrence**: <1st|2nd|Nth>

   ## What Happened
   <description>

   ## Root Cause
   <why the AI made this mistake>

   ## Detection Signal
   <how to catch this next time>

   ## Prevention
   - [ ] Added to HOTFIXES.md (if recurrence >= 2)
   - [ ] Updated relevant workflow/rule
   - [ ] Saved correction to memory/feedback/
   ```
7. Update `.ai/logs/incidents/INDEX.md`
8. If recurrence >= 2: auto-add to `.ai/HOTFIXES.md` and run `sync`
9. Report: "Incident logged. Recurrence: <N>." + hotfix promotion if applicable

---

### If ARGS starts with "hotfix"

**If ARGS is "hotfix add [description]":**
1. Parse description
2. Assign next HF-NNN number (read HOTFIXES.md for highest)
3. Append to `.ai/HOTFIXES.md`
4. Run `sync` to update CLAUDE.md
5. Report: "Added HF-<NNN> to HOTFIXES.md."

**If ARGS is "hotfix review":**
1. Read `.ai/HOTFIXES.md`
2. For each hotfix, check `.ai/logs/incidents/INDEX.md` for last recurrence date
3. Flag hotfixes with no recurrence in 30+ days
4. For each stale hotfix, ask user: "Graduate to RULES.md, or archive?"
5. Apply changes, run `sync`

---

### If ARGS starts with "snapshot"

**If ARGS is "snapshot list":**
1. Read `.ai/snapshots/INDEX.md`
2. For each entry, run: `git diff --name-only <git_hash> HEAD -- <watched_globs>`
3. Print table: topic | last updated | status (fresh/stale) | files changed

**If ARGS is "snapshot check [topic]":**
1. Read `.ai/snapshots/<topic>.md` frontmatter
2. Run: `git diff --name-only <git_hash> HEAD -- <watched_globs>`
3. Report: "fresh" or "stale (N files changed: ...)"

**If ARGS is "snapshot refresh [topic]":**
1. Read `.ai/snapshots/<topic>.md` frontmatter for `files_watched`
2. Spawn an Explore agent scoped to those file globs
3. Write exploration findings to `.ai/snapshots/<topic>.md` with fresh git_hash from `git rev-parse HEAD`
4. Update `.ai/snapshots/INDEX.md`
5. Report: "Snapshot <topic> refreshed."

**If ARGS is "snapshot refresh --all":**
1. Run snapshot list logic, find all stale snapshots
2. For each stale one, run snapshot refresh

**If ARGS is "snapshot create [topic]":**
1. Ask user: "What files/dirs should this snapshot cover?" (get globs)
2. Spawn Explore agent for that area
3. Write `.ai/snapshots/<topic>.md` with frontmatter: topic, description, git_hash, files_watched
4. Update `.ai/snapshots/INDEX.md`
5. Report: "Snapshot <topic> created."

---

### If ARGS starts with "snippet"

**If ARGS is "snippet [name]":**
1. Search `.ai/snippets/INDEX.md` for matching name/tags
2. If found: display it
3. If not found: ask user for code, tags, usage context
4. Write to `.ai/snippets/<name>.md`:
   ```markdown
   # Snippet: <Name>

   **Tags**: <tag1, tag2>
   **Used in**: <context>

   ## Code
   ```<lang>
   <code>
   ```

   ## When to Use
   <description>
   ```
5. Update `.ai/snippets/INDEX.md`

---

### If ARGS starts with "prompt"

**If ARGS is "prompt [name]":**
1. Search `.ai/prompts/INDEX.md` for matching name
2. If found: display it
3. If not found: ask user for template text, tags, workflow reference
4. Write to `.ai/prompts/<name>.md`:
   ```markdown
   # Prompt: <Name>

   **Tags**: <tag1, tag2>
   **Workflow**: <workflow reference or "none">
   **Behaviour**: <recommended behaviour>

   ## Template
   <prompt template text with [PLACEHOLDERS]>

   ## Variants
   <optional variant prompts>
   ```
5. Update `.ai/prompts/INDEX.md`

---

### If ARGS starts with "plan"

Manage feature plans. Plans live in `.ai/plans/features/`.

**If ARGS is "plan [title]"** — Create a new feature plan:
1. Parse title from ARGS, generate slug (kebab-case)
2. Ensure `.ai/plans/PLAN.md` exists — if not, create from `templates/plan-index.md`
3. Ensure `.ai/plans/features/` directory exists
4. Ask user for: goal, non-goals, requirements with priorities
5. Write to `.ai/plans/features/<slug>-plan.md` using `templates/plan.md`
6. Update `.ai/plans/PLAN.md` — add entry to Active Plans table
7. Report: "Plan created at plans/features/<slug>-plan.md"

**If ARGS is "plan list"** — Show all plans:
1. Read `.ai/plans/PLAN.md`
2. Display active and completed tables

**If ARGS is "plan archive [slug]"** — Archive a completed plan:
1. Move `.ai/plans/features/<slug>-plan.md` to `.ai/memory/plans/<slug>-plan.md`
2. Move entry from Active to Completed in PLAN.md
3. Report: "Plan archived to memory/plans/"

---

### If ARGS starts with "task"

Manage the task system. Tasks live as individual files in `.ai/tasks/` with a master checklist at `.ai/TASKS.md`.

**If ARGS is "task add [description]"** — Create a new task:
1. Parse description and infer priority (or ask user)
2. Determine next ID: scan `.ai/tasks/` and `.ai/memory/tasks/` for highest `task{N}_*.md`, next = N+1
3. Generate slug from description (kebab-case)
4. Write `.ai/tasks/task{id}_{slug}.md`:
   ```markdown
   ---
   id: {id}
   title: '<title>'
   status: pending
   priority: <high|medium|low>
   feature: '<feature or plan name if relevant>'
   dependencies: []
   created_at: "<timestamp>"
   ---

   ## Description
   <what needs to be done>

   ## Details
   - <specific requirements or steps>

   ## Test Strategy
   <how to verify this is done>
   ```
5. Update `.ai/TASKS.md` — append entry:
   `- [ ] **ID {id}: {Title}** (Priority: {priority})`
   `> {Description}`
6. Report: "Task {id} created: {title}"

**If ARGS is "task list"** — Show all tasks:
1. Read and display `.ai/TASKS.md`

**If ARGS is "task start [id]"** — Start working on a task:
1. Read task file, check dependencies are all completed
2. If dependencies not met → report which are blocking
3. **Complexity check** — before starting, assess if the task needs expansion:
   - Involves changes across 3+ unrelated files/modules?
   - Has 5+ acceptance criteria or detail items?
   - Would take more than ~2-3 hours of focused work?
   - Has high uncertainty or multiple distinct outcomes?
   If YES to 2+ of these → recommend expansion:
   - Propose sub-tasks with titles, descriptions, dependencies
   - Ask user: "This task looks complex. Expand into sub-tasks?"
   - If user agrees → create sub-tasks as `task{id}.1`, `task{id}.2`, etc.
   - Update parent task's Details section with sub-task list
   - If user declines → proceed normally
4. Update task YAML: `status: inprogress`, `started_at: <timestamp>`
5. Update `.ai/TASKS.md` entry: `[ ]` → `[-]`
6. Report: "Started task {id}: {title}"

**If ARGS is "task done [id]"** — Mark task completed:
1. If task has `## Test Strategy` → ask user: "Run tests or mark complete?"
2. Update task YAML: `status: completed`, `completed_at: <timestamp>`
3. Update `.ai/TASKS.md` entry: `[-]` → `[x]`
4. Report: "Completed task {id}: {title}"

**If ARGS is "task fail [id] [reason]"** — Mark task failed:
1. Update task YAML: `status: failed`, `error_log: <reason>`, `completed_at: <timestamp>`
2. Update `.ai/TASKS.md` entry → `[!] ... (Failed)`
3. Report: "Failed task {id}: {reason}"

**If ARGS is "task next"** — Start the next available task:
1. Read `.ai/TASKS.md`, find first `[ ]` (pending) entry
2. Check its dependencies
3. If met → start it (same as `task start`)
4. If not met → find next pending with met dependencies

**If ARGS is "task archive"** — Archive completed/failed tasks:
1. Scan `.ai/tasks/` for files with `status: completed` or `status: failed`
2. For each:
   - Read full content (title, description, dependencies)
   - Move file to `.ai/memory/tasks/`
   - Append to `.ai/memory/TASKS_LOG.md`:
     `- Archived **ID {id}: {Title}** (Status: {status}) on {timestamp}`
     `> {Description}`
   - Remove entry from `.ai/TASKS.md`
3. Report: "Archived {N} tasks to memory/tasks/"

**If ARGS is "task show [id]"** — Show task details:
1. Find `task{id}_*.md` in `.ai/tasks/` or `.ai/memory/tasks/`
2. Display full content

**TASKS.md icons:**
- `[ ]` pending
- `[-]` in-progress
- `[x]` completed
- `[!]` failed

---

### If ARGS starts with "handoff"

Generate a handoff document for downstream repos/teams.

**If ARGS is "handoff [plan-slug]"** — Generate handoff from a plan:
1. Read `.ai/plans/features/<slug>-plan.md`
2. If plan has no "Handoff Context" section or it's empty → ask user:
   - What downstream repo needs this?
   - What was built? (endpoints, schemas, events)
   - What's the interface contract? (response shapes, payloads)
   - Decisions that affect downstream?
   - How to sync? (SDK regen command, config update)
3. Fill the "Handoff Context" section in the plan
4. Also generate a standalone handoff file at `.ai/handoffs/<slug>.md`:
   ```markdown
   # Handoff: <Plan Title>

   **From**: <this repo>
   **To**: <downstream repo>
   **Date**: <today>
   **Plan**: plans/features/<slug>-plan.md

   ## What Was Built
   <endpoints, schemas, APIs, events>

   ## Interface Contract
   <response shapes, payloads, types>

   ## Decisions Affecting Downstream
   <key choices that downstream needs to know about>

   ## How to Sync
   <commands to run, config to update, deploy steps>

   ## Tasks for Downstream
   - [ ] <suggested task 1>
   - [ ] <suggested task 2>
   ```
5. Report: "Handoff generated. Share `.ai/handoffs/<slug>.md` with downstream repo."

**If ARGS is "handoff list"** — Show all handoffs:
1. List files in `.ai/handoffs/`

---

### If ARGS starts with "memory search"

Search feedback and decisions.

1. Parse query from ARGS after "memory search"
2. Grep `.ai/memory/feedback/` and `.ai/memory/decisions/` for the query
3. Display matching entries with file paths

---

### If ARGS starts with "status"

Show current system state.

1. Read active behaviour (from CLAUDE.md or behaviours/)
2. Count active hotfixes in HOTFIXES.md
3. Count recent incidents (last 30 days) from logs/incidents/INDEX.md
4. Check snapshot staleness (summary: N fresh, N stale)
5. Count feedback entries and decisions
6. Read last session from logs/sessions/INDEX.md
7. Print summary:
   ```
   codeskill status
   ─────────────────
   Behaviour: default
   Hotfixes: 3 active
   Incidents: 2 (last 30 days)
   Snapshots: 8 fresh, 2 stale
   Memory: 12 feedback, 5 decisions
   Last session: 2026-04-17 by @khanakia — "Add avatar upload"
   ```

---

### If ARGS starts with "health"

Score system health 1-10.

1. **Freshness (30%)**: Check dates on snapshots, last session, HOTFIXES
2. **Relevance (30%)**: Verify file paths referenced in memory/feedback still exist
3. **Completeness (20%)**: Check core files exist (RULES.md, taste.md, HOTFIXES.md, at least 1 snapshot)
4. **Actionability (20%)**: Check "Where We Left Off" in CLAUDE.md is present and specific
5. Compute weighted score
6. Print per-dimension scores and total
7. If < 5: suggest `/codeskill recover`

---

### If ARGS starts with "compress"

Run auto-compression on logs, memory, hotfixes.

1. Check line counts of:
   - `HOTFIXES.md` (cap: 75 lines)
   - `logs/sessions/INDEX.md` (cap: 150 lines)
   - `logs/activity/` individual files (cap: 150 lines each)
   - `memory/feedback/INDEX.md` (cap: 150 lines)
2. For each file over cap:
   - HOTFIXES: suggest graduating oldest to RULES.md or archiving
   - Session INDEX: archive entries older than 30 days
   - Activity logs: truncate oldest prompts, keep corrections
   - Memory INDEX: merge similar entries
3. Report: "Compressed N files. Removed N stale entries."

---

### If ARGS starts with "recover"

Recovery mode — rebuild from git + code.

1. Warn user: "Recovery mode will re-explore the codebase and verify all stored data. Continue?"
2. Scan project: read go.mod/package.json, README, directory structure
3. Read git history: `git log --oneline -20`
4. Re-explore and regenerate all stale snapshots
5. Verify file paths in memory/feedback/ and memory/decisions/ — flag broken refs
6. Rebuild ARCHITECTURE.md and STACK.md if missing
7. Run `sync` to regenerate CLAUDE.md
8. Report: "Recovery complete. Health score: <N>/10"

---

### If ARGS starts with "save-state"

Write emergency CONTINUATION.md.

1. Scan conversation for: current task, files being edited, what's done, what's in progress
2. Write `CONTINUATION.md` to project root (max 50 lines):
   ```markdown
   # Continue: <task>

   Resume from: `<file:line>` — <what was being done>

   ## State
   - [x] <completed items>
   - [ ] <in progress items>
   - [ ] <remaining items>

   ## Immediate Next Action
   <exact next step>
   ```
3. Report: "State saved to CONTINUATION.md. Next session will auto-load it."

---

### If ARGS starts with "export"

**If ARGS is "export cursor":**
1. Read `.ai/RULES.md`, `.ai/taste.md`, `.ai/PATTERNS.md`
2. Generate `.cursor/rules/` files from the content
3. Report: "Generated .cursor/rules/ from .ai/"

**If ARGS is "export copilot":**
1. Read `.ai/RULES.md`, `.ai/taste.md`, `.ai/PATTERNS.md`
2. Generate `.github/copilot-instructions.md`
3. Report: "Generated copilot-instructions.md from .ai/"

---

### If ARGS is empty or "help"

Show available commands:

```
codeskill — AI Project Intelligence System

Setup:
  init                    Scaffold .ai/ directory
  sync                    Regenerate CLAUDE.md from .ai/ files
  status                  Show current state

Behaviours:
  behaviour <name>        Switch mode (default, careful, review, debug, scaffold)
  behaviour list          List available behaviours

Session:
  session start           Load context, check staleness, report status
  session end             Write session log + activity log
  continue                Load CONTINUATION.md resume point
  save-state              Write emergency CONTINUATION.md

Learning:
  learn <lesson>          Learn NOW — save + apply immediately in current session
  feedback <correction>   Save user correction to memory
  decide <title>          Log a decision to memory
  incident <description>  Create incident report
  hotfix add <desc>       Add to active hotfixes
  hotfix review           Review stale hotfixes for graduation

Assets:
  snippet <name>          Find or create code snippet
  prompt <name>           Find or create prompt template

Planning:
  plan <title>            Create feature plan in plans/features/
  plan list               Show all active and completed plans
  plan archive <slug>     Move completed plan to memory/plans/
  handoff <plan-slug>     Generate handoff doc for downstream repo/team
  handoff list            Show all handoffs

Tasks:
  task add <description>  Create task file + add to TASKS.md
  task list               Show master checklist (TASKS.md)
  task start <id>         Start task (check dependencies first)
  task done <id>          Mark task completed
  task fail <id> <reason> Mark task failed with reason
  task next               Start next available pending task
  task archive            Archive completed/failed to memory/tasks/
  task show <id>          Show full task details

Memory:
  memory search <query>   Search feedback and decisions

Snapshots:
  snapshot list           Show all + staleness
  snapshot check <topic>  Check specific snapshot
  snapshot refresh <topic> Re-explore and update
  snapshot refresh --all  Refresh all stale
  snapshot create <topic> Create new snapshot

Maintenance:
  health                  Score system health 1-10
  compress                Auto-compress logs, memory, hotfixes
  recover                 Rebuild from git + code

Export:
  export cursor           Generate .cursor/rules/
  export copilot          Generate copilot-instructions.md
```

---

## Important Rules for This Skill

1. **Never modify CLAUDE.md directly** — always run `sync` to regenerate it
2. **Every feedback/decision entry needs**: date, user (@username), scope (project/global_candidate/global)
3. **Auto-save corrections**: When you detect user corrections during normal work (not just when this skill is invoked), save them to memory/feedback/
4. **Check before exploring**: Always check snapshots before spawning Explore agents
5. **Compact encoding**: All .ai/ files use tables > prose, file:line refs, one-line summaries
6. **Never store secrets** in any .ai/ file
