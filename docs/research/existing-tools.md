# Existing tools that overlap with Evidry

Research for issue #2: which tools already generate or verify unit tests, API tests or QA test cases, and where Evidry differs.

**Date checked: 2026-11-27.** All links were read, or queried, on that date.

## Conclusion

Each part of Evidry already exists somewhere. The combination was not found in any tool checked.

- Unit tests kept only on evidence: CoverUp (repeat runs and coverage), TestForge and MutantFix (planted bugs), Meta's internal tools.
- API tests checked against the spec: Schemathesis, EvoMaster, CATS, Specmatic and others do this well.
- Test cases linked to requirements: Xray, TestRail, Qase, VectorCAST Reqs2x, Kiro and several small projects.
- Closest overlaps: **VectorCAST Reqs2x** (requirements to unit tests, traced, own model, commercial, C/C++/Ada), **Kiro** (requirements to property tests, traced, IDE only), **Diffblue Testing Agent** (verified unit tests, own coding agent, commercial) and **api-vv-toolkit** (OpenAPI plus requirements, per-requirement verdicts, MIT, very small).

What Evidry would do that none of the checked tools do:

1. Keep **one set of numbered rules** used by unit, API and QA generation, with **one report** of which tests of all three kinds cover each rule.
2. Require **mode-specific evidence before keeping a test**, in all three modes, in one tool.
3. Offer that as an **open-source local CLI** with the user's own model key and no hosted service or paid licence.

This is a claim about the tools in this file, not proof that no such tool exists. Points 1 and 3 are the safer differentiators. "We verify generated tests" and "we trace tests to requirements" are not differentiators on their own.

Evidry should reuse spec-conformance checking, coverage, mutation testing and model access, and build the rule model, the evidence gate and the cross-mode report (see [Reuse](#what-evidry-could-reuse)).

## How to read the tables

- **Evidence** is what the tool does to show a test is good.
- **Req. link** is whether tests are linked to requirements. "None found" means the pages read did not describe it. It does not prove absence.
- **NV** means not verified.
- Dates for open-source projects are the latest default-branch commit and latest GitHub release, from the GitHub API (`repos/{owner}/{repo}`, `/commits`, `/releases`). Licences are GitHub's detection, or the licence file where detection was inconclusive.
- Most open-source claims come from each project's README. Tools were not installed or run.

## Unit test tools

### Open source

| Tool | Input | Evidence | Req. link | Languages | Licence | Maintenance |
|---|---|---|---|---|---|---|
| [Qodo Cover](https://github.com/qodo-ai/qodo-cover) | Source, existing tests, test command, Cobertura XML | Runs the test; keeps it if coverage rises | None found | Any that emits Cobertura XML | AGPL-3.0 | **Unmaintained** (README notice dated 2025-06-15); last commit 2025-06-24 |
| [CoverUp](https://github.com/plasma-umass/coverup) | Python package and tests | Runs alone, must pass and add coverage; repeats runs to catch flaky tests | None found | Python (pytest) | Apache-2.0 | Commit 2026-04-05; v0.6.3 (2025-07) |
| [Pynguin](https://github.com/se2p/pynguin) | Python module | Search for coverage; re-runs to drop unstable assertions; optional mode keeps only assertions that fail on a mutant ([docs](https://pynguin.readthedocs.io/latest/user/assertions.html)) | None found | Python (3.10; later versions experimental) | MIT | Active: commit 2026-11-26; 0.48.0 (2026-10). Describes itself as a research prototype |
| [EvoSuite](https://github.com/EvoSuite/evosuite) | Java classes | Search for coverage criteria; regression assertions record current behaviour; controls non-determinism ([tutorial](https://www.evosuite.org/documentation/tutorial-part-1/)) | None found | Java | LGPL-3.0 | Quiet: commit 2025-02; v1.2.0 (2021-09) |
| [Randoop](https://randoop.github.io/randoop/manual/index.html) | Java classes | Separates error-revealing tests (contract violations) from regression tests; configurable flaky handling | None found | Java | MIT | Active: commit 2026-11-27; v4.3.4 (2025-06) |
| [UnitTestBot](https://github.com/UnitTestBot/UTBotJava) | Java code | Symbolic execution and fuzzing to maximise path coverage | None found | Java | Apache-2.0 | Commit 2025-07; release 2025.9 |
| [TestSpark](https://github.com/JetBrains-Research/TestSpark) | Java/Kotlin code in IntelliJ (LLM, EvoSuite, Kex) | Shows pass/fail, line, branch and mutation coverage; user selects | None found | Java, Kotlin | MIT | Commit 2026-01; v0.4.2 |
| [ChatUniTest](https://github.com/ZJU-ACES-ISE/chatunitest-maven-plugin) | Maven project, class or method; model API key | Generate, execute, repair loop | None found | Java | MIT | Commit 2026-03; v2.0.0 (2024-12) |
| [TestPilot](https://github.com/githubnext/testpilot) | npm package | Optionally runs and re-prompts on failure | None found | JavaScript/TypeScript | MIT | **Archived** |
| [Mutahunter](https://github.com/codeintegrity-ai/mutahunter) | Source, tests, test command | LLM-made mutants; the current README documents mutation testing only | None found | Described as language-agnostic | AGPL-3.0 | Commit 2025-04 |
| [TestForge](https://github.com/Luz7818/testforge) | Python function | Must pass stably and kill a mutant that existing tests miss | None found | Python | MIT | Commit 2026-11-15; v0.6.0; 1 star |
| [MutantFix](https://github.com/Vikas-u-rao/mutantfix) | Project with Stryker report | Keeps a test only if it passes and kills its target mutant | None found | JavaScript (Jest) | MIT | Commit 2026-10; demo stage, 0 stars |

Related, not a test-case writer: [Hypothesis](https://github.com/HypothesisWorks/hypothesis) (property-based testing, Python, MPL-2.0, active) and [CrossHair](https://github.com/pschanely/CrossHair) (analysis of Python contracts, mainly MIT, active).

Meta's [TestGen-LLM](https://arxiv.org/abs/2402.09171) filters generated tests for building, passing reliably and adding coverage. Meta's [ACH](https://arxiv.org/abs/2501.12862) generates tests that catch targeted mutants (Android Kotlin). No code release was found on either abstract page. They show the evidence idea is established.

### Commercial

| Tool | Input | Evidence | Req. link | Languages | Licence | Maintenance |
|---|---|---|---|---|---|---|
| [Diffblue Testing Agent](https://docs.diffblue.com/llms-full.txt) | Git repo that builds with passing tests; the user's own coding agent (Claude Code, Copilot CLI, Gemini CLI, OpenCode) | Compiles and runs each test, removes failures, reports line/branch coverage. A separate quality workflow runs mutation testing (PIT; mutmut, experimental) | None found | Java 8+, Python 3.9+ | Proprietary, licence required | v1.2.2 (2026-07-24, [changelog](https://docs.diffblue.com/changelog.md)) |
| [GitHub Copilot testing for .NET](https://learn.microsoft.com/en-us/visualstudio/test/github-copilot-test-dotnet-overview) | C# solution, file, member or git diff | Builds and runs tests, reports coverage | None found | C# (xUnit, NUnit, MSTest) | Proprietary, paid Copilot plan, Visual Studio 2026 18.3+ | Page updated 2026-10-08 |
| [VectorCAST Reqs2x](https://www.vector.com/en/product/vectorcast/vectorcast-ai/) | Requirements (DOORS, Polarion, CSV) plus code | Human review of every mapping and test; nothing auto-accepted | **Yes**, requirement to test | C, C++, Ada (listed on related VectorCAST pages) | Proprietary; bring your own model | Currently marketed; release date NV |
| [Tusk](https://docs.usetusk.ai/automated-tests/test-generation.md) | PR diff, code context, PR and Jira/Linear ticket text | Runs in a sandbox, repairs errors, filters for relevance, de-duplicates | Ticket used as context; no per-requirement link found | [JS, TS, Python, Ruby, Go, Java, C/C++, C#, Rust, PHP](https://docs.usetusk.ai/requirements.md) | Proprietary hosted service | Docs live; release date NV |
| [Early](https://www.startearly.ai/docs/) | Repository / IDE code | NV | NV | Legacy IDE extension: JS/TS, Python | Proprietary | NV |
| [Qodo](https://www.qodo.ai/glossary/what-is-ai-test-automation/) | PRs | The current comparison page presents Qodo as code review, with tests run by the user's CI | NV | NV | Proprietary | Whether the IDE test generator is still offered: NV |
| [Amazon Q Developer `/test`](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-developer-ide-end-of-support.html) | IDE code | NV | NV | NV | Proprietary | **Ending**: IDE plugins end of support 2027-04-30 |

## API test tools

| Tool | Input | Evidence | Req. link | Targets / output | Licence | Maintenance |
|---|---|---|---|---|---|---|
| [Schemathesis](https://github.com/schemathesis/schemathesis) | OpenAPI, GraphQL, gRPC | [Checks](https://schemathesis.readthedocs.io/en/stable/reference/checks/) for 5xx, status, headers, body schema, negative and positive data, stateful and auth behaviour; replay; schema coverage | None found (custom checks possible) | Any running API; Python library and CLI | MIT | Very active: v4.34.3 (2026-11-27) |
| [EvoMaster](https://github.com/WebFuzzing/EvoMaster) | OpenAPI, GraphQL, RPC; running API | Search for coverage and faults; 30+ fault oracles including 500s and schema mismatches | None found | White-box: JVM. Black-box: any. Output: Python, JS, Java, Kotlin | LGPL-3.0 | Active: v6.2.0 (2026-09) |
| [CATS](https://github.com/Endava/cats) | OpenAPI 3.0/3.1 | Fuzzer verdicts, replay, quality gate | None found | Any REST API | Apache-2.0 | Active: 15.6.0 (2026-11) |
| [RESTler](https://github.com/microsoft/restler-fuzzer) | OpenAPI | Stateful fuzzing, bug buckets, replay logs | None found | Any REST API | MIT | Slower: commit 2026-02 |
| [RESTest](https://github.com/isa-group/RESTest) | OpenAPI 2.0 to 3.1 | Black-box testing; self-described experimental | None found | Any REST API | Apache-2.0 | Active: v2.5.0 (2026-11) |
| [RestTestGen](https://github.com/SeUniVr/RestTestGen) | OpenAPI 3 | Status-code and security oracles, API coverage; REST Assured output | None found | Any REST API | Apache-2.0 | Commit 2025-12 |
| [Specmatic](https://docs.specmatic.io/references/open-source-vs-enterprise) | OpenAPI, SOAP, others | Contract and schema-resiliency tests from the spec | None found | Any API | MIT core; paid Enterprise for AsyncAPI, GraphQL, gRPC contract tests and more | Active: 2.57.0 (2026-11) |
| [Portman](https://github.com/apideck-libraries/portman) | OpenAPI 3.0/3.1 | Injects contract and variation tests into Postman, run by Newman | None found | Postman collections | Apache-2.0 | Active: v1.35.3 (2026-11) |
| [Keploy](https://github.com/keploy/keploy) | Recorded traffic; OpenAPI for expansion | Deterministic replay with mocks, coverage. [Unit generation](https://keploy.io/docs/keploy-explained/supported-languages/) keeps tests that build, run and add coverage (Go only) | None found | API record/replay is language-agnostic | Apache-2.0; enterprise offering | Very active: v3.7.27 (2026-11-27) |
| [Dredd](https://github.com/apiaryio/dredd) | OpenAPI 2 (3 experimental) | Compares responses with the description | None found | Any HTTP API | MIT | **Archived** |
| [Step CI](https://github.com/stepci/stepci) | Its own YAML workflows (OpenAPI generation NV) | Assertions | None found | Any API | MPL-2.0 | Stale: commit 2024-08 |
| [Postman Agent Mode](https://learning.postman.com/docs/agent-mode/overview.md) | Collections, specs, natural language | Writes and debugs tests; acceptance gate NV | NV | Postman | Proprietary; can use own OpenAI/Anthropic key | Current docs |
| [Parasoft SOAtest](https://docs.parasoft.com/x/SxSZCQ) | OpenAPI | Generates scenarios; gate NV | Vendor advertises traceability; NV | SOAtest | Proprietary; OpenAI or Azure OpenAI account | NV (page has a placeholder release header) |

## QA test case tools

### Commercial test management

| Tool | Input | Evidence | Req. link | Licence | Notes |
|---|---|---|---|---|---|
| [Xray AI generation](https://docs.getxray.app/space/XRAYCLOUD/392921171/AI+Test+Case+Generation) | Requirement text; optional Jira issues and preconditions | Human review and edit | **Yes**, to Jira issues; [coverage status per issue](https://docs.getxray.app/space/XRAYCLOUD/44565185/Coverage+Analysis) | Proprietary | Manual or Cucumber cases |
| [TestRail AI](https://www.testrail.com/blog/ai-test-case-generation/) | Requirement text | Human picks and edits titles before steps are written | **Yes**, through the References field and a coverage report | Proprietary | The support article refused the fetch (403); this row rests on the vendor blog |
| [Qase](https://www.qase.io/requirements-traceability-matrix/) | Requirement text from Jira, GitHub and others | Human approves suggestions | **Yes**, requirement to case, with a matrix | Proprietary | Granularity looks issue-level (NV) |
| [BrowserStack Test Management](https://www.browserstack.com/docs/test-management/browserstack-ai/ai-generated-test-cases) | Prompts, PDFs, tickets, designs | Iterative refinement by the user | Per-case links not described on this page | Proprietary | Plain or BDD output |
| [TestSprite](https://docs.testsprite.com/mcp/concepts/key-terms) | PRD, or one inferred from code; OpenAPI for backend | Runs tests in its cloud against the live app ([CLI](https://github.com/TestSprite/testsprite-cli)) | PRD drives the plan; per-requirement report NV | CLI Apache-2.0; hosted service proprietary | CLI v0.18.0 (2026-11-26) |

Jira-native add-ons and other test managers also advertise AI generation. They were not examined. [Kiwi TCMS](https://github.com/kiwitcms/kiwitcms) (GPL-2.0, active) is an open-source test manager; AI generation in it was not looked for.

### Small open-source projects

These overlap closely in idea. All had 0 stars on the date checked, so treat them as prior art, not established competitors.

| Tool | Input | Evidence | Req. link | Licence | Maintenance |
|---|---|---|---|---|---|
| [api-vv-toolkit](https://github.com/muhib-karim/api-vv-toolkit) | OpenAPI 3.0/3.1 and requirements YAML | Runs API tests; a requirement is verified only when passing tests cover every response and check it names; LLM suggestions validated against the spec | **Yes**, matrix | MIT | v1.3.0; commit 2026-11-26 |
| [ai-testcase-generator (karthickpearl1)](https://github.com/karthickpearl1/ai-testcase-generator) | User stories or Gherkin, OpenAPI 3 | Schema and trace validation, execution, mutation score (only against its bundled demo API) | **Yes**, to acceptance criteria | MIT | Commit 2026-10 |
| [CaseFoundry](https://github.com/MadanMohan0537/requirements-traced-test-generator) | Numbered `[R1]` clauses | Review queue; seeded faults (sample app only) | **Yes** | MIT | Commit 2026-11; deterministic templates, no model yet |
| [TRACE](https://github.com/AchintSingh1303/trace) | Requirements with acceptance criteria | Human approval; marks cases stale when a requirement changes | **Yes**, matrix | No licence file found | Commit 2026-11 |
| [ai-test-case-generator (hambaliFadib)](https://github.com/hambaliFadib/ai-test-case-generator) | DOCX, PDF, Markdown, text; own model key | Deterministic validation, de-duplication, coverage audit | **Yes**, requirement reference per case | MIT | v1.3.2; commit 2026-11 |

## Spec-driven coding tools

| Tool | Input | Evidence | Req. link | Languages | Licence | Maintenance |
|---|---|---|---|---|---|---|
| [Kiro correctness](https://kiro.dev/docs/specs/correctness/) | EARS-format requirements | Property-based tests try to falsify each property; optional step | **Yes**, property to requirement and task | [Blog](https://kiro.dev/blog/property-based-testing/) names Hypothesis (Python); other languages NV | Proprietary | IDE only (not CLI or web) |

[Spec Kit](https://github.com/github/spec-kit) and [OpenSpec](https://github.com/Fission-AI/OpenSpec) (both MIT, active) manage specifications for coding agents. Whether they verify or trace tests was not examined.

## Overlap by Evidry promise

| Evidry promise | Already done by | Gap left |
|---|---|---|
| Passes repeatedly | CoverUp, TestForge, TestGen-LLM | Not combined with requirement rules |
| Adds coverage | Qodo Cover, CoverUp, Keploy (Go), TestGen-LLM | The main open CLI (Qodo Cover) is unmaintained |
| Catches a planted bug | TestForge, MutantFix, Pynguin (assertion mode), ACH, Diffblue (reported separately) | No established, maintained, multi-language open-source tool found that gates LLM tests this way |
| Matches the spec | Schemathesis, EvoMaster, CATS, Specmatic | Mature. Build on it, do not compete |
| Traces to a requirement rule | Xray, TestRail, Qase, Reqs2x, Kiro, small projects | Mostly links to a ticket or requirement document; one rule set shared by unit, API and QA output not found |
| Bring your own key, no hosted service | CoverUp, ChatUniTest, hambaliFadib, others | Not found together with the points above |

Risks to the differentiation:

- Test management tools can already show coverage per requirement. If they also count imported unit and API results (NV), a team using them gets part of Evidry's report at ticket level. Evidry's edge would then be rule-level granularity, automatic linking and the evidence behind each test.
- Diffblue, Kiro or a test manager could add the missing mode.
- General coding agents can write all three kinds of test when prompted. They were not assessed here.

## What Evidry could reuse

Licences matter because Evidry's repository is MIT. This is a practical note, not legal advice.

| Need | Candidate | Licence | Note |
|---|---|---|---|
| API test generation and conformance | [Schemathesis](https://github.com/schemathesis/schemathesis) | MIT | Strongest reuse: do not write conformance checks again |
| OpenAPI parsing and validation | [libopenapi](https://github.com/pb33f/libopenapi), [libopenapi-validator](https://github.com/pb33f/libopenapi-validator) (Go) | MIT | Choice depends on Evidry's language |
| Spec linting, mocking, diffing | [Spectral](https://github.com/stoplightio/spectral), [Prism](https://github.com/stoplightio/prism), [oasdiff](https://github.com/oasdiff/oasdiff) | Apache-2.0 | |
| Coverage | [coverage.py](https://github.com/coveragepy/coveragepy) (Apache-2.0), [c8](https://github.com/bcoe/c8) (ISC), [JaCoCo](https://github.com/jacoco/jacoco/blob/master/LICENSE.md) (EPL-2.0), [diff_cover](https://github.com/Bachmann1234/diff_cover) (Apache-2.0) | as listed | Details belong to issue #3 |
| Planted bugs | [mutmut](https://github.com/boxed/mutmut) (BSD-3), [cosmic-ray](https://github.com/sixty-north/cosmic-ray) (MIT), [StrykerJS](https://github.com/stryker-mutator/stryker-js) and [Stryker.NET](https://github.com/stryker-mutator/stryker-net) (Apache-2.0), [PIT](https://github.com/hcoles/pitest) (Apache-2.0), [cargo-mutants](https://github.com/sourcefrog/cargo-mutants) (MIT) | as listed | All committed to in 2026 |
| Mutation report format | [mutation-testing-elements](https://github.com/stryker-mutator/mutation-testing-elements) | Apache-2.0 | Shared result schema and viewer |
| Property-based tests | [Hypothesis](https://github.com/HypothesisWorks/hypothesis) | MPL-2.0 | |
| Gherkin input and output | [cucumber/gherkin](https://github.com/cucumber/gherkin) | MIT | |
| Traceability formats | [StrictDoc](https://github.com/strictdoc-project/strictdoc) (Apache-2.0), [sphinx-needs](https://github.com/useblocks/sphinx-needs) (MIT), [Doorstop](https://github.com/doorstop-dev/doorstop) (LGPL-3.0), [OpenFastTrace](https://github.com/itsallcode/openfasttrace) (GPL-3.0) | as listed | Consider import/export rather than embedding; review GPL before linking |
| Model access | [LiteLLM](https://github.com/BerriAI/litellm) (MIT outside `enterprise/`), [Vercel AI SDK](https://github.com/vercel/ai) (Apache-2.0), [llm](https://github.com/simonw/llm) (Apache-2.0) | as listed | |
| Reporting | JUnit XML; [Allure](https://github.com/allure-framework/allure2) | Apache-2.0 | Also export CSV for Xray, TestRail, Zephyr |

Design references to study rather than copy: CoverUp's loop (Apache-2.0), TestForge's acceptance gate (MIT), api-vv-toolkit's verification rule (MIT). Do not copy code from Qodo Cover or Mutahunter into an MIT project: both are AGPL-3.0.

## Not verified and limits

- No tool was installed or run. Vendor claims are as documented by the vendor.
- The search was not exhaustive. Other products (for example Katalon, mabl, testRigor, Jama, Polarion) were not examined. Browser end-to-end tools are out of scope for Evidry.
- "None found" for requirement links rests on the pages read, often a README or a single docs page.
- NV items in the tables, in particular: Early's validation, current Qodo test generation, Postman and SOAtest acceptance gates, Parasoft traceability, Kiro's property-test languages beyond Python.
- Whether Xray, TestRail or Qase count imported automated test results toward requirement coverage was not confirmed.
- TestRail's support documentation could not be fetched.
- Star counts and commit dates indicate activity, not quality.
