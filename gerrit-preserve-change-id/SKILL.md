---
name: gerrit-preserve-change-id
description: Preserves the Gerrit Change-Id trailer when amending or rewriting commit messages. Use when amending a commit, running git commit --amend, rewording a commit during interactive rebase, or any operation that rewrites a commit message — whether requested explicitly by the user or decided autonomously by Copilot.
---

# Gerrit Preserve Change-Id

## Quick start

Before rewriting any commit message, extract the `Change-Id` from the target
commit, append it to the new message, confirm with the user, then amend.

## Workflow

### Step 1 — Extract Change-Id from the target commit

```bash
# For HEAD:
git log -1 --format=%B HEAD

# For a specific commit (e.g. during interactive rebase):
git log -1 --format=%B <commit-sha>
```

Parse out the `Change-Id: I...` line.

- **No Change-Id found** → this is not a Gerrit repo; proceed with a normal
  amend, no warning needed.
- **Commit is not HEAD** (e.g. mid-rebase) → warn the user:
  *"Note: extracting Change-Id from non-HEAD commit `<sha>`."*

### Step 2 — Build the new message

Apply whatever rewording was requested. Do **not** include a `Change-Id` line yet.

### Step 3 — Resolve Change-Id conflicts

If **both** the new message and the original commit already contain a
`Change-Id` line, show the user both values and ask which one to keep.
Otherwise use the one extracted in Step 1.

### Step 4 — Append Change-Id as the last line

The final message must end with a blank line followed by the trailer:

```
<subject>

<body (optional)>

Change-Id: I<40-char-hex>
```

### Step 5 — Confirm and amend

Show the full final commit message to the user and wait for approval.
On confirmation, run:

```bash
# Single-line subject only:
git commit --amend -m $'<subject>\n\nChange-Id: I<hash>'

# Multi-line (use heredoc to avoid quoting issues):
git commit --amend -F - <<'EOF'
<subject>

<body>

Change-Id: I<hash>
EOF
```

## Notes

- Applies to `git commit --amend`, `git rebase -i` reword/fixup, and any other
  operation that rewrites a commit message.
- Never check for or require `.git/hooks/commit-msg`.
- Copilot must follow this workflow even when autonomously deciding to amend
  (e.g. as part of a larger task), not only when the user explicitly asks.
