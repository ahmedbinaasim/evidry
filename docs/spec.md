# Evidry first release: build spec

**Status:** accepted by the owner.

This brings together the decisions on the [planning map](https://github.com/ahmedbinaasim/evidry/issues/1). Words are used as defined in the [glossary](../GLOSSARY.md). Reasons for the less obvious choices are in [adr/](adr/). The [roadmap](roadmap.md) says what comes after, and the [build tasks](build-tasks.md) say in what order to build.

Nothing described here has been built or run. Statements about other tools come from their documentation. Points marked **[C1]**, **[C2]** and so on were filled in while writing this spec; they are listed [at the end](#choices-made-while-writing) and were accepted with it.

## 1. What it is

Evidry is an open-source (MIT) command-line tool that writes unit tests and QA cases from code and written requirements, and shows the evidence for each one. No test is presented as good because a model wrote it.

It succeeds when a developer or small team installs it and uses it on a real repository they care about.

Evidry never edits source code, never weakens an assertion, and never fixes bugs.

## 2. Supported

| | First release |
|---|---|
| Languages | TypeScript, JavaScript, Python, also mixed in one repository |
| Test runners | Vitest, Jest, pytest: current and previous major version; others get a warning and are still tried |
| Systems | Windows, macOS and Linux natively, without WSL for Evidry itself |
| Runtimes | Node 22 or newer; Python 3.11 or newer for Python projects |
| Code under `unit` | Functions and modules testable without a browser or device (no React or React Native components) |

## 3. Commands

| Command | Does |
|---|---|
| `evidry init` | Detects projects, languages, test runners and an available model key, then writes `evidry.yaml` for the user to edit. Nothing else. |
| `evidry qa <file>...` | Proposes rules and open questions from requirements and writes QA cases. |
| `evidry unit <path>...` | Generates and verifies unit tests for files or folders. A path is required; `.` means everything. `--feature <name>` limits the rules used. |
| `evidry diff <branch>` | The same as `unit`, only for functions the current branch changed. **[C1]** |
| `evidry report` | Re-runs linked tests and reports rule states. No model calls. |

Common to commands that call a model: `--dry-run` lists the files that would be sent and an estimated cost without calling a model. Later requests depend on model replies and cannot be shown exactly.

Terminal output is a short summary. Reasons go in files.

**Exit codes [C2]:** `0` nothing needs the user; `1` there are findings that need the user (suspected bugs, unexplained or unjudged failures, rule conflicts); `2` Evidry could not do its job.

## 4. Files in the user's repository

```text
evidry.yaml                 committed   all settings, no secrets
evidry/
  rules/<feature>.md        committed   edited by people
  cases/<feature>.md        committed   edited by people
  held/                     committed   failing generated tests
  state/<feature>.json      committed   Evidry's memory
  local/                    ignored     consent, request logs, latest results
  report.md                 ignored     rebuilt by `report`
  .gitignore                committed   ignores local/ and report.md  [C3]
```

A repository mixing languages has one `evidry.yaml`, one set of rules and one combined report.

Generated tests go in Evidry's own files beside the user's, in the project's framework and style (`cart.evidry.test.ts`, `test_pricing_evidry.py`). Evidry never adds to a file a person wrote.

Evidry only adds tests. It never edits or deletes an existing test, including its own. After a deliberate behaviour change the user deletes the outdated test and a later run writes a replacement.

Tests carry their rule ID, or the marker that they only record what the code does now:

```ts
// evidry: CHK-1
// evidry: records current behaviour
```

Python uses `# evidry:`. A person can add these markers by hand.

## 5. Rules

A rule is one testable statement of how the system must behave. It comes only from intent a person wrote down, never from the code ([ADR 0001](adr/0001-rules-are-never-inferred-from-code.md)). It names nothing in the code.

**Sources:** local Markdown or plain text requirement files, and rules typed by hand.

**Format:**

```markdown
# Checkout rules

## CHK-4: Discounts are capped at 100%
Status: approved
Source: docs/checkout.md, "Discounts"

A discount never reduces the total below zero, whatever percentage is entered.

## Open questions

- "Large orders ship faster": how large, how much faster?
```

- **Status** is `proposed`, `approved` or `retired`. A person approves by editing the line. Only an approved rule can be grounds for a suspected bug.
- **IDs** are a feature prefix and a sequence, never renumbered or reused. The model suggests the feature name and prefix; the user may change them before approving. Numbering already in a requirement is recorded in `Source`.
- **Rewording** an approved rule by hand keeps it approved and makes its tests and cases stale. A change into something unrelated retires the rule and issues a new ID.
- **Changed requirement documents** never overwrite an approved rule. A `> **Proposed change**` block is added under it; the old wording stays in force until the person replaces it or deletes the block.
- **Open questions:** a statement too vague to test, or one whose expected outcome is not stated, is listed as a question, not turned into a rule or case.
- **Checked by a person:** adding `Checked: name, date` under a rule marks it confirmed. The mark lapses if the rule wording, or the linked test or case, changes.
- **Conflicts:** both rules are flagged and neither can raise a suspected bug until one changes. Detection relies on the model and will miss some.
- **Retired rules:** their tests and cases are listed in the report and never deleted. They stop counting and cannot accuse the code.

### Rule states and flags

| State | Meaning |
|---|---|
| Confirmed | A linked test passes and fails when the rule is deliberately broken, or a person checked |
| Linked | A test or case claims the rule |
| Contradicted | An approved rule with a suspected bug against it |
| Not approved | Proposed |
| Retired | No longer applies |
| Untested | Approved with nothing linked **[C4]** |

Flags shown alongside: stale, code changed since, in conflict, manual cases only.

## 6. `qa`

One run proposes rules, raises open questions and writes cases. Cases for unapproved rules are labelled as depending on them.

A case has: an ID (`CASE-7`) **[C5]**, title, rule IDs, `Status`, preconditions, numbered steps with concrete data, expected result, and type (normal, boundary or invalid input). Output is Markdown only.

A case is kept only when it names a rule, has concrete data and an observable expected result, passes a second model check against its rule, and is not a duplicate (same rule, same kind of condition, same expected outcome, compared with cases already in Evidry's files). Dropped cases are listed with reasons.

Every case is proposed until a person approves it. Approved or hand-edited cases are never rewritten or deleted. When a rule's wording changes, its cases are marked stale and replacements are proposed beside them.

A case never confirms a rule on its own. Manual test runs and their results are not tracked.

## 7. `unit`

1. **Baseline.** Run the existing tests. Failing and flaky ones are left out of every comparison and listed. A project with no tests starts from nothing.
2. **Choose.** The model proposes which rules apply to each function; tests already carrying a rule ID are used first. Code matched to approved rules goes first, then the least covered. Show an estimated cost before starting.
3. **Generate.** Use approved QA cases as scenarios, adding others as needed. Proposed rules may guide generation; those tests are labelled as depending on unapproved rules.
4. **Repair** only what stops a test running: imports, setup, mocks, test data setup. Never change an expected value once the test has run.
5. **Repeat.** A test must pass 5 times (configurable), alone and with the existing tests. Mixed results mean flaky: dropped.
6. **Judge.** Keep a passing test only if it catches a planted bug that the baseline tests and already kept tests all miss, **or** confirms an approved rule nothing else has confirmed. Added coverage alone is not enough ([ADR 0003](adr/0003-coverage-alone-does-not-keep-a-test.md)). Coverage is still reported.
7. **Confirm rules.** The model writes a change that breaks that one rule, applied to a temporary copy. The rule is confirmed only if the linked test then fails on its expected outcome. The change is recorded and shown. If no suitable break can be made or the test still passes, the rule stays linked with the reason; a test kept only for that rule is dropped.
8. **Failures.** See below.
9. **Stop at the spending limit**, keep finished results and list what was not reached.

**Catching a bug** means a failure caused by changed behaviour. Timeouts, syntax or load errors and crashed runs do not count.

**Failing tests** are a **suspected bug** only when all hold: linked to an approved rule; fails the same way every run; fails on the expected outcome, not in setup or import; and a second model check, given the rule and the test, agrees the test states the rule correctly. Otherwise a repeatable failure is an **unexplained failure**, noting why. Neither is ever called confirmed by Evidry.

Both are written to `evidry/held/` with a `.held` ending and the reason at the top. The user accepts one by moving it into the test folder and removing the ending, or rejects it by deleting it. A rejected scenario (rule, function, inputs) is not written again.

**A kept test that fails later** is shown by `report` as "failing, not yet judged" and left where it is. The next `unit` or `diff` run applies the conditions above.

**Planted bugs:** StrykerJS for TypeScript and JavaScript. For Python, Evidry's own ([ADR 0002](adr/0002-evidry-plants-python-bugs-itself.md)): change a comparison, swap an arithmetic or `and`/`or` operator, negate a condition, change a constant, replace a return value, remove a statement; configurable cap per function.

## 8. `report`

Re-runs linked tests, makes no model calls, prints a summary and writes `evidry/report.md`: rule states and flags with their evidence, the change used to confirm each rule, suspected bugs and unexplained failures, tests failing and not yet judged, conflicts, stale items, retired rules still referenced, flaky or already failing tests, and what was dropped and why. Tests that record current behaviour are marked.

## 9. State and change detection

Committed state, one file per feature, holds only what cannot be worked out again from the files: what each approved rule, test and case looked like when last checked; rule confirmations including the breaking change; "checked" marks; rejected held tests; the highest ID used. Tests with no rule are recorded in `state/_unlinked.json` **[C6]**.

- Changes are detected by a fingerprint of the text, ignoring spacing and line endings. A typo fix counts.
- When code behind a confirmed rule changes, including a person-checked one, it stays confirmed and is flagged "code changed since" until the next `unit` or `diff` re-confirms it.
- Duplicate IDs or merge conflict markers stop Evidry and ask the user. It never renumbers or resolves by itself.

`evidry/local/` holds each person's consent to send data, the request logs of the last 20 runs (configurable) and the latest results. Results are saved as a versioned, machine-readable file. State files carry a format version too **[C7]**.

## 10. Settings

`evidry.yaml` holds every setting and no secrets. An unrecognised setting stops the run. Key names and defaults are **[C8]**:

| Setting | Default |
|---|---|
| `projects`: path, language, test runner | detected by `init` |
| `model` | written by `init`; no hidden default |
| `check_model` | same as `model` |
| `spend_limit_usd` (or `token_limit` with no known price) | 2 |
| `prices` | shipped table |
| `exclude` | common secret files |
| `isolation`: `container` or `none` | `container` |
| `network` | off |
| `env` (variables tests may see) | none beyond basics |
| `image` | built by Evidry |
| `test_timeout_seconds` | 60 |
| `memory_limit_mb` | 2048 |
| `repeat_runs` | 5 |
| `planted_bugs_per_function` | 20 |
| `keep_logs` | 20 |

## 11. Safe running

Applies to every test Evidry runs, including existing tests and those run by `report`.

- Tests, planted bugs and rule breaks run only in a temporary copy of the repository. Only finished test files and Evidry's own files are written back.
- Docker container by default ([ADR 0004](adr/0004-container-by-default-not-required.md)). Without it Evidry stops until `isolation: none` is set; it never falls back on its own. Under `none` each run says the protection is limited.
- No network by default. A container enforces this; without one it cannot be promised.
- Tests see a few basic environment variables plus those named, never the model key. `.env` and excluded files are not copied.
- Dependencies are installed from lock files into Evidry's image (network allowed then), or the user supplies an image. A project without a lock file gets a clear message **[C9]**. Without a container, what is installed is reused.
- Every test run has a time limit that kills everything it started, and a memory limit in a container.
- No databases or other services are started. The user may allow network access to their own test service.

## 12. Models, spending and data

- Anthropic, OpenAI, Google Gemini and any OpenAI-compatible endpoint, through a library behind Evidry's own interface (Vercel AI SDK is the candidate).
- Keys come only from environment variables and are never written to files or logs.
- Calls are used for: rules, open questions and conflicts; QA cases and their second check and duplicates; rule matching; test generation and repair; rule breaks; the second check on failures.
- Spending limit per run: each call is estimated first and not made if it would exceed the limit. The figure is an estimate, not the provider's bill.
- Only what the task needs is sent: target code and closely related files, its tests, relevant requirements, rules and cases, and test output. Excluded, Git-ignored and common secret files are never sent. Detected secrets are replaced with placeholders; detection is pattern-based and incomplete.
- Every run logs the exact requests locally.
- Before first sending anything from a repository, Evidry names the provider and asks. With nobody to ask it stops, unless already confirmed or given `--yes-send` **[C10]**.
- No telemetry. Prices are not downloaded.

## 13. Structure

Written in TypeScript. The core is a library; the command line is a layer over it, so later surfaces can use the same core. Nothing assumes every runnable test is a unit test.

- **Core** owns rules, keep-or-drop, suspected bugs, model calls, spending, state, reports and safe running.
- **Language plugins** (TypeScript/JavaScript; Python) detect projects, find testable and changed functions, gather context, write test files, install dependencies and answer "which planted bugs does this test catch?" in one format based on the StrykerJS report.
- **Test runner plugins** (Vitest, Jest, pytest) run tests, classify outcomes (pass, failed assertion, setup or load error, timeout) and read coverage.

Plugins report facts; the core decides. All are built in and the boundary is internal. Every plugin passes one shared set of checks against a sample project, and a written guide explains how to add one.

## 14. Distribution and website

- npm package `evidry`; Python users install Node. `npm install -g evidry`, with `npx` as an alternative.
- Evidry brings pinned versions of StrykerJS and the coverage tools, installed into its image or its own cache, without changing the project's manifests.
- Bundled dependencies are MIT, Apache-2.0, BSD or ISC, checked automatically. Anything else needs an explicit decision; no GPL or AGPL code.
- 0.x until file formats settle; published automatically from GitHub. No empty placeholder packages; an early pre-release marked not ready is published once something works.
- Website, part of the first release and live with it: `website/` in this repository, plain Astro with our own design, GitHub Pages at `ahmedbinaasim.github.io/evidry`. A front page and documentation (getting started, commands, settings, rules and cases, safety and data, languages, limits, contributing, roadmap), copyable install commands, no analytics or cookies, latest version only. Beautifully built and minimal; the design is chosen from mocks during the build. Search may be left out of a first version.

## 15. Ready for release

All of the following.

1. **Samples:** three small projects (TypeScript/Vitest, JavaScript/Jest, Python/pytest) with written requirements and expected results, covering obeyed and broken rules, a vague statement, conflicting rules, and weak, flaky and already failing tests. The same projects serve the shared plugin checks, and together they cover the mixed-repository case.
2. **Repeatable checks** with recorded model replies, all passing: keep-or-drop, suspected bugs, QA filtering, changed rules, changed code, team merges, spending limits, secret redaction. Run automatically on all three systems without a container and on Linux with one; containers on Windows and macOS checked by hand.
3. **Live runs**, three, with one named model: no suspected bug against an obeyed rule; every broken rule reported in at least two runs; the vague statement raised every time; each run within US$2. Other provider kinds only need to complete a run. The numbers are unmeasured guesses; changing them is a visible decision.
4. **Evidry on itself:** its requirements are written as rules, the flow runs on this repository, every approved rule is confirmed or says why not, and every suspected bug and held test has been judged.
5. The website is live and the install command works.

## 16. Not in this release

`api` mode, GitHub Action, MCP server, editor extension, HTML review page, public benchmark, other QA formats, other languages and runners, React components, other container tools, separately published plugins, starting services, hosted service or accounts, browser end-to-end, load and security testing, automatic bug fixing.

## 17. Unverified assumptions

Check these early; each has a task.

- StrykerJS and coverage tools can be attached to a project without changing its manifests.
- StrykerJS's Vitest runner is reliable enough. Its tracker has open reports: killed bugs scored as survived, wrong "no coverage" on Windows, not exiting on Windows, and a crash with Vitest 5 debug logging.
- Python coverage per test works. The research suggests forcing coverage.py's `ctrace` core on Python 3.14 and later.
- Evidry's own Python bug planting is accurate and fast enough.
- The model library supports all four provider kinds and reports token use well enough to enforce limits.
- A usable image can be built from real lock files, including mixed repositories.
- Automated Windows and macOS machines often cannot run Linux containers.
- npm's rule on placeholder packages (not confirmed).

## Choices made while writing

None of these was discussed in a ticket. All were accepted with this spec.

| | Choice |
|---|---|
| C1 | `diff <branch>` compares with the point where the current branch left `<branch>`, including uncommitted changes |
| C2 | Exit codes 0, 1 and 2 as in section 3 |
| C3 | Evidry ignores its local files with `evidry/.gitignore`, leaving the repository's own `.gitignore` alone |
| C4 | An approved rule with nothing linked is shown as "untested" |
| C5 | Case IDs are `CASE-n`, unique across the repository |
| C6 | Tests with no rule are recorded in `state/_unlinked.json` |
| C7 | State and result files carry a format version |
| C8 | Setting names and defaults as in section 10; only the spending limit, repeat runs and logs kept were agreed |
| C9 | No lock file: stop with a message rather than guess dependency versions |
| C10 | The consent flag is `--yes-send` |
