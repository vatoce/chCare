# Product Requirements Document (PRD)

## Church Care Platform — Pastoral Service, Visitation & Follow-up Management

| | |
|---|---|
| **Status** | Draft v0.3 (tech stack approved; served-person accounts & growth plans added) |
| **Type** | Non-profit, open-source |
| **Primary languages** | Arabic (RTL, default), English |
| **Document owner** | Project initiator |

---

## 1. Vision & Purpose

Churches serve hundreds or thousands of people, and the most valuable part of that service, personal pastoral care and visitation, is the hardest to scale. Servants rely on memory, paper lists and scattered spreadsheets, so people quietly drift away and families in need go unnoticed.

**Church Care Platform** is a multi-tenant, non-profit platform that gives every church a single place to:

1. Know its people and families.
2. Capture attendance automatically.
3. Be told, intelligently and explainably, **who needs a visit or a call first, and why**.
4. Engage children, youth and parents in spiritual and educational growth.
5. Care for people in material or emotional need with dignity and follow-through.

> The software supports shepherds. It never replaces them. Every recommendation is a suggestion that a human reviews.

### 1.1 Goals
- Reduce the number of people who disengage without anyone noticing.
- Eliminate manual attendance taking at liturgies and meetings.
- Give servants a prioritized, explainable visitation list.
- Let parents follow their children's participation remotely.
- Provide every church with an affordable (free) system and web presence.

### 1.2 Non-Goals (v1)
- Not a financial/accounting or donations platform.
- Not a replacement for confession or spiritual-father records. **No sacramental or confession data is ever stored.**
- Not a social network.
- Not a payroll/HR system.

---

## 2. Terminology

| Term | Meaning |
|---|---|
| **Eparchy / Diocese** (الإيبارشية / الأسقفية) | Top-level tenant group that oversees multiple churches |
| **Church / Parish** (الكنيسة) | Tenant. Each church has its own isolated data |
| **Served person** (مخدوم) | A person registered and followed up by the church |
| **Servant** (خادم) | A volunteer or clergy member who serves/visits |
| **Family / Household** (أسرة) | Group of related people living together or linked |
| **Visitation** (افتقاد) | A pastoral visit, call or message to a person/family |
| **Brothers of the Lord** (إخوة الرب) | People in need of material or moral support |
| **Stage / Class** (فصل / مرحلة) | Spiritual-education grouping by age or level |
| **Growth Plan** (خطة النمو) | Set of daily/monthly/yearly tasks assigned to a served person |
| **Task** (مهمة) | A spiritual, educational or service activity the person completes and tracks |

---

## 3. Users & Personas

| Persona | Description | Key needs |
|---|---|---|
| **Eparchy Admin** | Oversees multiple churches | Create churches, global reports, policy templates |
| **Church Admin / Priest** | Runs one church | Full church view, approvals, reports |
| **Service Leader** | Leads a stage/class/ministry | Their members, attendance, visitation lists |
| **Servant / Visitor** | Visits and follows up | Today's assignments, quick notes, mobile-first |
| **Care Coordinator** | Manages Brothers of the Lord | Case management, aid tracking |
| **Teacher / Tutor** | Runs classes & tutoring | Rosters, grades, activity logs |
| **Parent / Guardian** | Follows their children | Read-only dashboard, notifications |
| **Served person (child/youth/adult)** | Has their own account; follows a personal growth plan and uses the media library | Daily/monthly/yearly tasks, progress, streaks, videos, quizzes, points, badges |
| **Webmaster** | Church website | Public content & API |

---

## 4. System Architecture Overview

### 4.1 Multi-Tenancy
- **Hierarchy:** `Platform → Eparchy → Church → (Service / Class / Group)`.
- Each church is an isolated tenant. Data is never visible across churches unless explicitly shared by policy.
- **Eparchy-level access** is limited to aggregated/anonymized reports by default. Drill-down into personal data requires an explicit, audited grant from the church.
- Tenant isolation strategy: single database with a `tenant_id` on every row, enforced by **PostgreSQL Row-Level Security**, plus per-tenant encryption keys for sensitive fields.
- Each church can have a custom subdomain / custom domain, logo, language and calendar settings (including the Coptic calendar and liturgical seasons).

### 4.2 High-Level Components
```
 Mobile/Web Apps ──► API Gateway ──► Core Services ──► PostgreSQL (RLS)
 Parent Portal            │             ├─ People & Families        Redis (cache/queues)
 Church Website ◄─ Public API          ├─ Attendance               Object storage (media)
 Face/Biometric Devices ──► Edge       ├─ Care & Visitation Engine
        Connector ──────────►          ├─ Education & Activities
                                       ├─ Media & Gamification
                                       ├─ Brothers of the Lord
                                       └─ Notifications / Reports
```

---

## 5. Functional Requirements

Priority legend: **P0** = MVP, **P1** = next, **P2** = later.

### 5.1 Tenant, Roles & Access Control
| ID | Requirement | Pri |
|---|---|---|
| TEN-1 | Eparchy admin can create and manage churches under their eparchy | P0 |
| TEN-2 | Self-service church onboarding with approval by eparchy admin | P1 |
| TEN-3 | Role-based access control (RBAC) with per-church custom roles | P0 |
| TEN-4 | Scope-based permissions (e.g., a leader sees only their class) | P0 |
| TEN-5 | Full audit log of who viewed/edited sensitive records | P0 |
| TEN-6 | SSO / OAuth (Google, Apple) and optional 2FA | P1 |
| TEN-7 | Per-church configuration (languages, calendar, attendance rules, scoring weights) | P0 |

### 5.2 People, Families & Family Tree

**Goal:** Capture every person and automatically discover how people relate.

| ID | Requirement | Pri |
|---|---|---|
| PPL-1 | Create/edit person profiles: names, photo, gender, birth date, contacts, address (with map location), education, occupation, health notes (restricted), baptism/church status (optional), servant assigned | P0 |
| PPL-2 | Custom fields per church | P1 |
| PPL-3 | Create **families/households** with head of family, address and members | P0 |
| PPL-4 | Relationship model: parent, child, spouse, sibling, grandparent, guardian, relative, godparent, etc. (typed, directional, with start/end dates) | P0 |
| PPL-5 | **Interactive family tree** visualization (multi-generation, zoom, export as PDF/image) | P0 |
| PPL-6 | **Automatic family detection** (see 5.2.1) | P1 |
| PPL-7 | Handling of complex cases: divorce, remarriage, orphans, step/half siblings, guardians, families in multiple households | P1 |
| PPL-8 | Duplicate detection and merge tool with undo | P1 |
| PPL-9 | Bulk import (CSV/Excel) with field mapping and validation | P0 |
| PPL-10 | Life events timeline per person (joined, moved, graduated, ill, etc.) | P1 |
| PPL-11 | Tags, groups and saved filters | P0 |
| PPL-12 | Data lifecycle: archive, deceased, moved to another church (transfer with consent) | P1 |
| PPL-13 | Every served person can have **their own account** linked to their person record (see 5.12.1 for age-based rules) | P0 |

#### 5.2.1 Automatic Family Tree Detection
The system suggests links; humans confirm. Signals include:
- Same phone number, address, or location proximity.
- Shared surname/patronymic naming patterns (e.g., Arabic name chains: person + father + grandfather).
- Same guardian contact on children's records.
- Age gaps consistent with parent/child.
- Arrival/departure together on attendance data.

**Output:** a ranked list of *suggested relationships* with a confidence score and evidence ("Same mobile number + shared grandfather name"). A servant accepts, edits or rejects. Rejections are remembered so the same suggestion is not repeated. The model is rule-based first (explainable); ML-based ranking is a P2 enhancement.

### 5.3 Attendance & Biometric (Face Recognition) Integration

> The source requirement refers to facial biometrics ("بصمة الوجه"). The system is designed to support face recognition devices and, optionally, fingerprint, QR and NFC.

| ID | Requirement | Pri |
|---|---|---|
| ATT-1 | Define **events** (liturgies, meetings, classes, trips) with recurrence, location and eligible audience | P0 |
| ATT-2 | Manual attendance (mobile roll-call, QR check-in) as a fallback | P0 |
| ATT-3 | **Device integration layer**: a pluggable *Device Connector* interface supporting vendor SDKs/protocols (e.g., ZKTeco, Hikvision, Dahua, generic REST/webhook/ONVIF where possible) | P0 (framework) |
| ATT-4 | **Edge Connector** installed on a local church machine/Raspberry Pi to talk to on-premise devices, then securely sync to the cloud (works offline, queues events) | P1 |
| ATT-5 | Map device user IDs ↔ person records; enrollment workflow with consent | P0 |
| ATT-6 | Real-time attendance feed and live dashboards during an event | P1 |
| ATT-7 | Rules: late threshold, event time windows, anti-duplicate scans, kids "picked-up" check-out | P1 |
| ATT-8 | Attendance corrections with audit trail | P0 |
| ATT-9 | Attendance statistics: per person, family, class, event type, season | P0 |
| ATT-10 | Notify guardians on arrival/departure of children (opt-in) | P2 |

**Privacy rules for biometrics (mandatory):**
- Explicit, revocable, written/digital **consent**; for minors, consent from a guardian.
- Prefer storing **templates/IDs on the device**, not raw face images, in the cloud. Raw images are never stored by the platform unless the church explicitly enables it.
- Right to opt out with no penalty; the manual/QR alternative must always exist.
- Local legal compliance is the church's responsibility; the platform provides the tooling (consent logs, deletion, retention policies).

### 5.4 Pastoral Care & Smart Visitation Engine

**Goal:** Turn attendance and context data into a prioritized, explainable list of who should be visited or called, by whom, and when.

#### 5.4.1 Core concepts
- **Signal:** a measurable fact (e.g., "missed 4 consecutive Sundays").
- **Rule:** a configurable condition on signals producing a *score contribution* and a *reason text*.
- **Rule Pack:** a bundle of rules (e.g., Children, Youth, Adults, Elderly, Newcomers, Servants).
- **Priority Score & Tier:** sum of contributions → tier (Urgent / High / Normal / Low).
- **Task:** a suggested visitation assigned to a servant, with due date and status.

#### 5.4.2 Requirements
| ID | Requirement | Pri |
|---|---|---|
| CARE-1 | Rule engine with a **declarative rule format** (YAML/JSON) so rules are data, not code | P0 |
| CARE-2 | Ship a **library of pre-built rules** (target: 100+ over time, hundreds as the catalog matures) grouped by category (see 5.4.3) | P0 (initial set of ~30) |
| CARE-3 | Per-church enable/disable, tune thresholds and weights; create custom rules via UI | P1 |
| CARE-4 | **Explainability:** every recommendation shows "why" (rules triggered and their evidence) | P0 |
| CARE-5 | Daily/weekly auto-generated visitation list per servant/leader | P0 |
| CARE-6 | Smart assignment: servant ↔ person matching by gender, region, relationship, workload, prior contact | P1 |
| CARE-7 | Visitation lifecycle: suggested → assigned → contacted → completed/needs follow-up → closed, with outcome notes | P0 |
| CARE-8 | Visit logging via mobile (call, home visit, message), voice-note-to-text (P2), next follow-up date | P0 |
| CARE-9 | Prevent visit fatigue: cooldown periods and contact-frequency caps | P1 |
| CARE-10 | Context awareness: exams, illness, travel, bereavement, liturgical seasons (e.g., Lent), holidays | P1 |
| CARE-11 | Feedback loop: "Was this recommendation useful?" improves tuning | P1 |
| CARE-12 | Family-level view: combine members' signals into household priority | P1 |
| CARE-13 | Confidential notes with restricted visibility (priest/leader only) | P0 |
| CARE-14 | Effectiveness analytics: return-to-attendance rate after visits | P2 |
| CARE-15 | ML-assisted ranking layered on top of rules, always explainable and human-reviewed | P2 |

#### 5.4.3 Example rule categories
- **Attendance patterns:** consecutive absences, sudden drop vs. personal baseline, drop relative to family, irregular attendance, absence after a long streak.
- **Life stage:** child absent from class while parents attend, youth exam period, college transition, new baby, newly married, retirement, elderly living alone.
- **Circumstances:** reported illness, bereavement, job loss, relocation, family conflict flag, recent hospital stay.
- **Engagement:** stopped joining activities, no media interaction, no response to messages.
- **Newcomers:** first-time visitors, new families, lapsed returnees.
- **Servants:** servant burnout signals, missed service, requires encouragement.
- **Milestones:** birthdays, anniversaries, graduations, feast days.

**Example rule (illustrative format):**
```yaml
id: ATT-ABS-001
name: Four consecutive Sunday absences
category: attendance
applies_to: { age_min: 6 }
condition:
  event_type: sunday_liturgy
  consecutive_absences: { gte: 4 }
  exclude_if: [ "marked_travelling", "marked_ill_with_note" ]
score: 30
reason_en: "Missed the last {n} Sunday liturgies"
reason_ar: "غاب عن آخر {n} قداسات أحد"
suggested_action: call_then_visit
cooldown_days: 14
```

> **Ethics:** The engine is designed for care, not surveillance. Scores are visible only to authorized servants, never shown to the person or family, and are never used for punishment, exclusion or public lists.

### 5.5 Education, Activities & Classes

| ID | Requirement | Pri |
|---|---|---|
| EDU-1 | Manage spiritual education classes (Sunday school, youth, university, etc.) with stages, teachers, schedule, roster | P0 |
| EDU-2 | Manage **tutoring/remedial classes** for school subjects (subject, grade, teacher, schedule, fees waived/free) | P1 |
| EDU-3 | Generic **Activity** module (trips, camps, clubs, choirs, sports, retreats) with registration, capacity and attendance | P1 |
| EDU-4 | Curriculum & lesson plans; attach materials | P1 |
| EDU-5 | Assessments: grades, memorization (verses, hymns), participation, behavior notes | P1 |
| EDU-6 | Promotion between stages at year-end (bulk) | P1 |
| EDU-7 | Teacher/servant schedule and workload management | P2 |
| EDU-8 | Certificates generation | P2 |

### 5.6 Parent & Guardian Portal

| ID | Requirement | Pri |
|---|---|---|
| PAR-1 | Secure parent accounts linked to children via verified guardianship | P0 |
| PAR-2 | Dashboard: attendance, class participation, grades, upcoming events, points and badges | P0 |
| PAR-3 | Notifications (push/SMS/WhatsApp-compatible/email) for absences, events, announcements | P1 |
| PAR-4 | Parent ↔ servant messaging (moderated, logged) | P1 |
| PAR-5 | Permission slips and event registration approvals | P2 |
| PAR-6 | Parents control which data they receive; older teens can have limited visibility per church policy | P1 |
| PAR-7 | Native iOS & Android app for parents (see 5.11); web portal as secondary channel | P0 |

### 5.7 Church Website Integration

| ID | Requirement | Pri |
|---|---|---|
| WEB-1 | Versioned **Public REST/GraphQL API** per church for approved public data (schedule, events, announcements, media, contact) | P0 |
| WEB-2 | **Embeddable widgets** (events calendar, mass times, media library, registration forms) via script/iframe | P1 |
| WEB-3 | Optional **built-in website generator** (template-based, per-church subdomain) | P2 |
| WEB-4 | Public registration forms (new visitor, activity sign-up, prayer request) feeding the CRM with spam protection | P1 |
| WEB-5 | WordPress plugin as a reference integration | P2 |
| WEB-6 | API keys & scopes per church; public endpoints never expose personal data of minors | P0 |

### 5.8 Media Library & Gamification

| ID | Requirement | Pri |
|---|---|---|
| MED-1 | Upload/host or link media (video, audio, PDF, images); categories, tags, age-group, language, series | P0 |
| MED-2 | Content moderation/approval workflow before publishing | P1 |
| MED-3 | **Points system:** points for watching a video (verified by minimum watch-time/progress), answering attached quizzes, completing series | P0 |
| MED-4 | **Quizzes** attached to media: MCQ, true/false, short answer (manual review) | P0 |
| MED-5 | Anti-cheat: minimum watch percentage, attempt limits, cooldowns, randomization | P1 |
| MED-6 | Badges, levels, streaks, leaderboards (opt-in; class/age-based; never shaming low performers) | P1 |
| MED-7 | Rewards catalog managed by church (non-monetary, e.g., certificates, trips, books) | P2 |
| MED-8 | Child-safe mode: no external links, no comments, age-appropriate filtering | P0 |
| MED-9 | Analytics: popular content, completion rates, engagement feeds into care signals | P1 |
| MED-10 | Cross-church shared library at eparchy level (curated content share) | P2 |
| MED-11 | Respect copyright: content owners and licenses recorded per item | P1 |

### 5.9 Brothers of the Lord (Care for People in Need)

| ID | Requirement | Pri |
|---|---|---|
| BRO-1 | Confidential **case file** per person/family: needs (financial, medical, housing, food, education, emotional, employment), situation summary, documents | P0 |
| BRO-2 | **Strict privacy:** restricted roles, field-level encryption, access logging, no visibility in general lists | P0 |
| BRO-3 | Needs assessment & prioritization (configurable scoring, urgency) | P1 |
| BRO-4 | **Assistance tracking:** type, amount/item, date, donor/source (anonymous-capable), approver | P0 |
| BRO-5 | Recurring aid (monthly support), in-kind aid and inventory | P1 |
| BRO-6 | Visits and follow-up integrated with the Care engine | P0 |
| BRO-7 | Case workflow: intake → assessment → plan → aid → review → closure / graduation from need | P1 |
| BRO-8 | Referrals to external services (doctors, NGOs, job opportunities); partner directory | P2 |
| BRO-9 | Volunteer matching (doctors, lawyers, teachers, tradespeople) | P2 |
| BRO-10 | Impact reporting (anonymized): number helped, aid distribution, improvement outcomes | P1 |
| BRO-11 | Duplicate-aid prevention across churches (privacy-preserving, opt-in) | P2 |
| BRO-12 | Dignity principle: no public lists, photos or sharing without explicit consent | P0 |

### 5.10 Notifications, Communication & Reporting
- Channels: in-app, push, email, SMS, WhatsApp-style gateways (provider-pluggable). P1
- Announcement targeting by class, family, tag, age group. P1
- Reports: attendance trends, visitation coverage, class summaries, engagement, care outcomes, exportable (CSV/PDF/Excel). P0/P1
- Eparchy dashboards using aggregated data only. P1

---

### 5.11 Mobile Applications (iOS & Android)

The mobile app is a **core product surface**, not an add-on. Servants use it daily in the field (often with weak connectivity), and families use it as their main window into the church.

| ID | Requirement | Pri |
|---|---|---|
| MOB-1 | Native iOS and Android apps built from a **single TypeScript codebase** (React Native + Expo) | P0 |
| MOB-2 | **Offline-first** for servants: encrypted local database, background sync queue, conflict resolution (last-write-wins for simple fields, merge/prompt for notes and visit logs) | P0 |
| MOB-3 | Servant home: today's assigned visitations with the "why" behind each, one-tap call / WhatsApp / directions, quick visit logging with next follow-up date | P0 |
| MOB-4 | Camera **QR check-in** scanning and manual roll-call; tablet **kiosk mode** for entrance check-in | P0 / P1 |
| MOB-5 | Push notifications (APNs/FCM) with per-user preferences and quiet hours | P0 |
| MOB-6 | App unlock with device biometrics (Face ID / fingerprint). This is **local device security only**, separate from attendance biometrics | P0 |
| MOB-7 | Parent/guardian experience: children switcher, attendance, class progress, events, points & badges, messages | P0 |
| MOB-8 | Child-safe media player: no external links, no comments, age-appropriate content, quizzes and points | P1 |
| MOB-9 | Full Arabic RTL, accessibility (dynamic text, screen readers), and support for **low-end Android devices** (Android 8+, iOS 15+) | P0 |
| MOB-10 | Secure storage of tokens, encrypted local DB, remote session revoke / device wipe, screenshot protection on sensitive screens | P0 |
| MOB-11 | Over-the-air (OTA) updates for non-native changes, so fixes reach volunteers without store delays | P1 |
| MOB-12 | Deep links / universal links (open a person, event or task from a notification or message) | P1 |
| MOB-13 | Role-based experience in one app (servant, leader, parent, served person); option to ship separate store listings later via app variants | P0 |
| MOB-14 | Store compliance: Apple Kids/Parental-gate guidance, Google Play Families policy, privacy labels and data-safety forms | P1 |
| MOB-15 | Served-person home: today's tasks, streak and progress, one-tap completion offline, local reminders, monthly and yearly views (see 5.12) | P0 |
| MOB-16 | Home-screen widgets and quick actions for today's tasks (iOS/Android) | P2 |


### 5.12 Served-Person Accounts & Personal Growth Plan

Every served person (child, teen or adult) can sign in to their own account, follow a personalized plan of **daily, monthly and yearly tasks** (educational, spiritual and more) that the app assigns, and see their own progress over time. The goal is encouragement and habit-building, never pressure or comparison.

#### 5.12.1 Served-Person Accounts

| ID | Requirement | Pri |
|---|---|---|
| ACC-1 | Account creation by **invite** from the church (link/QR/code) tied to an existing person record; no open sign-up that could attach to the wrong person | P0 |
| ACC-2 | **Age tiers:** child (under 13) account is created and managed by a guardian; teen (13 to 17) has own login with guardian oversight per church policy; adult is fully independent. Age thresholds are configurable per church and locale | P0 |
| ACC-3 | Guardian approval and linkage for minors; guardians can see the child's plan and progress (aligned with PAR-2) | P0 |
| ACC-4 | Simple login for kids (PIN/QR/parent-assisted), and standard secure login for teens and adults; device biometric unlock (MOB-6) | P0 |
| ACC-5 | Profile and settings: language, reminder times, preferred task categories, privacy choices | P1 |
| ACC-6 | One person, several roles: a served person who is also a servant sees both experiences in the same app | P1 |
| ACC-7 | Account recovery that does not expose personal data (verification through the church/servant or guardian) | P1 |

#### 5.12.2 Task Model

- **Task Template:** reusable definition (title, description, category, cadence, completion method, points, age/stage targeting, optional media/quiz attachment).
- **Growth Plan:** a set of templates assigned to a stage, class, group or person for a period.
- **Task Assignment:** the concrete instance for one person for one period (a day, month or year).
- **Completion:** the record that the person (or the system) marked it done, with optional value, note and evidence.
- **Cadence:** daily, weekly, monthly, yearly, seasonal (e.g., Lent, Nativity fast, summer break) and one-off.
- **Categories (extensible per church):** Spiritual (prayer, scripture reading, hymns, fasting guidance, attending liturgy), Educational (lessons, memorization, homework, tutoring, reading), Service (volunteering, helping others), Personal growth (life skills, habits), Community (family time, visiting someone), plus custom categories.
- **Completion methods:**
  - *Self-check* (default for spiritual and personal tasks, honor-based).
  - *Counter/value* (e.g., chapters read, minutes prayed).
  - *Auto-verified* from system data: liturgy/meeting attendance (5.3), media watched or quiz passed (5.8), class assignment submitted (5.5).
  - *Confirmed* by a servant or guardian (for selected tasks).

#### 5.12.3 Requirements

| ID | Requirement | Pri |
|---|---|---|
| GRW-1 | Task template library managed by church admins/leaders, with an **eparchy-shared starter library** that churches can copy and adapt | P0 |
| GRW-2 | **Plans by stage/age:** each class or age group gets a default plan; a person's plan is derived automatically from their stage and can be adjusted | P0 |
| GRW-3 | **The app assigns tasks automatically** each day/month/year based on plan, age, level, liturgical calendar and recent completion history | P0 |
| GRW-4 | Adaptive suggestions: gentle difficulty progression, "easier day" mode after missed days, and suggestions for new tasks when the person is consistent (rule-based first, ML later) | P1 |
| GRW-5 | Personal goals: a person (or guardian) can add their own tasks and goals in addition to assigned ones | P1 |
| GRW-6 | **Daily view:** today's tasks with one-tap completion (works offline), reminders, and a short motivational or spiritual note | P0 |
| GRW-7 | **Monthly view:** monthly tasks and a month summary (completion %, streaks, highlights) | P0 |
| GRW-8 | **Yearly view:** yearly goals and a "year journey" summary (milestones reached, growth areas, books/lessons completed), exportable as a PDF certificate or report | P1 |
| GRW-9 | **Progress tracking:** completion rate per category, streaks with grace days (a missed day does not erase long progress), progress bars and milestones | P0 |
| GRW-10 | Integration with the **points & badges** system (5.8): points per task, badges for streaks and milestones, never shaming for missed tasks | P1 |
| GRW-11 | Liturgical-calendar awareness: seasonal plans (Lent, Nativity, Apostles' fast, feasts), church-defined dates, Coptic calendar display | P1 |
| GRW-12 | Visibility rules: the person sees everything; guardians see minors' plans and progress; servants and class leaders see **summary-level** progress for people they serve; adults may mark selected personal goals as private | P0 |
| GRW-13 | Servant tools: assign or adjust tasks for a person or class, add a personal encouragement message, view class-level progress and who might need encouragement | P1 |
| GRW-14 | **Care integration:** progress changes (e.g., a long streak that stops, a sudden drop in completion) can feed the visitation engine (5.4) as signals, only for categories and people covered by the church's privacy policy, always explainable | P1 |
| GRW-15 | Reminder engine: push and local (offline) notifications, per-user quiet hours and frequency caps, no guilt-based wording | P0 |
| GRW-16 | Task content in multiple languages; media or quiz attached to a task (reuses the media library) | P1 |
| GRW-17 | Group and family challenges (opt-in), e.g., a family reads together, a class completes a memorization series | P2 |
| GRW-18 | Reports for leaders: engagement per class/stage/period, most and least completed tasks (to improve the plans) | P1 |
| GRW-19 | Backfill and correction: a person can mark a task done for a recent day within a configurable window; all edits are logged | P1 |
| GRW-20 | Timezone-aware days (people travel, diaspora churches) | P1 |

#### 5.12.4 Principles and Safeguards
- **Encouragement, not surveillance.** Spiritual practice data is religious data and therefore sensitive. Defaults favor the person's dignity: no public leaderboards for spiritual tasks, no shaming, no comparison between individuals.
- **No sacramental tracking.** The platform never records confession, communion or anything covered by the confessional relationship. Tasks are practices like prayer, reading and service that the church chooses to encourage.
- **Honor system by default.** Self-reported completion is trusted; verification is applied only where the data already exists (attendance, quizzes) or the church chooses it.
- **Minors:** data minimization, guardian visibility, no free-text sharing with adults outside approved servants, and no behavioral advertising or third-party analytics in child accounts.
- **Consent and transparency:** persons (and guardians for minors) are told what servants can see and how progress may inform pastoral care.

---

## 6. Non-Functional Requirements

### 6.1 Security & Privacy (critical — data includes minors and vulnerable people)
- Encryption in transit (TLS 1.2+) and at rest; field-level encryption for sensitive data.
- Tenant isolation verified by automated tests (RLS) and periodic review.
- RBAC + least privilege; audit logging for all sensitive reads/writes.
- Data minimization, configurable retention, export and deletion on request (GDPR-style principles applied globally).
- Child-protection policies: background-checked servant accounts, two-adult communication rules (configurable), moderated messaging.
- Backups with tested restores; disaster recovery plan.
- Secure secrets management; dependency scanning; responsible disclosure policy (`SECURITY.md`).

### 6.2 Performance & Scalability
- Support up to ~10,000 active persons per church and ~1,000 churches per deployment in v1 targets.
- p95 API latency < 500 ms for standard reads; attendance ingestion handles bursts (hundreds of scans/minute at liturgy start).

### 6.3 Availability & Offline
- Target 99.5% in v1. Attendance capture and servant visitation logging must **work offline** (Edge Connector and the mobile app's local database) and sync later.

### 6.4 Usability & Accessibility
- Arabic-first, full RTL support; English secondary; i18n-ready for Coptic/other languages.
- Mobile-first, low-bandwidth friendly, simple for non-technical volunteers.
- WCAG 2.1 AA target. Support for the **Coptic calendar** and Hijri/Gregorian display options.

### 6.5 Maintainability & Open Source
- Modular monolith first (clear module boundaries), extractable services later.
- Automated tests (unit, integration, tenant-isolation), CI/CD, infrastructure as code, Docker-based dev setup.
- Documentation: API (OpenAPI), contributor guide, deployment guide for self-hosting.

### 6.6 Cost
- Designed to run cheaply (non-profit): single-VM self-host option and low-cost managed cloud option.

---

## 7. Technology Stack (Approved)

Guiding decision: **TypeScript everywhere in one monorepo**, so a small team of volunteers can move between backend, web, mobile and edge code. Details and rationale are in `docs/adr/`.

| Layer | Decision |
|---|---|
| Language / Monorepo | TypeScript, pnpm workspaces + Turborepo |
| Backend | NestJS, modular monolith, REST + OpenAPI (public API versioned) |
| Database | PostgreSQL with **Row-Level Security** for tenant isolation; `pg_trgm` for fuzzy matching |
| ORM / queries | Drizzle ORM (explicit SQL control for RLS) |
| Cache / Queue | Redis + BullMQ (rule evaluation, notifications, attendance ingestion) |
| Web | Next.js + Tailwind + shadcn/ui (RTL via CSS logical properties); church websites and web portal |
| **Mobile (iOS + Android)** | **React Native + Expo (dev builds + EAS Build/Update)**, TypeScript, Expo Router |
| Mobile offline data | SQLite-based local DB with a sync layer (WatermelonDB, evaluated against PowerSync in ADR-0003), SQLCipher encryption |
| Mobile extras | expo-camera (QR), expo-notifications (APNs/FCM), expo-local-authentication, expo-secure-store |
| Shared code | `packages/shared` (Zod schemas, types), `packages/api-client`, `packages/rules-engine` |
| Edge Connector | Node.js (TypeScript) in Docker for Raspberry Pi / small PC; vendor SDK adapters (native bridge if a device SDK requires it) |
| Family-tree UI | React Flow / family-chart (web), custom SVG view on mobile |
| Auth | Better Auth (OIDC-ready); Keycloak if institutional SSO is needed |
| Media | S3-compatible storage (Cloudflare R2 / Backblaze) + HLS streaming provider |
| Infra | Docker Compose on a VPS (Coolify), upgrade path to managed Kubernetes |
| CI/CD | GitHub Actions; EAS for mobile builds |
| Testing | Vitest, Playwright (web), Maestro/Detox (mobile), mandatory tenant-isolation tests |
| Observability | Sentry (backend, web, mobile), OpenTelemetry, Grafana/Prometheus later |

**Why React Native over Flutter?** Same language as the backend and web (shared validation schemas, API client and rule types), one hiring/volunteer pool, mature offline/notification/camera ecosystem, and OTA updates through Expo. Flutter remains the fallback if performance or device-SDK needs prove otherwise (revisit trigger recorded in ADR-0003).

---

## 8. Data Model (Initial Entities)

`Eparchy`, `Church (Tenant)`, `User`, `Role`, `Permission`, `Person`, `Family/Household`, `Relationship`, `Address`, `Event`, `EventSeries`, `AttendanceRecord`, `Device`, `DeviceEnrollment`, `Consent`, `Signal`, `Rule`, `RulePack`, `RuleEvaluation`, `CareTask`, `Visit`, `Note`, `Class`, `Stage`, `Enrollment`, `Assessment`, `Activity`, `MediaItem`, `Quiz`, `QuizAttempt`, `PointLedger`, `Badge`, `ServedAccount`, `GuardianLink`, `TaskTemplate`, `GrowthPlan`, `TaskAssignment`, `TaskCompletion`, `Streak`, `Milestone`, `CaseFile (Brothers of the Lord)`, `Aid`, `Announcement`, `Notification`, `AuditLog`, `ApiKey`.

---

## 9. Release Plan

| Phase | Scope | Outcome |
|---|---|---|
| **Phase 0 — Foundation** | Monorepo, CI, ADRs, tenant model + RLS, auth, RBAC, i18n/RTL base, mobile app shell (Expo) with login and push | Walking skeleton on web + iOS + Android |
| **Phase 1 — MVP** | People & families, manual family tree, events & QR/manual attendance, first care rules (~30), visitation list & visit logging, **servant mobile app with offline sync** | One pilot church using the app in the field |
| **Phase 2 — Automation** | Device connector framework + first device, Edge Connector, auto family suggestions, rule UI, smart assignment | Automated attendance & smarter care |
| **Phase 3 — Engagement** | Education/classes, **parent mobile experience**, **served-person accounts and growth plans (daily/monthly/yearly tasks, progress, reminders)**, notifications, public API & widgets | Families and served persons use the app daily |
| **Phase 4 — Growth** | Media library + gamification (child-safe player), adaptive task suggestions, yearly journey reports, task-to-care signals, Brothers of the Lord module | Full pastoral suite |
| **Phase 5 — Scale** | Eparchy dashboards, self-service onboarding, ML-assisted ranking, store-listing variants | Multi-church rollout |

---

## 10. Success Metrics

- % of attendance recorded automatically.
- Visitation coverage: share of "Urgent/High" recommendations acted on within SLA.
- Return-to-attendance rate after a visit.
- Number of families with a complete, confirmed family tree.
- Parent portal monthly active users.
- Media engagement (watch-completion, quiz completion).
- Served-person accounts activated, daily active users, task completion rate and 30/60/90-day streak retention.
- Time saved per servant per week (survey).
- Zero tenant-isolation or privacy incidents.

---

## 11. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Sensitive data leakage (minors, people in need) | RLS, encryption, audit logs, least privilege, security reviews |
| Biometric legal/ethical concerns | Consent-first, manual alternative, minimal storage, local-law guidance |
| Device vendor fragmentation | Connector abstraction; start with 1–2 popular devices |
| "Surveillance" perception by congregation | Transparency, explainable rules, never punitive, leader training |
| Poor recommendation quality | Start rule-based, feedback loop, per-church tuning |
| Volunteer-run maintenance | Simple architecture, good docs, open-source community |
| Low technical literacy of users | Simple UX, onboarding, Arabic-first, short training material |
| Task gamification creating pressure or comparison among children and youth | Private-by-default progress, grace days, no public rankings for spiritual tasks, positive wording reviewed by leaders |
| Mobile sync conflicts / data loss offline | Offline-first design with tested sync, conflict rules, visible sync status |
| App-store review for apps involving children | Follow Kids/Families policies early, parental gate, minimal data collection |

---

## 12. Open Questions

1. Which face-recognition/fingerprint devices are already in use at target churches (brand/model/SDK)?
2. Hosting preference: centralized SaaS run by the eparchy vs. self-hosted per church vs. both?
3. Which jurisdictions/legal frameworks must be considered for personal and biometric data?
4. Is Coptic-calendar liturgical-season awareness required in v1 rules?
5. Policy for who can see people in the "Brothers of the Lord" module at church vs. eparchy level?
6. Preferred notification channels and budget for SMS/WhatsApp gateways?
7. Licensing: open-source license choice (e.g., AGPL-3.0 to keep derivatives open vs. MIT/Apache-2.0)?
8. Who validates the pastoral rule library (priests, experienced servants) before release?
9. Who authors and approves the starter task library (spiritual and educational tasks) per age group, and what is the review process?
10. Default visibility of adults' task progress to servants: summary only, or opt-in only?
11. Age thresholds for child, teen and adult accounts, and any local legal limits on minors' accounts?
12. Should some tasks be mandatory for classes (e.g., memorization) with servant confirmation?

---

## 13. Appendix — Guiding Principles

1. **People first, data second.**
2. **Human in the loop** — the system suggests, shepherds decide.
3. **Explainable by default.**
4. **Privacy and dignity over convenience.**
5. **Works offline, works on cheap phones.**
6. **Free and open for churches.**
