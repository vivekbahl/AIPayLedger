# PayLedger: project guide for Claude Code

## What this is
A practice payments API (Node 20, TypeScript strict, Fastify, SQLite via better-sqlite3, Vitest, fast-check). It uses a MOCK processor only. There is no real card data anywhere.

## Source of truth
- Requirements and acceptance criteria: docs/SPEC.md
- Resolved ambiguities: docs/DECISIONS.md
- Test cases: docs/TEST_PLAN.md
Read these before planning or coding. If the spec is silent or contradictory, STOP and ask me. Do not invent behavior. Never edit the spec yourself unless I ask.

## Commands
- Install: npm install
- Dev server: npm run dev
- Tests: npm test
- Lint and types: npm run lint && npm run typecheck
Always run tests, lint and typecheck before saying a task is done. Report real output, not a summary of what you expect.

## Money rules (critical)
- All money is integer minor units (cents). Never use floats, toFixed, parseFloat, or unchecked Number() on money.
- Multiply before dividing. Round half up at exactly one defined place per rule in the spec.
- Tax is computed once per order on the subtotal, never per line.
- Refund tax portion: proportional, rounded half up, and the final refund takes the remainder so tax refunded equals tax charged.
- Enforce a maximum total of 99,999,999 cents and keep intermediate products within safe integer range.
- All money logic lives in src/money as pure functions with unit and property tests.

## Idempotency and concurrency rules (critical)
- Claim the idempotency key atomically (unique constraint, status in_progress) BEFORE any side effect. No check-then-act.
- Balance checks and the writes they protect happen in one database transaction.
- The processor reference equals our payment id. A retry must never create a second charge.
- Never use in-memory state for idempotency or balances.

## Testing rules
- Write failing tests first, from docs/TEST_PLAN.md, and show them to me before implementing.
- NEVER modify, weaken, skip or delete an existing test to make it pass. If you believe a test is wrong, tell me and explain.
- Tests assert exact values, not just that nothing threw. Compute expected numbers independently, do not copy the implementation formula.
- Include edge cases: rounding boundaries, retries, duplicates, concurrency, over-refund.

## Git workflow
- Never commit to main. One branch per slice: claude/<short-kebab-description>.
- Conventional Commits: feat, fix, test, docs, refactor, chore, ci. Imperative summary under 72 characters.
- Small, logical commits. Do not push until I say so if I ask for a local commit.
- Open PRs against main with gh pr create. Description must have: Summary, Changes, How to test, Risks and open questions, and end with: Created with Claude Code
- Never merge PRs, force-push or delete branches.

## Security and PCI hygiene
- Never read, print or commit .env files, secrets, keys or credentials.
- Reject any request fields named card_number, pan or cvv. Never log request bodies for payment routes. Mask tokens in logs.
- Never paste real card data into code, tests, prompts or fixtures. Use tok_success, tok_decline and tok_timeout only.

## Sensitive areas (flag in every PR that touches them)
src/payments, src/refunds, src/money, src/ledger, src/idempotency, auth. State clearly in the PR: SENSITIVE: needs careful line-by-line human review.

## Boundaries
- Ask before adding dependencies, changing CI config or changing the database schema.
- Ask before any destructive command.
- Keep diffs small and limited to the files in the approved plan. No unrelated refactors.
- At the end of each task, list your assumptions and anything not covered by tests.