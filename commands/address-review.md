Address review feedback on a PR: read comments, fix issues, update the PR.

If a PR number is provided ("$ARGUMENTS"), use that. Otherwise, detect the PR associated with the current branch by running `gh pr view --json number --jq '.number'`. If no PR is found, abort with a clear message.

## Phase 1: Gather Feedback

### 1.1 Fetch All Review Comments
- Get PR reviews: `gh api repos/{owner}/{repo}/pulls/<number>/reviews`
- Get review comments (inline): `gh api repos/{owner}/{repo}/pulls/<number>/comments`
- Get issue comments (general): `gh api repos/{owner}/{repo}/issues/<number>/comments`
- Identify the latest review round (comments after the last push).

### 1.2 Categorize Comments
For each comment/review, classify as:
- **Blocking**: explicit change requests, correctness issues, missing tests
- **Suggestion**: optional improvements, alternative approaches
- **Question**: reviewer asking for clarification (needs a reply, not necessarily a code change)
- **Resolved**: already addressed or acknowledged

### 1.3 Present Summary
- List all unresolved comments grouped by category (blocking first).
- For each, show: reviewer, file/line, comment text, and proposed action.
- **Wait for user approval** on the plan before making changes.

## Phase 2: Address Each Item

### 2.1 Fix Blocking Issues
- Check out the PR branch if not already on it.
- Address each blocking comment with a code change.
- For questions, draft a reply explaining the reasoning.

### 2.2 Consider Suggestions
- Apply suggestions that improve the code without over-engineering.
- For suggestions the user declined, draft a polite reply explaining why.

### 2.3 Build and Verify
- Build affected modules: `mvn install -B -pl <modules> -DskipTests`
- Run tests: `mvn test -B -pl <modules>`
- Run formatter: `mvn formatter:format impsort:sort -B -pl <modules>`
- Regenerate downstream artifacts if needed.
- Check `git status` and commit all changes.

## Phase 3: Update PR

### 3.1 Push Changes
- Commit with a message summarizing what was addressed: `CAMEL-XXXXX: Address review feedback`
- Push to the PR branch.

### 3.2 Reply to Comments
- For each addressed comment, reply confirming the fix with a brief explanation.
- For questions, post the drafted reply.
- Attribution: all replies must include "_Claude Code on behalf of Guillaume Nodet_".

### 3.3 Update PR Description
- Update the PR description using `gh pr edit <number> --body "..."` to reflect any changes in approach or scope resulting from the review.

## Phase 4: Report

- Summarize what was changed and which comments were addressed.
- List any comments that were intentionally not addressed (with reasons).
- Note: do NOT resolve review conversations — let the reviewer resolve them after checking.
