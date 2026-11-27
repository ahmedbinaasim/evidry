# Which coverage and mutation tools can Evidry build on?

Research for [issue #3](https://github.com/ahmedbinaasim/evidry/issues/3). **Date checked: 2026-11-27.**

## How to read this

- Sources are official docs, project repositories (source, CI workflows, issues), package registries and licence files.
- **Nothing was installed or run.** Every platform, behaviour and speed statement comes from reading sources.
- Platform evidence is graded:
  - **CI**: the project's own CI configuration has a job on that OS. This shows the OS is exercised, not that every feature works there.
  - **Doc**: the project says so, with no CI job found.
  - **Unverified**: neither was found.
- "Per-test" means the output says which test covered a line, or covered or killed a mutant.
- Statements marked *(inference)* are my reasoning, not something a source says.

## Summary

**Coverage**

| Language | Tool (latest) | Licence | Machine output | Per-test | One file | Windows / macOS / Linux |
|---|---|---|---|---|---|---|
| Python | coverage.py 7.17.5 + pytest-cov 7.1.0 | Apache-2.0 / MIT | JSON | **Yes** | Yes | CI / CI / CI |
| JS/TS | Vitest 5.1.2 or Jest 30.5.2 built-in; c8 12.0.0 | MIT / MIT / ISC | Istanbul JSON and others | None documented | Yes | Vitest, Jest: CI on all three. c8: CI / **Unverified** / CI |
| Java | JaCoCo 0.8.16 | EPL-2.0 | XML, CSV | No built-in | Filter by class | CI / **Unverified** / CI |
| C# | coverlet 10.1.1; Microsoft code coverage 18.12.0 | MIT; closed source | Cobertura and others | None documented | Filters | coverlet: CI on all three. Microsoft: Doc on all three |
| Go | `go test -cover` (Go 1.27.4) | BSD-3-Clause | text profile | No built-in | `-coverpkg` (package) | Doc (Go ships for all three) |

**Mutation**

| Language | Tool (latest) | Licence | Per-mutant output | Per-test | Limit to file / diff | Windows / macOS / Linux |
|---|---|---|---|---|---|---|
| Python | mutmut 3.8.1 | BSD-3-Clause | Internal files, text | Covering tests only | File: yes. Diff: no | **No (WSL only)** / Doc / CI |
| Python | cosmic-ray 8.9.0 | MIT | JSON lines | No | File: yes. Diff: filter | Doc (code) / **Unverified** / CI |
| JS/TS | StrykerJS 10.1.1 | Apache-2.0 | JSON | **Yes** | File and line range. Diff: caller computes ranges | CI / Doc (user reports) / CI |
| Java | PIT 1.30.2 | Apache-2.0 | XML, CSV | **Yes** (full matrix) | Class. Diff: commercial plugin | CI / CI / CI |
| C# | Stryker.NET 5.1.0 | Apache-2.0 | JSON | **Yes** | File and span. Diff: `--since`, whole files | CI / CI / CI |
| Go | Gremlins 0.6.0 | Apache-2.0 | JSON | No | Diff: yes, with open bugs. No include-one-file flag | CI, **but open Windows bugs** / CI / CI |

## Recommendation

**Least risky first language: JavaScript/TypeScript with StrykerJS** *(my judgement from the evidence below)*.

- It is the only combination here with all of: line-range scoping, a documented JSON schema with `coveredBy` / `killedBy` per mutant, a programmatic API, a `--testFiles` option to restrict which tests run, and Windows in CI.
- Its main risk is runner-specific bugs, especially in the Vitest runner. Support one runner first and pin versions. Jest is the more cautious choice *(inference: fewer relevant open reports seen; I did not do a full bug comparison)*.

**Second choice: Java with PIT.** The most clearly tested on all three operating systems, but scoping is per class, diff scoping is commercial, and Evidry would have to drive Maven or Gradle.

**Python** has the best coverage data but the weakest mutation story on native Windows. **Go** is the riskiest: no mature tool with trustworthy Windows behaviour was found.

If Evidry plants its own mutants and runs only the new test, the choice of mutation tool matters much less, and Python becomes attractive again *(inference; this is a design option, not researched here)*.

## Python

### Coverage: coverage.py and pytest-cov

- Active: coverage.py 7.17.5 released 2026-11-23 ([PyPI](https://pypi.org/project/coverage/)); requires Python 3.11+. pytest-cov 7.1.0 ([PyPI](https://pypi.org/project/pytest-cov/)). The coverage.py repository is now [coveragepy/coveragepy](https://github.com/coveragepy/coveragepy).
- Licences: Apache-2.0 ([LICENSE](https://github.com/coveragepy/coveragepy/blob/main/LICENSE.txt)); pytest-cov MIT ([repo](https://github.com/pytest-dev/pytest-cov)).
- Machine output: `coverage json`, with `--include` to limit files and `--show-contexts` to list contexts per covered line ([docs](https://coverage.readthedocs.io/en/latest/commands/cmd_json.html)).
- Per-test: `pytest --cov-context=test` records a context per test and phase, e.g. `test_x.py::test_addition|run` ([pytest-cov docs](https://pytest-cov.readthedocs.io/en/latest/contexts.html)).
- **Trap:** the `sysmon` core, default on Python 3.14+, does not support dynamic contexts ([config docs](https://coverage.readthedocs.io/en/latest/config.html), [FAQ](https://coverage.readthedocs.io/en/latest/faq.html)). The [changelog](https://github.com/coveragepy/coveragepy/blob/main/CHANGES.rst) says that contexts switched through the API, as pytest-cov does, were once dropped silently and now raise a warning. Evidry should force `core = ctrace` *(inference from the above)*. `ctrace` is slower than `sysmon`.
- Platforms: Windows, macOS and Linux in CI for both ([coverage.py workflow](https://github.com/coveragepy/coveragepy/blob/main/.github/workflows/testsuite.yml), [pytest-cov workflow](https://github.com/pytest-dev/pytest-cov/blob/master/.github/workflows/test.yml)).

### Mutation: mutmut

- Active: 3.8.1 on PyPI (2026-10-28), commits 2026-11-26 ([repo](https://github.com/boxed/mutmut)). BSD-3-Clause.
- **Does not run on native Windows.** The README says it needs `fork` and Windows users must use WSL ([README](https://github.com/boxed/mutmut/blob/main/README.rst)); the program prints that message and exits when `platform.system() == "Windows"` ([source](https://github.com/boxed/mutmut/blob/main/src/mutmut/__main__.py)). Native support is an open request ([#397](https://github.com/boxed/mutmut/issues/397)).
- CI runs Ubuntu only ([workflow](https://github.com/boxed/mutmut/blob/main/.github/workflows/tests.yml)). macOS is documented (a `setproctitle` workaround) but not in CI.
- Machine output: `mutmut export-cicd-stats` writes **counts only**. Per-mutant status is printed by `mutmut results`; it is also stored in `mutants/*.meta` JSON files, which are an internal cache format ([data.py](https://github.com/boxed/mutmut/blob/main/src/mutmut/mutation/data.py)). `mutmut tests-for-mutant` lists covering tests. No "which test killed it" output was found.
- Scoping: `only_mutate` file patterns, and mutant-name wildcards such as `mutmut run "my_module*"`. No diff option for source lines was found. mutmut 3 mutates functions only (README).
- Speed: uses `fork`, runs in parallel, runs only the tests that reach a function, and re-tests only changed functions. Default timeout is (test time + 1 s) x 15 (README).

### Mutation: cosmic-ray

- Active: 8.9.0 (2026-11-10), commits 2026-11-26 ([releases](https://github.com/sixty-north/cosmic-ray/releases)). MIT.
- Machine output: `cosmic-ray dump` prints one JSON `[WorkItem, WorkResult]` pair per line ([cli.py](https://github.com/sixty-north/cosmic-ray/blob/master/src/cosmic_ray/cli.py)), including position, outcome, diff and test output ([work_item.py](https://github.com/sixty-north/cosmic-ray/blob/master/src/cosmic_ray/work_item.py)). No per-test attribution.
- Scoping: `module-path` takes a file or directory ([tutorial](https://cosmic-ray.readthedocs.io/en/latest/tutorials/intro/index.html)). `cr-filter-git` skips mutants outside lines changed against a branch ([filters](https://cosmic-ray.readthedocs.io/en/latest/how-tos/filters.html)); it runs `git diff --relative -U0 <branch> .`, branch set in config, default `master` ([source](https://github.com/sixty-north/cosmic-ray/blob/master/src/cosmic_ray/tools/filters/git.py)).
- Platforms: CI is **Ubuntu only** ([workflow](https://github.com/sixty-north/cosmic-ray/blob/master/.github/workflows/run-tests.yml)). There is Windows-specific code. The fix that stops timed-out runs leaving child processes alive on Windows ([#619](https://github.com/sixty-north/cosmic-ray/issues/619), commit "clean up Windows test trees") is on `master` but **after the 8.9.0 release**.
- Windows risk *(inference from [testing.py](https://github.com/sixty-north/cosmic-ray/blob/master/src/cosmic_ray/testing.py))*: the test command is split with `shlex.split` in POSIX mode, so unquoted backslash paths would be mangled.
- Caveats for a driver: it edits the source file on disk and restores it ([mutating.py](https://github.com/sixty-north/cosmic-ray/blob/master/src/cosmic_ray/mutating.py)). `is_killed` is true for anything not `survived`, including `incompetent`, so read `test_outcome` directly. Timeouts are reported as killed.
- Speed: the `local` distributor runs mutants **one at a time** ([source](https://github.com/sixty-north/cosmic-ray/blob/master/src/cosmic_ray/distribution/local.py)); parallelism needs HTTP workers, each with its own copy of the code ([tutorial source](https://github.com/sixty-north/cosmic-ray/blob/master/docs/source/tutorials/distributed/index.rst)). Each mutant runs the whole configured test command.

## JavaScript / TypeScript

### Coverage

- **Vitest** 5.1.2 (MIT), providers `v8` (default) or `istanbul`; `coverage.include` globs, `coverage.changed=<ref>` for changed files, and a `json` reporter ([config](https://github.com/vitest-dev/vitest/blob/main/docs/config/coverage.md), [guide](https://github.com/vitest-dev/vitest/blob/main/docs/guide/coverage.md)). CI includes all three OSes ([workflow](https://github.com/vitest-dev/vitest/blob/main/.github/workflows/ci.yml)).
- **Jest** 30.5.2 (MIT), providers `babel` (default) or `v8`; `collectCoverageFrom`, `coverageReporters` including `json` ([config](https://jestjs.io/docs/configuration)); `--findRelatedTests`, `--changedSince`, `--json --outputFile` ([CLI](https://jestjs.io/docs/cli)). CI covers all three ([workflow](https://github.com/jestjs/jest/blob/main/.github/workflows/nodejs.yml)).
- **c8** 12.0.0 (ISC), wraps any Node command using V8 coverage, Istanbul reporters, `--include` ([README](https://github.com/bcoe/c8)). CI: Linux and Windows; no macOS job ([workflow](https://github.com/bcoe/c8/blob/main/.github/workflows/ci.yaml)).
- **nyc** 18.0.0 (ISC) still releases ([npm](https://www.npmjs.com/package/nyc)); CI Linux and Windows.
- Per-test: no per-test coverage report was found in the Vitest, Jest or c8 docs checked (the last part of the Jest configuration page was not read). To get it, run tests one at a time, or take it from StrykerJS results.

### Mutation: StrykerJS

- Active: 10.1.1 (2026-10-23), Node 22+ ([releases](https://github.com/stryker-mutator/stryker-js/releases), [npm](https://www.npmjs.com/package/@stryker-mutator/core)). Apache-2.0.
- Drive it: CLI, or the `Stryker` class whose `runMutationTest()` returns the mutant results ([stryker.ts](https://github.com/stryker-mutator/stryker-js/blob/master/packages/core/src/stryker.ts)).
- Machine output: the `json` reporter writes `reports/mutation/mutation.json` ([configuration](https://github.com/stryker-mutator/stryker-js/blob/master/docs/configuration.md)). The [schema](https://github.com/stryker-mutator/mutation-testing-elements/blob/master/packages/report-schema/src/mutation-testing-report-schema.json) has per-mutant `status` (Killed, Survived, NoCoverage, Timeout, CompileError, RuntimeError, Ignored, Pending), `location`, `replacement`, `coveredBy`, `killedBy`.
- By default a mutant stops at the first failing test; `disableBail` reports all killing tests (the Jest runner has a special case, described in the configuration doc).
- Scoping ([configuration](https://github.com/stryker-mutator/stryker-js/blob/master/docs/configuration.md)): `--mutate src/a.ts`, or ranges such as `src/a.ts:5-7` (ranges cannot be combined with globs). `--testFiles` limits which test files run. There is no git-diff flag in the options read, so Evidry would turn a diff into ranges. [Incremental mode](https://github.com/stryker-mutator/stryker-js/blob/master/docs/incremental.md) reuses earlier results, with listed limits (it ignores changes outside mutated and test files).
- Speed: `coverageAnalysis: perTest` (default) runs only covering tests; static mutants run all tests; concurrency defaults to cores minus one (all cores if four or fewer). The `command` runner has no coverage analysis and runs everything for every mutant. A dry run is always needed.
- Platforms: CI on **Ubuntu and Windows**, no macOS job ([workflow](https://github.com/stryker-mutator/stryker-js/blob/master/.github/workflows/ci.yml)). Users report running it on macOS (e.g. [#6335](https://github.com/stryker-mutator/stryker-js/issues/6335)).
- Open issues that matter to Evidry:
  - [#6335](https://github.com/stryker-mutator/stryker-js/issues/6335) Vitest runner can score a killed mutant as Survived when bail wins a race. This error rejects a good test rather than accepting a bad one. `disableBail` may avoid it *(inference, untested)*.
  - [#6329](https://github.com/stryker-mutator/stryker-js/issues/6329) Vitest runner reports covered mutants as NoCoverage on Windows with `isolate: false`.
  - [#6295](https://github.com/stryker-mutator/stryker-js/issues/6295) Stryker does not exit on Windows after finishing. Evidry needs its own process timeout.
  - [#6272](https://github.com/stryker-mutator/stryker-js/issues/6272) Vitest 5 with debug logging crashes. The runner's peer range is `vitest >=2.0.0`, and its package pins 4.1.12 for its own development.
- The Vitest runner does not support Browser Mode and always uses `perTest` ([doc](https://github.com/stryker-mutator/stryker-js/blob/master/docs/vitest-runner.md)).

## Java

### Coverage: JaCoCo (and OpenClover)

- JaCoCo 0.8.16 (2026-10-26) ([releases](https://github.com/jacoco/jacoco/releases)); EPL-2.0 ([LICENSE.md](https://github.com/jacoco/jacoco/blob/master/LICENSE.md)). Officially supports class files up to Java 27 ([FAQ](https://www.jacoco.org/jacoco/trunk/doc/faq.html)).
- Machine output: `report --xml` / `--csv` ([CLI](https://www.jacoco.org/jacoco/trunk/doc/cli.html)). The agent can limit classes with `includes` ([agent](https://www.jacoco.org/jacoco/trunk/doc/agent.html)).
- Per-test: no built-in report. The agent has session IDs and remote control over TCP or JMX ([agent](https://www.jacoco.org/jacoco/trunk/doc/agent.html)), so a driver could dump and reset between tests *(inference)*.
- Platforms: GitHub CI has Linux and Windows jobs ([workflow](https://github.com/jacoco/jacoco/blob/master/.github/workflows/ci.yml)); no macOS job found there or in its Azure pipeline. macOS is **unverified** from project sources, although it is pure Java.
- **OpenClover** 5.1.1 (2026-10-01, Apache-2.0) does record per-test coverage ([downloads](https://openclover.org/downloads), [docs](https://openclover.org/doc/manual/4.2.0/ant--test-results-and-per-test-coverage.html)). I found it documented for HTML reports, not as a per-line test list in XML; per-test collection does not support parallel test execution. XML per-test output and OS support are **unverified**. It instruments source and adds build complexity.

### Mutation: PIT (pitest)

- Active: 1.30.2 (2026-11-05) ([releases](https://github.com/hcoles/pitest/releases)). Apache-2.0.
- Drive it: [command line](https://pitest.org/quickstart/commandline/), [Maven](https://pitest.org/quickstart/maven/), Gradle (third-party plugin, not checked).
- Machine output: `outputFormats` XML or CSV. The XML has one record per mutant with `status`, class, method, line, mutator and `killingTest`. With `fullMutationMatrix` it lists `killingTests`, `succeedingTests` and `coveringTests` ([XMLReportListener](https://github.com/hcoles/pitest/blob/master/pitest-entry/src/main/java/org/pitest/mutationtest/report/xml/XMLReportListener.java)). No native JSON was found.
- Scoping: `targetClasses` and `targetTests` globs (classes, not files or lines).
- Diff: the git integration is [arcmutate](https://docs.arcmutate.com/docs/git-integration.html), which **requires a licence**. Free terms for open-source projects were not stated on the page checked.
- Speed: the [FAQ](https://pitest.org/faq/) says it is fast for a mutation tool but runs still take a while; it advises threads, narrowing scope and limiting to changed code. Tests with hidden order dependencies can fail the required green run. [Incremental analysis](https://pitest.org/quickstart/incremental_analysis/) reuses history, with documented accuracy assumptions.
- Platforms: CI on Linux, Windows (JDK 17) and macOS (JDK 17) ([workflow](https://github.com/hcoles/pitest/blob/master/.github/workflows/ci.yml)).
- JUnit 5 needs [pitest-junit5-plugin](https://github.com/pitest/pitest-junit5-plugin) (Apache-2.0). Its latest version on Maven Central, 1.2.3, dates from May 2025, and **JUnit 6 support is an open issue** ([#113](https://github.com/pitest/pitest-junit5-plugin/issues/113)).

## C#

### Coverage

- **coverlet** 10.1.1 (2026-11-15), MIT ([repo](https://github.com/coverlet-coverage/coverlet)). Outputs JSON, lcov, OpenCover, Cobertura; include/exclude filters ([README](https://github.com/coverlet-coverage/coverlet/blob/master/README.md)). CI on all three ([pipeline](https://github.com/coverlet-coverage/coverlet/blob/master/eng/azure-pipelines.yml)).
  - Trap: `coverlet.collector` and `coverlet.msbuild` work only with VSTest. Projects on Microsoft.Testing.Platform (MTP) need `coverlet.MTP`.
- **Microsoft code coverage** (`dotnet-coverage`, and `Microsoft.Testing.Extensions.CodeCoverage`) 18.12.0. Outputs `coverage`, XML, Cobertura ([tool](https://learn.microsoft.com/en-us/dotnet/core/additional-tools/dotnet-coverage), [MTP](https://learn.microsoft.com/en-us/dotnet/core/testing/microsoft-testing-platform-code-coverage)). **Closed source**, free to use ([repo README](https://github.com/microsoft/codecoverage)). `dotnet-coverage` collects telemetry from 18.10.0, with opt-out. Platforms: [supported OS list](https://github.com/microsoft/codecoverage/blob/main/docs/supported-os.md) (dynamic instrumentation is not available on some Linux Arm builds or macOS Arm64; static is).
- Per-test: neither documents it. `dotnet-coverage snapshot --reset` could support a run-and-snapshot loop *(inference)*.

### Mutation: Stryker.NET

- Active: 5.1.0 (2026-11-10) ([releases](https://github.com/stryker-mutator/stryker-net/releases)). Apache-2.0. Needs the .NET 10 runtime or newer; the project under test may target older versions ([getting started](https://github.com/stryker-mutator/stryker-net/blob/master/docs/getting-started.md)).
- Machine output: `--reporter json` ([reporters](https://github.com/stryker-mutator/stryker-net/blob/master/docs/reporters.md)). Each mutant carries `Status`, `Location`, `Replacement`, `CoveredBy` and `KilledBy` ([JsonMutant.cs](https://github.com/stryker-mutator/stryker-net/blob/master/src/Stryker.Core/Stryker.Core/Reporters/Json/SourceFiles/JsonMutant.cs)).
- Scoping ([configuration](https://github.com/stryker-mutator/stryker-net/blob/master/docs/configuration.md)): `--mutate` globs, optionally with character spans (`File.cs{10..100}`, not line numbers). `--since:<ref>` selects by diff but marks **every mutant in a changed file** ([SinceMutantFilter.cs](https://github.com/stryker-mutator/stryker-net/blob/master/src/Stryker.Core/Stryker.Core/MutantFilters/SinceMutantFilter.cs)). One project under test per run.
- Speed: `coverage-analysis: perTest` by default; `perTestInIsolation` is more accurate but slower to start; concurrency defaults to half the logical processors.
- Platforms: unit tests in CI on all three ([workflow](https://github.com/stryker-mutator/stryker-net/blob/master/.github/workflows/unit-test.yaml)). .NET Framework projects need NuGet and Visual Studio build targets, which is a Windows setup.
- Risks: the MTP runner is **preview**, with open correctness issues ([#4018](https://github.com/stryker-mutator/stryker-net/issues/4018) mutant outcome depends on order; [#3956](https://github.com/stryker-mutator/stryker-net/issues/3956) timeouts ignore slow tests). [#4032](https://github.com/stryker-mutator/stryker-net/issues/4032) reports broken references in solution mode.

## Go

### Coverage: the Go toolchain

- `go test -coverprofile` with `-coverpkg` and `-run` ([flags](https://pkg.go.dev/cmd/go/internal/test)); `go test -json` for test events. `go tool covdata` converts and merges binary coverage data ([docs](https://go.dev/doc/build-cover)). BSD-3-Clause. Current releases 1.27.4 and 1.26.11 ([downloads](https://go.dev/dl/)).
- The profile is a text format, not JSON. File limiting means filtering the profile; `-coverpkg` works on packages.
- Per-test: not built in. Run each test alone with `-run` and its own profile *(inference)*.

### Mutation: Gremlins

- Maintenance is **slow**: latest release 0.6.0 (2025-12-06), latest commit 2026-03-30 ([repo](https://github.com/go-gremlins/gremlins)). Apache-2.0. Still 0.x, so flags may change; the README says it does not work well on very big modules.
- Machine output: `--output` JSON with per-file, per-mutant line, column, type and status ([unleash docs](https://github.com/go-gremlins/gremlins/blob/main/docs/docs/usage/commands/unleash/index.md)). No per-test data.
- Scoping: `--diff <ref>` and `--exclude-files` regex. No include-one-file flag in the docs ([request #289](https://github.com/go-gremlins/gremlins/issues/289)). Open diff bugs silently skip mutants: [#301](https://github.com/go-gremlins/gremlins/issues/301), [#296](https://github.com/go-gremlins/gremlins/issues/296), [#278](https://github.com/go-gremlins/gremlins/issues/278).
- Platforms: CI on all three ([workflow](https://github.com/go-gremlins/gremlins/blob/main/.github/workflows/ci.yml)) and Windows binaries, **but** open reports say it finds no coverage on Windows ([#222](https://github.com/go-gremlins/gremlins/issues/222), 2024) and is far slower there, 17 minutes against 40 seconds on one machine ([#81](https://github.com/go-gremlins/gremlins/issues/81), 2022). Whether either still happens is **unverified**.
- Speed: runs the mutated package's tests only (`--integration` runs the whole suite); parallel workers.

### Other Go candidates

| Tool | State | Notes |
|---|---|---|
| [avito-tech/go-mutesting](https://github.com/avito-tech/go-mutesting) (MIT) | v2.3.1 2025-12; commits 2026-10 | Takes files, dirs or packages; optional `report.json`. The built-in executor calls an external `diff` program and swaps the source file in place ([main.go](https://github.com/avito-tech/go-mutesting/blob/master/cmd/go-mutesting/main.go)), so stock Windows is a problem *(inference)*. CI Linux only. |
| [gomutants](https://github.com/gomutants/gomutants) (MIT) | Created 2026-04; v0.7.3 2026-11-10 | Per its README: JSON and Stryker-format output, changed-code scope, covering-test routing, `go test -overlay`. Its README says **Windows is not covered by automated tests**. Young, 19 stars. |
| [gomu](https://github.com/sivchari/gomu) (MIT) | v0.3.0 2026-10 | JSON output, file/dir targets, base-SHA scoping. Workflows seen run Linux; Windows **unverified**. |
| [ooze](https://github.com/gtramontina/ooze) (MIT) | commits 2026-08 | A library run from a Go test; tested on all three in CI; no machine-readable report documented. |
| [zimmski/go-mutesting](https://github.com/zimmski/go-mutesting) | last commit 2021 | Unmaintained in practice. |

Newer projects such as [mutago](https://github.com/quality-gates/mutago) and [gooze](https://github.com/gooze-dev/gooze) exist; I did not assess them beyond noting that no Windows job appeared in the CI files sampled.

## What this means for Evidry's keep-or-reject gate

These are design observations *(inference)*.

1. **"Passes repeatedly"** needs only the test runner. None of these tools is required.
2. **"Adds coverage"** is feasible in all five languages by comparing a run without and with the new test. Per-test data is not required for this.
3. **"Catches a planted bug"** must be attributed to the *new* test. Options: all killing tests (StrykerJS `disableBail`, PIT `fullMutationMatrix`), or restrict the run to the new test (StrykerJS `--testFiles`, PIT `targetTests`, cosmic-ray `test-command`). Stryker.NET reports `killedBy`, but I did not check whether it can list *all* killers.
4. A mutant that survived the old suite and is killed with the new test is the strongest signal. This needs stable mutant identity across two runs.
5. Treat `Timeout`, `RuntimeError`, `CompileError`, `incompetent` and `suspicious` as **not** proof that a test caught a bug, even where a tool counts them as detected.
6. Several tools edit files in place or leave processes running. Run them in a copy or worktree, with an outer timeout that kills the process tree.

## Not verified

- No tool was installed or executed, on any OS.
- No benchmark was run. Speed notes are the projects' own statements or user reports.
- No CI job for: macOS in StrykerJS, c8, nyc, JaCoCo; macOS or Windows in cosmic-ray and mutmut.
- OpenClover's OS support and XML per-test output.
- Whether `--testFiles` works with every StrykerJS runner.
- Gradle plugins for PIT and JaCoCo.
- Third-party dependency licences. Only the tools' own licences were considered. If Evidry calls a tool as a separate process rather than bundling it, licence obligations are likely lighter, but that is not legal advice and was not researched.
