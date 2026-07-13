# cloud-itonami-isco-8342

Open Occupation Blueprint for **ISCO-08 8342**: Earthmoving and Related Plant Operators.

**Maturity: `:implemented`** — EarthmovingAdvisor ⊣ EarthmovingGovernor
as a langgraph StateGraph (`intake → advise → govern → decide →
commit/hold`, human-approval interrupt), modeled on
cloud-itonami-isco-4311's bookkeeping actor. 14 tests / 29 assertions
green. The governor never dispatches hardware — it only gates what
the survey/grade-checking robot below may execute.

The excavation HARD invariants — arithmetic and set membership, not a
scheduling choice:

1. **Cleared-depth ceiling** — the proposed excavation depth must not
   exceed the site's registered utility-cleared ceiling (digging past
   it is a strike risk, not a scheduling choice).
2. **Cleared-zone membership** — the proposed excavation zone must be
   a member of the site's registered cleared-zones set (unlocated
   ground is not fair game).

`:approve-unlocated-utility-excavation` and
`:approve-occupied-zone-entry` **always** escalate to human sign-off
regardless of confidence, per this repo's Trust Controls
(business-model.md).

This repository designs a forkable OSS business for an independent earthmoving plant operator: a survey and grade-checking robot performs site surveying and grade-verification under a governor-gated actor, so the operator keeps their own site and utility-clearance records instead of renting a closed heavy-equipment SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a survey and grade-checking robot performs site surveying and grade-verification alongside the operated plant equipment under an actor that proposes
actions and an independent **Earthmoving Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
operating near underground utilities, or in occupied work zones) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
site plan + utility-locate report + grading specification
        |
        v
Earthmoving Advisor -> Earthmoving Governor -> excavate/grade, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `8342`). Required capabilities:

- :robotics
- :telemetry
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
