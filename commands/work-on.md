Work on a JIRA issue end-to-end: from investigation to PR creation.

The JIRA issue key must be provided as "$ARGUMENTS" (e.g., `CAMEL-12345`). If not provided, abort with a clear message.

## Phase 1: Claim the Issue

### 1.1 Fetch Issue Details
- Fetch the JIRA issue using `curl` with Bearer token auth (`$JIRA_TOKEN`):
  ```
  curl -s -H "Authorization: Bearer $JIRA_TOKEN" "https://issues.apache.org/jira/rest/api/2/issue/CAMEL-XXXXX"
  ```
- Display: summary, description, component(s), reporter, current status, and any linked issues.

### 1.2 Verify Availability
- If the issue is already **assigned** to someone other than `gnodet`, abort: "Issue is assigned to <name>. Not taking over."
- If the issue is already **Closed** or **Resolved**, abort: "Issue is already closed."

### 1.3 Assign and Transition
- Assign to `gnodet`:
  ```
  curl -s -X PUT -H "Authorization: Bearer $JIRA_TOKEN" -H "Content-Type: application/json" \
    -d '{"name":"gnodet"}' "https://issues.apache.org/jira/rest/api/2/issue/CAMEL-XXXXX/assignee"
  ```
- Transition to "In Progress" (transition id `4`):
  ```
  curl -s -X POST -H "Authorization: Bearer $JIRA_TOKEN" -H "Content-Type: application/json" \
    -d '{"transition":{"id":"4"}}' "https://issues.apache.org/jira/rest/api/2/issue/CAMEL-XXXXX/transitions"
  ```

## Phase 2: Investigate

Do NOT jump to implementation. Camel is a large, long-lived project — code often looks "wrong" but exists for good reasons.

### 2.1 Understand the Code
- Identify the affected module(s) and file(s) from the issue description.
- Read the relevant source code.

### 2.2 Check History
- Run `git log --oneline -20 -- <affected-files>` to see recent changes.
- Run `git blame` on key areas to understand who wrote the code and why.
- Read commit messages and linked JIRA tickets for prior changes.

### 2.3 Search for Context
- Search JIRA for related tickets (same component, similar keywords) to find prior discussions, rejected approaches, or intentional design decisions.
- Check the `proposals/` directory for design docs related to the affected area.

### 2.4 Validate the Issue
- Confirm the reported problem is real. Question assumptions in the issue description.
- If the issue involves a module that replaced or deprecated another, understand why.
- If the proposed fix would effectively revert a prior intentional commit, flag this.

### 2.5 Present Findings
- Summarize your investigation to the user:
  - Is the issue valid?
  - What is the root cause?
  - Are there any risks or conflicts with prior decisions?
  - What approach do you recommend?
- **Wait for user approval before proceeding to implementation.**

## Phase 3: Implement

### 3.1 Create Branch
- Create a branch from `main`: `CAMEL-XXXXX-brief-description`
  ```
  git checkout -b CAMEL-XXXXX-brief-description main
  ```

### 3.2 Implement Changes
- Write the fix/feature with proper tests and documentation.
- Follow Camel conventions (Component/Endpoint/Producer/Consumer patterns, annotations, etc.).

### 3.3 Build and Verify
- Build the affected module(s): `mvn install -B -pl <modules> -DskipTests`
- Run tests: `mvn test -B -pl <modules>`
- Run formatter: `mvn formatter:format impsort:sort -B -pl <modules>`
- Regenerate downstream artifacts if needed:
  ```
  mvn install -B -pl catalog/camel-catalog -DskipTests
  mvn generate-sources -B -pl dsl/camel-endpointdsl -DskipTests
  mvn generate-sources -B -pl dsl/camel-componentdsl -DskipTests
  ```
- Only stage files related to your change (ignore unrelated diffs from other components).
- Check `git status` and commit ALL changes including generated files.

## Phase 4: Self-Review

- Run the `/review-pr` process against the current branch diff (`git diff main...HEAD`), reviewing your own changes as if you were an independent reviewer.
- Fix ALL issues found (blocking, major, minor, nits) and repeat Phase 3.3. New code should be clean from the start.
- If the review uncovers important pre-existing issues unrelated to the current change, use `/create-issue` to file them separately rather than fixing them in this PR.

## Phase 5: Create PR

### 5.1 Push Branch
- Push to the `gnodet` fork:
  ```
  git push -u gnodet CAMEL-XXXXX-brief-description
  ```

### 5.2 Create PR
- Create the PR targeting `apache/camel:main`:
  ```
  gh pr create --repo apache/camel --head gnodet:CAMEL-XXXXX-brief-description \
    --title "CAMEL-XXXXX: Brief description" --body "..."
  ```
- The PR body should include:
  - Summary of changes and why
  - Link to the JIRA issue
  - Any notable implementation decisions

### 5.3 Request Reviewers
- Identify relevant reviewers:
  ```
  git log --format='%an' --since='1 year' -- <affected-files> | sort | uniq -c | sort -rn | head -10
  ```
- Cross-reference with active committers and request at least 2 reviewers:
  ```
  gh pr edit <number> --add-reviewer <reviewer1>,<reviewer2>
  ```

### 5.4 Monitor CI
- Run `/monitor-ci` to watch the CI run, diagnose any failures, and fix them.

### 5.5 Report
- Display the PR URL and a summary of what was done.
