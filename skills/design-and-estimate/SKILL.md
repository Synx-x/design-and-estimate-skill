---
name: design-and-estimate
description: >
  Generates a Design & Estimate (D&E) proposal document for a new feature in
  whatever repo the session is running in. Use this skill whenever the user
  invokes /design-and-estimate or /Design&Estimate, asks for a D&E, wants to
  estimate a feature, needs a proposal for a new screen or report, or says
  something like "draft me a proposal for X". The skill discovers the repo's own
  architecture before writing, so it produces grounded file paths and layer
  breakdowns in the repo's own vocabulary, not generic boilerplate. It always
  asks first whether the output should be a plain-English summary or a detailed
  technical breakdown. Always invoke this skill before attempting to write a D&E
  manually.
author: Synx-x
license: MIT
compatibility: "Any repo and any agent host that can read files and run shell commands. Uses AskUserQuestion for the style prompt when the host provides it."
---

# Design & Estimate Skill

Produce a grounded D&E document for a feature in the current repo. The output is
a .txt file written to the scratchpad and opened immediately.

This skill is stack-agnostic. Every project name, layer name, file path, and
convention in the finished document must come from the repo the session is
running in, discovered in Step 2. Do not carry a name from another repo, from a
previous conversation, or from an example in this file.

## Step 1: Gather requirements

Requirements come from one of three sources (in priority order):

1. **Ticket or doc URL in args or conversation**: fetch it. For a ClickUp doc,
   `mcp__claude_ai_ClickUp__clickup_get_document_pages` takes document_id (first
   ID after `/v/dc/`) and page_id (second ID). Read every page. For Jira, Azure
   DevOps, Linear, or GitHub, use whichever connector or CLI the session has.
   Confirm the tool exists in the session's tool list before relying on it.
2. **Free-text description in args**: use as-is.
3. **Nothing provided**: ask one focused question: "What feature needs a D&E?"

Do not proceed to Step 2 until requirements are clear enough to name at least
two concrete deliverables.

## Step 2: Discover the repo's architecture

Establish where the session actually is before researching. Read the working
directory, then the nearest solution or manifest file (`*.sln`, `package.json`,
`pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`) to learn the real project
and module names. A monorepo or a multi-project solution may hold sibling
folders outside the one the session started in, so check the parent directory
when a referenced project is not where expected.

Then spawn a code-exploration subagent and hand it every question at once, not
one at a time. Use `scaffold:code-explore` when available. If it comes back
off-topic, garbled, or reading like cached instructions, do not fall back to
`Explore` and do not retry the same subagent a third time. Some harnesses
block `Explore` outright and will just burn another round trip. After one
retry still fails, gather the same answers directly with Read/Glob/Grep/Bash
instead of continuing to retry a subagent.

- The layers or projects present, and what each owns: frontend, API, domain or
  application logic, data access, shared contracts.
- Where a new UI screen belongs: the folder convention, the routing mechanism,
  and two nearby screens to mirror for naming.
- Where new endpoints belong, and the existing controller, module, or service
  they would extend.
- The data entities or tables relevant to this feature, and the layer each one
  is defined in.
- How authorization is expressed: attribute, decorator, policy, middleware,
  permission constant, or database role table. Name the existing one covering
  this area.
- How navigation entries get registered: config file, database rows, a route
  manifest, or hardcoded markup.
- The existing pattern for the specific thing being built: export, upload,
  background job, report, notification, integration.
- The UI component library and styling approach in use.
- How schema changes ship: ORM migrations, hand-written SQL, embedded scripts,
  or a separate DDL repo.
- The test setup, and whether a new feature normally arrives with tests.

Where the repo has no answer for one of these, that absence is itself a finding.
Record it in the document rather than describing a layer that does not exist.

After the discovery pass, check for `references/known-stacks.md` next to this
file. Apply only the entry matching the repo just identified. Those notes are
shortcuts for repos already mapped, they carry their own staleness warning, and
they do not override what the subagent found.

Without this research the D&E will carry wrong file paths. Do not skip it.

## Step 3: Ask which output style (HARD requirement)

Do not pick the style yourself. Before writing a single line of the document,
call `AskUserQuestion` with:

- header: `Output style`
- question: `Summary or detailed breakdown for this D&E?`
- option 1: **Summary**: phase-grouped plain-English list with inline ETAs. No
  file paths, no class names. For clients and non-technical stakeholders.
- option 2: **Detailed**: full structure breakdown by layer with exact file
  paths, endpoints, interfaces, data changes, nav and authorization,
  infrastructure, and a phase table. For developers and internal planning.

This holds even when the user's request sounds obviously technical, and even
when a previous D&E in the same conversation used one style. Ask every time.

The only exception: the user already named the style in their own words
("give me the short version", "full technical breakdown"). Then use it and say
which one was taken.

## Step 4: Write the D&E

Fill every placeholder with facts from Steps 1 and 2. Do not invent file paths.
If something is genuinely unknown, write `[CONFIRM: <the unknown>]` in that spot
rather than guessing. Phrase it as the missing item, not as a question.

### Template A: SUMMARY

Plain technical English. Rules that apply to this template only:

- Describe what a person can do on a screen, not what the screen is built from.
  "A page where an admin uploads invoices and sees what matched" beats a
  sentence naming the view engine and the grid widget.
- No file paths, project names, class names, interface names, table names, or
  component names. If a sentence needs one to make sense, the sentence belongs
  in Template B.
- One sentence per deliverable. Under 20 words each.
- ETA inline under each deliverable, AI-accelerated first, manual in brackets.
- Ranges are fine and usually honest: `1 - 2 hr (manual: 3 - 4 days)`.

```
[FEATURE NAME]

Phase 1:
Front-End setup:
[What the user can do on screen one.]
ETA: [X] hr (manual: [Y] days)
[What the user can do on screen two.]
ETA: [X] hr (manual: [Y] days)

Phase 2:
Backend setup:
[Environment or service setup, one line.]
[Services behind the front-end functionality, one line.]
ETA: [X] hr (manual: [Y] days)

Total: [X] hours (manual: [Y] days)
```

Skip a phase heading that has no work in it. Add a Phase 3 only if the work
genuinely does not fit under front-end and back-end. Rename the phase headings
when the repo has no front-end or back-end split, for example a library, a CLI,
or a data pipeline.

### Template B: DETAILED

Section headings carry the repo's real project names in brackets. A repo with
one deployable drops the layer split and uses a single structure section.

```
[FEATURE NAME] D&E
==================
Feature: [one-line description]
Date: [today's date]


1. STRUCTURE
============

[Two-sentence summary of where this fits. Which projects or modules, new area
versus existing area, reuses existing patterns versus introduces new ones.]

FRONTEND ([real project or package name])
-----------------------------------------
[List each new screen. For each: file path, filters and controls, components
used, client-side script path. Mirror the naming convention of the nearby
screens found in Step 2.]

[Describe any existing controller or route handler that gets new actions, with
the exact file path.]

BACKEND ([real project or service name])
----------------------------------------
[List new endpoints: HTTP verb, route, what it returns. One bullet per endpoint.]

[Name the interfaces or modules that get new signatures, with exact file paths.]

[Describe the implementation location alongside existing equivalents.]

[Name any new service, repository, or query methods needed, and which data joins
they perform.]

DATA
----
[New tables, columns, or documents. How the schema change ships in this repo.
Whether existing entities suffice.]

NAV AND AUTHORIZATION
---------------------
[Name the existing permission, policy, or role that covers this, with its real
value. State whether a new one is needed. State how the navigation entry gets
registered in this repo and how many artifacts that takes.]


2. INFRASTRUCTURE SETUP
=======================

Already exists, no changes needed:
  [Bullet list: running services, existing registrations, existing entities,
  existing patterns this feature rides on. Use the repo's real names.]

Gets added:
  [Bullet list: new files, new signatures, new schema artifacts. Be specific.]

Gets changed:
  [Bullet list: existing files that get modified rather than created. Shared
  interfaces, registration files, layouts, manifests.]


3. ETA
======

[X] working days manual, [Y] AI-accelerated, [N] phases. Ranges are preferred
over single figures where the work is genuinely uncertain (`2-3 days`,
`0.5-1 hr`):

+---+-------------------------------+---------+----------------+-----------------------------------+
| # | Phase                         | Manual  | AI-Accelerated | Deliverable                       |
+---+-------------------------------+---------+----------------+-----------------------------------+
| 1 | [phase name]                  | [X] day | [Y] hr         | [what ships]                      |
+---+-------------------------------+---------+----------------+-----------------------------------+
...
+---+-------------------------------+---------+----------------+-----------------------------------+
|   | TOTAL                         | [X] days| ~[Y] day       |                                   |
+---+-------------------------------+---------+----------------+-----------------------------------+

[One short paragraph: what AI acceleration covers, in this repo's terms
(scaffolding matched to sibling modules, boilerplate, schema artifacts, data
binding). What manual time concentrates on (testing with real data, complex UI
interaction, end-to-end validation).]


UNRESOLVED DECISIONS
====================
[Numbered list of genuine blockers, each a declarative either/or statement plus
one line on what it changes. "Ageing origin date: due date versus period end.
Changes the query for accounts on a non-standard billing cycle." Not a
question, and not first-person. Skip the section when there are none.]
```

Detailed-only sections (`UNRESOLVED DECISIONS`, data, nav and authorization,
infrastructure) do not appear in Template A. A summary with unresolved blockers
lists them in chat instead, not in the document.

---

## ETA sizing

Size the phases from the actual work, then let the total fall out of the sum.
Do not back-solve phases from a target total.

Sanity band, set by the user on 2026-10-20 as the realistic timing for this
work. Size each deliverable line on its own:

- A single screen or one service or endpoint group in an established area of a
  mature repo: 1-4 hours AI-accelerated, 0.25-1 working day manual.
- A screen plus its backend together: roughly 3-6 hours AI-accelerated,
  roughly 1 working day manual.
- A large item (a full module such as quoting, or a new external integration
  such as a two-way calendar sync): 4-5 hours AI-accelerated, about 1 working
  day manual on the line itself.

Calibration case: a mobile PWA migration plus To-Do, calendar filters, contacts,
quotes and an Outlook integration, about 20 deliverable lines, landed at 40-60
hours AI-accelerated and 12.5 working days (2.5 weeks) manual. The manual
figure sits close to 2x the accelerated hours, so do not inflate manual days to
a multiple of the hours. Per-line manual figures below 0.5 day are acceptable.

A heavier case applies when the work adds real infrastructure (a new
container, a queue, a background worker): an invoice-matching feature with a
processing sidecar and a job queue was sized at 15-20 days manual and 4-8 hours
accelerated across eight phases. Treat it as an upper marker, not a default.

Adjust the band for the repo. A greenfield or thinly-conventioned codebase costs
more per screen than one with a dominant pattern to copy, because there is no
sibling module to mirror. That adjustment is judgement, not a measured figure.

Phase count follows the work breakdown, typically 4-6 for a screen, up to 8-10
when infrastructure is involved. Use ranges where the work is genuinely
uncertain. State the total in both units, accelerated and manual.

## Writing past a prose guard

Some sessions run Write/Edit hooks that block prose files for style,
grounding, or chat-residue violations. If the D&E write gets blocked, fix
exactly what the error names rather than rewriting the whole document from
scratch:

- Avoid semicolons in prose. Pick one name per entity and keep that same name
  for it everywhere in the document, rather than switching terms partway
  through.
- Cite factual claims inline with a path or URL, not in a trailing sources
  list. If a citation in `C:\Users\...` form is not recognized, retry with
  forward slashes (`C:/Users/...`) instead. Some grounding checks only match
  Unix-style absolute paths.
- Do not narrate the session: no first person, no chat-address phrasing, no
  open questions phrased as questions. State the fact, or the declarative
  either/or, instead. Template B's UNRESOLVED DECISIONS section already gives
  this rule. It applies to anything else written into the file too, including
  a Summary's incidental mentions of open items.

## Step 5: Export and open

Write the document to the session scratchpad directory:

```
[scratchpad]/[FeatureName]_DE.txt
```

Then open it with the platform's opener: `start "" "[path]"` on Windows, `open
"[path]"` on macOS, `xdg-open "[path]"` on Linux. Also deliver it with
`SendUserFile`, so the content reaches the user without depending on a GUI
opener working.

Tell the user the file path, which style was written, and note any
`[CONFIRM: ...]` placeholders that need answers before the D&E is final.

If the summary was chosen, offer the detailed version as a follow-up in one
line. Do not write both unasked.
