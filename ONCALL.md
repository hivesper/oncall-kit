<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# ONCALL.md — standing on-call policy (Vesper · team-web · #oncall)

<!-- Copy to your repo root. Every {{...}} is filled during oncall-setup
     Phase 2 (Interview). Mined suggestions may appear as comments; a human
     sets every policy value. This file is policy: changes go by PR.
     Filled 2026-10-16 from the Claude Tag oncall setup run in
     #oncall-playground, with answers from john
     (https://vesper-global.slack.com/archives/C0C9Q1W13JA/p1792137690434749).
     Every value still in {{...}} is marked "unknown — fill in": nobody has
     set it yet. Review of this PR is the policy sign-off. -->

## What this covers

Vesper's production web stack as seen by team-web: the Vesper API
(`hivesper/vesper-api`), the web app (`hivesper/vesper-web-app`), the
marketplace API and app (`hivesper/vesper-marketplace-api`,
`hivesper/vesper-marketplace-app`), plus the workers and mobile app whose
errors land in the same Sentry org. The current focus is Sentry issue
reports and alerts that the Sentry bot posts to #oncall (C0C65AGAYUE).
#oncall-playground (C0C9Q1W13JA) is for trying things out. There is no
rotation and no paging tool: whoever is in #oncall picks things up.

Incident severity levels, incident command, and stakeholder communications
remain your team's existing incident-management process — this kit plugs
into that process and does not replace it: the Confluence Incident
Management Process,
https://vesper-global.atlassian.net/wiki/spaces/DD/pages/1134821377
(possibly outdated — the team's own words in the #oncall thread win over
it). Vesper has no severity levels; a **major incident** is: an API
unreachable, a user-facing app not responding, or the tool unusably slow.
PMs own all internal and external communications. Claude never runs an
incident and never communicates to stakeholders beyond this channel.

## Incident records

An incident exists when a human declares one: anyone in #oncall can
declare an incident (john, 2026-10-16), in a thread in #oncall
{{exact convention, e.g. "start a thread beginning 🔴 INC:" — unknown,
fill in; must match the `incidents` binding in STACK.md}}. This replaces
the Confluence-era rule that a PM declares in #incident-management.
Claude may propose that something deserves an incident; it never
declares one itself.

## Incident lifecycle

Four states. `active` and `stale` are the open pair — stale means "this
looks abandoned: fixed-but-never-closed, or genuinely dropped," a
nobody's-watching alarm, not a closure. `mitigated` (fix in, still
watching) and `resolved` (done) are the closed pair; keep them distinct.
An archived or deleted incident channel/thread means resolved regardless
of what its frozen text says — nobody works an incident in a place that
no longer exists. Claude never changes a state (rule 2); it applies these
rules when deciding what to *report*:

- **Stale takes three legs, never silence alone:** (1) the record/thread
  quiet ≥ {{24h}} (unknown — fill in); (2) nothing in the alert channels
  naming it in the last hour; (3) its specific symptom absent from current
  data — and leg 3 only counts if the relevant run/window actually
  *completed* ("not failing yet" ≠ "passing"). If your checks can't see
  the incident's actual symptom, don't call it stale — a healthy proxy
  covers only the path it measures ("checkouts are flowing" proves the
  checkout path, not the whole payment provider). Every "looks stale"
  nudge carries its one-line evidence ("thread quiet 26h; merges flowing;
  no related alerts").
- **Promotions climb a ladder; demotions don't.** Suggesting a stale or
  closed incident is live again requires evidence **newer than the
  verdict it would overturn** — and your own earlier posts, or re-reading
  an unchanged status line, never count as activity (or your sweep keeps
  every zombie alive forever). Treating an incident as *quieter* may rest
  on older evidence: a wrong "quieter" is cheap to fix; a wrong
  resurrection is the failure mode this ladder exists to prevent.
- **No verdict is a verdict.** "Mitigated?" with a question mark,
  "duplicate of INC-…", an unparseable status line: none of these changes
  what you report. Absent evidence never moves state. And someone
  *asking* "any update?" is a question, not an update — don't let it
  reset the quiet clock.
- **Zombie sweep:** the morning sitrep lists every open incident with no
  update in {{24h}}, and the weekly handoff lists those quiet {{72h}} —
  "record still says active, thread silent since yesterday: close it or
  update it." (Both windows unknown — fill in. No morning sitrep or
  weekly handoff is scheduled yet; see Handoff and sitreps.)

## Paging policy

Deterministic alerts page humans directly — always. Claude is never in
the detection path. These criteria serve two purposes: they are the
values Claude uses when **proposing** deterministic rules (starter rules
for new services, tuning proposals after incidents), and they define
page-severity for **Claude's own findings** during triage (a correlation
discovery, an unalerted symptom) — with every page/no-page decision
logged per CLAUDE.md rule 3a. Written in plain English on purpose —
judgment clauses like "not during a known deploy window" are allowed.

**Vesper today:** there is no paging tool and no rotation (john,
2026-10-16). Sentry posts issue alerts to #oncall; nobody gets paged.
Claude never pages. No paging thresholds have been set.

**New services** get conservative starter rules before launch — proposed
by Claude by analogy to the closest existing service, installed by a
human — never a monitoring routine. Coarse rules that cannot die
silently beat clever monitoring that can.

| Signal | Page when | Sustain | Exempt when |
|---|---|---|---|
| {{e.g. pipeline error rate}} | {{> N%}} | {{M min}} | {{known deploy window; see Deploy windows}} |
| {{…}} | {{…}} | {{…}} | {{…}} |

<!-- Unknown — fill in. No paging thresholds set (no pager in use). The
     "major item" trigger below (5+ Sentry events in 24h) is the team's
     proactive-fix threshold, not a paging threshold. -->

Anything observed that does **not** trip these criteria: log to `lessons.md`
for the morning sitrep instead (CLAUDE.md rule 3).

## Severity norms

No severity levels at Vesper. What the kit uses instead:

- **Page** (any hour): none — no paging tool. The closest tier is a
  **major item**: matches the major-incident definition above (an API
  unreachable, a user-facing app not responding, the tool unusably slow),
  or a Sentry issue with 5+ events in 24h. Major items get the proactive
  fix rule below.
- **Business-hours ping**: {{definition}} (unknown — fill in)
- **Morning log line**: everything else

## Deploy windows

How to check whether a deploy is in progress: {{the `deploys` capability
binding, a channel, a calendar — name it}} (unknown — fill in; STACK.md
notes #deployment-announcements as an unverified candidate). Check
deploy times first for any vesper-api 500 burst: each vesper-api deploy
can briefly 500 while the route cache rebuilds (see lessons.md).

## Routing tree

<!-- Drafted by Mine phase from CODEOWNERS × who-actually-responded;
     conflicts resolved in Interview. Group handles must be real.
     Set by john, 2026-10-16. Sentry team slugs read from
     GET /api/0/organizations/vespertool/teams/ and /projects/. -->

Routing here means **who a Sentry issue is assigned to** (by Sentry
project), not who gets @-mentioned.

| Failure class | Owner | Notes |
|---|---|---|
| worker (Sentry project `worker`) | Sentry team `team-data` | the data team; Sentry team created by john 2026-10-16 |
| web-app, web-api | Sentry team `team-web` (display name "team-webapp") | Team Web |
| marketplace-api, marketplace-app | Sentry team `team-actnlearn` | Team Web Act & Learn |
| vesper-mobile-app | Sentry team `team-decide` | Team Web Decide |
| any other Sentry project | leave unassigned | |
| Anything company-blocking | {{@escalation}} | escalate immediately — unknown, fill in |

Claude mentions an owner only when this tree names one for the diagnosed
class (CLAUDE.md rule 12). This tree names no Slack handles, so Claude
@-mentions nobody on its own; the team's Slack group is @team-web
(S08TKNC8JG1), used only when a person asks.

**Escalation timeout:** a page-severity finding with no human
acknowledgment within {{N minutes}} escalates to {{next handle up the
ladder}}. An acknowledgment is an explicit affirmative from a person
("ack", "on it", or {{designated reaction}}) — bot posts, alert traffic,
and passive emoji don't count. One escalation only — never a loop.
**Terminal step** if the escalation also goes unacked: {{repeat-page via
the pager's escalation policy / post to #wider-channel / accepted
"unattended until morning" posture — the team chose deliberately}}.
(Timeout, ladder, reaction and terminal step: unknown — fill in.)

The ladder has more states than "paged" and "resolved" — each gets a
`paging-log.md` line:

- **Ack-then-silence:** acked, then nothing for {{30 min}} while the page
  condition persists → one (and only one) in-thread nudge naming the
  still-firing signal. Never re-page an acked incident on your own.
- **Re-fire after ack:** the signal cleared and tripped again → a *new*
  paging decision with a new log entry; the old ack doesn't cover a new
  firing.
- **Page unconfirmed:** the page call returned success but the pager
  shows no incident within {{2 min}} → treat as page-failed and take the
  fallback path, noting both in the log.
- **Two page-severity findings at once:** two pages, two entries — unless
  correlation says one cause, in which case one page naming both symptoms
  (triage step 3b).

**Fallback alerting:** if a page-severity finding cannot page (no pager
bound, or the page call fails), Claude instead {{@-mentions @escalation
in this channel / posts to #always-watched-channel / holds for the
morning log}} (unknown — fill in; no pager is bound, so this is the only
path). Chosen in the Interview; never improvised mid-incident.
Known limitation the team accepted: Slack @-mentions don't penetrate
Do-Not-Disturb — an @-mention fallback is business-hours-grade coverage.

**Out-of-band path (Slack itself down):** {{human-owned note: where the
team coordinates when Slack is unavailable — e.g., the pager's own
incident notes + a video-call bridge. Claude can't help here; this line
exists so the team decides this before an outage, not during one.}}
(Unknown — fill in. For prod DB/API disaster recovery, see Confluence
"In case of emergency":
https://vesper-global.atlassian.net/wiki/spaces/DD/pages/1198063624.)

## Fix workflow and the proactive fix rule

The team's fix workflow (john, 2026-10-16): check Sentry → assign team
(or user) → convert into a Jira bug ticket → create a branch prefixed
with the Jira ID → resolve the issue (coding) → open a PR → create a
staging via ScaSta (`/scasta`, vesper plugin) → QA / replicate (is it
solved?) → move the ticket to the code review lane in Jira → wait for
human QA → merge to prod.

**Proactive fix rule** (john, confirmed 2026-10-16). For a **major item**
in #oncall — matching the major-incident definition above, or a Sentry
issue with 5+ events in 24h — Claude starts unasked and does:

1. assign the issue in Sentry per the routing tree;
2. create the Jira bug;
3. create a branch prefixed with the Jira ID;
4. open a **draft** PR with the fix.

A ScaSta staging is created only once someone in the thread says go.
Moving to the code review lane, human QA and the merge stay with people.
Claude never marks a Sentry issue resolved and never merges.

For everything else, Claude does write steps only when a person in the
thread asks.

## Postmortems

The Incident Owner (the dev team lead) writes the incident report, with
root cause and solution, in the Confluence DD Incidents folder
(https://vesper-global.atlassian.net/wiki/spaces/DD/folder/1935900695)
using the template
https://vesper-global.atlassian.net/wiki/spaces/Marketplac/pages/1630502952.
The Incident Manager (any PM) holds the post-incident review and decides
on an internal vs external postmortem. Claude may draft; a person
publishes.

## Alert-rule proposals

Format: proposals are paste-ready {{monitor JSON / Terraform / PromQL /
UI steps}} for Sentry alert rules (format unknown — fill in).
Install mode: **human installs** (default — Claude drafts, a human
pastes; kit stays fully read-only toward alerting). The alert-editor
extension is not opted in.

## Read-only guarantee

This agent never changes the state of any monitored system. Its only
outputs are: messages in the channel; entries in the repo's log files
(`lessons.md`, `paging-log.md`, `eval/shadow-log.md`); updates to the
bound weather report page (only if opted in above); proposed diffs as
PRs; and (where a pager is bound) pages. There is no allowlist
and nothing to configure in this section — it exists so every reader of
this policy knows the contract. All mitigation is performed by humans.

**Team exception (john, confirmed 2026-10-16):** the proactive fix rule
above lets Claude, for major items only, assign a Sentry issue, create a
Jira bug, push a branch and open a draft PR. These are the only writes;
nothing is restarted, rolled back, silenced, resolved, staged without a
go, or merged. This deviates from CLAUDE.md rule 1 as written — the
reviewers of this file own that decision.

## Status on demand — and the optional weather report

Default: no standing status service. When someone asks for status, answer
from the open incident records plus these health signals: {{e.g.
merge-queue depth, deploy lag on critical branches, error rates}}
(unknown — fill in; today the only signal source is Sentry's unresolved
issues by frequency, see STACK.md). Fresh-reader rules apply (CLAUDE.md
rule 10).

Opt-in: the `weather` skill (routine 6 in `templates/routines.md`) — an
event-gated standing report. Not opted in (default).

## Handoff and sitreps

Weekly handoff cadence: none scheduled — handoff on request (john,
2026-10-16) · Posted to: the #oncall thread where it was asked
Morning sitrep: off (default) — no routine set up. A daily
unanswered-alert review was offered 2026-10-16 and left unscheduled.
