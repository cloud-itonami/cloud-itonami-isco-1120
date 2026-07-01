# cloud-itonami-isco-1120

Open Occupation Blueprint for **ISCO-08 1120**: Managing Directors and Chief Executives.

This repository designs a forkable OSS business for an independent small-business executive (managing director/CEO of a single small enterprise): a site-monitoring robot performs walkthrough and compliance capture under a governor-gated actor, so the executive keeps their own decision and compliance records instead of renting a closed ERP/board management SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a site-monitoring robot performs walkthrough checks and compliance evidence capture across the operator's small-business sites under an actor that proposes
actions and an independent **Executive Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
decisions affecting worker safety, payroll, or regulatory filings) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
strategic plan + delegation scope + compliance requirement
        |
        v
Executive Advisor -> Executive Governor -> decide, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `1120`). Required capabilities:

- :robotics
- :identity
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
