# Regex editing regression evidence

Issue: https://github.com/PostHog/posthog/issues/44054

Tested source: `85cbd64dcdd6d7e5e4a35b1258539112f90f2ec5`.

The running local Storybook renders the actual `OperatorValueSelect` with an invented `example.com` regex. Opening a stored regex at 650px pushes the edit button offscreen before the fix. The popover's fixed 800px maximum overrides its inline viewport limit.

The retained browser regression checks the edit button's bounds and clicks it. Before the fix, the check fails: expected right edge <=642px, received 802px. With the fix, the right edge is 636px.

The before and after images use the same story, 650x850 viewport, and device scale factor 2. They contain synthetic data only.

| Viewport | Theme | Edit button right edge after fix |
| --- | --- | --- |
| 520px | Light | 506px |
| 650px | Light | 636px |
| 1200px | Light | 810px; existing 800px dropdown maximum retained |
| 650px | Dark | 636px |

Argent browser automation clicked the visible edit button, entered an optional trailing slash in the regex, and verified the edited filter value persisted.

Checks completed:

- `pnpm --filter @posthog/storybook exec test-storybook --url http://localhost:6006 --maxWorkers=1 --index-json -- --testPathPatterns=filters-propertyfilters-operatorvalueselect` exited 0. All nine stories in the file passed, including the new play test. The runner generated local light/dark snapshots; these were new baselines, not image comparisons, and were not committed.
- `corepack pnpm --filter=@posthog/frontend typescript:check` reported no errors.
- Targeted `oxlint` and `oxfmt --check` passed for both changed files.
- Committed `hogli ci:preflight --strict --against origin/master` reported two changed files and zero failures. It reported existing component complexity and a branch freshness advisory.
- Normal commit and push hooks passed.

The full backend application and database-backed suites were not run. This reproduction exercises the affected frontend component in Storybook. No database, Docker, or new production dependency was needed.
