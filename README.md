# Claude Code Commands

Custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) commands for Apache Camel development workflows.

These commands encode repeatable, multi-step development workflows that I use daily when contributing to [Apache Camel](https://github.com/apache/camel). Each command is a Markdown file that Claude Code interprets as a structured prompt.

## Commands

| Command | Description |
|---|---|
| [`/work-on`](commands/work-on.md) | Work on a JIRA issue end-to-end: claim, investigate, implement, self-review, create PR |
| [`/review-pr`](commands/review-pr.md) | Thorough PR code review: correctness, thread safety, security, performance, conventions |
| [`/address-review`](commands/address-review.md) | Address review feedback: categorize comments, fix issues, reply to reviewers |
| [`/monitor-ci`](commands/monitor-ci.md) | Monitor CI for a PR, diagnose failures, fix and re-push (up to 3 iterations) |
| [`/create-issue`](commands/create-issue.md) | Create a JIRA issue for a side problem found while working on something else |
| [`/merge-pr`](commands/merge-pr.md) | Merge a PR after verifying CI, approvals, and unresolved discussions |

## Installation

Copy the commands to your Claude Code commands directory:

```bash
# Global (available in all projects)
cp commands/*.md ~/.claude/commands/

# Project-specific
mkdir -p .claude/commands
cp commands/*.md .claude/commands/
```

Then invoke them in Claude Code with `/work-on CAMEL-12345`, `/review-pr 123`, etc.

## How it works

Each command file is a Markdown document that describes a multi-phase workflow. When you invoke a command, Claude Code reads the file and follows the instructions step by step, asking for confirmation at key decision points.

Key design principles:

- **Investigation before implementation** — the agent gathers context (git history, JIRA, blame) before writing any code
- **Human-in-the-loop** — the agent presents findings and waits for approval at critical steps
- **Self-review** — the `/work-on` command runs `/review-pr` on its own changes before creating a PR
- **Composability** — commands reference each other (e.g., `/work-on` calls `/monitor-ci` and `/review-pr`)

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub CLI](https://cli.github.com/) (`gh`) — for PR and CI operations
- A JIRA Bearer token in `$JIRA_TOKEN` — for JIRA operations
- Apache Camel development environment (Maven, JDK, etc.)

## Adapting for other projects

These commands are tailored for Apache Camel, but the patterns are generic. To adapt them:

1. Replace JIRA API URLs with your issue tracker
2. Adjust build commands (`mvn` → your build tool)
3. Update project conventions (component patterns, formatting rules, etc.)
4. Adjust the merge policy (approval count, squash vs. merge, etc.)

## License

Apache License 2.0
