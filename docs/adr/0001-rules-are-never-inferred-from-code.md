# Rules are never inferred from code

Rules come only from intent a person wrote down (requirements, stories, or a rule typed by hand) and are never derived from the code, even though the code is always available and many projects have no written requirements. A rule read from the code would turn existing bugs into requirements, so tests would confirm the bug instead of reporting it.

## Consequences

A project with no written requirements has no rules until someone writes some. `unit` still keeps tests on coverage and planted bugs there, but reports failures as unexplained rather than as suspected bugs. Docstrings, comments and API spec constraints are not sources for now and may be revisited.

Decided in [What is a rule?](https://github.com/ahmedbinaasim/evidry/issues/5).
