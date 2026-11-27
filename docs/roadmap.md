# Roadmap

**Status:** accepted by the owner. Order only; no dates.

## First release

`init`, `qa`, `unit`, `diff` and `report` for TypeScript, JavaScript and Python, plus the website. See the [spec](spec.md).

## Then, in this order

1. **`api` mode:** tests from an API spec. This completes the three connected modes. Build on existing conformance tools (the research points to Schemathesis).
2. **GitHub Action:** comments on pull requests with new tests and untested rules.
3. **MCP server:** a thin layer for coding agents.
4. **Java.**
5. **More QA formats:** Gherkin, CSV, test-management import.
6. **HTML review page** for approving and rejecting.
7. **Public benchmark** of projects with planted bugs.
8. **VS Code extension.**

## Later, not yet ordered

- C# and Go, and other languages, possibly from contributors
- React and React Native components
- Mocha, `unittest` and other test runners
- Podman and other container tools
- A PyPI wrapper
- Separately published plugins
- Requirements from GitHub issues, Jira, Word and PDF
- Docstrings, comments and API spec constraints as rule sources (to be revisited)
- Starting databases and other services for tests

## Not planned

Hosted service, accounts or dashboard; browser end-to-end, load and security testing; automatic bug fixing.

## What the first release leaves room for

The core is separate from the command line; results are saved in a versioned, machine-readable file; exit codes distinguish findings from failures; nothing assumes every runnable test is a unit test. Nothing else about later items is designed yet.
