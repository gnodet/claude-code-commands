Monitor CI for a PR, diagnose failures, and fix them.

If a PR number is provided ("$ARGUMENTS"), use that. Otherwise, detect the PR associated with the current branch by running `gh pr view --json number --jq '.number'`. If no PR is found, abort with a clear message.

## Phase 1: Check Current Status

- Run `gh pr checks <number>` to get the status of all checks.
- If all checks have passed, report "CI is green" and stop.
- If checks are still pending, watch them: `gh run watch <run_id> --exit-status` (with a 10-minute timeout).
- If checks have failed, proceed to Phase 2.

## Phase 2: Diagnose Failures

For each failed job:

### 2.1 Get Failure Details
- Get the job ID from `gh pr checks` output.
- Fetch job logs: `gh api repos/{owner}/{repo}/actions/jobs/<job_id>/logs`
- Look for the error: search for `FAILURE`, `ERROR`, `BUILD FAILURE`, `❌`, or the exit code section.

### 2.2 Categorize the Failure
Identify which category the failure falls into:
- **Uncommitted generated files**: "Fail if there are uncommitted changes" step failed → need to regenerate and commit
- **Compilation error**: code doesn't compile → fix the code
- **Test failure**: a test failed → analyze and fix
- **Formatting**: import order or code format issues → run `mvn formatter:format impsort:sort`
- **Docs build**: `yarn install` or `gulp` failure in docs module → check lockfile, node issues
- **Flaky/infra**: timeout, OOM, network issue → consider re-running
- **Other**: unknown failure → report details and ask user

### 2.3 Present Diagnosis
- Report which job(s) failed and why.
- Propose a fix. Wait for user approval if the fix involves code changes beyond formatting/regeneration.

## Phase 3: Fix

### 3.1 Apply Fix
- Check out the PR branch if not already on it.
- Apply the fix (regenerate files, fix formatting, fix code, etc.).
- Build and verify: `mvn install -B -pl <modules> -DskipTests`
- Run formatter: `mvn formatter:format impsort:sort -B -pl <modules>`
- Check `git status` for any remaining uncommitted changes.

### 3.2 Commit and Push
- Commit the fix with a descriptive message.
- Push to the PR branch.

## Phase 4: Re-monitor

- After pushing, watch the new CI run: `gh run watch <run_id> --exit-status`
- If it passes, report "CI is now green."
- If it fails again, go back to Phase 2 (max 3 iterations, then ask the user for help).
