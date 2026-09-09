Reference material for structure and information density only. Not for copying visual styling, colours, icons, terminology or wording. All strings in Perdura are original.

---

## How to use this file

Each screenshot below has a one-line description of what it shows, followed by a
blank line for behaviour notes. Add notes under the relevant entry — what should
happen on click, what the empty state does, what is derived vs. entered, what is
missing from the reference and needs designing fresh.

Screenshots live in `reference/screens/`, numbered in product-flow order:
onboarding → dashboard → getting started → organisation → risk → plans →
exercises → incidents → compliance → settings.

---

## Onboarding

**01-onboarding-create-organisation.png** — First-run organisation form: name, industry, company size, country, and a ticked "preload a sample workspace" option, with a trial note above the primary action.


**02-onboarding-size-and-country-pickers.png** — The same form with both the company-size band list and the searchable country list open at once.


## Dashboard

**03-dashboard-overview.png** — Dashboard top: four metric cards (seats used, plan, process count, BIAs submitted), a trial-ended banner, and a tick-list of programme milestones with completed items struck through.


**04-dashboard-organisation-switcher.png** — Organisation switcher dropdown listing the user's organisations with role labels and a "new organisation" action.


**05-dashboard-account-menu.png** — Account menu from the avatar: name and email header, profile and settings links, sign out.


**06-dashboard-tier-distribution.png** — Dashboard scrolled: process counts per criticality tier as four cards, plus the frameworks in scope as chips.


## Getting started

**07-getting-started-progress.png** — Guided path page: circular percent-complete ring, "N of 6 steps" summary, then step cards each with a state badge, an explanation and a direct action button.


**08-getting-started-steps-scrolled.png** — Steps 3–6 of the same list: risk scoring, plan build and approve, exercise, compliance and evidence export.


**09-getting-started-footer-actions.png** — Bottom of the guided path: the final step card plus a footer panel with secondary actions (documentation, run a drill).


## Organisation

**10-org-processes-list.png** — Organisation section with tab strip (processes, departments, people, assets); process table showing department, derived tier badge, RTO and BIA status per row.


**11-org-new-process-dialog.png** — Create-process modal: name, description, department and owner selects, single primary action.


**12-org-new-department-dialog.png** — Create-department modal: name and description only.


**13-org-departments-list.png** — Flat department list with a per-row delete affordance and a count above the table.


**14-org-people-list.png** — People list: name with an emergency-contact badge where set, and a secondary line of job title · department · email.


**15-org-add-person-dialog.png** — Add-person modal: name, email, phone, job title, department select (open), and an emergency-contact checkbox that explains where the flag surfaces.


**16-org-assets-list.png** — Asset list: name with a type badge and a short description line per row.


**17-org-new-asset-dialog.png** — Create-asset modal: name, type select (open, showing the type enum), vendor, and a recovery option / workaround field.


## Risk

**18-risk-register-heatmap.png** — Risk register: 5×5 heat map with axis labels, a band legend and a plotted-count line, above a table of risks with linked process, numeric score with band, and status.


**19-risk-new-threat-library.png** — New-risk modal with the threat library dropdown open, showing a scrollable list of pre-defined threat types.


**20-risk-new-process-picker.png** — New-risk modal with the "process at risk" select open, listing the organisation's processes.


**21-risk-new-likelihood-scale.png** — New-risk modal with the likelihood select open, showing the 1–5 scale, alongside impact and status selects.


**22-risk-edit-populated.png** — Edit-risk modal with all fields populated, including a free-text treatment plan.


## Plans

**23-plans-list-single-approved.png** — Plans list: plan name, type abbreviation, version and status badge, with a create action.


**24-plans-new-plan-type-picker.png** — New-plan modal with the plan-type select open, showing the plan taxonomy.


**25-plans-new-plan-linked-process.png** — New-plan modal showing title, type and an optional linked-process select.


**26-plan-editor-blocks.png** — Plan detail in draft: title with type and version badges, export and submit-for-review actions, then stacked block cards each with its own save and delete.


**27-plan-editor-unsaved-rows.png** — Same editor with per-block "unsaved" badges and expanded row editors: call-tree rows (name/role/phone), recovery steps (step/owner), communications (audience/channel/message).


**28-plan-editor-add-block-menu.png** — Add-block menu listing the available block types, including a live-resolved contact list and a constrained free-text block.


**29-plans-list-two-statuses.png** — Plans list with two rows at different lifecycle states and versions.


## Exercises

**30-exercise-new-dialog.png** — New-exercise modal: title, type select, scheduled date with an explicit format hint.


**31-exercise-new-type-picker.png** — New-exercise modal with the exercise-type select open.


**32-exercise-detail-scheduled.png** — Scheduled exercise: generated scenario paragraph, injects listed against +0/+15/+30/+45 offsets, a start action, and a findings panel with a severity select.


**33-exercise-in-progress-findings.png** — Exercise in progress: same scenario and injects, plus a decision/observation logger, an empty timeline, and findings and corrective-actions panels.


**34-exercise-log-entry-type.png** — Exercise log with the entry-type select open (decision vs. observation).


## Incidents

**35-incidents-empty-state.png** — Incidents empty state explaining what to do next, with a visually distinct destructive-styled declare action.


**36-incident-declare-severity.png** — Declare-incident modal with the severity select open, showing four labelled severity levels.


**37-incident-declare-plan-picker.png** — Declare-incident modal with the plan select open and a drill/test checkbox that states messages will be marked.


**38-incident-command-centre.png** — Live incident: severity and status badges, live checklist panel (empty because no plan was linked), SITREP composer, notify-responders panel explaining acknowledgement links, acknowledgement counter, and a timeline.


**39-incident-sitrep-posted.png** — The same page immediately after posting a SITREP, with a success toast and the entry appended to the timeline.


**40-incident-timeline-after-sitrep.png** — Post-toast steady state showing the SITREP entry in the timeline.


**41-incident-resolved.png** — Resolved incident: status badge changed, notify panel gone, and a post-incident report action in its place.


**42-incident-post-incident-report-print.png** — Generated post-incident report rendered as a print/PDF document: header line, org and severity metadata with declared/resolved timestamps, then timeline, checklist and acknowledgements sections.


## Compliance

**43-compliance-iso22301-controls.png** — Compliance page with a standing note that statuses say "evidence present", not "compliant"; per-framework card with a large readiness number and a clause-referenced control list with state badges.


**44-compliance-iso22301-score-working.png** — Same card scrolled to the bottom, showing the score arithmetic written out in full beneath the control list, and a gap row carrying a specific next action.


**45-compliance-dora-and-nis2.png** — Second and third framework cards, each with its own readiness number, article-referenced controls and per-row gap actions.


**46-compliance-nis2-score-working.png** — The third framework card in full, including its score working.


## Settings

**47-settings-organisation.png** — Settings with a tab strip (organisation, members, billing, API, profile); organisation tab showing name, industry, size and country.


**48-settings-members-and-seats.png** — Members tab: statement of which roles consume paid seats, an invite-by-email row with a role select, and the current member list with role badges.


**49-settings-billing-plans.png** — Billing tab: current plan panel with days remaining, then a monthly/annual toggle and three priced plan cards with feature lists and a "most popular" marker.


**50-settings-api-gate.png** — API tab as an entitlement gate: explains which plan includes API access and links to upgrade rather than showing a disabled key list.


**51-settings-profile.png** — Profile tab: full name, email shown read-only with an explanation of why it cannot be edited.

