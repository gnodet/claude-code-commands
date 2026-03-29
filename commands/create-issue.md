Create a JIRA issue for a problem found in passing (flaky test, unrelated bug, tech debt, doc issue).

The issue description must be provided as "$ARGUMENTS". If not provided, ask the user to describe the problem.

## Phase 1: Gather Information

### 1.1 Analyze the Problem
From the description and current context, determine:
- **Summary**: concise one-line title (under 80 chars)
- **Issue type**: Bug, Improvement, Task, or Wish
- **Component(s)**: identify the affected Camel component(s) from the module path (e.g., `camel-kafka` → `camel-kafka`)
- **Description**: expand the user's input into a proper issue description with:
  - What the problem is
  - How to reproduce (if applicable)
  - Expected vs actual behavior (if applicable)
  - Any relevant code references (file paths, test class names)
- **Priority**: Major (default), Critical (if blocking), Minor (if cosmetic/nit)

### 1.2 Check for Duplicates
- Search JIRA for existing issues with similar keywords:
  ```
  curl -s -H "Authorization: Bearer $JIRA_TOKEN" \
    "https://issues.apache.org/jira/rest/api/2/search?jql=project=CAMEL+AND+text+~+\"<keywords>\"+AND+status+not+in+(Closed,Resolved)&maxResults=5"
  ```
- If potential duplicates are found, show them to the user and ask whether to proceed or link to an existing issue.

## Phase 2: Create the Issue

### 2.1 Confirm with User
- Present the issue details (summary, type, component, priority, description) and ask for approval before creating.

### 2.2 Create in JIRA
- Create the issue:
  ```
  curl -s -X POST -H "Authorization: Bearer $JIRA_TOKEN" -H "Content-Type: application/json" \
    -d '{
      "fields": {
        "project": {"key": "CAMEL"},
        "summary": "<summary>",
        "issuetype": {"name": "<type>"},
        "components": [{"name": "<component>"}],
        "priority": {"name": "<priority>"},
        "description": "<description>"
      }
    }' "https://issues.apache.org/jira/rest/api/2/issue"
  ```
- Do NOT assign the issue (leave it unassigned for anyone to pick up, unless the user says they want to work on it).

### 2.3 Report
- Display the created issue key and URL: `https://issues.apache.org/jira/browse/CAMEL-XXXXX`
- If the user is currently working on a PR, mention: "Created CAMEL-XXXXX — not linked to current work."
