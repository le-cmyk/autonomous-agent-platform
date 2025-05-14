# Branch Protection Configuration

This repository is configured to prevent direct pushes to the main/master branches. This ensures that all changes go through pull requests and code reviews before being merged.

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
