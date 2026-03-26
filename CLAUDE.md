# Todo App — Actions Benchmark

GitHub Actions benchmark using the v2 agent pipeline. Each phase is a separate PR with automated review + QA.

## Pipeline

```
Issue created → @claude implement → PR opened
                                       ↓
                          [review.yml] + [qa.yml] (parallel)
                                       ↓ (both must pass)
                          Branch protection blocks merge
                                       ↓ human merges
                          [benchmark-score.yml] → score + next issue
```

## Agents

| Agent | Trigger | File |
|---|---|---|
| Senior | `@claude implement #N` in issue comment | `.claude/agents/senior.md` |
| Review | Automatic on PR open | `.claude/agents/review.md` |
| QA | Automatic on PR open | `.claude/agents/qa.md` |
| Benchmark | Automatic on PR merge | `.claude/agents/benchmark.md` |

## Dev Commands

```bash
npm run dev      # Start dev server (port 3000)
npm run build    # Compile TypeScript
npm test         # Run Vitest tests
npx tsc --noEmit # Type check only
npx eslint .     # Lint
```

## Tooling

| Tool | Purpose |
|---|---|
| TypeScript + ts-node | Language |
| Vitest | Tests |
| ESLint + @typescript-eslint | Linting |
| Playwright | QA browser testing (CI: headless, no sandbox) |
