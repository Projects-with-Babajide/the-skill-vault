# Ship Chore

> **PREREQUISITE:** This skill requires `.claude/skill-config.md` in the project root. Step 0 checks for it. If the file is missing, the skill MUST stop immediately and will not proceed under any circumstances.

For ad-hoc changes that don't have a ticket yet — config tweaks, tooling fixes, skill updates, refactors made on the fly. Creates a lightweight ticket, branches off it, commits, pushes, opens a PR, monitors CI, and merges.

## Usage

- `/ship-chore` — auto-detect changes, create ticket, branch, commit, PR, CI watch, merge

---

## Step 0: Load Config

**This is the first and mandatory step. Do not proceed to any other step until this check passes.**

Run:
```
test -f .claude/skill-config.md || echo "CONFIG_MISSING"
```

**If the output is `CONFIG_MISSING`:** Output this message exactly, then STOP. Do not execute Step 1 or any subsequent step under any circumstances.
> "Cannot run `/ship-chore`: no `.claude/skill-config.md` found in this project. Create the config file first — see the user-level skill for the required format."

**If the command produces no output:** Read `.claude/skill-config.md` and extract:
- **Issue tracker** and **Issue prefix** (Project Identity section)
- **Linear team name** and **Linear project name** (Project Identity section)
- **Base branch** (Project Identity section)
- **Branch naming convention** (Branch Naming Convention section)
- **Deployment** section (optional) — if present, extract: Health check URL, Health check expected status, Deploy timeout, Rollback on failure, and any Deployment status tools (List services tool, List deployments tool, Get logs tool, Services to monitor). If the section is absent, set `deployment_configured` to false and skip all deployment steps later.

---

## Step 1: Inspect Changes

1. Run `git status` to identify modified, staged, and untracked files.
2. Run `git diff` and `git diff --cached` to read what changed.
3. Run `git branch --show-current` to check the current branch.
4. If on a feature/chore branch (not the **Base branch** from config), warn the user:
   > "You're on `<branch>`, not `<base-branch>`. This skill is designed for changes sitting on the base branch without a ticket. Should I continue from here?"
   Wait for confirmation before proceeding.

If no changes are found (clean working tree, nothing staged or untracked), inform the user there is nothing to ship and stop.

---

## Step 2: Determine Ticket Type

Analyze the changes and pick the most appropriate type:

| Type | When to use |
|------|-------------|
| `chore` | Config, tooling, scripts, CI, non-functional changes |
| `fix` | Bug fix in existing code |
| `refactor` | Code restructuring without behavior change |
| `docs` | Documentation only |
| `feature` | New functionality |

Default to `chore` when in doubt.

---

## Step 3: Draft and Create Ticket

Based on what the diff shows, draft a lightweight ticket:

- **Title** — concise, imperative (e.g., "Fix allow-list entries and remove HEREDOC from ship-and-watch")
- **Description** — 2–4 sentences: what changed, why it was needed. No need for full ticket standard sections — this is not an implementation ticket.
- **State** — set to `In Progress` (the work is already done)
- **Priority** — Normal (3) unless clearly otherwise

Create the ticket using `create_issue` with:
- `team`: the **Linear team name** from config
- `project`: the **Linear project name** from config
- `state`: In Progress

Display the created ticket identifier to the user before continuing.

---

## Step 4: Create Branch

Using the ticket identifier from Step 3 and the **Branch naming convention** from config, derive the branch name:

- `<type>` — match what you chose in Step 2
- `<issue-id>` — the ticket identifier, lowercased (e.g., `oru-79`)
- `<short-description>` — 2–4 words in kebab-case describing the change

Examples:
- `chore/oru-79/fix-allow-list-heredoc`
- `fix/oru-80/sqlite-null-return`
- `refactor/oru-81/module-runner-cleanup`

Run: `git checkout -b <branch-name>`

---

## Step 5: Stage and Commit

Stage only the relevant changed files — NOT `git add -A` or `git add .`:
- Read `git status` to identify files to stage
- Exclude any files that look like secrets (`.env`, `*.key`, `*.pem`, `credentials.*`) — warn and skip them

Use multiple `-m` flags for the commit message. Do NOT use HEREDOC or command substitution; they trigger a Claude Code safety check:

```
git commit \
  -m "<commit title — imperative, under 72 chars>" \
  -m "<body paragraph if needed — bullet list of changes>" \
  -m "Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

If the commit only touches one file or is trivially small, a single `-m` plus the co-author line is fine.

---

## Step 6: Push and Create PR

Check upstream: `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
- If no upstream: `git push -u origin <branch-name>`
- If upstream exists: `git push`

If push is rejected, report the error. Do NOT force-push.

Create the PR using an inline multiline string for `--body`. Do NOT use HEREDOC or command substitution:

```
gh pr create --title "<title>" --body "## Summary
<what changed and why — 1–3 bullet points>

## Changes
<key files changed and what was done to each>

## Test plan
- [ ] CI pipeline passes

---
Generated with [Claude Code](https://claude.com/claude-code)"
```

Capture and display the PR URL.

---

## Step 7: Monitor CI

Poll `gh pr checks <pr-number> --json name,state,conclusion,detailsUrl` every 30 seconds.

- Report a status line only when something changes: `"CI: check 'test' — in progress..."`
- Wait until all checks have a non-pending state.

**All passed** → proceed to Step 8.

**One or more failed** → fetch logs with `gh run view <run-id> --log-failed 2>&1 | tail -200`, then report to the user:

```
## CI Failure: <check-name>
- **Error:** <error message / failed test>
- **Root cause:** <analysis>
- **Proposed fix:** <specific change>
```

Ask: **"Should I apply this fix?"** Wait for confirmation before touching any code. After applying and pushing, go back to Step 7.

**Stuck** (no progress after 10 minutes) — inform the user and ask whether to keep waiting or investigate.

---

## Step 8: Merge PR

After all CI checks pass, merge **without** deleting the branch:

```
gh pr merge <pr-number> --merge
```

Do NOT pass `--delete-branch`. The branch stays alive until deployment is verified (Step 8b) or, if no deployment is configured, until the end of this step.

**If the Deployment section exists in `skill-config.md`:** proceed to Step 8b. Do NOT delete the branch or mark the ticket yet.

**If no Deployment section exists:** delete the branch, mark the ticket as Done, and proceed to the Final Summary:

```
git push origin --delete <branch-name>
git branch -d <branch-name>
```

### Mark the ticket as Done

1. Fetch the issue: use `get_issue` with the ticket identifier.
2. Find the "Done" state: use `list_issue_statuses` with the team from the issue.
3. Use `update_issue` with `state: Done`.
4. Confirm to the user: > "<ticket_id> marked as Done."

Then provide the Final Summary.

---

## Step 8b: Verify Deployment (only when Deployment section exists in config)

This step runs only when the **Deployment** section is present in `skill-config.md`. It verifies the merged code deploys successfully before cleaning up.

### 1. Wait for deploy to trigger

After merge, wait **30 seconds** for the platform's auto-deploy to pick up the new commit.

### 2. Check deployment status via configured tools

If the config includes a **Deployment status tools** subsection, use the listed tools to monitor deployment progress before falling through to the health check.

1. If a **List services tool** is configured, call it with the provided parameters to discover service IDs for each service in **Services to monitor**.
2. If a **List deployments tool** is configured, call it per service to find the most recent deployment (started after the merge timestamp).
3. Poll every **30 seconds** until all deployments reach a terminal state (success or failure), or the **Deploy timeout** is reached.
4. Report status changes:
   > `"Deploy: <service> — <status> (<elapsed>)"`
5. If any deployment fails and a **Get logs tool** is configured, call it for the failed service to retrieve recent logs. Present the logs to the user (see "Deployment failed" below). Do NOT proceed to health check.
6. If all deployments succeed, proceed to the health check (step 3).

If no deployment status tools are configured, skip straight to the health check.

### 3. Verify health check

The **Health check URL** is the final confirmation. Even if the status tools report success, the health check must pass.

- Make an HTTP GET request to the **Health check URL** using `curl` via Bash.
- Expect the **Health check expected status** (default 200).
- **Retry logic:** after the first successful status code, retry 2 more times with 10-second intervals. All 3 must return the expected status to guard against rolling deploy blips.
- If the health check has not passed after **Deploy timeout**, treat it as a failure.
- Report each attempt:
  > `"Health check: <url> returned <status> (<elapsed>)"`

### 4. Interpret results

**Deployment and health check both pass:**
1. Report success:
   > "Deployment verified: health check at `<url>` passed."
2. Delete the branch:
   ```
   git push origin --delete <branch-name>
   git branch -d <branch-name>
   ```
3. Mark the ticket as Done (same logic as Step 8), then proceed to the Final Summary.

**Deployment or health check failed:**
1. If a **Get logs tool** is configured and logs were not already fetched, call it for each service in **Services to monitor**.
2. Present the failure to the user:
   ```
   ## Deployment Failed

   **Failed service(s):** <service names, or "health check only">
   **Health check:** <url> returned <actual status> (expected <expected status>)
   **Elapsed:** <time since merge>

   ### Logs
   <logs from configured tool, or "No logs tool configured">

   ### What was preserved
   - Branch `<branch-name>` has NOT been deleted
   - Ticket has NOT been marked as Done

   ### Suggested next steps
   - Check the logs above for the root cause
   - Push a follow-up fix to the same branch, or investigate on the platform directly
   ```
3. Do NOT delete the branch. Do NOT mark the ticket as Done.
4. If **Rollback on failure** is `yes`: offer to trigger a rollback. If `no`: just flag the issue.
5. Ask:
   > "Deployment failed. Branch preserved for follow-up. Would you like to investigate, push a fix, or stop here?"

**Health check returns unexpected response shape (e.g., 200 but empty body):**
- Treat as a pass if the status code matches. The skill checks status codes, not response bodies.

---

## Final Summary

```
## Ship Chore Summary

- **Ticket:** <ticket-id> — <title>
- **Branch:** <branch-name> (deleted | preserved — see deployment)
- **PR:** <url>
- **CI:** passed (<n> checks)
- **Merged:** yes
- **Deployment:** Verified | Failed | Skipped (no config)
- **Ticket:** <ticket-id> marked Done (or "Not marked — deployment failed")

### Committed
- `<file-path>` — <what changed>
```

---

## Rules

- NEVER push directly to the base branch.
- NEVER force-push without explicit user request.
- NEVER commit secrets (`.env`, `*.key`, `*.pem`, `credentials.*`).
- NEVER skip pre-commit hooks (`--no-verify`).
- NEVER amend the previous commit — always create a new commit after a hook failure.
- NEVER use HEREDOC or command substitution in git or gh commands — use multiple `-m` flags for commits and inline multiline strings for PR bodies.
- NEVER create a full ticket-standard ticket for this skill — keep the ticket lightweight.
- Follow all conventions from `CLAUDE.md` at all times.
