# PROTOTYPE: a first session with Evidry

Throwaway mock for [What does using Evidry look like, from first command to report?](https://github.com/ahmedbinaasim/evidry/issues/11). **Nothing here exists or has been run.** Every file, number and message is invented so you can react to it.

**Verdict:** all nine recommendations (R1 to R9) were accepted. The marks **(R1)** to **(R9)** point at the questions asked in chat.

The example project is a small shop: a TypeScript cart in `web/` and a Python pricing service in `api/`.

---

## 1. Set up

```text
$ evidry init

Found
  TypeScript   web/   Vitest
  Python 3.12  api/   pytest
  Model        key found in your environment

Wrote evidry.yaml. Check it, then run:  evidry qa <requirements file>
```

`evidry.yaml` (model, spending and sharing settings are placeholders, decided in other tickets):

```yaml
projects:
  - path: web
    language: typescript
    tests: vitest
  - path: api
    language: python
    tests: pytest
model: <provider and name>
spend_limit_per_run: <amount>
exclude: [".env*", "secrets/"]
```

## 2. What lands in the repository (R1)

```text
evidry.yaml                      committed
evidry/
  rules/checkout.md              committed   you read and edit these
  cases/checkout.md              committed   you read and edit these
  held/                          committed   failing tests, waiting for you
  state.json                     committed   Evidry's memory; don't edit
  report.md                      ignored     rebuilt by `evidry report`
web/src/cart.ts
web/src/cart.test.ts             yours, never touched
web/src/cart.evidry.test.ts      written by Evidry          (R4)
api/tests/test_pricing.py        yours, never touched
api/tests/test_pricing_evidry.py written by Evidry          (R4)
```

## 3. From requirements to rules and cases

(CHK-7 to CHK-9 in the later report are rules someone typed by hand afterwards.)

```text
$ evidry qa docs/checkout.md

Rules      6 proposed  (CHK-1 to CHK-6)
Cases      9 proposed
Dropped    3 cases
             2 duplicates of a kept case
             1 the second check said it did not test CHK-3
Questions  1  "Large orders ship faster": how large, how much faster?

Nothing is approved yet. Review:
  evidry/rules/checkout.md
  evidry/cases/checkout.md
```

### Rules file (R2)

````markdown
# Checkout rules

## CHK-1: Total is the sum of the line prices
Status: proposed
Source: docs/checkout.md, "Cart"

The cart total equals the sum of price times quantity for every line.

## CHK-4: Discounts are capped at 100%
Status: proposed
Source: docs/checkout.md, "Discounts"

A discount never reduces the total below zero, whatever percentage is entered.

## Open questions

- "Large orders ship faster" (docs/checkout.md, "Shipping"): how large, how much faster?
````

**To approve**, change `Status: proposed` to `Status: approved`. **To add a rule**, type a new section with the next ID.

**When the requirements document changes**, `qa` does not overwrite an approved rule. It adds a block under it (R3):

```markdown
## CHK-4: Discounts are capped at 100%
Status: approved
Source: docs/checkout.md, "Discounts"

A discount never reduces the total below zero, whatever percentage is entered.

> **Proposed change** (docs/checkout.md was edited)
> A discount of more than 50% is rejected.
```

You accept by replacing the old sentence with the new one, or reject by deleting the block.

### Cases file

````markdown
# Checkout cases

## CASE-7: A 120% discount does not go below zero
Rules: CHK-4
Type: invalid input
Status: proposed

Before you start: a cart with one item priced 40.00.

1. Enter a discount of 120%.
2. Look at the total.

Expected: the total is 0.00, never negative.
````

Approval works the same way: change `Status`.

## 4. Unit tests

```text
$ evidry unit web/src/cart.ts api/pricing.py

Existing tests    41 pass, 1 flaky (left out of comparison)
Generating        19 candidate tests
Running each 5x   16 stable
Planting bugs     ........................

Kept            7
Dropped         9   6 caught no bug your tests miss
                    2 flaky
                    1 could not be made to run
Suspected bugs  1   CHK-4
Unexplained     2

line coverage       58% -> 81%
planted bugs caught 22 of 31
spent this run      <amount> of <limit>

Details: evidry report
```

A kept test, in your framework and style:

```ts
// evidry: CHK-1
it('totals price times quantity across lines', () => {
  expect(total([{ price: 10, qty: 2 }, { price: 5, qty: 1 }])).toBe(25)
})

// evidry: records current behaviour
it('returns 0 for an empty cart', () => {
  expect(total([])).toBe(0)
})
```

## 5. A test that fails (R5)

It is saved in `evidry/held/`, with a `.held` ending so no test runner picks it up:

```python
# evidry/held/test_chk4_discount_cap.py.held
#
# SUSPECTED BUG  CHK-4: Discounts are capped at 100%
# apply_discount(40.0, 120) returned -8.0; the rule says never below zero.
# Failed 5 of 5 runs. Second check: the test states the rule correctly.
#
# If the code is wrong: fix it, move this to api/tests/ and drop ".held".
# If the test is wrong: delete this file. Evidry will not write it again.

# evidry: CHK-4
def test_discount_never_goes_below_zero():
    assert apply_discount(40.0, 120) >= 0
```

An unexplained failure looks the same, headed `UNEXPLAINED` with why no rule could judge it.

## 6. Only what a branch changed

```text
$ evidry diff main

Changed against main: api/pricing.py (apply_discount, shipping_cost)
Generating for those functions only ...
```

The results read the same as `unit`.

## 7. The report (R8)

```text
$ evidry report

Running linked tests (no model calls) ...

Rules   2 confirmed   4 linked   1 contradicted   1 not approved   1 retired
Needs you   1 suspected bug, 2 unexplained, 1 rule conflict, 1 stale case

Full report: evidry/report.md
```

`evidry/report.md`:

> ### Rules
>
> | Rule | State | Evidence |
> |---|---|---|
> | CHK-1 Total is the sum of lines | **Confirmed** | 2 tests; failed when the rule was broken |
> | CHK-2 Email lists every item | Linked | manual cases only (2) |
> | CHK-3 Removed items stop counting | Linked | no break the test caught |
> | CHK-4 Discounts capped at 100% | **Contradicted** | suspected bug in `apply_discount` |
> | CHK-5 Expired codes are rejected | Not approved | 1 test, cannot accuse the code |
> | CHK-6 One code per order | Retired | 1 test and 1 case still point here (R7) |
> | CHK-7 Shipping is free from 50 | Linked | conflicts with CHK-8 |
> | CHK-8 Orders under 60 pay shipping | Linked | conflicts with CHK-7 |
> | CHK-9 Gift cards are never discounted | **Confirmed** (by you) | marked checked |
>
> ### How CHK-1 was confirmed
>
> Evidry broke the rule in a temporary copy and the linked test failed:
>
> ```diff
> - return lines.reduce((sum, l) => sum + l.price * l.qty, 0)
> + return lines.reduce((sum, l) => sum + l.price, 0)
> ```
>
> ### Conflicting rules (R6)
>
> CHK-7 "Shipping is free from 50" and CHK-8 "Orders under 60 pay shipping" disagree for orders from 50 to 59.99. Until one changes, neither can raise a suspected bug.
>
> ### Stale
>
> CASE-3 was written for earlier wording of CHK-2. A replacement, CASE-11, is proposed next to it.
>
> ### Left out
>
> `cart.test.ts > rounds totals` is flaky. 3 cases and 9 tests were dropped, reasons listed.

## 8. Marking a rule checked

For a rule Evidry cannot confirm, try it yourself, then add one line in the rules file:

```markdown
## CHK-2: The confirmation email lists every item
Status: approved
Checked: Sam, 2026-11-27
```

On the next run Evidry records what you checked. If the rule's wording, or its test or case, changes, the report says the check has lapsed.
