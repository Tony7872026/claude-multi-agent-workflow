---
name: release
description: Orchestrate a complete release workflow analyzing changes and bumping the version
---

Run the release workflow to prepare a new version: analyze changes, determine version bump, and update files.

## Workflow orchestration

This command runs two agents in sequence with the changelog analysis running first, feeding its output to the version bumper.

### Phase 1 — Analyze (Parallel)
Start with the changelog-analyzer agent to scan recent commits and categorize changes into Features, Fixes, and Breaking Changes. This agent works read-only, examining git history and code without making modifications.

*Agent: changelog-analyzer*

### Phase 2 — Bump (Sequential, depends on Phase 1)
Once the changelog summary is ready, run the version-bumper agent. It receives the changelog analysis, determines the appropriate semantic version bump (patch/minor/major), and updates version files.

*Agent: version-bumper* — requires output from Phase 1

## Result

After the workflow completes:
1. You have a detailed changelog of what changed
2. Version files are updated with the new semantic version
3. Review the changes in your working tree before committing
