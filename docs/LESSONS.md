# Lessons Learned

_A growing reference updated as the team learns about the system. Read at session start alongside ARCHITECTURE.md._

## Process

- Batch specs with exact code snippets work best. Ambiguity in specs leads to wrong implementations.
- Coding agents report completion but don't always commit. Always check git status.
- Completed specs get `x-` prefix after QA.

## Agent Behavior

- Agents suffer "time blindness" — they'll spend hours running tests instead of making progress. Set speed expectations in specs.
- Scope creep kills sessions. If an agent discovers something adjacent, it should stop and report, not improvise.
