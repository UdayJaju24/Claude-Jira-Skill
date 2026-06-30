# Installation Guide

This skill is designed to be installed **globally** so it works across all your projects.

## Quick Install (5 Minutes)

### Step 1: Clone the Repository

```bash
git clone https://gitlab.cee.redhat.com/ujaju/Jira-Claude-Skill.git
cd Jira-Claude-Skill
```

### Step 2: Install Skill Globally

```bash
# Create global skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill to global location
cp -r .claude/skills/update-jira ~/.claude/skills/update-jira
```

✅ **The skill is now installed and available in ALL your projects!**

### Step 3: Configure Global Settings

```bash
# Copy settings template to global location
cp .claude/settings.json ~/.claude/settings.json
```

If `~/.claude/settings.json` already exists, **merge** the env variables instead:

```bash
# Open your existing global settings
code ~/.claude/settings.json
# OR
nano ~/.claude/settings.json
```

Add these env variables (merge with existing content):

```json
{
  "env": {
    "JIRA_BASE_URL": "https://redhat.atlassian.net",
    "JIRA_EMAIL": "your-email@redhat.com",
    "JIRA_API_TOKEN": "your-api-token-here",
    "GITHUB_TOKEN": "",
    "GITLAB_TOKEN": ""
  }
}
```

### Step 4: Add Your Personal Tokens

**Replace the placeholder values in `~/.claude/settings.json`:**

```json
{
  "env": {
    "JIRA_BASE_URL": "https://redhat.atlassian.net",
    "JIRA_EMAIL": "ujaju@redhat.com",              ← Your Red Hat email
    "JIRA_API_TOKEN": "ATATTxxxxxxxxxxxxxxxx",     ← Your Jira token
    "GITHUB_TOKEN": "",                             ← Optional (leave empty if not using GitHub)
    "GITLAB_TOKEN": "glpat_xxxxxxxxxxxxx"          ← Your GitLab token
  }
}
```

**See [README.md](README.md#setup)** for detailed instructions on getting these tokens.

### Step 5: Test It Works

```bash
cd ~/any-project-directory
claude

# In Claude Code:
/update-jira TEST-123 comment "Testing installation"
```

---

## Why Global Installation?

**Global (`~/.claude/skills/`):**
- ✅ Works in ALL your projects
- ✅ Update once, available everywhere
- ✅ Perfect for team-shared skills like this Jira integration

**Project-specific (`.claude/skills/` in repo):**
- ❌ Only works in that specific project
- ❌ Need to copy to each project manually
- ✅ Good for project-specific customizations

---

## File Locations Reference

After installation, here's where everything lives:

```
~/.claude/
├── skills/
│   └── update-jira/          # ← The skill (copied from repo)
│       └── SKILL.md
└── settings.json             # ← Your settings with personal tokens

/path/to/Jira-Claude-Skill/   # ← The cloned repo
├── .claude/
│   ├── skills/
│   │   └── update-jira/      # ← Source (keep for updates)
│   │       └── SKILL.md
│   └── settings.json         # ← Template (with placeholders)
├── README.md
└── INSTALLATION.md
```

---

## Updating the Skill

When updates are released:

```bash
# Pull latest changes
cd /path/to/Jira-Claude-Skill
git pull

# Copy updated skill to global location
cp -r .claude/skills/update-jira ~/.claude/skills/update-jira
```

Your tokens in `~/.claude/settings.json` remain safe (not overwritten)!

---

## Team Setup

**The skill is already configured for Red Hat Jira:**

The template includes `JIRA_BASE_URL: "https://redhat.atlassian.net"` so all Red Hat users can use it as-is.

**For projects with a different Jira instance:**

Create project-specific settings in `<project>/.claude/settings.json`:

```json
{
  "env": {
    "JIRA_BASE_URL": "https://different-company.atlassian.net"
  }
}
```

Project settings override global settings for that project only.

---

## Security Notes

🔒 **Your `~/.claude/settings.json` is PERSONAL - not in any git repo!**

- ✅ `~/.claude/settings.json` - Your global settings (not tracked by git)
- ✅ Safe to add tokens here - this file is on your machine only
- ❌ Never commit `.claude/settings.json` from this repo with real tokens

**The template in this repo:**
- `.claude/settings.json` in repo - Contains placeholders only
- Safe to commit because it has no real tokens
- Each user copies it and fills in their own values

---

## Troubleshooting

### "Skill not found"

Make sure you copied to the right location:

```bash
ls -la ~/.claude/skills/update-jira/SKILL.md
```

Should show the file. If not, repeat Step 2.

### "Authentication failed"

Check your tokens in `~/.claude/settings.json`:

```bash
cat ~/.claude/settings.json
```

Verify:
- `JIRA_EMAIL` matches your Atlassian account email exactly
- `JIRA_API_TOKEN` was copied completely (starts with `ATATT`)
- No extra spaces, quotes, or newlines
- Values are inside double quotes

### "Skill works in one project but not others"

You might have project-specific settings overriding global ones. Check:

```bash
# In the project directory
cat .claude/settings.json
cat .claude/settings.local.json
```

---

## Uninstallation

To remove the skill:

```bash
rm -rf ~/.claude/skills/update-jira
```

Your tokens in `~/.claude/settings.local.json` remain for other skills.

---

## Need Help?

1. Check [README.md](README.md) for token setup guides
2. See [Troubleshooting section](#troubleshooting) above
3. Open an issue on GitLab

---

**You're all set!** The skill is now available in every project.

Test it: `/update-jira YOUR-TICKET-ID`
