
# Indian School LMS — Product + Architecture Blueprint

## 1) Executive summary
Build a fast, offline-capable, India-native LMS that fixes grading friction, brings parents into the loop via WhatsApp, plugs into UPI Autopay/eNACH for fees, issues verified report cards to DigiLocker, and reuses DIKSHA/NDEAR where possible. Mobile-first. Simple for teachers. Investable because unit economics are clear and integrations reduce churn.

## 2) Product goals
- Delight teachers with a 1-screen Gradebook and question-wise grading.
- Be reliable on patchy internet with offline-first clients.
- Speak India: UPI Autopay, DigiLocker, DIKSHA QR content, multilingual UI, WhatsApp alerts.
- Privacy-first for minors and audit-ready.
- Launch a credible pilot with 3–5 schools in 90 days.

**Non-goals for MVP**
- Full ERP replacement.
- Complex proctoring or advanced adaptive learning.
- Microservices at day 1 (keep modular-monolith).

## 3) Personas
- **Teacher (Priya)**: low setup tolerance, wants quick grading, reuse of last year’s class.
- **Student (Aman)**: uses low-end Android, needs offline viewing and simple submissions.
- **Parent (Rita)**: wants reminders and report cards, prefers WhatsApp/SMS and her language.
- **School Admin (Mr. Sharma)**: cares about fees collection, attendance, audit, and uptime.

## 4) Core use cases & flows
- Create class → import syllabus/template → add DIKSHA content by scanning QR → publish.
- Assignment → students submit (PDF, photo, doc, audio) → teacher grades in grid view → release marks → parent notified.
- Attendance via roll call or QR/RFID → syncs when online → alerts to absentees’ parents.
- Fees: set plan → send UPI Autopay/eNACH mandate → auto reminders → reconciliation.
- Reports: term report cards → issue to DigiLocker → parents view and download.

## 5) Feature scope (MVP vs v1.1)
**MVP**
- Classes, assignments, submissions, Grade-by-Question, Grade-by-Student.
- Offline-capable mobile app and PWA.
- WhatsApp/FCM notifications.
- Google/Microsoft Drive import.
- Basic attendance and simple timetable.
- UPI Autopay and eNACH for recurring fees.
- DigiLocker issuance of report cards.
- DIKSHA QR scanning for content fetch.
- Basic analytics dashboards.

**v1.1 (post-pilot)**
- AI TA (chat over course material), rubric-based grading assist.
- Role-based SSO (OIDC/SAML), LTI 1.3 tool launch.
- xAPI telemetry + LRS integration for deeper analytics.
- Content marketplace and reusable question banks.
- State board templates and bulk importers.

## 6) Architecture (modular-monolith, extract later)
**Client**
- **Mobile**: Flutter, SQLite, background sync, file compression for uploads, offline queue.
- **Web**: Next.js PWA, IndexedDB via RxDB, installable, works on lab PCs.
- **Notifications**: FCM push; WhatsApp Business API via an Indian BSP (Gupshup/Karix).

**Server**
- **Runtime**: TypeScript + NestJS (recommended to start). Optional high-throughput services in Go later.
- **Storage**: PostgreSQL (primary), Redis (cache/queues), Object Storage (S3-compatible) for media, OpenSearch for search.
- **Streaming**: Kafka/Redpanda for events and analytics.
- **Media**: HLS adaptive streaming via Mux or AWS IVS; optional DRM (Widevine) for premium content.
- **Interops**: LTI 1.3, xAPI/SCORM. Optional Learning Locker as LRS.
- **Integrations**: UPI Autopay/eNACH, DigiLocker (issuer), DIKSHA (QR/content), SMS fallback via an Indian aggregator.
- **Infra**: Host in India regions with CDN, blue/green deploys, IaC via Terraform, Observability with OpenTelemetry.

**Module boundaries**
- Identity & Orgs | Rostering | Courses & Content | Assignments & Submissions | Gradebook | Attendance & Timetable | Fees & Invoicing | Messaging | Media | Search | Analytics.

## 7) Data model (high-level)
- `Organisation` → `School` → `Classroom` → `Roster (Teacher, Student, Parent)`
- `Course` → `Module` → `ContentItem` (source: upload, Drive, DIKSHA)
- `Assignment` → `Submission` → `Grade` → `Rubric` → `Feedback`
- `AttendanceSession` → `AttendanceRecord`
- `Invoice` → `PaymentIntent` → `Mandate` → `ReconciliationEntry`
- `Certificate`/`ReportCard` → `DigiLockerArtifact`
- `Event` (xAPI) for analytics

## 8) Public API sketches (REST)
- `POST /v1/classes` create class
- `POST /v1/assignments` create assignment
- `POST /v1/submissions` upload or reference file
- `POST /v1/grades/bulk` grade-by-question bulk update
- `POST /v1/fees/mandates` create UPI Autopay or eNACH mandate
- `POST /v1/digilocker/issue` issue a report card artifact
- `POST /v1/notifications/whatsapp` template send
- `GET /v1/content/diksha?q=qr_id` pull DIKSHA resource

## 9) Security, privacy, compliance
- DPDP for minors: verifiable parental consent, data minimization, privacy notices, parental dashboards, easy withdrawal.
- CERT-In: 180-day logs, 6-hour incident reporting playbook, audit trails.
- PII encryption at rest, TLS in transit, RBAC, ABAC for teacher access to classes.
- Regular 3rd party VAPT, backups with periodic restore drills.
- Data residency in India. Delete and retention policies documented.

## 10) AI features (guardrailed)
- **AI TA**: RAG over course files and DIKSHA content, with citations. Rate limited per class. 
- **Grading assist**: rubric-aware suggestions for short answers; teacher must confirm.
- **Content co-pilot**: lesson plan generator aligned to board templates; multilingual outputs.
- **Governance**: on-by-default logging of prompts and outputs (redacted for PII). Models hosted in India where possible.

## 11) Observability and SRE
- SLOs: login p95 < 600 ms; assignment create p95 < 800 ms; submission upload success > 99.5% weekly.
- RUM + backend traces via OpenTelemetry; alerts on SLO burn.
- Error budgets and simple runbooks for top 10 incidents.

## 12) Pricing and unit economics (hypothesis)
- SaaS per-student per-month with slabs and annual discount.
- Add-ons: payments, premium media/DRM, AI packs, SMS.
- School pilot: free 3 months for lighthouse schools in exchange for case studies.

## 13) GTM for India (first 6 months)
- Target affordable private schools and CBSE-affiliated mid-tier schools in Tier 1–2.
- Channel partners: local school IT resellers, tutoring chains.
- Proofs: 3 lighthouse pilots, before-after metrics on grading time, fee recovery, parent engagement.
- Content hooks: DIKSHA QR import, Google Drive one-click, report cards into DigiLocker.

## 14) QA & launch-readiness
- Test matrix: low-end Android, shared-PC labs, offline-to-online sync, 2G/3G throttling.
- Security tests: authz bypass, IDOR, broken object level authorization, SSRF, file upload AV scan.
- Privacy review: data map, DPIA, consent and age flows, data export and deletion.

## 15) Demo script (10 minutes)
1) Teacher creates class from CBSE template, scans DIKSHA QR to add content.
2) Publishes assignment; student submits by photo and Drive doc; offline submit queues.
3) Teacher grades question-wise in one screen; releases marks.
4) Parent gets WhatsApp notification, views report card in DigiLocker.
5) Admin checks fee mandates and attendance dashboard.

---

# 90-day execution plan (week-by-week)

## Month 1 — Foundation and MVP skeleton
- **Week 1**: Repo setup, CI/CD, IaC, core DB schema, auth + orgs, classroom & roster models, basic Next.js PWA shell, Flutter app shell.
- **Week 2**: Assignments + submissions APIs, object storage uploads, offline queue on web and mobile, basic notifications via FCM.
- **Week 3**: Gradebook data model and service, Grade-by-Question UI v1, Drive importers, simple attendance.
- **Week 4**: Payments service MVP (UPI Autopay/eNACH sandbox), WhatsApp templates integration, observability baseline, test harness.

## Month 2 — India integrations and polish
- **Week 5**: DigiLocker issuer flow + templated report cards, QR scanner in Flutter + DIKSHA fetch, parent mode.
- **Week 6**: HLS upload-and-play pipeline, image/PDF compression on device, multilingual UI toggle.
- **Week 7**: Admin dashboards, reconciliation for fees, CSV importers, role permissions pass.
- **Week 8**: Privacy & security hardening, DPDP consent flows, log retention, incident runbooks, pilot school onboarding kit.

## Month 3 — Pilot hardening and investor demo
- **Week 9**: Field testing in 1–2 schools, fix offline/edge cases, teacher training deck.
- **Week 10**: AI TA (RAG) over course files and DIKSHA content, grading assist alpha.
- **Week 11**: SLO tuning, load tests, blue/green rollout, DR drill.
- **Week 12**: Final demo build, pricing sheet, case study templates, investor deck.

**Milestones**
- **M1 (end of Week 4)**: Create class, assign, submit, grade, notify, fees sandbox.
- **M2 (end of Week 8)**: DigiLocker + DIKSHA QR + WhatsApp live, parent app usable.
- **M3 (end of Week 12)**: Pilot-ready, AI assist alpha, investor demo.

---

# Tech choice: Go vs TypeScript (for you)
- You already work in TypeScript/C#; picking **NestJS (TypeScript)** for the monolith will make you fastest to MVP.
- **Go** is excellent for high-throughput, low-latency services and simple concurrency. Add Go later for “hot paths” (e.g., media workers, realtime grade sync, webhook fanout) behind a message bus. 
- Rule of thumb: if a NestJS endpoint p95 < 250 ms and scales horizontally fine, keep it. If you hit CPU-bound or goroutine-friendly workloads, carve that part out to Go.

**Recommendation**: Start with **TypeScript + NestJS**. Introduce Go for specific services only when metrics prove it.

---

# Appendices

## A) Success metrics
- Teacher time to grade 30 submissions for 5 questions reduced by 50 percent.
- Submission success rate > 99.5 percent weekly.
- Fee collection improvement +10 percent with Autopay.
- Parent monthly active users > 60 percent.

## B) Risk log (top 5)
- WhatsApp template approvals and BSP throughput limits.
- DigiLocker issuer onboarding timelines.
- Variance in DIKSHA API reliability.
- School data migration complexity.
- Training and change management effort.

## C) Pilot checklist
- 2 classes per grade, 3 subjects, 2 assignments per week for 4 weeks.
- At least 1 fee cycle with mandates created.
- One term report card issued to DigiLocker.
- Post-pilot survey with NPS and qualitative teacher feedback.

