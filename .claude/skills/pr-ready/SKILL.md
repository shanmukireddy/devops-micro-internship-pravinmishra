---
name: pr-ready
description: Analyzes staged diffs for security risks and generates PR drafts.
allowed-tools: [Bash, Read, Grep]
disable-model-invocation: true
---

# PR Ready Check

1. Run `git diff --cached` to gather all staged changes.
2. Analyze the diff for hardcoded credentials, debug logs, or missing documentation.
3. Output a Risk Assessment Report flagging any findings.
4. Draft a Pull Request Title and Description.
