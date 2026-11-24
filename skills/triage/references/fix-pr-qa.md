<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# Triage reference: QA on a draft fix PR

> Not a symptom class: load this after triage has produced a draft fix PR.
> It applies only where `ONCALL.md` records the team's standing ask for
> proactive fix PRs with staging QA; otherwise stop at the proposed diff
> (CLAUDE.md rule 1). The environment tool is bound as `staging` in
> `STACK.md`. Mined from the first runs, 2026-11-19 to 2026-11-20 (the
> Vesper team's staging tool is ScaSta); entries marked unverified are
> hypotheses.

## QA step

1. Create one staging per ticket after the draft PR exists. Label = the Jira
   key, lower-cased. Use the PR's branch for the repo it changes and the
   default branch for every other repo. Mark it agent-created and
   not keep-alive. Stagger creates a few minutes apart; a staging takes
   ~12–14 min to provision.
2. Add the staging URL to the PR body only once it is provisioned.
3. Run in-depth QA: browser automation on the web-app, plus direct API calls
   on the fixed path. Report what was run and what was seen, not just
   "QA passed".
4. Destroy the staging when QA, including the feedback loop below, is done.
   Then remove the staging line from the PR body. Always destroy; never
   leave it to expire.

## Feedback loop

For every issue found:

1. Take a screenshot of it.
2. Attach the screenshot to the Jira ticket with a feedback comment: what
   was done, what was expected, what was seen.
3. Post the same feedback as a PR comment, with the Claude Code footer.
4. Fix on the PR branch, push, redeploy the staging, QA again.

Loop until a round is clean. After 3 rounds without traction, stop and ask a
person. Never merge: review, QA sign-off and merge stay with people.

*(Provenance: policy set 2026-11-19. First loop test 2026-11-20, two PRs,
both clean in round 1, so the screenshot-to-ticket step is untested.)*

## "Ready to Deploy" label

When QA, including the feedback loop, finishes **clean**, add the
`Ready to Deploy` label to the fix PR. Clean means QA actually ran on the
staging and the latest round found no remaining issues.

- Use the exact label name that exists in that repo. If the repo lacks it,
  don't create it; tell the channel.
- No label if QA didn't run, is still pending, or issues remain.
- If a later push changes the PR, re-run QA. If QA is no longer clean, remove
  the label.
- The label is a signal, not a sign-off. Never merge or approve: review, QA
  sign-off and merge stay with people.

*(Provenance: requested by Jason Viljoen, 2026-11-23.)*

## "Low risk" label

When Claude opens or handles a fix PR, it assesses the risk of the change. If
the PR is low risk, add the `Low risk` label.

- Provisional criteria: a tiny, local diff (a null, guard or type fix, a
  one-line eager load, an extra match); no behaviour change on the happy
  path; no migrations; no auth, billing or data writes; no change to query
  semantics. When in doubt, no label.
- Use the exact label name that exists in that repo (`vesper-api` and
  `vesper-web-app` have it). If another repo lacks it, don't create it; tell
  the channel.
- Jason Viljoen will provide a more comprehensive checklist for "low risk"
  later. It replaces the provisional criteria above.
- The label is a signal only. It does not skip review, QA or the "Ready to
  Deploy" rule. Never merge or approve.

*(Provenance: requested by Jason Viljoen, 2026-11-24.)*

## Environment gotchas

- **Nightly data refresh blocks staging changes.** Our shared database
  snapshot refreshes 21:30–~01:00 Amsterdam (about 3h15–3h25, growing ~10 min
  a week). Creates and redeploys inside it fail after ~9 min with
  `FATAL: could not acquire global lock`. Create and redeploy only outside
  the window. If it hits anyway, wait and retry after 01:00 Amsterdam; do not
  escalate. Around 00:45–01:00 a brief `golden mysqld socket missing` is
  expected while the database restarts; if it persists, tell the tool's
  owner.
- **Missing integration keys cause 500s before your code runs.** Staging
  lacks the recommendation API key, so home-feed cards return 500; stub them
  in the browser tests. A third-party data endpoint can fail the same way
  when its credentials are absent: use a variant that skips it.
- **The web-app talks to `<label>.v2.vesperstaging.com/api/`** with
  `X-App-ID: web`, not to the API's own host.
- **No query profiler on staging.** N+1 fixes can't be checked there; rely on
  a CI query-count test, written at service level (an endpoint-level one
  broke CI for an unseen reason, *seen 1×, unverified*).
- **Admin panel (Nova): click only exact selectors.** A fallback click once
  hit Delete. Wait for list links to render before reading them.
- **Credentials stay out of everything.** Login values come from the
  environment; never put them in files, PRs, tickets or Slack. Environment
  variable names are fine to cite.

## Escalation

- Provisioning fails twice outside the refresh window: ask a person.
- Same finding after 3 fix rounds: ask a person (see feedback loop).
- Anything that looks like it needs a merge, a data change or a credential:
  a person's decision, not yours.
