---
name: opencode-update
description: Check for new OpenCode releases and update the Dockerfile version if a newer version is available
---

# OpenCode Version Update

Check if there is a new release of OpenCode and update the Dockerfile
if a newer version is available.

## Instructions

1. Navigate to the cloned `kubeopencode` directory

2. Check the current OpenCode version in `agents/opencode/Dockerfile`:
   - Look for the line `ARG OPENCODE_VERSION=x.y.z`

3. Check the latest OpenCode release on GitHub:
   ```bash
   curl -s https://api.github.com/repos/anomalyco/opencode/releases/latest | jq -r '.tag_name'
   ```

4. Compare versions:
   - If the GitHub version (without 'v' prefix) is newer than the Dockerfile version, proceed
   - If versions are the same, exit successfully with message "OpenCode is already up to date"

5. If update needed:
   - Update the `ARG OPENCODE_VERSION=x.y.z` line in `agents/opencode/Dockerfile`
   - Create a new branch: `chore/bump-opencode-<version>`
   - Commit with message: `chore(agents): bump opencode version to <version>`
   - Use signoff flag: `git commit -s`
   - Push the branch and create a PR

## Important

- Only update if there is actually a newer version
- Do not make any other changes to the file
- Use the exact version number from the GitHub release (without 'v' prefix)
