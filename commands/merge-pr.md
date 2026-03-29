Merge a PR after verifying all requirements are met.

If a PR number is provided ("$ARGUMENTS"), use that. Otherwise, detect the PR associated with the current branch by running `gh pr view --json number,url --jq '.number'`. If no PR is found, abort with a clear message.

## Phase 1: Pre-merge Checks

Run ALL of the following checks. If ANY check fails, abort immediately and report which checks failed.

### 1.1 CI Status
- Run `gh pr checks <number>` and verify ALL checks have passed (no failures or pending checks).
- If any check is failing or still pending, abort: "CI is not green. Failing checks: ..."

### 1.2 Unresolved Discussions
- Run `gh pr view <number> --json reviewDecision,reviews,comments` and check for unresolved review threads.
- Use `gh api repos/{owner}/{repo}/pulls/<number>/reviews` to find any reviews with state `CHANGES_REQUESTED` that haven't been followed by an `APPROVED` from the same reviewer.
- If there are unresolved conversations or outstanding change requests, abort: "Unresolved discussions found: ..."

### 1.3 Approvals
- Check that the PR has at least **2 approvals** from distinct reviewers.
- Use `gh pr view <number> --json reviews --jq '[.reviews[] | select(.state=="APPROVED")] | map(.author.login) | unique'`.
- If fewer than 2 approvals, abort: "Only N approval(s) found. Need at least 2."

## Phase 2: Prepare Merge Content

### 2.1 Understand the Full Changeset
- Read the PR description and all commit messages (`gh pr view <number> --json commits`).
- Read the full diff: `gh pr diff <number>`.
- Identify the JIRA ticket number from the branch name or PR title (pattern: `CAMEL-XXXXX`).
- If a JIRA ticket is found, fetch its details for context.

### 2.2 Write the Commit Message
Compose a squash-merge commit message with this format:
```
CAMEL-XXXXX: Short title (under 70 chars)

<Description of the changes in 200 words max. Explain WHAT changed and WHY.
Focus on the purpose and impact, not a file-by-file listing.>

Closes #<PR number>
```
If no JIRA ticket is found, omit the `CAMEL-XXXXX:` prefix.

### 2.3 Update PR Description
- Update the PR description using `gh pr edit <number> --body "..."` to reflect the final state of the changeset. Include:
  - A summary of what was changed and why
  - Link to the JIRA ticket (if any)
  - Any notable implementation decisions

### 2.4 Update JIRA Issue
- If a JIRA ticket was identified, update its description or add a comment summarizing the changes and linking to the PR.
- Set `fixVersions` to the current project version (check `pom.xml` for the version).

## Phase 3: Merge

- Present the commit message to the user for approval before proceeding.
- Once approved, squash-merge using:
  ```
  gh pr merge <number> --squash --subject "<title>" --body "<body>" --delete-branch
  ```
- Verify the merge succeeded by checking `gh pr view <number> --json state`.

## Phase 4: Cleanup

- Confirm the remote branch was deleted.
- If the local branch matches the PR branch, switch to `main` and pull.
- Report: "PR #XXXX merged successfully. Branch deleted."
