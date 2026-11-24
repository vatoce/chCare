# Product Requirements Document (PRD)

## Church Care Platform — Pastoral Service, Visitation & Follow-up Management

| | |
|---|---|
| **Status** | Draft v0.1 |
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
| **Served person (child/youth/adult)** | End user of media library | Videos, quizzes, points, badges |
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
| PAR-7 | Mobile-first PWA, later native apps | P0 (PWA) |

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
- Target 99.5% in v1. Attendance capture must **work offline** (Edge Connector and mobile queue) and sync later.

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

## 7. Suggested Technology Stack (open to change)

| Layer | Proposal |
|---|---|
| Backend | TypeScript (NestJS) **or** Python (Django/FastAPI) — modular monolith |
| Database | PostgreSQL with Row-Level Security; `pg_trgm` for fuzzy matching |
| Cache/Queue | Redis + background workers (BullMQ/Celery) |
| Frontend | React/Next.js (PWA, RTL via CSS logical properties) |
| Mobile | PWA first; React Native/Flutter later |
| Media | S3-compatible storage + HLS streaming |
| Edge Connector | Small Go/Node service, Docker/Raspberry Pi-ready |
| Auth | OIDC provider (Keycloak/Auth0/self-built) |
| Infra | Docker, GitHub Actions, Terraform |
| Observability | OpenTelemetry, Prometheus/Grafana, Sentry |

---

## 8. Data Model (Initial Entities)

`Eparchy`, `Church (Tenant)`, `User`, `Role`, `Permission`, `Person`, `Family/Household`, `Relationship`, `Address`, `Event`, `EventSeries`, `AttendanceRecord`, `Device`, `DeviceEnrollment`, `Consent`, `Signal`, `Rule`, `RulePack`, `RuleEvaluation`, `CareTask`, `Visit`, `Note`, `Class`, `Stage`, `Enrollment`, `Assessment`, `Activity`, `MediaItem`, `Quiz`, `QuizAttempt`, `PointLedger`, `Badge`, `CaseFile (Brothers of the Lord)`, `Aid`, `Announcement`, `Notification`, `AuditLog`, `ApiKey`.

---

## 9. Release Plan

| Phase | Scope | Outcome |
|---|---|---|
| **Phase 0 — Foundation** | Repo, CI, architecture decisions, tenant model, auth, RBAC, i18n/RTL base | Walking skeleton |
| **Phase 1 — MVP** | People & families, family tree (manual), events & manual/QR attendance, basic reports, first care rules (~30), visitation list & visit logging | One pilot church usable |
| **Phase 2 — Automation** | Device connector framework + first device integration, Edge Connector, auto family suggestions, rule UI, assignments | Automated attendance & smarter care |
| **Phase 3 — Engagement** | Education/classes, parent portal, notifications, public API & widgets | Parents and website connected |
| **Phase 4 — Growth** | Media library + gamification, Brothers of the Lord module | Full pastoral suite |
| **Phase 5 — Scale** | Eparchy dashboards, self-service onboarding, ML-assisted ranking, native apps | Multi-church rollout |

---

## 10. Success Metrics

- % of attendance recorded automatically.
- Visitation coverage: share of "Urgent/High" recommendations acted on within SLA.
- Return-to-attendance rate after a visit.
- Number of families with a complete, confirmed family tree.
- Parent portal monthly active users.
- Media engagement (watch-completion, quiz completion).
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

---

## 13. Appendix — Guiding Principles

1. **People first, data second.**
2. **Human in the loop** — the system suggests, shepherds decide.
3. **Explainable by default.**
4. **Privacy and dignity over convenience.**
5. **Works offline, works on cheap phones.**
6. **Free and open for churches.**
