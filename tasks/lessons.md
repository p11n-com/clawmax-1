
## 2026-07-09: Scope questions must be phrased in product terms
- Correction: user answered a scope question with "you aren't explaining properly".
- Pattern: I asked about "runtime selection granularity (per-agent vs global)" using architecture jargon (env vars, IDENTITY.md fields) before establishing what the feature looks like to the user.
- Rule for myself: when asking a scope/design question, first describe the current user experience and the two candidate future experiences in plain product language (what the user sees/clicks, what changes), grounded in the app's own UI concepts (e.g. "the BYOK dialog"), THEN name the technical mechanism. Use AskUserQuestion previews/mockups for UI-shape choices.

## Lesson: default to short answers (2026-08-03)

**Correction:** "too much info" — responses were too long (multi-section
summaries with tables, verification detail, and staged next steps).

**Pattern:** I front-loaded every finding I had instead of the decision the
user needed. Verification evidence and rationale are *available on request*,
not the default payload.

**Rule for myself:** Answer in <=5 lines unless asked to expand. Lead with the
result + the one decision needed. Put evidence behind an offer ("details if you
want them"), not in the message. Long-form only for: explicit "explain",
review findings the user asked to see, or a plan they must approve.

## Lesson: state the current version before asking the user to pick one (2026-08-13)

**Correction:** I asked "which base should the branch merge onto: rc33 or
origin/main HEAD?" and the user replied "whats the latest RC?" — the question
was unanswerable without a fact I already had and had not surfaced.

**Pattern:** I framed a choice using internal identifiers (commit SHAs, branch
names) without first reporting the state that makes the choice meaningful —
what the latest release actually is, how far behind the current deployment is,
and what lands in between.

**Rule for myself:** Before asking the user to choose between versions,
branches, or baselines, state in the same message: what the latest is, what
they are on now, the gap between them, and the cost of each option. If I have
to look something up to make the question answerable, look it up first.

## Lesson: write choice options in the user's language, not the system's (2026-08-13)

**Correction:** "I don't understand the options" — my AskUserQuestion described
a launchd job reconciling `deployment_env.CLAWMAX_IMAGE` in `state.json`.

**Pattern:** I named implementation internals (daemon labels, config keys, file
paths) as if they were the user's mental model. The user experiences this as
"the thing that keeps my app running", not as a plist and a JSON key.

**Rule for myself:** Options describe *what will happen to the user's system*
and *what it costs them* — "pause the helper that restarts the app; nothing on
disk changes; a reboot undoes it" — not the mechanism. Internals belong in the
technical detail below the question, if anywhere. If an option needs a config
key or daemon name to be understood, it is written wrong.
