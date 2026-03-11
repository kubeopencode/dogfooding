---
name: github-respond
description: Handle GitHub @mentions - answer questions, make code changes, create PRs, and respond to discussions
---

# GitHub @Mention Handler

Triggered when someone @mentions the bot in an Issue, PR, or Discussion comment.

## Immediate Acknowledgment

**IMPORTANT**: Since analyzing requests often takes a long time, you MUST
immediately acknowledge receipt of the request before starting any analysis.

When you receive a request, FIRST post a quick reply like:
- "Received, analyzing..."
- "Got it! Working on this now..."

Then proceed with your analysis and detailed response.

## Step 1: Gather Context

Always read the full context before responding:
- Read the issue/PR/discussion history
- Understand the conversation before replying

### For Issues
```bash
gh issue view ${NUMBER} --comments
```

### For PRs
```bash
gh pr checkout ${NUMBER}
gh pr view ${NUMBER} --comments
gh pr diff ${NUMBER}
```

### For PR Review Comments (Inline)
```bash
gh pr checkout ${NUMBER}
gh pr view ${NUMBER} --comments
gh api repos/${REPO}/pulls/${NUMBER}/comments \
  --jq '.[] | "[\(.user.login) on \(.path):\(.line)] \(.body)"'
cat ${FILE_PATH}
```
Pay attention to `${FILE_PATH}` and `${LINE}` from the event.

### For Discussions
Run this command to get the discussion details (just run it, don't modify):
```bash
gh api graphql -f query='
  query {
    repository(owner: "${OWNER}", name: "${REPO_NAME}") {
      discussion(number: ${NUMBER}) {
        title
        body
        comments(first: 50) {
          nodes {
            author { login }
            body
          }
        }
      }
    }
  }
'
```
**Note**: The output will be raw JSON. That's fine - just read and understand it directly.
Do NOT try to format it with jq.

## Step 2: Analyze User Intent

Determine what the user is asking:

| Intent | Description | Action |
|--------|-------------|--------|
| **Question** | Asking about code, features, or implementation | Analyze code and answer |
| **Guidance Request** | Seeking help on how to do something | Provide step-by-step guidance |
| **Code Change Request** | Asking to fix, update, add, or modify code | **Make the changes** (see below) |
| **Review Request** | Asking for code review or feedback | Provide review |
| **Feature Suggestion** | Proposing new functionality | Acknowledge and guide to open an Issue |
| **No Clear Request** | Just mentioned the bot | Ask how you can help |

**IMPORTANT**: Only proceed with code changes if the user EXPLICITLY requested them.
Look for clear action words like: "please fix...", "can you update...", "add X to...",
"remove...", "change...". If unsure, ask for clarification first.

**Privileged users only**: Code changes are only performed for privileged users
(OWNER, MEMBER, CONTRIBUTOR, COLLABORATOR).

## Step 3: Respond Appropriately

### For Questions
- Read and analyze relevant code
- Provide a clear, helpful answer
- Include code examples if helpful

### For Code Change Requests

#### On Issues (Create New PR)
1. Create branch:
   ```bash
   git checkout -b fix/issue-${NUMBER}-<short-description>
   ```
2. Make the requested changes
3. Commit with signoff:
   ```bash
   git add .
   git commit -s -m "fix: <description>

   Fixes #${NUMBER}"
   ```
4. Push and create PR:
   ```bash
   git push -u origin HEAD
   gh pr create --title "fix: <description>" --body "Fixes #${NUMBER}

   ## Changes
   - [describe changes]

   ---
   *PR by kubeopencode-agent*"
   ```
5. Enable auto-merge (optional):
   ```bash
   gh pr merge --auto --squash
   ```
6. Reply to the issue with the PR link

#### On PR Comments (Push to Existing PR)
1. Check if PR is from a fork:
   ```bash
   gh pr view ${NUMBER} --json headRepositoryOwner,baseRepository \
     --jq '{head: .headRepositoryOwner.login, base: .baseRepository.owner.login}'
   ```
2. If same-repo PR (head owner == base owner):
   ```bash
   gh pr checkout ${NUMBER}
   # Make changes
   git add .
   git commit -s -m "fix: <description>"
   git push
   ```
   Then reply confirming the changes.
3. If fork PR (different owners):
   Reply explaining you cannot directly push to fork PRs.

#### On Discussions (Create New PR)
Same as Issues workflow - create a new branch and PR.

#### On PR Review Comments (Inline)
Same as PR Comments workflow, but focus changes on the specific
`${FILE_PATH}` and `${LINE}` mentioned in the event.

### For Feature Suggestions
```
Thanks for your suggestion!

To track this properly:
1. Please open a GitHub Issue with details
2. Or submit a Pull Request if you'd like to implement it

I'm happy to answer questions about the implementation approach!
```

### Response Format

Always structure responses as:
```
> [Quote the user's comment briefly]

@${COMMENTER} [Your response]
```

**IMPORTANT**: Always post your response using the appropriate `gh` command.
Do not just print to stdout.
