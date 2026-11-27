# Build tasks for the first release

**Status:** accepted by the owner. Ordered by dependency; no estimates. Section numbers refer to the [spec](spec.md).

A task can start when every task in **Needs** is done. Tasks within a stage that do not need each other can run in parallel.

## Stage 0: prepare

| ID | Task | Done when | Needs |
|---|---|---|---|
| T01 | Merge the planning documents into `main` | Spec, glossary and decision records are on `main`; ticket links to them updated | |
| T02 | Set up the TypeScript project: build, Vitest, lint, CI on Windows, macOS and Linux, licence check | An empty command runs and CI passes on all three | T01 |

## Stage 1: prove the risky parts first (section 17)

Throwaway experiments. Each ends with a written result; a failed one reopens the decision it rests on.

| ID | Task | Done when | Needs |
|---|---|---|---|
| T03 | Attach StrykerJS and coverage to a Vitest and a Jest project without changing their manifests, on all three systems | Per-test results obtained, or the limits written down | T02 |
| T04 | Plant the six kinds of Python bug and run one test against each, on all three systems | Accurate results and measured speed | T02 |
| T05 | Check the model library with all four provider kinds | Calls, token counts and errors work, or another library is chosen | T02 |
| T06 | Build a container image from npm and Python lock files and run tests in it with no network | Works for a mixed project; Windows and macOS tried by hand | T02 |
| T07 | Kill a runaway test and everything it started on all three systems | No process left behind | T02 |

## Stage 2: samples and foundations

| ID | Task | Done when | Needs |
|---|---|---|---|
| T08 | Write the three sample projects with requirements and expected results (15.1) | Their own tests run; expected results listed | T02 |
| T09 | Settings: `evidry.yaml` loading and refusal of unknown settings (10) | Tested | T02 |
| T10 | Rules and cases files: read, write without losing hand edits, IDs, statuses, markers (4, 5) | Round trips tested | T02 |
| T11 | State and local files: fingerprints, versions, conflict and duplicate detection (9) | Tested, including a merged conflict | T10 |
| T12 | Safe runner: temporary copy, container and `none`, environment, network, limits (11) | Protections tested | T06, T07, T09 |
| T13 | Model access: providers, consent, spending limit, exclusions, redaction, logs, dry run (12) | Tested with recorded replies | T05, T09 |
| T14 | Record-and-replay of model replies for tests | A run repeats with no model call | T13 |
| T15 | Plugin boundary and shared plugin checks (13) | Checks run against a stub plugin | T08 |

## Stage 3: plugins

| ID | Task | Done when | Needs |
|---|---|---|---|
| T16 | Python language plugin, including bug planting | Shared checks pass | T04, T12, T15 |
| T17 | pytest runner plugin | Shared checks pass | T12, T15 |
| T18 | TypeScript/JavaScript language plugin with StrykerJS | Shared checks pass | T03, T12, T15 |
| T19 | Vitest runner plugin | Shared checks pass | T03, T12, T15 |
| T20 | Jest runner plugin | Shared checks pass | T03, T12, T15 |

## Stage 4: commands

One language path first (T22 to T27 can be built against Vitest, which Evidry itself needs), then the others.

| ID | Task | Done when | Needs |
|---|---|---|---|
| T21 | `init` (3) | Writes settings for each sample | T09, T16, T17, T18, T19, T20 |
| T22 | `qa`: rules, open questions, conflicts, proposed changes (5) | Expected results on samples with recorded replies | T10, T11, T14 |
| T23 | `qa`: cases, second check, duplicates, stale replacements (6) | Same | T22 |
| T24 | `unit`: baseline, rule matching, generation, repair, repeats, planted-bug judging (7 steps 1 to 6) | Keep-or-drop checks pass | T11, T14, T18, T19 |
| T25 | `unit`: rule breaks and confirmation (7 step 7) | Obeyed rules confirmed on the TypeScript sample | T22, T24 |
| T26 | `unit`: failures, second check, held tests, rejections, later failures (7) | Broken rules reported, obeyed rules never accused | T25 |
| T27 | `report` and the results file, exit codes (3, 8, 9) | Every state and flag shown on the samples | T26, T23 |
| T28 | `diff` (3) | Only changed functions are worked on | T24 |
| T29 | `unit` and `diff` working with Jest and with Python, including a mixed repository | All three samples pass | T16, T17, T20, T27, T28 |

## Stage 5: release

| ID | Task | Done when | Needs |
|---|---|---|---|
| T30 | Complete the repeatable release checks (15.2) | All pass on the platforms listed | T29, T21 |
| T31 | Choose the release model and run the live checks (15.3) | Pass marks met, or a visible decision to change them | T30 |
| T32 | Write Evidry's own requirements as rules and approve them | Approved rule files committed (the owner approves) | T22 |
| T33 | Run Evidry on itself and judge every finding (15.4) | Every approved rule confirmed or explained | T27, T32 |
| T34 | Owner: create the npm account | Publishing access exists | |
| T35 | Automated publishing from GitHub | A pre-release marked not ready is published | T34, T27 |
| T36 | Design mocks for the website; owner picks one | A design is chosen | |
| T37 | Build the website and write the documentation (14) | Site builds with all listed pages | T36, T29 |
| T38 | Write the plugin guide for contributors | A reader can follow it | T29 |
| T39 | Release: publish package and site together | `npm install -g evidry` works from the site's command | T31, T33, T35, T37, T38 |

T34 and T36 depend on nothing and can be done at any time. A first pre-release (T35) is possible as soon as the TypeScript path reports (T27), which also secures the package name earliest.
