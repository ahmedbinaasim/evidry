# A container is the default, not a requirement

Evidry runs code a model wrote, so it runs tests in a container by default. It does not require one: containers on Windows need WSL 2 or another virtual machine, which conflicts with working natively on all three systems, and Docker Desktop needs a paid subscription in larger companies. Without a container, Evidry stops until the user sets `isolation: none` in `evidry.yaml`; it never falls back on its own.

## Consequences

Two ways of running have to be built and tested. Under `isolation: none` the remaining protections (a temporary copy, a reduced environment, time limits) reduce accidents but cannot stop badly behaved code from reaching files or the network, and Evidry has to say so.

Docker facts checked 2026-11-27 against [its Windows install page](https://docs.docker.com/desktop/setup/install/windows-install/).

Decided in [How do generated tests run safely?](https://github.com/ahmedbinaasim/evidry/issues/9).
