# Branch Protection Configuration

This repository is configured to prevent direct pushes to the main/master branches. This ensures that all changes go through pull requests and code reviews before being merged.

## Manual Setup (GitHub Interface)

Branch protection is set up directly through the GitHub web interface:

1. Go to your repository on GitHub
2. Click on "Settings" > "Branches"
3. Under "Branch protection rules", click "Add rule"
4. In "Branch name pattern", enter `main` or `master` (depending on your default branch)
5. Enable the following options:
   - "Require a pull request before merging"
   - "Require approvals" (set to 1 or more as needed)
   - "Dismiss stale pull request approvals when new commits are pushed"
6. Click "Create" or "Save changes"

## Pushing Changes

With branch protection rules in place, you'll need to:

1. Create a new branch for your changes
   ```bash
   git checkout -b feature/my-feature-name
   ```
2. Make your changes and commit them
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```
3. Push the branch to GitHub
   ```bash
   git push origin feature/my-feature-name
   ```
4. Create a pull request using GitHub CLI
   ```bash
   gh pr create --title "Feature: My new feature" --body "Description of the changes"
   ```
5. Get the required approvals
6. Merge the pull request
7. Delete the feature branch after merging
   ```bash
   git checkout master
   git pull
   git branch -d feature/my-feature-name
   git push origin --delete feature/my-feature-name
   ```

## GitHub CLI Setup

To use GitHub CLI for PR creation:

1. Install GitHub CLI
   - Windows: `winget install --id GitHub.cli` or `choco install gh`
   - macOS: `brew install gh`
   - Linux: Follow instructions at https://github.com/cli/cli/blob/trunk/docs/install_linux.md

2. Authenticate with your GitHub account
   ```bash
   gh auth login
   ```

3. Follow the prompts to complete authentication

For more information, see [GitHub's documentation on protected branches](https://docs.github.com/en/github/administering-a-repository/defining-the-mergeability-of-pull-requests/about-protected-branches) and [GitHub CLI documentation](https://cli.github.com/manual/).
