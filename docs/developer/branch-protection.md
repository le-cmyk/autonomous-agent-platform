# Branch Protection Configuration

This repository is configured to prevent direct pushes to the main/master branches. This ensures that all changes go through pull requests and code reviews before being merged.

## Automated Setup (GitHub Actions)

The repository contains GitHub Actions workflows that set up branch protection rules automatically. For these workflows to work, you need to:

1. Create a GitHub Personal Access Token (PAT) with the `repo` scope
2. Add this token as a repository secret named `ADMIN_GITHUB_TOKEN` 

### Creating and Adding the Secret:

1. Go to your GitHub profile settings > Developer settings > Personal access tokens
2. Generate a new token with the `repo` scope
3. Navigate to your repository settings > Secrets and variables > Actions
4. Create a new repository secret named `ADMIN_GITHUB_TOKEN` with the token value

After adding the secret, manually trigger the workflows from the "Actions" tab in your repository.

## Manual Setup (GitHub Interface)

You can also set up branch protection rules manually:

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
2. Make your changes and commit them
3. Push the branch to GitHub
4. Create a pull request
5. Get the required approvals
6. Merge the pull request

For more information, see [GitHub's documentation on protected branches](https://docs.github.com/en/github/administering-a-repository/defining-the-mergeability-of-pull-requests/about-protected-branches).
