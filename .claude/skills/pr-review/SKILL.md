---
name: pr-review
description: Perform automated code review on pull requests - check for bugs, security issues, style violations, and submit inline comments
---

# PR Review Handler

Triggered when a new Pull Request needs code review.

## Workflow

1. **Navigate to the cloned `kubeopencode` directory**

2. **Checkout the PR:**
   ```bash
   gh pr checkout ${PR_NUMBER}
   ```

3. **Get changed files:**
   ```bash
   gh pr diff ${PR_NUMBER} --name-only
   gh pr diff ${PR_NUMBER}
   ```

4. **Review each file** for:
   - Code correctness and potential bugs
   - Error handling and edge cases
   - Security vulnerabilities
   - Style violations and consistency
   - Performance concerns
   - Missing error handling
   - Test coverage

5. **Submit review** using the PR Review method with inline comments:
   ```bash
   gh api repos/${REPO}/pulls/${PR_NUMBER}/reviews \
     -f event="COMMENT" \
     -f body="## Review Summary

   [Your summary here]

   ---
   *Review by kubeopencode-agent*" \
     -f 'comments[0][path]=path/to/file.go' \
     -f 'comments[0][line]=42' \
     -f 'comments[0][body]=Your inline comment'
   ```

   If no issues found, submit a summary-only review.

6. **Add the `ai-reviewed` label** after submitting the review:
   ```bash
   gh pr edit ${PR_NUMBER} --add-label "ai-reviewed"
   ```

## Important

- Be constructive and specific in feedback
- Suggest code improvements with examples
- Do NOT approve or request changes, use COMMENT event only
- Always add the `ai-reviewed` label after review
- Write review comments in English
