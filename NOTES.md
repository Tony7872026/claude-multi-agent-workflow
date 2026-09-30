# Release Plugin — Notes

## What the plugin does

The release plugin automates the release management workflow. It orchestrates two specialized agents to:

1. **Analyze changes** — Scan git history and code to extract what's new, fixed, or breaking since the last release
2. **Bump version** — Determine the appropriate semantic version bump (patch/minor/major) and update version files

The plugin also includes pre-release validation checks and a skill for formatting release notes into publication-ready markdown.

### Installation

Add the plugin marketplace and install:
```bash
/plugin marketplace add <your-repo-url>
/plugin install release@<marketplace-name>
```

Then trigger releases with:
```bash
/release
```

---

## Scoping Decision: Changelog Analyzer Uses Haiku

**Why Haiku for changelog-analyzer:**

The changelog analyzer is a straightforward data-extraction task: read git logs, examine diffs, categorize commits. It doesn't require complex reasoning or multi-step decision-making — it's pattern matching and summarization.

Haiku is fast, cheap, and sufficient for this read-only, exploratory work. Larger models would waste capacity on a task that doesn't benefit from reasoning depth. The agent's tools are intentionally limited (Read, Bash only — no edit/write), so it can only gather information, not modify files.

If the changelog contained ambiguous or conflicting changes requiring judgment calls, a larger model would make sense. But that's exactly why the version-bumper (which *does* make judgment calls about versioning) uses Sonnet.

---

## Orchestration Decision: Sequential Steps (Analyze → Bump)

**Why changelog-analyzer and version-bumper run sequentially, not in parallel:**

The version-bumper fundamentally depends on the changelog-analyzer's output. It receives the categorized changes (Features/Fixes/Breaking) and uses that summary to decide whether to bump patch, minor, or major version.

Running them in parallel would be wasteful:
- The version-bumper would have no input and would have to redundantly re-examine git history
- It would require passing results between agents anyway, making parallelism illusory

The workflow *could* have added a parallel step if, for example, it also fetched external release notes, documentation updates, or changelog templates in parallel with change analysis. But with the current scope, sequential execution is correct.

The dependency is explicit and unavoidable: **analyze first, then decide and update based on what you learned.**

---

## Testing & Validation

- Pre-release hook validates that the working tree is clean and you're on main before running the workflow
- Release notes skill can be called separately to format and validate changelog output before publishing
- Both agents are tested against the course API in `course-api/` to ensure they work on real code
