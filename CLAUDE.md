# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ralph is an autonomous AI agent loop that runs [Amp](https://ampcode.com) repeatedly until all PRD items are complete. Each iteration spawns a fresh Amp instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

## Commands

```bash
# Run Ralph (default 10 iterations)
./ralph.sh [max_iterations]

# Flowchart dev server
cd flowchart && npm install && npm run dev

# Build flowchart for production
cd flowchart && npm run build

# Lint flowchart
cd flowchart && npm run lint

# Check PRD status
cat prd.json | jq '.userStories[] | {id, title, passes}'

# View progress/learnings
cat progress.txt
```

## Architecture

```
ralph/
├── ralph.sh           # Bash loop spawning fresh Amp instances
├── prompt.md          # Instructions given to each Amp iteration
├── prd.json           # User stories with passes status (created per feature)
├── progress.txt       # Append-only learnings between iterations
├── skills/
│   ├── prd/           # Amp skill for generating PRDs
│   └── ralph/         # Amp skill for converting PRDs to JSON
└── flowchart/         # React Flow visualization (Vite + TypeScript)
```

### Memory Between Iterations

Each Ralph iteration starts fresh with no LLM memory. Context carries forward only through:
1. **Git history** - commits from previous iterations
2. **progress.txt** - learnings and patterns discovered
3. **prd.json** - which stories are complete (`passes: true`)

### The Loop

1. Check branch matches PRD `branchName`
2. Pick highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks
5. Commit if checks pass
6. Update `prd.json` to mark `passes: true`
7. Append learnings to `progress.txt`
8. Exit with `<promise>COMPLETE</promise>` when all stories pass

## Story Sizing

Stories must be small enough to complete in one context window. If too big, the LLM runs out of context and produces broken code.

**Right-sized:** Database migration, single UI component, server action update, filter dropdown

**Too big (split these):** "Build entire dashboard", "Add authentication", "Refactor the API"

## Skills

Install globally to `~/.config/amp/skills/` for use across projects:
- **prd skill:** Generates PRDs with clarifying questions, outputs to `tasks/prd-[name].md`
- **ralph skill:** Converts PRD markdown to `prd.json` format

## Flowchart

Interactive React Flow visualization at `flowchart/`. Built with Vite, React 19, TypeScript, and @xyflow/react. Deployed to GitHub Pages.
