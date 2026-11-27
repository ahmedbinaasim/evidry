# Evidry

Evidry writes test cases from code, requirements or an API spec, and shows the evidence for each one.

## Language

**Requirement**:
Text a person wrote saying what the system must do, such as a story or a requirements document.
_Avoid_: Spec (reserved for an API spec)

**Rule**:
One testable statement of how the system must behave, taken from intent a person wrote down and never inferred from the code.
_Avoid_: Requirement, assertion, constraint

**Proposed rule**:
A rule no person has approved yet.
_Avoid_: Draft rule

**Approved rule**:
A rule whose current wording a person has accepted. Only an approved rule can be grounds for a suspected bug.
_Avoid_: Verified rule, confirmed rule

**Retired rule**:
A rule that no longer applies. Its ID is never reused.
_Avoid_: Deleted rule

**Linked**:
Said of a rule that a test claims to cover.
_Avoid_: Covered, traced

**Confirmed**:
Said of a rule with a linked test that passes and fails when the rule is deliberately broken, or that a person has checked.
_Avoid_: Verified, proven

**Planted bug**:
A small deliberate change to a copy of the code, made to see whether a test notices.
_Avoid_: Mutant, mutation

**Rule break**:
A deliberate change to a copy of the code that violates one particular rule, made to see whether the rule's linked test notices.
_Avoid_: Planted bug

**Kept test**:
A generated test that passes reliably and either catches a planted bug no other test catches or confirms a rule nothing else has confirmed.
_Avoid_: Proven test, verified test

**Stale test**:
A test linked to a rule whose wording changed after the test was last checked.
_Avoid_: Outdated test

**Suspected bug**:
A repeatable failure, on the outcome an approved rule describes, of a test that states that rule correctly, which no person has judged yet.
_Avoid_: Bug, defect

**Unexplained failure**:
A failing test where it is not established whether the code or the test is wrong, because no approved rule applies or the test may not state its rule correctly.
_Avoid_: Suspected bug

**Open question**:
A statement in a requirement too vague to become a rule until a person clarifies it.
_Avoid_: Ambiguity, TODO
