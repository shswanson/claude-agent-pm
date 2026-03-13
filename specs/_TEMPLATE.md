# Batch N: [Title] (Px)

| Field        | Value                         |
|--------------|-------------------------------|
| **Batch**    | N                             |
| **Repo**     | `repo-name`                   |
| **Priority** | Px — [brief justification]    |
| **Status**   | Ready                         |
| **Files**    | `path/to/file1.ext`, `path/to/file2.ext` |

---

## Problem

[2-3 sentences: What's wrong or what's needed. Why this matters.]

## Intent

[What does "done" look like from the user's perspective? Describe the desired end state, not the implementation. If the coding agent finds a better path than specified below, this is what they optimize for.]

## Architecture

[How the solution fits into the existing system. What approach was chosen and why.]

---

## Task 1: [Title]

**File:** `path/to/file.ext`

[Description of what to change.]

<current_code file="path/to/file.ext" line="N">
```language
// existing code that will be modified
```
</current_code>

<new_code file="path/to/file.ext">
```language
// what it should become
```
</new_code>

<notes>
- [Gotchas, edge cases, things the agent might get wrong]
</notes>

---

## Task 2: [Title]

**File:** `path/to/file.ext`

[Same structure as Task 1]

---

<do_not_touch>
## What NOT to Touch

- `path/to/unrelated-file.ext` — looks related but should be left alone because [reason]
- [Other files that might be tempting to modify]
</do_not_touch>

---

<verification>
## Verification Checklist

- [ ] `file1.ext` has [expected change]
- [ ] `file2.ext` has [expected change]
- [ ] [Lint/syntax check command] passes
- [ ] [Functional verification command] returns expected result
- [ ] Only N files changed: [list them]

### Verification commands:

```bash
# Syntax check
[language-specific lint command]

# Functional check
curl -s 'https://staging-url/affected-page/' | grep -c 'expected-element'

# Negative check (ensure nothing broke)
curl -s 'https://staging-url/unrelated-page/' | grep -c 'still-present'
```
</verification>

---

## Commit Message

```
[Short description of what this batch does]

[Optional longer description with context]
```

---

## Risks & Mitigations

- **[Risk]**: [What could go wrong] — **Mitigation**: [How the spec prevents it]
