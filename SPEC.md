# Perdura — Business Continuity Management Platform
## Product specification v2 (commercial)

> **How Claude Code should use this file.**
>
> This is the authoritative source for the build. Read it fully before writing code. Reference screenshots may be pasted into the session per feature — use them for *layout structure and information density only*. Never copy visual identity, iconography or wording from them. Every string in this product is written fresh.
>
> Start each session with: *"Read SPEC.md and PROGRESS.md. Work only on the next unstarted phase. Open one PR when done."*
>
> Before scaffolding anything, summarise the architecture and data model back to the user and list every ambiguity you find. Wait for confirmation.

---

## 1. Product

Perdura is a multi-tenant SaaS platform for business continuity management, aligned to ISO 22301. It replaces the spreadsheet-and-Word-document approach most organisations use with a connected data model, so an impact analysis flows into recovery objectives, objectives flow into plans, plans flow into exercises and incidents, and all of it produces audit evidence automatically.

**Delivery:** one codebase, three surfaces.
- Responsive web application (primary)
- Installable PWA — desktop and mobile, with offline access to approved plans
- Microsoft Store listing (PWA packaging), for distribution and credibility

**Positioning:** deterministic and auditable. No AI anywhere in the product. Every derived number traces to a rule a user can read and an auditor can verify. In a compliance product this is a feature — buyers must be able to explain how a recovery objective was arrived at.

### Non-negotiable constraints

| Constraint | Requirement |
|---|---|
| Development environment | Built entirely through Claude Code cloud sessions. No local machine. All verification via CI and preview deployments. |
| Infrastructure cost | Must run on free tiers whose terms permit commercial use |
| AI / LLM | None. No provider SDKs, no API keys, no stubs, no "coming soon" affordances. |
| Offline | Approved plans and contact lists readable with no network |
| Mobile | Every responder-facing screen fully usable one-handed on a phone |

---

## 2. Architecture

### Stack

- **Framework:** Next.js 15+, App Router, TypeScript strict mode
- **Hosting:** Cloudflare Workers via `@opennextjs/cloudflare` (free tier permits commercial use; Vercel's Hobby tier does not)
- **Database:** Neon serverless Postgres, free tier, with the HTTP driver
- **ORM:** Drizzle ORM + drizzle-kit
- **Tenancy:** Postgres Row-Level Security, plus an application-layer scoping rule (§4)
- **Auth:** own email/password implementation with `@node-rs/argon2` or `bcryptjs`, signed HTTP-only session cookies, sessions in Postgres. Microsoft Entra ID and Microsoft Account OIDC added in Phase 9.
- **Styling:** Tailwind CSS, custom design tokens
- **Components:** hand-written in `components/ui/`. No component library.
- **Icons:** `lucide-react`, individually imported
- **Validation:** `zod` at every boundary
- **Forms:** `react-hook-form`
- **Dates:** `date-fns`
- **Charts:** hand-written SVG. No charting library.
- **PDF:** `/print/*` routes with `@media print` and `window.print()`. No headless browser.
- **ZIP:** `fflate`
- **Email:** Resend free tier behind a transport interface, with a console transport for development
- **Offline:** Workbox-free custom service worker + IndexedDB via `idb`
- **Tests:** `vitest` for derivation logic and tenancy isolation

### CI/CD — build this in Phase 1, not later

Without it you are building a UI you cannot see.

- GitHub Actions on every PR: typecheck, lint, unit tests, build
- Cloudflare preview deployment per PR, with the URL posted as a PR comment
- Neon database branch per preview, seeded automatically
- Production deploys on merge to `main`
- Never merge a PR whose preview you have not opened on your phone

---

## 3. Design system

**Identity.** Perdura. Latin *perdurare* — to endure. The tone is calm, precise, institutional. This is software people open on the worst day of their quarter.

**Palette**

```
--ink            #0F172A   headings, primary text
--ink-muted      #475569   secondary text
--surface        #FFFFFF   cards
--canvas         #F8FAFC   page background
--line           #E2E8F0   borders
--brand          #0F766E   primary actions, active nav
--brand-soft     #CCFBF1   selected states
--tier-0         #BE123C   mission critical
--tier-1         #EA580C   critical
--tier-2         #CA8A04   important
--tier-3         #0891B2   standard
--ok             #059669
--warn           #D97706
--danger         #E11D48
--drill          #7C3AED   drill / test mode — must be visually unmistakable
```

Full dark-mode token set required. Persist the preference per user.

**Type:** Inter, self-hosted via `next/font`, weights 400/500/600 only. Tabular numerals for all metrics (`font-variant-numeric: tabular-nums`) — RTO tables must align.

**Logo:** an SVG keystone glyph plus the wordmark, generated in-repo. Never a third-party mark.

**Layout:** collapsible left navigation on desktop, bottom tab bar on mobile. Content max-width 1280px. 4px spacing scale. Cards with 1px borders and no shadows except on overlays.

**Interaction quality — this is what "high end" means in practice**
- Command palette on `⌘K` / `Ctrl+K`: jump to any process, plan, risk, incident; run actions
- Optimistic updates on every mutation, with rollback and a toast on failure
- Skeleton loaders, never spinners
- Every empty state explains what the object is, why it matters, and offers the action that creates one
- Every destructive action confirms and is undoable for 10 seconds
- Full keyboard navigation; visible focus rings
- WCAG 2.2 AA: contrast, labels, roles, reduced-motion respect
- No layout shift on data load

**Voice.** Formal, plain, short. British spelling. No exclamation marks, no "Oops", no "Awesome". Errors state what failed and what to do. All copy original.

---

## 4. Tenancy, roles, security

### Isolation — two independent layers

**Layer 1, Postgres RLS.** Every tenant table has `org_id NOT NULL` and an RLS policy `USING (org_id = current_setting('app.org_id')::uuid)`. Every request opens its transaction with `SET LOCAL app.org_id`. Enable `FORCE ROW LEVEL SECURITY` so the table owner is not exempt.

**Layer 2, application.** All queries live in `lib/db/queries/*.ts`. Every function takes `orgId` first and includes it in the WHERE clause. Nothing outside that directory imports Drizzle.

`orgId` is read from the session server-side and is **never** accepted from a body, param or header. A vitest suite seeds two organisations and asserts every exported query returns empty for the wrong tenant, with RLS on and again with RLS off.

### Roles

| Role | Scope |
|---|---|
| Owner | Everything; billing, deletion, ownership transfer |
| Admin | Everything except deleting the organisation |
| Planner | Create and edit all program content. Cannot approve plans. |
| Process Owner | Edit BIAs for assigned processes only |
| Responder | View plans naming them; acknowledge notifications |
| Auditor | Read-only everywhere, including the audit log |

`can(role, action, resource)` in `lib/auth/permissions.ts`, enforced server-side in every action and route handler. Hide the UI too, but never rely on hiding.

### Security baseline

- Argon2id or bcrypt cost ≥12; never store plaintext
- Sessions: HTTP-only, Secure, SameSite=Lax, 30-day sliding expiry, revocable, listed in settings with device and last-seen
- Rate limiting on login, password reset, invite acceptance and the acknowledgement endpoint
- Email verification on signup; password reset by single-use expiring token
- TOTP two-factor, optional per user, enforceable per organisation
- CSP, HSTS, `X-Content-Type-Options`, `Referrer-Policy` via middleware
- Every input validated with zod before it reaches the database
- Full audit log, append-only, no UI delete path
- Data export (JSON + CSV) and account deletion self-service — a DPDP and GDPR requirement, not a nice-to-have

---

## 5. Data model

Drizzle schema in `lib/db/schema.ts`. UUID primary keys. `orgId`, `createdAt`, `updatedAt` on every tenant table. Soft-delete (`deletedAt`) on processes, plans, risks and people; hard-delete elsewhere.

```
organisations       id, name, slug, industry, sizeBand, country, timezone,
                    logoUrl, brandColour, planTier, trialEndsAt,
                    subscriptionStatus, createdAt
users               id, email(unique, citext), passwordHash, name, avatarUrl,
                    emailVerifiedAt, totpSecret, theme, createdAt
memberships         id, orgId, userId, role, status(invited|active|suspended),
                    invitedByUserId, invitedAt, acceptedAt
sessions            id, userId, orgId, userAgent, ip, lastSeenAt, expiresAt
invitations         id, orgId, email, role, token, expiresAt, acceptedAt

departments         id, orgId, name, description, headPersonId, parentId
people              id, orgId, name, email, phone, altPhone, departmentId,
                    jobTitle, isEmergencyContact, escalationOrder, linkedUserId
locations           id, orgId, name, address, type(office|plant|dc|remote)

processes           id, orgId, name, ref, description, departmentId,
                    ownerPersonId, locationId, status(active|retired),
                    tier, rtoMinutes, rpoMinutes, mtpdMinutes,
                    criticalityScore, minimumServiceLevel, peakPeriods,
                    biaCompletedAt, biaReviewDueAt
assets              id, orgId, name, type(application|vendor|equipment|data|
                    facility|people|utility), description, criticality,
                    vendorName, vendorContact, contractRef, slaSummary,
                    recoveryNotes, singlePointOfFailure(bool)
process_assets      processId, assetId, dependencyType(critical|supporting),
                    notes
process_dependencies upstreamProcessId, downstreamProcessId

bia_assessments     id, orgId, processId, assessedByUserId, assessedAt,
                    method(manual|template), templateKey, notes,
                    status(draft|complete), supersededById
bia_scores          id, biaId, category, horizon, score, justification

risks               id, orgId, ref, title, description, category,
                    ownerPersonId, likelihood, impact, inherentScore,
                    treatment, controls, residualLikelihood, residualImpact,
                    residualScore, status, reviewDueAt, lastReviewedAt
process_risks       processId, riskId
asset_risks         assetId, riskId

plans               id, orgId, title, ref, type, scopeDescription,
                    processId, departmentId, status, version,
                    ownerPersonId, approvedByUserId, approvedAt,
                    reviewDueAt, archivedAt
plan_blocks         id, planId, type, position, title, content(jsonb),
                    config(jsonb)
plan_versions       id, planId, version, snapshot(jsonb), dataAsOf,
                    approvedByUserId, approvedAt, changeSummary

exercises           id, orgId, title, type, objective, scenarioKey,
                    scenario(text), scheduledAt, durationMinutes, status,
                    planId, facilitatorUserId, participantIds(jsonb),
                    completedAt, effectivenessRating
exercise_injects    id, exerciseId, position, offsetMinutes, title, content,
                    expectedResponse, deliveredAt
exercise_log        id, exerciseId, at, authorUserId, type, note
findings            id, orgId, ref, source(exercise|incident|review|audit),
                    exerciseId, incidentId, title, description, severity,
                    ownerPersonId, dueAt, status, closedAt, closureEvidence

incidents           id, orgId, ref, title, description, severity(1-4),
                    status, category, declaredByUserId, declaredAt,
                    activatedPlanId, commanderPersonId, resolvedAt,
                    closedAt, isDrill, impactSummary
incident_tasks      id, incidentId, position, description, ownerPersonId,
                    status, dueAt, completedAt, notes
incident_timeline   id, incidentId, at, authorUserId, type, body
notifications       id, orgId, incidentId, personId, channel, isDrill,
                    subject, body, token, sentAt, deliveredAt,
                    acknowledgedAt, ackStatus, ackNote

frameworks          id, key(iso22301|dora|nis2), name, version, description
controls            id, frameworkId, clauseRef, title, intent, evidenceRule,
                    weight
control_status      id, orgId, controlId, state, score, computedAt,
                    evidenceSummary(jsonb), manualNote, overrideState,
                    overrideByUserId
readiness_snapshots id, orgId, frameworkId, score, capturedAt, breakdown(jsonb)

documents           id, orgId, title, type(policy|procedure|evidence|other),
                    linkedType, linkedId, url, notes, uploadedByUserId
audit_log           id, orgId, userId, at, action, entityType, entityId,
                    summary, before(jsonb), after(jsonb), ip, userAgent
api_keys            id, orgId, name, prefix, hash, lastUsedAt, revokedAt,
                    createdByUserId
```

**Copyright note on framework seed data.** ISO 22301 clause text is copyrighted, and so is much of the guidance around DORA and NIS2 implementation. Seed **clause references and your own paraphrased titles and intent statements only**. A row reads `8.4.2 — Plan content and structure` with an intent line you wrote. Never paste standard text into a seed file or into the UI.

---

## 6. Business Impact Analysis

The analytical core. Pure functions in `lib/bia/derive.ts`, no React and no database imports, exhaustively unit-tested.

### Input matrix — fixed 4 × 5

Categories: `financial`, `operational`, `legal`, `reputational`
Horizons: `h1` (1h), `h4` (4h), `h24` (24h), `h72` (72h), `h168` (1 week)
Scale: 0 none, 1 negligible, 2 minor, 3 moderate, 4 major, 5 severe

The matrix is fixed and not configurable. Comparable tiers across processes are what make the readiness score meaningful. Do not build a matrix editor.

Each cell may carry an optional one-line justification. Justifications appear in the exported BIA report and are what an auditor reads.

### Derivations

```ts
horizonMinutes = { h1: 60, h4: 240, h24: 1440, h72: 4320, h168: 10080 }

worst(h) = max(financial[h], operational[h], legal[h], reputational[h])
```

**MTPD** — first horizon where `worst(h) >= 4`. If none, `null` ("beyond 1 week").

**RTO** — the horizon immediately before the MTPD horizon.
- MTPD at `h1` → RTO 30 minutes
- MTPD `null` → RTO 10080 minutes

**RPO**
```ts
climb = max(operational.h4, financial.h4) - max(operational.h1, financial.h1)

climb >= 3 -> 15    climb == 2 -> 60
climb == 1 -> 240   climb <= 0 -> 1440

RPO = min(RPO, RTO)
```

**Criticality score, 0–100**
```ts
weights = { h1: 5, h4: 4, h24: 3, h72: 2, h168: 1 }     // sum 15
raw     = Σ ( worst(h) × weights[h] )                    // max 75
score   = round(raw / 75 × 100)
```

**Tier**
```
>= 75  Tier 0 — Mission critical
>= 50  Tier 1 — Critical
>= 25  Tier 2 — Important
<  25  Tier 3 — Standard
```

### Behaviour

- Derived values update live in the browser as cells change. Same pure function client and server.
- The editor shows a small impact-curve sparkline per category and a combined worst-case curve
- Below the results, a generated plain-English rationale: *"Legal impact becomes major at 72 hours, so disruption is tolerable for no more than three days and recovery must complete within 24 hours."*
- On save, write objectives back to the process and create an immutable superseded record of the previous assessment
- BIAs carry a 12-month review date; overdue BIAs surface on the dashboard and reduce the readiness score
- Durations always rendered human-readably

---

## 7. Risk

5 × 5. `inherentScore = likelihood × impact` (1–25); `residualScore` likewise after controls.

Bands: 1–4 Low, 5–9 Medium, 10–14 High, 15–25 Critical.

**Heat map:** CSS-grid 5 × 5, likelihood vertical, impact horizontal, cell counts, click-to-filter. A toggle switches between inherent and residual view — seeing the two side by side is the point of the exercise.

Risks link to processes and to assets. A process detail page shows its aggregate risk exposure; an asset flagged `singlePointOfFailure` with a Critical linked risk is escalated on the dashboard.

Risk register supports bulk CSV import — a real adoption blocker if missing, because every prospect already has a register in Excel.

---

## 8. Modules

### 8.1 Dashboard

Not a vanity screen. Answers: *what is not ready, and what is overdue?*

- Resilience posture: readiness score per framework, with 90-day trend from `readiness_snapshots`
- Tier 0/1 processes with no approved plan — named, linked
- Plans past review date, BIAs past review date
- Open findings past due
- Risks in the Critical band with no treatment recorded
- Days since last exercise
- Recent activity feed from the audit log

Every item deep-links to the fix.

### 8.2 Onboarding

A guided first-run wizard, then a persistent checklist reading live data:

1. Organisation profile
2. Departments and people
3. Business processes
4. First BIA
5. Risk register
6. First plan
7. Submit and approve
8. Export PDF

Progress computed from queries, never a stored flag. Offer sample-data preload at signup — a fully populated demo organisation the user can explore and then delete.

### 8.3 Organisation

Departments (hierarchical), people, locations, processes, assets. Process list sortable by tier with inline RTO/RPO/owner/BIA status.

**Dependency map:** an SVG graph of process → asset → vendor and process → process, with single points of failure highlighted. Hand-drawn layout, no D3.

People flagged as emergency contacts flow automatically into call trees and wallet cards, resolved live at render and export time.

### 8.4 Plan builder

Structured blocks only, never freeform documents.

| Block | Contents |
|---|---|
| Purpose & scope | Objective, covered processes, exclusions |
| Activation criteria | Triggers, thresholds, decision authority |
| Roles & responsibilities | Named people against named roles |
| Call tree | Ordered escalation tiers, live-resolved |
| Recovery procedure | Ordered steps: owner, target duration, description, dependencies |
| Resources required | Linked assets, alternates, fallback arrangements |
| Communications | Audience, channel, template, timing, approver |
| Contact list | Live query by department or emergency flag |
| Dependencies | Upstream and downstream processes |
| Appendix | Constrained free text |

Drag to reorder using native HTML5 drag events plus keyboard-accessible move buttons. No drag library.

**Lifecycle**
1. **Draft** — editable
2. **In review** — locked, approver notified, reviewer comments captured
3. **Approved** — Owner/Admin only. Serialises an immutable snapshot with all live blocks resolved, stamps `dataAsOf`, increments version, sets a 12-month review date, writes the audit entry.
4. **Archived**

Approved plans cannot be edited. Editing forks a new draft at version *n+1*. Every prior version stays retrievable, with a version-comparison view showing what changed between approvals — auditors ask for exactly this.

### 8.5 Exercises

Scenario generation from templates plus the organisation's own data (§9.3). Six skeletons: ransomware, data-centre outage, key-supplier failure, extended power loss, loss of premises, mass staff unavailability.

Facilitator console: running clock, injects surfacing at their offsets, decision/observation log with timestamps, participant attendance, findings capture. Post-exercise report as branded PDF. Completion updates readiness and the exercise-recency metric.

### 8.6 Incidents

Declare with severity 1–4, optional linked plan. Linked plan's recovery steps are **copied** into `incident_tasks` — copied, not referenced, so the plan cannot change under a live incident.

Command centre: task checklist with owners and completion, SITREP composer, chronological timeline, impact summary, linked notifications with a live acknowledgement rate.

**Notifications**
- Email via Resend, behind a transport interface. Every send recorded with delivery state, visible in an Outbox — silent failure is unacceptable in this product.
- One-tap acknowledgement at `/ack/[token]`, no login: *I'm safe*, *Available*, *Unavailable*, plus optional note
- Drill mode: `[TEST]` prefix, `--drill` purple treatment throughout, excluded from real statistics
- Post-incident review captures findings into the same tracker as exercises

### 8.7 Compliance

ISO 22301, DORA and NIS2. Each control carries a named `evidenceRule` evaluated against live data:

```
processes_have_bia               ≥80% of active processes have a complete, current BIA
critical_processes_have_plan     every Tier 0/1 process covered by an approved plan
plans_approved_current           ≥1 approved plan; none past review date
plans_reviewed_annually          all approved plans reviewed within 12 months
risk_register_populated          ≥5 risks with treatment recorded
risks_reviewed                   no risk past its review date
exercise_within_12m              ≥1 completed exercise in the last 12 months
exercise_findings_closed         no exercise finding past due
incident_process_defined         ≥1 incident or drill recorded
post_incident_review             every closed incident has a review
emergency_contacts_defined       ≥1 emergency contact; call tree populated
assets_mapped                    every Tier 0/1 process has ≥1 critical asset
spof_identified                  single points of failure flagged and risk-assessed
dependency_map_complete          no Tier 0/1 process without upstream dependencies recorded
audit_trail_active               audit entries within 90 days
roles_assigned                   every process has an owner
```

Weighted score per framework, 0–100. States: `evidence_present`, `partial`, `gap`.

Every gap renders a specific action from the query result — *"Three Tier 1 processes have no approved plan: Payroll, Order intake, Client onboarding"* — with names and links, never a generic hint.

**Statuses read "evidence present", never "compliant".** Display a standing note that Perdura surfaces evidence and gaps and that certification is the auditor's determination. This is both honest and legally protective.

Snapshot the score nightly into `readiness_snapshots` for trend charts.

### 8.8 Exports and evidence

- Branded plan PDF via print route: cover page, version, approval block, `data as of` stamp, page numbers, table of contents
- Wallet card, credit-card size, activation triggers plus escalation contacts
- BIA report per process with justifications
- Risk register, process register, asset register, findings register as CSV
- Post-exercise and post-incident reports
- **Audit evidence pack:** `fflate` ZIP with an `index.html` manifest, every approved plan, all registers, the compliance report per framework, exercise records, and an audit-log extract for the period

### 8.9 Offline / PWA

The feature that justifies the architecture. A continuity tool unreachable during an outage is worthless.

- Web app manifest, maskable icons, standalone display, theme colour
- Custom service worker precaching the app shell
- On login, cache the user's approved plans, their call trees and contact lists into IndexedDB
- Offline mode is **read-only** and clearly badged, showing the cache timestamp
- Acknowledgement submissions queue offline via Background Sync and flush on reconnect
- Installable from browser, and packaged for the Microsoft Store

### 8.10 Read-only API — Phase 10

`GET /api/v1/processes`, `/risks`, `/plans`, `/openapi`. Bearer key `pdk_<id>_<secret>`, shown once, stored hashed. `{ "data": [...] }` envelopes, `{ "error": "..." }` on failure, 120 req/min per key. No write endpoints, ever.

---

## 9. Deterministic generators (the AI replacement)

Every generator produces an editable draft, labelled as a starting point, never presented as authoritative.

### 9.1 BIA templates

`lib/bia/presets.ts` — ten complete 4×5 matrices with impact curves reflecting typical operational shapes: customer-facing transactional, support desk, payroll and finance, regulatory reporting, manufacturing, IT platform, HR administration, sales, logistics, generic. Picker shows a curve preview and one-line description. Every cell editable after applying.

### 9.2 Recovery step outlines

`lib/plans/generate.ts` reads tier, RTO, RPO, top three linked risks and linked asset types, then assembles ordered steps from a phase-keyed template library:

```
Detection & assessment → Escalation & activation → Containment
→ Recovery execution → Verification → Communication → Stand-down & review
```

Conditional rules: Tier 0/1 adds a 15-minute escalation step and crisis communications; RPO under 60 minutes adds data-loss assessment and transaction replay; a Critical linked risk adds a containment step naming it; vendor assets add supplier notification. Target durations apportioned across steps as fractions of RTO. Process owner pre-filled as default step owner.

### 9.3 Exercise scenarios

`lib/exercises/generate.ts` merges a skeleton with real records — highest-tier process, its owner, top linked risk, a vendor asset, the affected department — into parameterised injects at T+0, T+15, T+45, T+90, T+150. Output reads as a specific scenario about the customer's own operation, assembled entirely from templates plus their data.

---

## 10. Commercial layer

- **Plans:** Trial (14 days, full features, watermarked exports), Starter, Growth, Business. Seat-based for Owner/Admin/Planner. **Process Owners, Responders and Auditors are free and unlimited** — this is the pricing model, and it must be enforced in seat counting.
- **Entitlements:** `lib/billing/entitlements.ts` maps plan tier to limits (processes, plans, API access, SSO). Checked server-side. Exceeding a limit blocks creation with a clear upgrade path, never a silent failure.
- **Trial expiry:** read-only grace mode. Everything remains exportable. Never hold a customer's continuity plans hostage — say so in the UI.
- **Legal pages required before any listing:** Terms of Service, Privacy Policy, Data Processing Addendum, subprocessor list, security overview, support policy. Privacy Policy URL is a hard requirement for Microsoft Store submission.
- **Phase 9 auth additions:** Microsoft Entra ID and Microsoft Account OIDC — prerequisites for any future Microsoft Marketplace transactable listing.

---

## 11. Seed data

`npm run seed` builds **Northgate Logistics**, a mid-size distribution firm: 6 departments, 24 people (8 emergency contacts), 3 locations, 16 processes across all tiers with complete BIAs and justifications, 28 assets with dependencies and two single points of failure, 22 risks across all bands with treatments, 4 plans (one approved at v3, one approved at v1, one in review, one draft), 2 completed exercises with injects, logs and 5 findings (2 open, 1 overdue), 1 closed incident with timeline and acknowledgements, 1 drill, and computed compliance across all three frameworks.

The result must be genuinely mixed — around 70% readiness, with real gaps — so the compliance and dashboard screens have something honest to display. Print demo credentials at the end of the run.

---

## 12. Build phases

One phase per PR. Each fully working end to end before the next begins. Update `PROGRESS.md` every phase.

| # | Phase | Done when |
|---|---|---|
| 1 | Foundation | Next.js on Cloudflare, Neon connected, RLS enabled, auth complete, design tokens, UI component set, app shell, CI green, preview URL live and openable on a phone |
| 2 | Organisation | Departments, people, locations, processes, assets, dependency links. Tenancy tests passing both layers. |
| 3 | BIA | Derivation module fully tested, matrix editor with live results, templates, write-back, review dates, BIA report export |
| 4 | Risk | Register, dual heat map, process and asset linking, treatments, CSV import |
| 5 | Plans | Block builder, all block types, live resolution, review and approval, versioning, comparison view, print routes, wallet card |
| 6 | Exercises | Generator, facilitator console, findings tracker |
| 7 | Incidents | Declaration, task checklist, timeline, notifications, acknowledgements, drill mode, post-incident review |
| 8 | Compliance & evidence | Framework seeding, evidence rules, weighted scores, gap actions, snapshots, evidence pack |
| 9 | PWA, offline, commercial | Manifest, service worker, offline plans, background sync, entitlements, trial states, legal pages, Entra ID and MSA SSO |
| 10 | Store & API | Store packaging and submission assets, read-only API, final accessibility and performance audit |

---

## 13. Standing instructions

- **Server-side enforcement always.** `requireSession()` then `can()` in every action and route handler.
- **`orgId` from session only.** Reading it from a request is a tenancy break — stop and flag it.
- **zod at every boundary.**
- **Pure derivation logic**, no React or database imports, always tested.
- **No new dependency without asking** — state the package, its size, and why nothing present will do.
- **Ask when the spec is silent.** Never invent a feature to fill a gap; surface the gap.
- **All copy original.** Never reproduce wording, labels or structure from any existing product's interface or documentation, including from reference screenshots.
- **No AI, no stubs for AI.**
- **Mobile-first.** Every responder screen must work one-handed on a phone. Verify on the preview URL before opening the PR.
- **Report your uncertainty.** At the end of each phase, list what you are least confident is correct — unenforced permissions, unvalidated inputs, unhandled failure paths. Do not report everything as done.
