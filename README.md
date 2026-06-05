# Georgina Palliative Coordination (PaTz Pilot Prototype)

> **Southlake Health × Ontario Health atHome × Georgina FHT**
> An interactive HTML prototype for a shared palliative care coordination platform, designed around the **PaTz model**, **Geisinger 7-day ACP gate**, **Kaiser dynamic risk stratification**, and **PHIPA circle-of-care** constraints.

---

## Table of Contents

1. [What this is](#1-what-this-is)
2. [Why it exists — the problem](#2-why-it-exists--the-problem)
3. [Design principles](#3-design-principles)
4. [How to run](#4-how-to-run)
5. [Roles & demo accounts](#5-roles--demo-accounts)
6. [Architecture overview](#6-architecture-overview)
7. [Page-by-page walkthrough](#7-page-by-page-walkthrough)
8. [Cross-cutting features](#8-cross-cutting-features)
9. [Data model](#9-data-model)
10. [Keyboard shortcuts](#10-keyboard-shortcuts)
11. [Suggested 90-second demo script](#11-suggested-90-second-demo-script)
12. [Known limitations](#12-known-limitations)
13. [Roadmap](#13-roadmap)

---

## 1. What this is

A **single-file HTML prototype** (`index.html`) that simulates a working palliative care coordination platform across four user roles:

- **Care Manager** (Maria, RN) — owns the panel, closes gaps, runs huddles
- **Primary Care Physician** (Dr. Lai) — signs ACPs and care-plan approvals
- **Family / Substitute Decision Maker** (Helen, John's daughter) — gentle, warm portal
- **Manager / Supervisor** (Dr. Patel) — aggregate dashboard, PHIPA audit, ROI

There is **no backend** — all data is in-memory JavaScript. The goal is to demonstrate **what the experience feels like** for each role, not to ship production code.

---

## 2. Why it exists — the problem

Palliative patients in Georgina (a rural area in York Region, Ontario) live across **siloed systems**: hospital EMRs, FHT EMRs, atHome (home-care) records, hospice systems, and ED notes. The result:

- Care managers run their panel out of **Excel + sticky notes**
- ED nurses **fax** discharge summaries to GPs that arrive days late
- Families don't know **who to call first** when the patient deteriorates at home
- ACPs get signed late or never (Geisinger's 7-day gate is missed)
- Managers have **no visibility** into panel-level coordination health

This prototype shows a **"replaces, not adds"** approach: every module explicitly displays what existing tool or workflow it eliminates.

---

## 3. Design principles

| Principle | How it's expressed |
|---|---|
| **Human-in-the-loop AI** | Every AI suggestion has Accept / Modify / Override + a "Why?" chip with 3 sources |
| **Policy → Pixel** | Geisinger gates, Kaiser stratification, PHIPA circle-of-care are visible UI elements, not hidden logic |
| **Replaces, not adds** | Each module has a `↻ Replaces ...` badge naming the legacy workflow it removes |
| **Tone-shifting** | Same data, two voices: clinical for staff, warm for family |
| **Auditable provenance** | Three-color status: 🟢 signed · 🟡 AI-inferred · 🔴 required |
| **Privacy by design** | n≥5 aggregation, identity hidden even from managers, audio auto-delete in 24h |

---

## 4. How to run

```bash
# No build step. No dependencies. Just open the file.
open index.html
# or double-click in your file manager
```

Tested in Chrome, Safari, Firefox, Edge (modern versions). Best viewed at **≥1280px width**.

---

## 5. Roles & demo accounts

On the login screen, click any demo chip — or type the credential into the input.

| Login | Role | Lands on | Why try this view |
|---|---|---|---|
| `e10482` | **Maria Okafor, RN** (Care Manager) | Worklist | The "main" daily-work view. Start here. |
| `dr.lai@southlake.ca` | **Dr. Karen Lai** (GP) | Care Plan (Rita) | See pending signature workflows |
| `e88312` | **A. Singh, RN** (ED nurse) | Loopback form | See the 5-field after-hours form |
| `helen.karras` | **Helen Karras** (SDM — daughter) | Family portal | See the family-facing voice |

> Once logged in, use the **role switcher** at the top of the page, or press **`Tab`** to cycle through all four roles. Press **`Esc`** to close any open drawer.

---

## 6. Architecture overview

```
┌─────────────────────────────────────────────────────────────┐
│  LOGIN SCREEN                                                │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  APP SHELL                                                   │
│  ┌──────────┬──────────────────────────────────────────┐   │
│  │ Sidebar  │  Topbar (role switcher + breadcrumb)     │   │
│  │ (role-   ├──────────────────────────────────────────┤   │
│  │  aware)  │                                          │   │
│  │          │  Active view (one of 11)                 │   │
│  │          │                                          │   │
│  └──────────┴──────────────────────────────────────────┘   │
│                                                              │
│  Floating: 📬 Mailbox FAB · 🌿 Signal banner                │
│  Modal: Patient 360° drawer · "Why?" popover · dialogs      │
└─────────────────────────────────────────────────────────────┘
```

**11 active views**, organized by role:

- **Worker views** (Care Manager, GP, ED): Worklist · Register · Care Plan · Intake · One-pager · Loopback · Huddle · Capacity · Team & Access
- **Family view**: Family Portal
- **Manager view**: Manager Dashboard

---

## 7. Page-by-page walkthrough

### 7.1 🏠 Worklist (`commandView`) — Care Manager's home

**Who:** Maria (Care Manager), Dr. Lai (GP)
**Purpose:** The prioritized daily action list. "What needs me, right now?"

#### Sections (top → bottom)

1. **🚨 Alert banner** — high-priority count + jump-to-register
2. **🌿 Panel-level 7-step closed care loop** — for each of the 7 PaTz steps (Identify → Register → Assign → Care plan → Handoff → Loopback → Review), shows how many patients on your panel have cleared it (e.g., `4/5`). Color codes:
   - 🟢 all done · 🟡 some watching · 🔴 some blocked
   - **Click any tile** to see per-patient detail
3. **4 KPI metrics** — panel size, today's actions, overdue count, ACP gate compliance %
4. **"Maria's next actions" worklist** — the main prioritized list
   - Tabs: All / Overdue / Event flags / Gaps / Stale
   - Filters: Today + overdue / This week / Overdue only
   - **Auto-surfaced "Derived" rows** (dashed amber border, ⚡ icon) — patients whose ESAS-r symptoms jumped ≥2 points in <7 days. Clicking opens the ESAS tab directly.
5. **Selected patient panel** (bottom-left) — recommended next step + risk explainer + care team mini-list
6. **Why these surfaced** (bottom-right) — the nightly rules engine criteria + activity feed

#### Key interactions

- **Click a row** → selects that patient for the panel below
- **Click the patient name** (dotted underline) → opens **Patient 360° drawer** (slide-in from right)
- **Click the risk pill** → opens **risk score breakdown popover** (Kaiser-style line-item composition)
- **Click "why?" chip** → opens **AI reasoning popover** with 3 sources + raw numbers
- **"Mark top action done"** → simulates closing an action

---

### 7.2 📋 Register (`registerView`) — Shared palliative roster

**Who:** All worker roles + Manager
**Purpose:** The single shared list of palliative patients in Georgina, keyed on OHIP.

#### What you can do

- **Search** by name, OHIP, action, owner, source
- **Filter** by risk (high/medium/low) or category (event/gap/stale)
- **Click any row** to select that patient (selection persists across views)
- **Bulk reassign high-risk → Maria** (one-click ownership transfer)
- **+ Add patient** → opens quick-intake dialog (full intake flow is at `intakeView`)
- **Export for huddle** → mock PDF export

#### Why it matters

Replaces 4 siloed EMR lists (hospital, FHT, atHome, hospice). PHIPA circle-of-care is enforced: each user only sees patients in their `visiblePatientIds`.

---

### 7.3 🩺 Care Plan (`carePlanView`) — The patient's full record

**Who:** Care manager, GP
**Purpose:** Everything about one patient, organized for action.

#### Header

- Name, age, OHIP, conditions
- **Risk pill** (clickable → score breakdown)
- **🎯 Geisinger gate strip** — visualizes 5 compliance milestones:
  - Day 0: Register ✓
  - Day 7: ACP signed (Geisinger's hard gate)
  - Day 14: SDM confirmed
  - Day 21: Med reconciliation
  - Day 30: Escalation plan signed
- Buttons: View one-pager · Open ConnectingOntario · Resolve top blocker

#### Tabs (6)

| Tab | Contents |
|---|---|
| **Care plan** | 8 locked fields with 🟢/🔴 provenance, edit-with-signature workflow |
| **ESAS-r symptoms** | 9-symptom Edmonton scale, with trend arrows (↑↓→) and `why?` chips on jumps |
| **SPICT history** | **Baseline vs Current diff** showing which indicators are new — instant deterioration visibility |
| **Contact log** | Vertical timeline of every interaction |
| **Voice note** | Tap-to-record (3s mock) → AI generates **3 versions**: 🏥 Clinical (EMR), ♡ Family (warm letter), ⚡ Brief (5-line handoff). Audio auto-deletes in 24h. |
| **Documents** | Linked PDFs (ACP, escalation plan, SPICT baseline, ED summary, one-pager) |

#### Right rail

- **Open tasks** (checkable)
- **Reference window** — live snapshot of DHDR (meds), acCDR (encounters), CHR (problem list) from ConnectingOntario
- **Peer playbooks** — anonymized "what worked for similar cases"
- **Care team** with phone numbers

#### Closed care loop visualization

Below the recommendation banner: 7 cards showing this patient's status through each PaTz step. Color-coded (green/amber/red), clickable.

---

### 7.4 ⚡ New Identification (`intakeView`) — 30-second palliative screening

**Who:** GP at FHT visit, ED nurse at discharge
**Purpose:** Adds a patient to the shared register in under 30 seconds.

#### 4-step flow

1. **Surprise Question** — "Would you be surprised if this patient died in 12 months?"
2. **Universal entry questions** — admissions, decline, comfort-focus
3. **Tool selection** — SPICT (default, with pre-checked EMR-extracted items) / HOMR / Other
4. **System suggests** — segment + care manager + auto-notifies them

Replaces ad-hoc, inconsistent palliative identification judgement.

---

### 7.5 📄 One-pager (`onePagerView`) — The travel document

**Who:** Anyone in circle of care
**Purpose:** The artifact that travels with the patient — same view for ED nurse, paramedic, SDM.

#### Contents

- Patient header + QR code (links to live version)
- **4 contact cards** (care manager + on-call are highlighted as "call FIRST before 911")
- **🚨 Escalation plan** with DNR line, step-by-step instructions for: breathlessness, pain, agitation, last resort
- Place-of-care preference + key conditions
- Footer with version + signing physician

#### What you can do

- 🖨 **Print** — uses print-specific CSS to strip nav/buttons
- **Generate wallet card** — mock PDF
- Replaces faxed care summaries that arrive too late.

---

### 7.6 📋 Loopbacks (`loopbackView`) — ED → community structured handback

**Who:** ED nurse (primary), care manager (review)
**Purpose:** When an ED visit happens to a registered palliative patient, the ED nurse fills in a **5-field structured form** instead of faxing a summary.

#### 5 fields

1. **F1 — Reason for contact** (free text, pre-filled from triage)
2. **F2 — Action taken** (free text)
3. **F3 — Outcome** (radio: home / plan worked / ED / admitted / other)
4. **F4 — Follow-up needed** (radio: no / next-day / urgent)
5. **F5 — Note to care manager** (free text — the most clinically valuable field)

Submit → care manager sees it in mailbox + worklist within minutes (not days).

---

### 7.7 👥 Huddle Prep (`huddleView`) — Weekly team meeting

**Who:** Care manager (chair), supervisor, GPs, nurses
**Purpose:** Auto-generated agenda for Thursday's 45-min huddle.

#### Sections

- **Auto-generated agenda** — pulled from register: new, deteriorating, gap-flagged, stale, deaths-since-last
- **Create huddle task** — patient + task type + priority + due + owner + notes
- **Decisions captured today** — running log
- **▶ Meeting mode** — full-screen card-by-card walkthrough during the meeting

Replaces whiteboard agenda rebuilt fresh each week.

---

### 7.8 🏨 Capacity (`capacityView`) — Resource directory

**Who:** Care manager
**Purpose:** Know where there's a bed, a nurse, a pharmacy kit — without making 12 phone calls.

#### Contents

15–25 entries (hospice / community nursing / pharmacy / equipment / family services), each with:
- Availability
- Owner contact
- **Freshness status**: 🟢 fresh (<7d confirmed) · 🟡 stale (>7d) · 🔴 blocked (>14d)
- Last month's usage stats

#### Actions

- **Suggest best option for selected patient** — based on patient's needs
- **Send Monday confirm-emails** — weekly batch verification
- **Confirm now** / **Flag issue** per entry

---

### 7.9 👤 Team & Access (`teamView`) — Provider onboarding

**Who:** Manager, care-manager lead
**Purpose:** Manage who has access to which patients, under PHIPA circle-of-care.

#### Contents

- **Onboarded providers** with credential status (ONE ID / ConnectingOntario) and permission chips
- **Audit log** — every access change is recorded
- **Reassign panel** — move routine tasks while keeping ACP signatures with original owner
- **Onboard provider** dialog

---

### 7.10 ♡ Family Portal (`familyPortalView`) — Helen's view

**Who:** SDM / family member (Helen, John's daughter)
**Purpose:** A warm, gentle, anxiety-reducing portal — *not* a clinical record viewer.

#### Sections

1. **Warm welcome card** — *"How is your father today?"*
2. **🌿 Take a quiet moment** — 90-second breathing pause, no setup
3. **💌 Letter from Maria** — personal note (not a clinical summary)
4. **Today's plan · John** — 4 cards: "what we're watching, what's coming up"
5. **How are *you*, Helen?** — 4-button mood check: 🌹 okay · 🌿 a bit tired · 💧 heavy · ⚠ need help now
6. **Comfort toolkit** — 4 step-by-step guides: breathlessness · pain · confusion · last days. **Written in plain language, no jargon.**
7. **Connect with the team** — message Maria, reschedule, alert on-call, share a moment
8. **From other families walking this path** — anonymous quotes

#### Voice & tone

Compare:
- Clinical: *"ESAS pain 5→7, dyspnea PRN protocol"*
- Family: *"His pain is down to about a 4 out of 10, which is much better than last week."*

Same data, different voice.

#### Bi-directional signals

When Helen taps "a bit tired" or opens the breathlessness toolkit 3x, **Maria's view shows a gentle signal banner** in the top-right — without exposing Helen's identity unnecessarily.

---

### 7.11 📊 Manager Dashboard (`managerView`)

**Who:** Dr. Patel (palliative supervisor)
**Purpose:** Aggregate panel health, PHIPA-audited, **never exposing individuals below n=5**.

#### Sections

1. **Hero stats** — 23 active panel · 78% ACP compliance · $847K FY ER savings
2. **🌿 Coordination health · 7-step grid** — fraction-done at each closed-loop step (clickable for drill)
3. **🔒 PHIPA audit stream** — last 7 days of access changes, signatures, AI edits, consent events
4. **🎯 Strategic pillars 2024–29** — Reduce ED visits / ACP compliance / Family confidence / Staff wellbeing, with evidence
5. **🌹 Team wellbeing** — anonymous, aggregated only when n ≥ 5
   - Wellness score · After-hours email % · Protected time honoured % · Anonymous "need support" signals
   - Identity hidden even from the manager (peer SOS pairings handle individual escalation)
6. **💰 ROI** — 63 ED visits diverted · $847K saved · 8 hospice referrals · 78% ACP within gate

Every tile is **clickable** for a drill-down dialog.

---

## 8. Cross-cutting features

### 📬 Mailbox (FAB, bottom-right)

Available in worker + GP views. Tabs: All · Loopbacks · Signatures · From family · Drafts · Sent. Unread items have a red left border.

### 🔍 Patient 360° drawer

Click any **patient name with dotted underline** to slide in a right-side drawer:
- Portrait + tagline
- At-a-glance facts
- Top blocker
- Recent timeline (color-coded: alert/warm/gentle)
- Cross-platform footprint (care plan / loopback / capacity / huddle)
- Next-step actions

### ❓ "Why?" reasoning popover

Click any purple `why?` chip on:
- AI recommendations → 3 sources + raw numbers
- Risk score pills → Kaiser-style line-item composition (e.g., +28 for ED visit, -6 for caregiver presence)
- Care-plan field statuses → why it's flagged unsigned/inferred

Every popover ends with **"Trust it"** or **"This doesn't fit"** — feedback loops back to the AI.

### 🔁 Role switcher (top center)

Switch to any of 4 roles. Press **`Tab`** to cycle. Preserves nothing between roles — gives a fresh perspective.

### 🌿 Bi-directional signal banner

Family actions surface gently in worker view (top-right). Dismissible, auto-fades.

### Toast notifications

Bottom-center, 2.6s auto-dismiss. Used for every meaningful action.

---

## 9. Data model

All state is in-memory JavaScript at the top of the `<script>` block.

### Core entities

```javascript
patients[]          // Array of patient objects (5 demo patients)
  ├─ id, name, ohip, age, sex, conditions
  ├─ segment, risk, category (event/gap/stale/check)
  ├─ action, actionDetail, due, owner, source
  ├─ riskScore, carePlanScore, sla, blocker, recommendation
  ├─ riskFactors[]
  ├─ team[]         // [role, name, phone]
  ├─ fields[]       // 8 care-plan fields with provenance
  ├─ esas[]         // 9 symptoms with current/prev/trend
  ├─ spictIndicators { baseline[], current[] }   // diff source
  ├─ tasks[]
  ├─ references { dhdr, accdr, chr }             // ConnectingOntario mocks
  ├─ loop[]         // 7 closed-care-loop steps with status
  ├─ timeline[], loopbacks[], crossBed[]
  └─ portrait, tagline

staffProfiles{}     // Login → user profile mapping
agenda[]            // Huddle items
capacity[]          // 15-25 capacity directory entries
providers[]         // Onboarded providers
mailboxItems[]      // Inbox
auditLog[]          // PHIPA audit trail
WHY_CLAIMS{}        // AI reasoning library, keyed by claim ID
RISK_BREAKDOWNS{}   // Kaiser stratification line items per patient
```

### State variables

```javascript
currentUser       // active profile object
currentRole       // 'worker' | 'gp' | 'family' | 'manager'
selectedId        // currently selected patient ID
activeFilter      // register filter chip
activeRange       // worklist date range
activeWLTab       // worklist tab
activeCPTab       // care plan tab
activeVoiceVersion // 'clinical' | 'family' | 'brief'
selectedLoopStep
mailboxFilter
```

---

## 10. Keyboard shortcuts

| Key | Action |
|---|---|
| `Tab` (outside inputs) | Cycle to next role |
| `Esc` | Close any open drawer/popover/dialog |

---

## 11. Suggested 90-second demo script

Use this when presenting to evaluators.

1. **(0:00–0:10) Login as Maria** — "This is the care manager's daily start. Notice the 7-step loop strip at top: 4 of 5 patients have cleared the care-plan step; one is blocked."
2. **(0:10–0:25) Click John's risk pill** — "Risk score 92 isn't a black box. It's a sum of 7 rules — ED visit +28, escalation unsigned +20, caregiver presence offsets -6."
3. **(0:25–0:40) Open John's Care Plan → SPICT tab** — "Baseline vs current diff. The amber row 'persistent dyspnea' is new since baseline. That's the trigger."
4. **(0:40–0:55) Open Voice note tab → click record → wait 3s** — "Same recording becomes 3 versions: clinical for EMR, warm letter for Helen, 5-line brief for handoff."
5. **(0:55–1:10) Press `Tab` → switch to Family view** — "This is Helen's view. Notice the language: 'How is your father today?' not 'Patient assessment.' The comfort toolkit is plain English."
6. **(1:10–1:25) Tap 'a bit tired' → press `Tab` back to Maria** — "A gentle signal appears in Maria's view. Bi-directional, but respectful."
7. **(1:25–1:30) Press `Tab` to Manager view** — "Aggregate only. Identity hidden even from the manager. $847K in ED diversions, all auditable."

---

## 12. Known limitations

- **No backend** — refreshing the page resets all state. Demo data only.
- **No real EMR integration** — DHDR / acCDR / CHR are static strings.
- **Single-file** — `index.html` is ~2700 lines. Not maintainable beyond prototype phase; production version should be component-based (React/Vue).
- **Accessibility gaps** — color carries significant meaning (partial icon-augmentation only); no full focus trap in dialogs; small base font (14px) for clinical screens.
- **No mobile layout** — designed for ≥1100px. Below that, grids collapse but density is wrong.
- **AI is mocked** — "Why?" reasoning is a static lookup table, not a live model.
- **Voice recording is mocked** — 3-second timer, no real audio capture, pre-written outputs.
- **Print** — only the one-pager has proper print CSS; other views print poorly.

---

## 13. Roadmap

### Short-term (next prototype iteration)

- [ ] Add a **guided tour** on first login (spotlight overlay)
- [ ] Add **AI feedback trend** to manager dashboard (Accept/Modify/Override ratio over time)
- [ ] Add **Family confidence trend line** (currently a static 4.6/5)
- [ ] Implement proper **focus trap** in dialogs + `aria-label`s
- [ ] Color-blind safe mode (icons + patterns, not just hue)

### Medium-term (productization)

- [ ] Split into components (React + Vite)
- [ ] Backend: FHIR-compliant API mapping to ConnectingOntario
- [ ] Real **OAuth/ONE ID** integration
- [ ] **PHIPA-compliant** audit log persisted server-side
- [ ] Mobile companion for community nurses (offline-first)
- [ ] Real AI service with retrievable provenance (Claude / GPT-4 + RAG over PaTz protocols)

### Long-term (deployment)

- [ ] Pilot in Georgina (12-month evaluation)
- [ ] Measure: ED diversion rate, ACP compliance, family confidence, staff burnout
- [ ] Expand to other Ontario Health regions

---

## License & credits

- **Project context:** Ivey MSc capstone — palliative care coordination design
- **Partner organizations:** Southlake Health, Ontario Health atHome, Georgina FHT
- **Frameworks referenced:** PaTz (NL), Geisinger ProvenHealth, Kaiser Permanente stratification, PHIPA (Ontario)

> Built as a learning artifact. Not for clinical use.

