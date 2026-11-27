# Evidry plants Python bugs itself

For TypeScript and JavaScript, Evidry uses StrykerJS to plant bugs. For Python, it plants them itself, because the first release must work natively on Windows, macOS and Linux and neither existing tool fits: mutmut needs WSL on Windows, and cosmic-ray runs one planted bug at a time and is tested only on Linux by its own project.

## Consequences

Evidry has to build and maintain its own Python bug planting. Counts of planted bugs caught are not strictly comparable between Python and the other two languages.

This rests on reading the tools' documentation and source, checked 2026-11-27; neither tool was run. Revisit if one of them gains native, tested support on all three systems.

Decided in [Which language comes first, and what is Evidry written in?](https://github.com/ahmedbinaasim/evidry/issues/7).
