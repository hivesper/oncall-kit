<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# lessons.md — the living incident log (Vesper · team-web)

<!-- Seeded by oncall-setup Phase 1 from your incident history; appended by
     Claude after every incident without asking (CLAUDE.md rule 8); pruned
     by humans whenever. Newest first, below the status banner. -->

Status: seeded 2026-10-16 at oncall setup (team-web, #oncall) with one
imported fact; no incidents logged yet. Watching: vesper-api deploy 500s.

<!-- The banner is the resume point: anyone (or any session) picking this
     file up reads the first five lines and knows where things stand.
     Claude updates it whenever it appends an entry. -->

## Entry formats — three kinds

**Incident entry** (the default — a resolved incident or a died hypothesis):

```
## {{date}} · {{one-line title}} · #{{failure-class-tag}} · {{INC-nnn or thread link}}
- What happened: {{one fresh-reader sentence}}
- Root cause: {{mechanism, with evidence link}}
- Fix: {{what resolved it, who decided}}
- Gotcha / rule: {{the reusable lesson — the reason this file exists}}
```

**Investigation entry** (a long or inconclusive investigation worth its
trail — so no future session re-walks the same dead ends):

```
## {{date}} · Investigation: {{what}} · #{{tag}} · {{thread link}}
| step | result |
|---|---|
| {{check run}} | {{✅/❌ + link}} |
- Checked out on paper: {{everything that SHOULD make it work, each
  verified ✅ — the signature move before declaring a mystery}}
- Could not verify: {{what, why, and the exact one-line command the owner
  should run — hand the human a grep, not a mystery}}
- Rabbit holes eliminated: {{hypothesis → the evidence that killed it}}
- Confidence: {{~N%, and what the remaining % is, named}}
```

**GOTCHA one-liner** (a tool or environment surprise, not an incident):

```
- GOTCHA ({{date}}): {{the surprise, and the rule it implies — one line}}
```

## Graduation

When a tag accumulates **3 entries with the same mechanism**, or an entry
stops being a story and becomes a *procedure* (a checklist someone could
follow cold), promote it into that class's reference file by PR
(CLAUDE.md rule 9) and replace the entries with one pointer line. Record
the promotion itself as an entry — the reference file's provenance points
back here.

---

## 2026-10-07 · vesper-api deploys briefly 500 during route-cache rebuild · #deploy-rollout · https://vesper-global.slack.com/archives/C0C0C17L12Q/p1791366507115869?thread_ts=1791366118.175699&cid=C0C0C17L12Q
- What happened: right after a vesper-api deploy, requests briefly
  returned 500, enough to tip the low-traffic data-api 5xx-rate alarm
  over 1% in #monitoring-infra-warning. (seen 1×, unverified — imported
  at setup 2026-10-16)
- Root cause: a race while the route cache rebuilds on deploy:
  `require bootstrap/cache/routes-v7.php` fails with "Failed to open
  stream" (Sentry WEB-API-24A, WORKER-2KE).
- Fix: none recorded; the errors stop once the cache is rebuilt.
- Gotcha / rule: for a short vesper-api 500 burst, check deploy times
  first before investigating further.
