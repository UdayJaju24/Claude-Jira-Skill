# Jira Ticket Manager - Claude Skill

A powerful Claude Code skill that manages Jira tickets with code changes, comments, MR linking, and automatic ticket relationship management.

## 📦 Installation

**New user?** See **[INSTALLATION.md](INSTALLATION.md)** for complete setup instructions.

**Quick install:**
```bash
# Clone the repo
git clone https://github.com/UdayJaju24/Claude-Jira-Skill.git
cd Jira-Claude-Skill

# Install globally
cp -r .claude/skills/update-jira ~/.claude/skills/update-jira
cp .claude/settings.json ~/.claude/settings.json

# Edit ~/.claude/settings.json and add your tokens
```

See [Setup](#setup) section below for token details.

## Features

🎯 **Update Descriptions** - Auto-generates ticket descriptions from code changes  
💬 **Add Comments** - Quickly add comments to tickets  
🔗 **Link MRs/PRs** - Auto-detect and link merge requests  
🔗 **Link Related Tickets** - Automatically link dependencies and blockers  
✅ **Validation** - Prevents errors and accidental overwrites  
🤖 **Smart Detection** - Auto-detects create vs append mode  

## Quick Start

```bash
# Update ticket description (auto-detects create vs append)
/update-jira PROJ-123

# Add a comment
/update-jira PROJ-123 comment "Ready for review"

# Link MR/PR automatically
/update-jira PROJ-123 link-mr

# Update with context and automatic ticket linking
/update-jira PROJ-123 update "persona: admin, blockers: AUTH-100, depends: INFRA-50"
```

---

## Table of Contents

- [Setup](#setup)
- [Usage](#usage)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

---

## Setup

### 1. Jira Credentials (Required for All Features)

#### How to Get Your Jira API Token

**Step 1:** Go to https://id.atlassian.com/manage-profile/security/api-tokens

**Step 2:** Click **"Create API token"**

**Step 3:** Enter a label
- Label: `Claude Code Skill` (or any name you prefer)

**Step 4:** Click **"Create"**

**Step 5:** **IMPORTANT:** Copy the token immediately!
- ⚠️ You'll only see it once
- It looks like: `ATATTxxxxxxxxxxxxxxxxxxx`

**Step 6:** Find your Jira details:
- **JIRA_BASE_URL:** Look at your browser when logged into Jira
  - Example: `https://yourcompany.atlassian.net`
  - Example: `https://jira.yourcompany.com`
- **JIRA_EMAIL:** Your Atlassian account email (the one you log in with)

**Step 7:** Add to `.claude/settings.json`:
```json
{
  "env": {
    "JIRA_BASE_URL": "https://yourcompany.atlassian.net",
    "JIRA_EMAIL": "your-email@company.com",
    "JIRA_API_TOKEN": "ATATTxxxxxxxxxxxxxxxxxxx"
  }
}
```

**Step 8:** Test it works:
```bash
/update-jira TEST-123 comment "Testing setup"
```

**Common Issues:**
- ❌ 401 Unauthorized → Check JIRA_EMAIL matches your Atlassian account
- ❌ 404 Not Found → Check JIRA_BASE_URL is correct
- ❌ Token not working → Generate a new token (old one may have been copied incorrectly)

### 2. GitHub/GitLab Access (Optional - Only for MR/PR Linking)

**Only needed if you want to use `/update-jira TICKET-ID link-mr`**

Choose the option that fits your workflow:

---

#### Option A: GitHub Personal Access Token

**When to use:** You work with GitHub repositories and want automatic PR detection

**Step 1:** Choose token type:
- **Fine-grained token (Recommended):** More secure, repo-specific access
- **Classic token:** Simpler, broader access

**Step 2a: Fine-Grained Token (Recommended)**

1. Go to https://github.com/settings/tokens?type=beta
2. Click **"Generate new token"**
3. Configure:
   - **Token name:** `Claude Jira Skill`
   - **Expiration:** `90 days` (recommended)
   - **Repository access:** Choose "Only select repositories"
   - Select the repositories you work with
4. Under **"Repository permissions"**:
   - **Pull requests:** `Read-only` ✅
   - **Metadata:** `Read-only` (auto-granted)
5. Click **"Generate token"**
6. Copy token (starts with `github_pat_...`)

**Step 2b: Classic Token (Alternative)**

1. Go to https://github.com/settings/tokens
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. Configure:
   - **Note:** `Claude Jira Skill`
   - **Expiration:** `90 days`
   - **Scopes:** 
     - ✅ `repo` (for private repos)
     - OR ✅ `public_repo` (for public repos only)
4. Click **"Generate token"**
5. Copy token (starts with `ghp_...`)

**Step 3:** Add to `.claude/settings.json`:
```json
{
  "env": {
    "GITHUB_TOKEN": "github_pat_xxxxx"
    // OR
    "GITHUB_TOKEN": "ghp_xxxxx"
  }
}
```

---

#### Option B: GitLab Personal Access Token

**When to use:** You work with GitLab repositories and want automatic MR detection

**Step 1:** Go to https://gitlab.com/-/profile/personal_access_tokens

**Step 2:** Click **"Add new token"**

**Step 3:** Configure:
- **Token name:** `Claude Jira Skill`
- **Expiration date:** Choose date (90 days recommended)
- **Scopes:** Check one of these:
  - ✅ `read_api` (Minimal - read-only access, recommended)
  - OR ✅ `api` (Full API access - if you need more than just reading)

**Step 4:** Click **"Create personal access token"**

**Step 5:** Copy token (starts with `glpat_...`)
- ⚠️ You'll only see it once!

**Step 6:** Add to `.claude/settings.json`:
```json
{
  "env": {
    "GITLAB_TOKEN": "glpat_xxxxx"
  }
}
```

**For GitLab Self-Hosted:**
Use your GitLab instance URL instead of `https://gitlab.com`

---

#### Option C: Manual Entry

**When to use:** You don't want to set up tokens, or only use link-mr occasionally

**How it works:**
- No setup required
- When you run `/update-jira TICKET-ID link-mr`, the skill will ask you to paste the MR/PR URL
- Just copy the URL from your browser

**Example:**
```bash
/update-jira PROJ-123 link-mr
# Skill asks: "Please provide the MR/PR URL manually:"
# You paste: https://github.com/company/repo/pull/456
```

---

#### Complete Configuration Examples

**Minimal (Jira only):**
```json
{
  "env": {
    "JIRA_BASE_URL": "https://yourcompany.atlassian.net",
    "JIRA_EMAIL": "your-email@company.com",
    "JIRA_API_TOKEN": "ATATTxxxxxxxxxx"
  }
}
```

**With GitHub:**
```json
{
  "env": {
    "JIRA_BASE_URL": "https://yourcompany.atlassian.net",
    "JIRA_EMAIL": "your-email@company.com",
    "JIRA_API_TOKEN": "ATATTxxxxxxxxxx",
    "GITHUB_TOKEN": "github_pat_xxxxxxxxxx"
  }
}
```

**With GitLab:**
```json
{
  "env": {
    "JIRA_BASE_URL": "https://yourcompany.atlassian.net",
    "JIRA_EMAIL": "your-email@company.com",
    "JIRA_API_TOKEN": "ATATTxxxxxxxxxx",
    "GITLAB_TOKEN": "glpat_xxxxxxxxxx"
  }
}
```

**With Both (if you use GitHub and GitLab):**
```json
{
  "env": {
    "JIRA_BASE_URL": "https://yourcompany.atlassian.net",
    "JIRA_EMAIL": "your-email@company.com",
    "JIRA_API_TOKEN": "ATATTxxxxxxxxxx",
    "GITHUB_TOKEN": "github_pat_xxxxxxxxxx",
    "GITLAB_TOKEN": "glpat_xxxxxxxxxx"
  }
}
```

**Security:** 
- ⚠️ Never commit tokens to git!
- Add `.claude/settings.json` to `.gitignore`
- Rotate tokens every 90 days

---

## Usage

### Action 1: UPDATE - Update Ticket Description

**Analyzes code changes and updates Jira ticket description in standardized format.**

```bash
# Basic usage (auto-detects create vs append)
/update-jira PROJ-123

# With context
/update-jira PROJ-123 update "persona: admin user, value: security compliance"

# With dependencies and blockers (auto-links tickets!)
/update-jira PROJ-123 update "persona: customer, blockers: AUTH-100, AUTH-101, depends: INFRA-50, value: faster checkout"
```

**What it does:**
1. ✅ Validates ticket ID format and existence
2. ✅ Analyzes all code changes (uncommitted, staged, committed)
3. ✅ Auto-detects create mode (empty ticket) vs append mode (existing description)
4. ✅ **Automatically links related tickets** mentioned in `blockers:` and `depends:`
5. ✅ Generates/updates description in standardized format
6. ✅ Asks for confirmation before overwriting existing content

**Context options:**
- `persona:` - Who uses this (admin, customer, developer)
- `value:` - Why it matters to the business
- `blockers:` - Blocking tickets (e.g., `AUTH-100, AUTH-101`) → Creates "blocks" Jira links
- `depends:` - Dependencies (e.g., `INFRA-50`) → Creates "relates" Jira links
- `acceptance:` - Specific success criteria

**Automatic Ticket Linking Example:**
```bash
/update-jira PAYMENT-100 update "blockers: AUTH-50, AUTH-51, depends: INFRA-200"
```
Creates Jira links:
- AUTH-50 **blocks** PAYMENT-100
- AUTH-51 **blocks** PAYMENT-100  
- INFRA-200 **relates to** PAYMENT-100

**Overwrite Protection:**
If ticket has existing content, you'll be asked:
```
⚠️  Warning: This ticket already has a description (320 characters).

Do you want to:
1. APPEND new changes (preserves existing) ← Recommended
2. REPLACE completely (overwrites everything)
3. CANCEL
```

**Generated Description Format:**
```
Definition
A unit of customer or user value.

Story (Required)
As a <persona>, I want <capability> so that <value>.

Description (Required)
<Technical details from code analysis>

--- Update: 2026-04-25 ---  [if appending]
<New technical details>

Dependencies
- AUTH-100: OAuth integration (blocker)
- INFRA-50: Database migration (dependency)

Acceptance Criteria
- Login completes successfully with valid credentials
- Failed login shows clear error message

Done Checklist
- [ ] Code complete and reviewed
- [ ] Tests automated and passing
- [ ] Documentation updated
- [ ] Acceptance criteria met
```

### Action 2: COMMENT - Add Comment to Ticket

**Quickly add comments to tickets.**

```bash
# With comment text
/update-jira PROJ-123 comment "Ready for QA testing"

# Interactive (asks for comment)
/update-jira PROJ-123 comment
```

**Common use cases:**
```bash
# Code review status
/update-jira PROJ-123 comment "Code review completed, LGTM ✅"

# Deployment notification
/update-jira PROJ-123 comment "Deployed to staging: https://staging.example.com"

# Testing results
/update-jira PROJ-123 comment "Tested on Chrome, Firefox, Safari - all passing"

# Blocker updates
/update-jira PROJ-123 comment "Unblocked: received API keys from vendor"
```

### Action 3: LINK-MR - Link Merge/Pull Request

**Auto-detects and links your MR/PR to the ticket.**

```bash
/update-jira PROJ-123 link-mr
```

**What it does:**
1. ✅ Auto-detects git platform (GitHub/GitLab/Bitbucket)
2. ✅ Finds MR/PR for current branch
3. ✅ Adds formatted comment with MR link
4. ✅ Optionally transitions ticket to "In Review"

**Auto-detection methods (tries in order):**
1. CLI tools: `gh pr view` or `glab mr view`
2. API with tokens: Uses `GITHUB_TOKEN` or `GITLAB_TOKEN` from settings
3. Manual entry: Asks you to paste the URL

**Output:**
```
✅ MR/PR linked to PROJ-123
🔗 MR: https://github.com/company/repo/pull/42
📊 Status: Open
👤 Author: ujaju
✅ Ticket transitioned to 'In Review'
```

---

## Examples

### Example 1: Feature with Dependencies

```bash
# Make changes
git commit -m "Integrate Stripe payments"

# Update ticket with context
/update-jira PAYMENT-100 update "persona: customer, value: secure checkout, blockers: INFRA-50, depends: STRIPE-10, STRIPE-20"
```

**Result:**
```
✅ Description updated (CREATE mode)
✅ 3 related tickets linked:
   - INFRA-50 (blocks): Waiting for database migration
   - STRIPE-10 (relates): Stripe API key configuration
   - STRIPE-20 (relates): Webhook endpoint setup
🔗 View: https://yourcompany.atlassian.net/browse/PAYMENT-100
```

### Example 2: Iterative Development

```bash
# Week 1: Initial work
git commit -m "Add login form"
/update-jira AUTH-200 update "persona: user, value: secure authentication"
# → Creates description

# Week 2: More work on same ticket
git commit -m "Add password reset"
/update-jira AUTH-200
# → Asks: "APPEND or REPLACE?" → Choose APPEND
# → Preserves Week 1 work, adds "--- Update: 2026-04-25 ---" section
```

### Example 3: Code Review Workflow

```bash
# Create MR
git push origin feature/notifications

# Link MR to ticket
/update-jira NOTIF-50 link-mr
# → Auto-detects MR URL
# → Adds comment with link
# → Asks: "Transition to 'In Review'?" → Yes
# → Ticket status updated

# After review
/update-jira NOTIF-50 comment "Addressed review feedback, ready for re-review"
```

### Example 4: Bug Fix with Multiple Blockers

```bash
git commit -m "Fix race condition in order processing"

/update-jira BUG-500 update "persona: customer, value: prevents double-charging, blockers: BUG-499, BUG-498, acceptance: order processing is idempotent"
```

**Result:**
- Description created with bug fix details
- BUG-499 and BUG-498 linked as blockers
- Specific acceptance criteria added

---

## Troubleshooting

### Token Issues

#### "Authentication failed" (Jira)
```
❌ Error: Authentication failed. Please check your JIRA_EMAIL and JIRA_API_TOKEN.
```
**Fix:**
1. Verify `JIRA_EMAIL` matches your Atlassian account email (case-sensitive)
2. Check `JIRA_API_TOKEN` was copied completely (no extra spaces)
3. Verify `JIRA_BASE_URL` is correct (check your browser URL when logged into Jira)
4. Generate a new Jira API token at https://id.atlassian.com/manage-profile/security/api-tokens
5. Restart Claude Code to reload settings

**Test token manually:**
```bash
curl -u "your-email@company.com:YOUR_TOKEN" \
  https://yourcompany.atlassian.net/rest/api/3/myself
```
Should return your user info if token is valid.

---

#### "Could not auto-detect MR/PR URL" (GitHub/GitLab)
```
⚠️ Could not auto-detect MR/PR URL.
```

**For GitHub:**
1. Check `GITHUB_TOKEN` is set in `.claude/settings.json`
2. Verify token has `Pull requests: Read` permission
3. Test token:
   ```bash
   curl -H "Authorization: Bearer YOUR_GITHUB_TOKEN" \
     https://api.github.com/user
   ```
   Should return your GitHub user info

**For GitLab:**
1. Check `GITLAB_TOKEN` is set in `.claude/settings.json`
2. Verify token has `read_api` or `api` scope
3. Test token:
   ```bash
   curl -H "PRIVATE-TOKEN: YOUR_GITLAB_TOKEN" \
     https://gitlab.com/api/v4/user
   ```
   Should return your GitLab user info

**Alternative:** Just paste the MR/PR URL manually when prompted

---

### Ticket Issues

#### "Invalid ticket ID format"
```
❌ Error: Invalid ticket ID format. Expected format: PROJECT-123
```
**Fix:** Use format like `PROJ-123`, `AUTH-456` (uppercase project, dash, number)

---

#### "Ticket does not exist"
```
❌ Error: Ticket PROJ-999 does not exist in Jira.
```
**Fix:** 
1. Verify ticket ID in Jira web interface
2. Check you have permission to view the ticket
3. Ensure you're using the correct Jira instance (check JIRA_BASE_URL)

---

#### "Ticket is assigned to someone else"
```
⚠️ Warning: Ticket DW-2306 is assigned to John Doe (john@company.com)
```
**This is a safety feature!**
- If you see this, confirm you want to update their ticket
- Or ask them to reassign it to you first
- Or choose "No, cancel" and update your own ticket

---

### Configuration Issues

#### Settings not loading
**Fix:**
```bash
# Validate JSON syntax
cat .claude/settings.json | jq .

# Should output formatted JSON, not an error
# If error, fix the JSON syntax
```

**Common JSON errors:**
- Missing comma between properties
- Trailing comma on last property
- Wrong quote type (use `"` not `'`)
- Unescaped special characters

**Then restart Claude Code**

---

#### Token visible in git commits
**Fix:**
1. Add `.claude/settings.json` to `.gitignore`:
   ```bash
   echo ".claude/settings.json" >> .gitignore
   ```
2. If already committed, remove from git history:
   ```bash
   git rm --cached .claude/settings.json
   git commit -m "Remove settings.json from git"
   ```
3. Rotate all exposed tokens immediately!

---

### Still Having Issues?

**Check this checklist:**
- [ ] All tokens copied completely (no spaces/newlines)
- [ ] JIRA_EMAIL matches Atlassian account exactly
- [ ] JIRA_BASE_URL doesn't have trailing slash
- [ ] `.claude/settings.json` has valid JSON syntax
- [ ] Claude Code restarted after changing settings
- [ ] Tokens haven't expired (regenerate if old)

**Get debug info:**
```bash
# Check which variables are set
env | grep -E "JIRA|GITHUB|GITLAB"
```

**99% of issues are:**
1. Typo in email or token
2. Settings file not reloaded (restart Claude Code)
3. Wrong Jira base URL

---

## Validation Features

All actions include automatic validation:

### ✅ Ticket ID Format
- Valid: `PROJ-123`, `AUTH-456`
- Invalid: `proj-123`, `PROJ`, `PROJ-ABC`

### ✅ Ticket Existence
Checks Jira API before any operation

### ✅ Overwrite Protection
Confirms before replacing existing descriptions >200 characters

### ✅ Authentication
Validates Jira credentials and provides clear error messages

---

## All Features Summary

| Feature | Command | Needs Jira | Needs Git Auth |
|---------|---------|-----------|---------------|
| Update description | `/update-jira PROJ-123` | ✅ | ❌ |
| Update with context | `/update-jira PROJ-123 update "..."` | ✅ | ❌ |
| Auto-link tickets | `blockers: AUTH-100, depends: INFRA-50` | ✅ | ❌ |
| Add comment | `/update-jira PROJ-123 comment "..."` | ✅ | ❌ |
| Link MR/PR | `/update-jira PROJ-123 link-mr` | ✅ | ✅ |

---

## File Structure

```
.
├── .claude/
│   ├── settings.json          # Your credentials (update this!)
│   └── skills/
│       └── update-jira.md     # The skill definition
├── .gitignore                 # Protects API tokens
├── README.md                  # This file
└── main.py
```

---

## Security Notes

- ✅ Add `.claude/settings.json` to `.gitignore`
- ✅ Never commit tokens to version control
- ✅ Use CLI tools (most secure) or `.claude/settings.local.json` for personal tokens
- ✅ Rotate tokens every 90 days
- ⚠️ Tokens in settings.json are plain text

**Recommended setup for teams:**

**.claude/settings.json** (committed):
```json
{
  "env": {
    "JIRA_BASE_URL": "https://yourcompany.atlassian.net"
  }
}
```

**.claude/settings.local.json** (gitignored, personal):
```json
{
  "env": {
    "JIRA_EMAIL": "your-email@company.com",
    "JIRA_API_TOKEN": "your-personal-token",
    "GITHUB_TOKEN": "your-personal-github-token"
  }
}
```

---

## Quick Setup Checklist

**Minimum (UPDATE and COMMENT):**
- [ ] Get Jira API token
- [ ] Update `.claude/settings.json`
- [ ] Test: `/update-jira TEST-123 comment "test"`

**Full (including LINK-MR):**
- [ ] Complete minimum setup
- [ ] Choose: Install `gh`/`glab` CLI OR add tokens to settings
- [ ] Test: `/update-jira TEST-123 link-mr`

---

## Need Help?

- Check that `.claude/settings.json` has valid JSON syntax
- Validate with: `cat .claude/settings.json | jq .`
- Restart Claude Code after changing settings
- 99% of issues are invalid tokens or typos in settings.json

---

**You're ready to go! Start with:**
```bash
/update-jira YOUR-TICKET-ID
```
