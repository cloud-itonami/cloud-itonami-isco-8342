# cloud-itonami-isco-8342

Open Occupation Blueprint for **ISCO-08 8342**: Earthmoving and Related Plant Operators.

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
