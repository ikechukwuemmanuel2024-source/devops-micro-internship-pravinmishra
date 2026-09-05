---
name: pr-ready
description: Reviews staged Git changes and drafts a PR title, description, and risk report. Never commits, pushes, or opens PRs.
allowed-tools: Bash, Read, Grep
disable-model-invocation: true
---

# PR Ready

Review the currently staged Git changes and prepare a pull-request readiness report.

## Review steps

1. Inspect the staged file list.
2. Inspect the staged diff.
3. Look for:
   - hardcoded secrets or credentials
   - private keys
   - debug statements
   - accidental files
   - oversized files
   - suspicious or unsafe changes
4. Summarize the risks found.
5. Draft:
   - a PR title
   - a PR description
   - a concise risk report

## Safety rules

- Never commit changes.
- Never push changes.
- Never open or create a pull request.
- Never modify files.
- Never reveal or reproduce real credentials or secrets.
- Report suspicious secret-like values without exposing the complete value.

## Output format

### PR Title
Draft a concise PR title.

### PR Description
Summarize the changes and relevant testing or validation.

### Risk Report
List each risk found and its severity.

If no significant risks are found, state that the staged changes appear ready for review.