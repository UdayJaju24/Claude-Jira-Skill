---
name: update-jira
description: Manage Jira tickets - update descriptions, add comments, link MRs, and link related tickets
args:
  ticket_id:
    description: The Jira ticket ID (e.g., PROJ-123)
    required: true
  action:
    description: "Action to perform: 'update' (default), 'comment', 'link-mr'. If not specified, defaults to 'update'"
    required: false
  context:
    description: "Context for the action. For 'update': persona, business value, dependencies, blockers. For 'comment': the comment text to add"
    required: false
---

# Jira Ticket Manager

Manage Jira tickets with multiple actions: update descriptions, add comments, link merge requests, and link related tickets.

## Input Parameters
- **ticket_id**: The Jira ticket identifier (e.g., PROJ-123)
- **action**: What to do (update | comment | link-mr) - defaults to 'update'
- **context**: Additional context based on action

## Actions Overview

### 1. UPDATE (Default)
Updates ticket description with code changes
```
/update-jira PROJ-123
/update-jira PROJ-123 update "persona: admin, blockers: AUTH-100"
```

### 2. COMMENT
Adds a comment to the ticket
```
/update-jira PROJ-123 comment "This is ready for review"
/update-jira PROJ-123 comment
```
(Without text, asks what to comment)

### 3. LINK-MR
Adds MR/PR link as a comment
```
/update-jira PROJ-123 link-mr
```
Auto-detects the MR/PR URL and adds it as a comment

---

## VALIDATION (All Actions)

**Before performing ANY action, always validate:**

### 1. Validate Ticket ID Format
Check if ticket ID matches standard Jira format: `[A-Z]+-[0-9]+`

Examples:
- ✅ Valid: PROJ-123, AUTH-456, PAYMENT-1
- ❌ Invalid: proj-123, PROJ, 123, PROJ-ABC

If invalid format:
```
Error: Invalid ticket ID format. Expected format: PROJECT-123
Example: AUTH-456, PROJ-100
```

### 2. Check Ticket Exists
Fetch the ticket to verify it exists:

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" \
  "${JIRA_BASE_URL}/rest/api/3/issue/${ticket_id}" \
  -w "\n%{http_code}"
```

**Handle responses:**
- **200**: Ticket exists ✅ Proceed to step 3
- **404**: Ticket not found ❌ Show error:
  ```
  Error: Ticket ${ticket_id} does not exist in Jira.
  Please check the ticket ID and try again.
  Link: ${JIRA_BASE_URL}/browse/${ticket_id}
  ```
- **401/403**: Authentication issue ❌ Show error:
  ```
  Error: Authentication failed. Please check your JIRA_EMAIL and JIRA_API_TOKEN.
  ```
- **Other**: Network/API issue ❌ Show error with status code

### 3. Verify Ticket Assignment

**Extract assignee from the ticket response and verify:**

```bash
# Parse the response from step 2
ASSIGNEE_EMAIL=$(echo "$response" | jq -r '.fields.assignee.emailAddress // "unassigned"')

if [ "$ASSIGNEE_EMAIL" == "unassigned" ] || [ "$ASSIGNEE_EMAIL" == "null" ]; then
  # Ticket is unassigned - Ask user for confirmation
  echo "⚠️  Warning: Ticket ${ticket_id} is currently unassigned."
  echo ""
  echo "Do you want to proceed with updating this ticket?"
  echo "1. Yes, proceed (I'm taking ownership)"
  echo "2. No, cancel"
elif [ "$ASSIGNEE_EMAIL" != "$JIRA_EMAIL" ]; then
  # Ticket is assigned to someone else - Show warning and ask
  ASSIGNEE_NAME=$(echo "$response" | jq -r '.fields.assignee.displayName // "Unknown"')
  echo "⚠️  Warning: Ticket ${ticket_id} is assigned to someone else!"
  echo ""
  echo "Assigned to: ${ASSIGNEE_NAME} (${ASSIGNEE_EMAIL})"
  echo "Your email: ${JIRA_EMAIL}"
  echo ""
  echo "Do you want to proceed anyway?"
  echo "1. Yes, proceed (I'm collaborating with them)"
  echo "2. No, cancel"
else
  # Ticket is assigned to current user - Proceed
  echo "✅ Ticket ${ticket_id} is assigned to you"
fi
```

**Confirmation required if:**
- Ticket is unassigned
- Ticket is assigned to a different user

Only proceed after user confirms. This prevents accidentally updating someone else's work.

### 4. Prevent Accidental Overwrites (UPDATE action only)
When updating description, if existing description has substantial content (>200 chars):

**Ask for confirmation:**
```
⚠️  Warning: This ticket already has a description (542 characters).

Current description preview:
---
[First 200 chars of existing description]...
---

Do you want to:
1. APPEND new changes (preserves existing description) ← Recommended
2. REPLACE completely (overwrites everything)
3. CANCEL
```

Only proceed after user confirms their choice.

---

## ACTION: UPDATE

Updates the ticket description with code changes.

### Process

#### 1. Parse User Context (if provided)

Extract from context parameter:
- **persona**: Who will use this feature
- **business_value** / **value**: Why this matters
- **blockers**: Tickets or issues blocking this work
- **depends** / **dependencies**: External dependencies
- **acceptance** / **criteria**: Specific acceptance criteria

#### 2. Fetch and Analyze Existing Description

After validation passes, fetch ticket details:
- Parse existing description (if any)
- Identify what's already documented
- Extract mentioned commits, dependencies, tickets

**Auto-detect mode:**
- Empty/minimal description → **CREATE MODE**
- Substantial content → **APPEND MODE** (with confirmation)

#### 3. Analyze Code Changes

Run these commands in parallel:
```bash
git status
git diff
git diff --staged
git log origin/main..HEAD --oneline
git diff origin/main...HEAD
```

For APPEND mode: Identify only NEW changes not mentioned in existing description.

#### 4. Extract Related Tickets

**From code changes:**
- Scan commit messages for ticket references: `PROJ-123`, `AUTH-456`
- Scan code comments for ticket mentions

**From user context:**
- Extract from `blockers:` parameter
- Extract from `depends:` parameter

**Create list of related tickets** to link later.

#### 5. Understand Changes and Gather Context

**From code:** functionality, technical details, technical dependencies
**From user context:** persona, business value, blockers, acceptance criteria

**If critical info missing**, use AskUserQuestion for:
- Persona (if can't infer)
- Business value (if not obvious)
- Dependencies/blockers (if any)

#### 6. Generate or Update Description

**CREATE MODE:**
```
Definition

A unit of customer or user value.

Story (Required)

As a <persona>, I want <capability> so that <value>.

Description (Required)

<Technical details and context>

Dependencies

<List dependencies with links to tickets>

Acceptance Criteria

- <Clear, testable conditions>

Done Checklist

- [ ] Code complete and reviewed
- [ ] Tests automated and passing
- [ ] Documentation updated
- [ ] Acceptance criteria met
```

**APPEND MODE:**
Preserve existing content, add:
```
--- Update: YYYY-MM-DD ---
<New technical details>
```

Add new dependencies and acceptance criteria.

#### 7. Link Related Tickets

For each ticket found in blockers/dependencies:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" \
  -d '{
    "type": {
      "name": "<link_type>"
    },
    "inwardIssue": {
      "key": "${ticket_id}"
    },
    "outwardIssue": {
      "key": "${related_ticket}"
    },
    "comment": {
      "body": "<explanation>"
    }
  }' \
  "${JIRA_BASE_URL}/rest/api/3/issueLink"
```

**Link types and descriptions:**

| Context | Link Type | Description |
|---------|-----------|-------------|
| `blockers: AUTH-123` | `Blocks` | "${ticket_id} is blocked by ${related_ticket}" |
| `depends: INFRA-50` | `Relates` | "${ticket_id} depends on ${related_ticket}" |
| Found in commits | `Relates` | "${ticket_id} is related to ${related_ticket}" |

Example:
```
blockers: AUTH-100, AUTH-101
```
Creates links:
- AUTH-100 blocks PROJ-123 - "Waiting for OAuth integration"
- AUTH-101 blocks PROJ-123 - "Pending API authentication setup"

#### 8. Update Ticket Description

```bash
curl -X PUT \
  -H "Content-Type: application/json" \
  -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" \
  -d '{
    "fields": {
      "description": {
        "type": "doc",
        "version": 1,
        "content": [...]
      }
    }
  }' \
  "${JIRA_BASE_URL}/rest/api/3/issue/${ticket_id}"
```

#### 9. Confirm Success

Show:
- ✅ Description updated
- ✅ X related tickets linked (list them)
- 🔗 Link to view: `${JIRA_BASE_URL}/browse/${ticket_id}`

For APPEND mode:
- What was added vs preserved
- New dependencies/criteria

---

## ACTION: COMMENT

Adds a comment to the Jira ticket.

### Usage
```
/update-jira PROJ-123 comment "Ready for review"
/update-jira PROJ-123 comment "QA can start testing"
/update-jira PROJ-123 comment
```

### Process

#### 1. Validation
Run standard validation (format, ticket exists)

#### 2. Get Comment Text

**If context parameter provided:**
Use it as the comment text.

**If context parameter empty:**
Ask user: "What comment would you like to add to ${ticket_id}?"

#### 3. Add Comment

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" \
  -d '{
    "body": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "paragraph",
          "content": [
            {
              "type": "text",
              "text": "<comment_text>"
            }
          ]
        }
      ]
    }
  }' \
  "${JIRA_BASE_URL}/rest/api/3/issue/${ticket_id}/comment"
```

#### 4. Confirm Success

```
✅ Comment added to ${ticket_id}
💬 "${comment_text}"
🔗 View: ${JIRA_BASE_URL}/browse/${ticket_id}
```

---

## ACTION: LINK-MR

Automatically detects MR/PR URL and adds it as a comment to the Jira ticket.

### Usage
```
/update-jira PROJ-123 link-mr
```

### Process

#### 1. Validation
Run standard validation (format, ticket exists)

#### 2. Detect MR/PR URL

**Auto-detection using API tokens:**

Search order: Origin repository first, then upstream (for fork workflows)

**GitHub API:**
```bash
# Requires GITHUB_TOKEN or GH_TOKEN environment variable
if [ -n "$GITHUB_TOKEN" ] || [ -n "$GH_TOKEN" ]; then
  TOKEN="${GITHUB_TOKEN:-$GH_TOKEN}"
  BRANCH=$(git branch --show-current)
  ORIGIN_REPO=$(git config --get remote.origin.url | sed 's/.*[:/]\(.*\)\.git/\1/')
  REPO_OWNER=$(echo "$ORIGIN_REPO" | cut -d'/' -f1)
  
  # Step 1: Search origin repository first
  # For direct repos: searches PRs in the main repo
  # For forks: searches PRs within your fork (rare but possible)
  PR_URL=$(curl -s -H "Authorization: Bearer ${TOKEN}" \
    "https://api.github.com/repos/${ORIGIN_REPO}/pulls?head=${REPO_OWNER}:${BRANCH}&state=open" \
    | jq -r '.[0].html_url // empty')
  
  # Step 2: If not found in origin, try upstream (fork workflow)
  # This finds PRs from your fork to the upstream repository
  if [ -z "$PR_URL" ] || [ "$PR_URL" == "null" ]; then
    if git remote get-url upstream &>/dev/null; then
      UPSTREAM_REPO=$(git config --get remote.upstream.url | sed 's/.*[:/]\(.*\)\.git/\1/')
      
      PR_URL=$(curl -s -H "Authorization: Bearer ${TOKEN}" \
        "https://api.github.com/repos/${UPSTREAM_REPO}/pulls?head=${REPO_OWNER}:${BRANCH}&state=open" \
        | jq -r '.[0].html_url // empty')
      
      if [ -z "$PR_URL" ] || [ "$PR_URL" == "null" ]; then
        echo "" # No PR found in upstream either
      else
        echo "$PR_URL"
      fi
    fi
  else
    echo "$PR_URL"
  fi
fi
```

**GitLab API:**
```bash
# Requires GITLAB_TOKEN environment variable
if [ -n "$GITLAB_TOKEN" ]; then
  BRANCH=$(git branch --show-current)
  ORIGIN_PROJECT=$(git config --get remote.origin.url | sed 's/.*[:/]\(.*\)\.git/\1/' | sed 's/\//%2F/g')
  
  # Step 1: Search origin repository first
  # For direct repos: searches MRs in the main project
  # For forks: searches MRs within your fork (rare but possible)
  MR_URL=$(curl -s -H "PRIVATE-TOKEN: ${GITLAB_TOKEN}" \
    "https://gitlab.com/api/v4/projects/${ORIGIN_PROJECT}/merge_requests?source_branch=${BRANCH}&state=opened" \
    | jq -r '.[0].web_url // empty')
  
  # Step 2: If not found in origin, try upstream (fork workflow)
  # This finds MRs from your fork to the upstream project
  if [ -z "$MR_URL" ] || [ "$MR_URL" == "null" ]; then
    if git remote get-url upstream &>/dev/null; then
      UPSTREAM_PROJECT=$(git config --get remote.upstream.url | sed 's/.*[:/]\(.*\)\.git/\1/' | sed 's/\//%2F/g')
      
      MR_URL=$(curl -s -H "PRIVATE-TOKEN: ${GITLAB_TOKEN}" \
        "https://gitlab.com/api/v4/projects/${UPSTREAM_PROJECT}/merge_requests?source_branch=${BRANCH}&state=opened" \
        | jq -r '.[0].web_url // empty')
      
      if [ -z "$MR_URL" ] || [ "$MR_URL" == "null" ]; then
        echo "" # No MR found in upstream either
      else
        echo "$MR_URL"
      fi
    fi
  else
    echo "$MR_URL"
  fi
fi
```

**Manual Input (Fallback):**

If auto-detection fails (no token set or no MR/PR found), ask user for URL:

```
Could not auto-detect MR/PR URL.

To enable auto-detection, add to .claude/settings.json:
- GitHub: Set GITHUB_TOKEN environment variable
- GitLab: Set GITLAB_TOKEN environment variable

Please provide the MR/PR URL manually:
> _
```

#### 3. Extract MR/PR Details

From the URL or API, get:
- MR/PR number
- Title
- Status (open/merged/closed)
- Author

#### 4. Add Comment with MR/PR Link

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" \
  -d '{
    "body": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "paragraph",
          "content": [
            {
              "type": "text",
              "text": "Merge Request: "
            },
            {
              "type": "text",
              "text": "<mr_url>",
              "marks": [{"type": "link", "attrs": {"href": "<mr_url>"}}]
            }
          ]
        },
        {
          "type": "paragraph",
          "content": [
            {
              "type": "text",
              "text": "Status: <status> | Author: <author>"
            }
          ]
        }
      ]
    }
  }' \
  "${JIRA_BASE_URL}/rest/api/3/issue/${ticket_id}/comment"
```

#### 5. Optional: Update Ticket Status

Ask user: "Would you like to transition this ticket to 'In Review'?"

If yes:
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" \
  -d '{
    "transition": {
      "id": "<transition_id>"
    }
  }' \
  "${JIRA_BASE_URL}/rest/api/3/issue/${ticket_id}/transitions"
```

(Get transition ID from `/rest/api/3/issue/${ticket_id}/transitions`)

#### 6. Confirm Success

```
✅ MR/PR linked to ${ticket_id}
🔗 MR: <mr_url>
📊 Status: <status>
🔗 View ticket: ${JIRA_BASE_URL}/browse/${ticket_id}
```

---

## Important Notes

### Environment Variables
- **JIRA_BASE_URL**: Your Jira instance URL (e.g., https://yourcompany.atlassian.net)
- **JIRA_EMAIL**: Your Jira account email (required for authentication)
- **JIRA_API_TOKEN**: Your Jira API token

**Validation:**
Before making any API call, verify all three environment variables are set:
```bash
if [ -z "$JIRA_BASE_URL" ] || [ -z "$JIRA_EMAIL" ] || [ -z "$JIRA_API_TOKEN" ]; then
  echo "Error: Missing required environment variables"
  echo "Required: JIRA_BASE_URL, JIRA_EMAIL, JIRA_API_TOKEN"
  echo ""
  echo "Add them to .claude/settings.json:"
  echo '{'
  echo '  "env": {'
  echo '    "JIRA_BASE_URL": "https://yourcompany.atlassian.net",'
  echo '    "JIRA_EMAIL": "your-email@company.com",'
  echo '    "JIRA_API_TOKEN": "your-api-token"'
  echo '  }'
  echo '}'
  exit 1
fi
```

### Error Handling
- Always validate before making changes
- Show clear error messages with next steps
- Provide ticket links for easy access
- Handle API errors gracefully

### Ticket Linking
- Automatically link tickets mentioned in blockers/dependencies
- Use appropriate link types (Blocks, Relates, Depends on)
- Add descriptive comments to links
- Show summary of linked tickets

### Smart Detection
- Auto-detect update vs append mode
- Auto-detect MR/PR URLs
- Extract ticket IDs from commits and context
- Infer persona/value when obvious

### Confirmation Prompts
- Confirm before overwriting substantial descriptions
- Confirm before transitioning ticket status
- Ask for missing information when needed

### Git Platform Support
- GitHub (via gh CLI or API)
- GitLab (via glab CLI or API)
- Bitbucket (via API)
- Fork workflows (checks origin first, then upstream remote)
- Generic git remotes (build URL from remote)
