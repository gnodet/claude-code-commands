As an experienced senior software developer and long-time Apache Camel committer, conduct a thorough review of a PR.

If a PR number is provided ("$ARGUMENTS"), use that. Otherwise, detect the PR associated with the current branch by running `gh pr view --json number,url --jq '.number'`. If no PR is found, abort with a clear message.

## Phase 1: Context & Justification
- Read the PR description, referenced JIRA issue, and any linked discussions
- Understand the problem being solved and whether it's actually needed
- Check git history for related changes or prior attempts
- Assess whether this is the right approach or if there's a simpler/better alternative

## Phase 2: Code Review
Analyze the diff for:
- **Correctness**: logic errors, edge cases, off-by-one, null handling
- **Thread safety**: concurrent access, shared mutable state, proper synchronization
- **Security**: injection risks, input validation at system boundaries, credential handling
- **Performance**: unnecessary allocations, N+1 patterns, blocking in async paths, resource leaks
- **Camel conventions**: proper use of Component/Endpoint/Producer/Consumer patterns, annotation usage (@UriParam descriptions), error handling via Exchange
- **Java best practices**: resource management (try-with-resources), proper use of Optional, immutability where appropriate
- **Test coverage**: are new features/fixes tested? are edge cases covered? are tests deterministic?
- **Documentation**: are new options/features documented? are adoc files updated?
- **Generated files**: are catalog, DSL, and schema files regenerated if needed?

## Phase 3: Report
Provide a structured summary:
1. **Overview**: what the PR does in 2-3 sentences
2. **Verdict**: approve / request changes / needs discussion
3. **Issues**: list of problems found, ranked by severity (blocking, major, minor, nit)
4. **Suggestions**: optional improvements that aren't blocking

Use sub-agents to parallelize independent research (e.g., checking JIRA, reviewing test coverage, analyzing the main code changes).
