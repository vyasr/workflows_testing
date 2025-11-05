# workflows_testing

This repository is for testing and demonstrating various GitHub Actions workflows.

## Automated Issue and PR Release Management

This repository includes GitHub Action workflows that automatically manage the Release field for issues and pull requests based on Priority values and VERSION file contents.

### Workflows

#### 1. Issue Release Automation (`.github/workflows/issue-release-automation.yml`)

Automatically sets the Release field on issues based on their Priority field:

- **P0 or P1**: Release field set to current release (from VERSION file, e.g., `25.12`)
- **P2 or higher**: Release field set to `Backlog`

**Triggers:**
- When an issue is opened or edited

**Note:** Since GitHub Projects V2 don't emit repository-level events when fields are changed directly in the project view, this workflow only runs when the issue itself is edited. For Priority changes made directly in the project, see the scheduled workflow below.

#### 2. Issue Release Scheduled Sync (`.github/workflows/issue-release-scheduled.yml`)

Periodically checks all open issues and syncs their Release field based on Priority. This catches Priority changes made directly in the GitHub Project view.

- Runs every hour on the hour
- Only updates issues where Release field doesn't match expected value based on Priority
- Adds comments only when updates are made
- Can be manually triggered via GitHub Actions UI for immediate sync

**Triggers:**
- Schedule: Every hour (`0 * * * *`)
- Manual: `workflow_dispatch` (can be triggered manually from Actions tab)

#### 3. PR Release Automation (`.github/workflows/pr-release-automation.yml`)

Automatically sets the Release field on pull requests based on the VERSION file in the PR's base branch.

- Release field set to the first 2 segments of the VERSION file (e.g., `25.12.00` → `25.12`)

**Triggers:**
- When a PR is opened, edited, synchronized (new commits), or reopened

### Setup Instructions

#### Step 1: Create a Personal Access Token (PAT)

1. Go to GitHub Settings → Developer settings → [Personal access tokens → Tokens (classic)](https://github.com/settings/tokens)
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "Project Automation Workflows")
4. Select the following scopes:
   - `repo` (Full control of private repositories)
   - `project` (Full control of projects)
5. Click "Generate token"
6. **Copy the token value** (you won't be able to see it again!)

#### Step 2: Add PAT as Repository Secret

1. Go to this repository's Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `PROJECT_TOKEN`
4. Value: Paste the PAT you created in Step 1
5. Click "Add secret"

#### Step 3: Configure GitHub Project

Your GitHub Project must have the following fields:

1. **Priority** (Single select field)
   - Options: `P0`, `P1`, `P2`, `P3`, etc.
   - Used to determine release assignment for issues

2. **Release** (Text field or Single select field)
   - Will be automatically populated by workflows
   - Values: Version numbers (e.g., `25.12`) or `Backlog`

#### Step 4: Add Issues/PRs to Project

For the automation to work:
- Issues must be added to a GitHub Project with Priority and Release fields
- PRs must be added to a GitHub Project with a Release field
- The workflows will automatically update the Release field based on the rules above

### How It Works

#### Version Parsing

The VERSION file contains a 3-segment version (e.g., `25.12.00`), but the Release field only uses 2 segments (e.g., `25.12`). The workflows automatically parse and truncate the version.

#### Issue Priority Logic

```
Priority → Release Mapping:
├─ P0 → Current version (e.g., 25.12)
├─ P1 → Current version (e.g., 25.12)
└─ P2+ → Backlog
```

#### PR Release Logic

All PRs get their Release field set based on the VERSION file in their **base branch** (not the PR branch). This ensures PRs are tagged with the release they're targeting.

### Workflow Notifications

Both workflows provide feedback via comments on issues/PRs:

- ✅ Success: Confirms the Release field was updated
- ⚠️ Warning: Missing fields or project not configured
- ❌ Error: Provides error details and troubleshooting tips

### Troubleshooting

**Issue: "Could not find Release field in project"**
- Ensure your GitHub Project has a field named exactly "Release"
- Check that issues/PRs are added to the project

**Issue: "Failed to update Release field" with permission error**
- Verify the `PROJECT_TOKEN` secret is set correctly
- Ensure the PAT has `repo` and `project` scopes
- Check that the PAT hasn't expired

**Issue: Workflow doesn't run**
- Check that issues/PRs are in a GitHub Project
- Verify the workflow files are in `.github/workflows/` directory
- Check the Actions tab for any workflow errors

### Development

The VERSION file in this repository controls the release version used by the workflows. To change the current release, update the VERSION file and commit the change.
