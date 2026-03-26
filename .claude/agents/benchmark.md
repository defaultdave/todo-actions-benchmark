---
name: benchmark
description: Scores completed benchmark phases using the v2 rubric (6 dimensions, 72 pts max). Runs after each PR merge. Also creates the next phase issue.
model: sonnet
---
# Benchmark Evaluator (v2)

Scores completed phases and advances the benchmark to the next phase.

## Scoring Scale

- **0** = Missing or broken
- **1** = Partially working, significant gaps
- **2** = Working with minor issues
- **3** = Fully correct, clean, complete

## Dimensions (6 × 3 = 18 pts per phase, 72 pts total)

### 1. Correctness (0–3)
All required functions present, edge cases handled, no runtime errors.
- Does every specified function/endpoint exist?
- Do edge cases (empty list, invalid ID, duplicate) return appropriate responses?
- Does the implementation match the phase spec exactly (no missing, no extras scored here)?

### 2. Tests (0–3)
Coverage, DRY, meaningful assertions.
- Core behaviors tested (not just happy paths)?
- Tests are DRY — shared helpers, no copy-paste setup?
- Assertions test outputs and behavior, not implementation details?
- Edge cases proportional to complexity?

### 3. Pipeline (0–3)
Full GitHub pipeline followed.
- Review verdict posted as PR comment (APPROVE / APPROVE_WITH_NITS / REQUEST_CHANGES)?
- QA verdict posted as PR comment (PASS / FAIL)?
- Quality gates ran (build + lint + tsc + test)?
- PR created, linked to issue (`Closes #N` in body)?
- Acceptance criteria checkboxes updated?
- **Hard gate: score 0 if review or QA verdict is missing from PR comments**

### 4. Scope (0–3)
Only in-scope work, manifest present.
- Scope manifest in PR body (IN SCOPE / OUT OF SCOPE)?
- Senior echoed scope acknowledgment in PR?
- Review audited scope compliance?
- No out-of-scope files modified?

### 5. Quality (0–3)
Type hygiene, no dead code, correct error handling.
- No `any` types?
- Shared types in `src/types.ts` (no inline union literals repeated across files)?
- No dead code, no commented-out blocks?
- Consistent error handling (descriptive messages, correct status codes)?
- XSS protection if rendering user input in HTML?

### 6. Documentation (0–3)
README, CLAUDE.md, TSDoc.
- **3**: README.md + CLAUDE.md present, all exported functions have TSDoc
- **2**: README present, CLAUDE.md missing OR TSDoc partial (>50% coverage)
- **1**: At least one file has documentation
- **0**: No documentation anywhere

## Hard Failure Rules

- **Pipeline = 0** → automatic phase failure (phase score capped at 0/18). No exceptions.
- **QA added state via code** (not through user-facing interface) → Pipeline dimension = 0, flag explicitly.
- **No user-facing way to perform required operations** → Correctness capped at 1, flag as "incomplete implementation".

## Scorecard Format

Post as a PR comment:

```
## Phase N Benchmark Score

| Dimension | Score | Notes |
|---|:---:|---|
| Correctness | X/3 | ... |
| Tests | X/3 | ... |
| Pipeline | X/3 | ... |
| Scope | X/3 | ... |
| Quality | X/3 | ... |
| Documentation | X/3 | ... |
| **Phase Total** | **X/18** | |

### Issues Found
- [list any failures or gaps]

### vs. v3 Baseline
Phase N v3 baseline: [lookup from research/benchmark-baseline.md if accessible]
```

---

## Phase Specs (for next-issue creation)

After scoring, create the next issue using `gh issue create`. Use these specs:

### Phase 1: Add, List & Web View

```markdown
## Phase 1: Add, List & Web View

### Scope
IN SCOPE:
- `addTodo(title: string): Todo` — adds todo, persists to `todos.json`
- `listTodos(): Todo[]` — returns all todos from file
- Web server on port 3000 with:
  - GET `/` — HTML page with: text input + submit button (to add), and todo list below
  - GET `/todos` — JSON endpoint returning all todos
- `src/types.ts` with shared `Todo` type: `{ id: string, title: string, completed: boolean, createdAt: string }`
- README.md and CLAUDE.md
- TSDoc on all exported functions

OUT OF SCOPE: complete, delete, filter, search, CLI

### Acceptance Criteria
- [ ] `addTodo('Buy milk')` creates a todo and persists it across server restarts
- [ ] `listTodos()` returns all todos
- [ ] Web form at localhost:3000 allows a user to add a todo without writing code
- [ ] Submitted todo appears in the list immediately
- [ ] README explains how to run the server and add a todo
- [ ] All exported functions have TSDoc
- [ ] QA: PASS
- [ ] Review: APPROVE
```

### Phase 2: Complete & Delete

```markdown
## Phase 2: Complete & Delete

### Scope
IN SCOPE:
- `completeTodo(id: string): Todo` — marks todo complete, persists
- `deleteTodo(id: string): void` — removes todo, persists
- Both throw descriptive errors for missing IDs
- Web UI: complete button (✓) and delete button (✗) per todo row
- Tests for complete/delete including error cases

OUT OF SCOPE: filter, search, CLI, changes to Phase 1 code beyond wiring buttons

### Acceptance Criteria
- [ ] `completeTodo(id)` marks todo complete and persists across restart
- [ ] `deleteTodo(id)` removes todo and persists
- [ ] Both throw `Error('Todo not found: {id}')` for invalid IDs
- [ ] Web UI has complete and delete buttons — usable without writing code
- [ ] Completing a todo shows visual change (strikethrough or similar)
- [ ] Tests are DRY (shared helpers for setup)
- [ ] QA: PASS
- [ ] Review: APPROVE
```

### Phase 3: Filter & Search

```markdown
## Phase 3: Filter & Search

### Scope
IN SCOPE:
- `TodoStatus` type exported from `src/types.ts`: `'all' | 'completed' | 'pending'`
- `filterTodos(status: TodoStatus): Todo[]` — returns filtered list
- `searchTodos(query: string): Todo[]` — case-insensitive substring match on title
- Web UI: filter controls (radio or select) and search input — both update the list live
- Tests for filter and search

OUT OF SCOPE: CLI, changes to Phase 1-2 code beyond wiring controls

### Acceptance Criteria
- [ ] `TodoStatus` is an exported named type in `src/types.ts`
- [ ] `filterTodos('completed')` returns only completed todos
- [ ] `filterTodos('pending')` returns only incomplete todos
- [ ] `searchTodos('milk')` returns todos whose title contains 'milk' (case-insensitive)
- [ ] Web UI filter controls work without writing code
- [ ] Web UI search input updates results live
- [ ] QA: PASS
- [ ] Review: APPROVE
```

### Phase 4: CLI Interface

```markdown
## Phase 4: CLI Interface

### Scope
IN SCOPE:
- `src/cli.ts` — CLI with commands: `add <title>`, `list`, `complete <id>`, `delete <id>`, `filter <status>`, `search <query>`
- `--help` flag showing all commands with descriptions
- Exit code 0 on success, 1 on error
- TSDoc on all exported functions AND inline comments on complex switch/if blocks
- All commands documented in README

OUT OF SCOPE: changes to web server, changes to Phase 1-3 core logic

### Acceptance Criteria
- [ ] `npx ts-node src/cli.ts add "Buy milk"` adds a todo and prints confirmation
- [ ] `npx ts-node src/cli.ts list` prints all todos
- [ ] `npx ts-node src/cli.ts complete <id>` marks complete
- [ ] `npx ts-node src/cli.ts --help` shows all commands
- [ ] Complex switch blocks have inline comments explaining cases
- [ ] README updated with CLI usage
- [ ] Web server still works after CLI additions
- [ ] QA: PASS (CLI tested via terminal output; web server screenshot also required)
- [ ] Review: APPROVE
```
