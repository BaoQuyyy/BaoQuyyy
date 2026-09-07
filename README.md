## Huy Phan

I build the failure paths in n8n — the branches that only matter when something
breaks. Vietnam, UTC+7.

Most automation work is judged on a screenshot of a canvas. These three repos are
the opposite: source open, each brought up and verified by **one command** on
your own machine, with **84 assertions** between them. The suites drive the
failures rather than describing them.

### [wf1-resilient-ingest](https://github.com/BaoQuyyy/wf1-resilient-ingest)

A webhook ingest pipeline where no event is ever silently lost and no event is
ever processed twice.

Idempotency is enforced by a `PRIMARY KEY`, not by a check-then-insert pair — the
latter looks equivalent and loses the race under concurrency. `429` and `5xx`
back off exponentially with jitter and obey `Retry-After`; `4xx` fails fast to a
dead-letter queue, because the same payload will be rejected identically forever
and retrying it only delays the moment a human finds out. A replay workflow
drains that queue — releasing the idempotency claim first, since a queue you
cannot drain is just a slower way of losing data.

### [wf2-llm-cost-controlled](https://github.com/BaoQuyyy/wf2-llm-cost-controlled)

LLM grading that treats the model as what it is: an unreliable, metered
dependency.

Well-formed JSON is not correct JSON. Scores out of range or a missing criterion
are caught before anything reaches a user, and the specific errors are fed back
as a correction turn rather than the same prompt being retried and hoped at.
Every call is metered **including the rejected ones** — a rejected response costs
exactly as much as an accepted one, and a rising invalid-output count is the
earliest signal of prompt or model drift.

### [wf3-bidirectional-sync](https://github.com/BaoQuyyy/wf3-bidirectional-sync)

Two-way sync without infinite loops and without quietly destroying edits.

No origin markers — they leak the moment a write is retried, and then the echo
loop returns intermittently, which is the worst way for it to return. Instead the
pipeline keeps a merge base and diffs both sides against it, so echo suppression
falls out of the data model. Conflicts resolve per field, because record-level
last-write-wins discards edits that never conflicted. Reconciliation reports
drift and never silently repairs it: the drifted value may be the only surviving
copy of a real edit.

---

Also written down: [seven n8n traps that fail
silently](https://github.com/BaoQuyyy/wf1-resilient-ingest/blob/main/docs/NOTES.md),
including the one where an imported workflow activates, reports success, and
never binds its webhook because `triggerCount` is `0`.

**Stack:** n8n · JavaScript · Python · SQL · REST and webhook integration

**Honest about the gaps:** no client references yet — which is exactly why the
code above is open rather than described in adjectives.

📧 phanlebaohuy1989@gmail.com
