# Coverage alone does not keep a test

A generated test that passes repeatedly is kept only if it catches a planted bug that the existing passing tests and the tests already kept all miss, or if it confirms an approved rule that nothing else has confirmed. Added coverage does not count, because a test can run a line without checking anything about it.

This is stricter than the starting vision, which accepted added coverage or a caught planted bug, and than tools that keep a test when coverage rises.

## Consequences

Fewer tests are kept, and judging each one costs more runs. Where no bug can be planted, only rule confirmation can keep a test. Coverage is still reported.

Decided in [What does proven mean for a unit test, and when is a failure a suspected bug?](https://github.com/ahmedbinaasim/evidry/issues/8).
