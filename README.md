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

## Reference implementation

`src/exec_practice/{store,governor}.cljc` is a minimal but real
implementation of the Core Contract above (pure cljc, no external deps):

- `exec-practice.store` — `Store` protocol + `MemStore`: engagements
  (with a per-engagement `spending-limit`), decisions, spending-approval
  events. A decision/spending-approval can only be recorded against a
  registered engagement (engagement provenance).
- `exec-practice.governor` — `ExecPracticeGovernor`: `assess` gates a
  proposal against the engagement env. Hard invariants force `:hold` (no
  engagement, direct-write instead of `:propose`, or a spending-approval
  over the engagement's `spending-limit` below `:high` safety-class); a
  spending-approval over the limit always requires `:high`+ safety-class
  and thus `:human-approval` — it can never be auto-approved;
  low-confidence proposals also escalate.

```bash
clojure -M:test   # 8 tests, 13 assertions, green
```

This repo's own `blueprint.edn` currently declares `:itonami.blueprint/maturity
:blueprint`, not `:implemented`: `store`/`governor` are real, but there is no
compiled `langgraph-clj` StateGraph, Advisor protocol, or audit ledger wired
around them yet, so the actor cannot yet run an end-to-end proposal ->
governor -> commit/hold cycle. Any `:implemented`-tier listing for this repo
in [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
predates that correction and should be treated as stale until the missing
StateGraph/Advisor/ledger layer is built.

## License

AGPL-3.0-or-later.
