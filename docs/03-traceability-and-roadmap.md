# UniPortal Traceability & Roadmap

## Table of Contents
1. [Traceability Matrix](#traceability-matrix)
2. [Release Roadmap](#release-roadmap)
3. [Effort Estimates](#effort-estimates)

---

## Traceability Matrix

The traceability matrix demonstrates end-to-end alignment from business rules through functional requirements to technical implementation. This ensures every requirement is grounded in business policy and has a clear implementation path.

### Key Traceability Chains

| FR ID | Functional Requirement | Business Rule | API Endpoint | Data Entities | UI Component |
|-------|------------------------|---------------|--------------|---------------|--------------|
| **FR-REG-001** | Course Search | N/A | `GET /api/courses/search?term={term}&dept={dept}` | Course, Section, Term | CourseSearch.jsx |
| **FR-REG-004** | Prerequisite Validation | BR-001 (Prerequisite Enforcement) | `POST /api/enrollment/validate-prerequisites` | Course, Prerequisite, Enrollment, Student | CartValidation.jsx |
| **FR-REG-005** | Time Conflict Detection | BR-003 (Time Conflict Prevention) | `POST /api/enrollment/validate-schedule` | Section, Enrollment | ScheduleConflict.jsx |
| **FR-REG-006** | Real-Time Seat Availability | BR-004 (Enrollment Capacity Management) | `GET /api/sections/{id}/availability` (WebSocket) | Section | SeatCounter.jsx |
| **FR-REG-009** | Waitlist Enrollment | BR-005 (Waitlist Priority Order) | `POST /api/waitlist/enroll` | WaitlistEntry, Student, Section | WaitlistButton.jsx |
| **FR-REG-010** | Waitlist Auto-Enrollment | BR-005 (Waitlist Priority Order) | Background Job: `process_waitlist_queue` | WaitlistEntry, Enrollment, Section | N/A (automated) |
| **FR-BIL-001** | Tuition Calculation | BR-007 (Tuition Refund Schedule) | `POST /api/billing/calculate-tuition` | Invoice, Student, Section, Term | TuitionCalculator.jsx |
| **FR-BIL-005** | Refund Calculation | BR-007 (Tuition Refund Schedule) | `POST /api/billing/calculate-refund` | Invoice, Payment, Enrollment | RefundPreview.jsx |
| **FR-BIL-006** | Financial Hold Application | BR-008 (Registration Hold System) | Background Job: `apply_financial_holds` | Hold, Invoice, Student | HoldNotification.jsx |
| **FR-GRD-002** | Grade Entry | BR-012 (Grade Submission Deadline) | `POST /api/grades/submit` | Enrollment, Section, Faculty | GradeRoster.jsx |
| **FR-GRD-003** | Grade Submission Deadline Enforcement | BR-012 (Grade Submission Deadline) | Background Job: `send_grade_reminders` | Enrollment, Section, Term | EmailNotification |
| **FR-GRD-004** | Grade Change Request | BR-013 (Grade Change Authorization) | `POST /api/grades/change-request` | Enrollment, GradeChangeLog, Faculty | GradeChangeForm.jsx |
| **FR-GRD-005** | Student Grade View | BR-009 (GPA Calculation Method) | `GET /api/students/{id}/grades` | Enrollment, Student | GradeTranscript.jsx |
| **FR-POR-001** | Dashboard Overview | BR-008 (Registration Hold System), BR-010 (Academic Standing) | `GET /api/students/{id}/dashboard` | Student, Enrollment, Hold, Invoice | Dashboard.jsx |

### Detailed Traceability Example

#### Registration Flow: From Business Rule to Implementation

**Business Rule:** BR-001 (Prerequisite Enforcement)
> A student may only register for a course if they have successfully completed all prerequisite courses with a grade of C- or better.

**Functional Requirement:** FR-REG-004 (Prerequisite Validation)
> System shall prevent adding courses to cart if student has not completed prerequisites.

**API Endpoint:** `POST /api/enrollment/validate-prerequisites`
```json
Request:
{
  "student_id": 12345,
  "course_id": 450
}

Response (Success):
{
  "valid": true,
  "prerequisites_met": ["CS 341", "MATH 310"]
}

Response (Failure):
{
  "valid": false,
  "missing_prerequisites": [
    {
      "course_code": "CS 341",
      "title": "Data Structures & Algorithms",
      "status": "not_completed"
    }
  ],
  "message": "Missing prerequisites: CS 341"
}
```

**Data Entities:**
- `Student` — Student attempting to enroll
- `Course` — Target course with prerequisites defined
- `Prerequisite` — Junction table linking Course ↔ Required Course
- `Enrollment` — Historical enrollments to check completed prerequisites

**Database Query:**
```sql
SELECT p.required_course_id, rc.course_code, e.grade
FROM prerequisite p
JOIN course rc ON p.required_course_id = rc.course_id
LEFT JOIN enrollment e ON e.course_id = rc.course_id
  AND e.student_id = ?
WHERE p.course_id = ?
  AND (e.grade IS NULL OR e.grade > 'C-')
```

**UI Component:** `CartValidation.jsx`
- Displays error modal when prerequisite check fails
- Shows missing prerequisite courses with links to course catalog
- Suggests courses to take next term

---

## Release Roadmap

UniPortal will launch in four phases over 18 months, with each phase building on the previous. MVP focuses on core registration and billing; later phases add advanced features like degree audit and mobile apps.

### Phase 1: MVP (Months 1-6) — Core Student Services

**Goal:** Replace legacy SIS for course registration, grading, and billing. Minimum viable product for Fall 2026 registration period.

#### Features
- **Registration Module**
  - FR-REG-001: Course search and catalog browsing
  - FR-REG-002: Course catalog display with seat availability
  - FR-REG-003: Shopping cart management
  - FR-REG-004: Prerequisite validation
  - FR-REG-005: Time conflict detection
  - FR-REG-006: Real-time seat availability
  - FR-REG-007: Registration confirmation
  - FR-REG-008: Drop course functionality

- **Grading Module**
  - FR-GRD-001: Grade roster access for faculty
  - FR-GRD-002: Grade entry
  - FR-GRD-003: Grade submission deadline enforcement
  - FR-GRD-005: Student grade view and GPA display

- **Billing Module**
  - FR-BIL-001: Tuition calculation
  - FR-BIL-002: Invoice generation
  - FR-BIL-003: Payment processing (Stripe integration)
  - FR-BIL-005: Refund calculation
  - FR-BIL-006: Financial hold application

- **Student Portal**
  - FR-POR-001: Dashboard overview
  - FR-POR-002: Weekly schedule view
  - FR-POR-005: Hold resolution guidance

- **Administration**
  - FR-ADM-001: Term setup
  - FR-ADM-002: Course catalog management
  - FR-ADM-003: Section creation
  - FR-ADM-005: User role management

#### Technical Deliverables
- React SPA frontend with responsive design
- Django REST API backend
- PostgreSQL database with migration scripts
- Redis cache for seat counts
- Authentication via university LDAP
- Email notifications via SendGrid
- Basic admin dashboard

#### Success Metrics
- 8,000 students can register for Fall 2026 without system crashes
- Page load time < 3 seconds on 3G network
- 99.5% uptime during registration period
- Zero double-bookings (seat overselling)
- 95% student satisfaction score on post-registration survey

#### Effort Estimate: **480 developer-days** (6 months × 2 developers)

---

### Phase 2: v1.1 (Months 7-10) — Waitlist & Advanced Registration

**Goal:** Add waitlist functionality, payment plans, and enhanced reporting for Spring 2027 registration.

#### Features
- **Registration Enhancements**
  - FR-REG-009: Waitlist enrollment
  - FR-REG-010: Waitlist auto-enrollment with priority logic
  - Shopping cart save/share functionality
  - Schedule optimizer (suggest alternative sections to avoid conflicts)

- **Billing Enhancements**
  - FR-BIL-004: Payment plan enrollment
  - FR-BIL-007: 1098-T tax form generation
  - Late fee automation
  - Financial aid award display (read-only integration)

- **Grading Enhancements**
  - FR-GRD-004: Grade change request workflow
  - Incomplete grade tracking and resolution
  - Mid-term grade entry for early alerts

- **Administration**
  - FR-ADM-004: Enrollment reports and analytics
  - Bulk section creation via CSV import
  - Room conflict detection for scheduling

#### Technical Deliverables
- RabbitMQ message queue for async tasks
- Celery workers for waitlist processing
- Enhanced caching strategy for performance
- Audit logging for compliance (FERPA, PCI DSS)
- Scheduled jobs for reminder emails

#### Success Metrics
- Waitlist auto-enrollment latency < 60 seconds
- 1098-T forms generated for 100% of students by Jan 31
- Payment plan adoption rate > 40%
- Report generation time < 10 seconds for 10,000 records

#### Effort Estimate: **240 developer-days** (4 months × 1.5 developers)

---

### Phase 3: v1.2 (Months 11-14) — Degree Audit & Self-Service

**Goal:** Empower students with degree progress tracking and reduce advising workload.

#### Features
- **Degree Audit**
  - FR-POR-004: Degree audit display with requirements checklist
  - What-if analysis (simulate major/minor change)
  - Graduation eligibility prediction
  - Automated degree clearance for registrar

- **Self-Service Enhancements**
  - FR-POR-003: Transcript request and delivery
  - Address change and emergency contact update
  - Enrollment verification letter generation
  - Student photo upload for ID cards

- **Academic Advising Tools**
  - Advising notes and appointment scheduling
  - Degree plan builder (4-year course sequence)
  - Major declaration workflow
  - Academic standing notifications (probation, dean's list)

- **LMS Integration**
  - Canvas roster sync (bidirectional)
  - Auto-create Canvas course shells when section is created
  - Grade passback from LMS to SIS (optional)

#### Technical Deliverables
- Degree audit rules engine
- Document generation service (PDF transcripts, verification letters)
- Canvas API integration
- Student self-service portal enhancements
- Notification center with read/unread tracking

#### Success Metrics
- 80% of students use degree audit (track engagement)
- Transcript request fulfillment time < 2 business days
- Advising hold clearance time reduced by 50%
- LMS roster sync accuracy 100% (zero manual corrections)

#### Effort Estimate: **320 developer-days** (4 months × 2 developers)

---

### Phase 4: v2.0 (Months 15-18) — Mobile & Advanced Analytics

**Goal:** Launch native mobile apps and business intelligence dashboards for data-driven decision making.

#### Features
- **Mobile Apps (iOS & Android)**
  - Mobile-optimized registration flow
  - Push notifications for grades, payment due, and waitlist
  - Mobile ID card (barcode for library, dining, gym access)
  - Offline schedule view

- **Business Intelligence**
  - Real-time enrollment dashboards for deans
  - Retention analytics (at-risk student identification)
  - Financial forecasting (tuition revenue projections)
  - Course demand prediction for future term planning
  - IPEDS reporting automation

- **Faculty Enhancements**
  - Attendance tracking integration
  - Early alert system for struggling students
  - Faculty workload reporting (teaching hours, advisees)

- **Advanced Features**
  - Multi-language support (Spanish, Mandarin)
  - Accessibility improvements (WCAG 2.1 AAA compliance)
  - API marketplace for third-party integrations

#### Technical Deliverables
- React Native mobile apps
- GraphQL API for mobile clients
- Tableau/Power BI integration for analytics
- Machine learning model for retention prediction
- WebSocket real-time notifications

#### Success Metrics
- 60% of students use mobile app for at least one transaction
- Mobile app store rating ≥ 4.5 stars
- At-risk student identification accuracy 75%
- BI dashboard adoption by 100% of department chairs

#### Effort Estimate: **400 developer-days** (4 months × 2.5 developers)

---

## Effort Estimates Summary

### Total Project Timeline: 18 Months

| Phase | Duration | Team Size | Developer-Days | Key Deliverables |
|-------|----------|-----------|----------------|------------------|
| **Phase 1: MVP** | 6 months | 2 devs | 480 days | Registration, Grading, Billing, Portal |
| **Phase 2: v1.1** | 4 months | 1.5 devs | 240 days | Waitlist, Payment Plans, Reports |
| **Phase 3: v1.2** | 4 months | 2 devs | 320 days | Degree Audit, Transcripts, LMS Integration |
| **Phase 4: v2.0** | 4 months | 2.5 devs | 400 days | Mobile Apps, Analytics, Advanced Features |
| **TOTAL** | **18 months** | Avg 2 devs | **1,440 days** | Full-featured SIS |

### Team Composition (Recommended)

- **Backend Engineers (2):** Django/Python, PostgreSQL, REST API design
- **Frontend Engineer (1):** React, Tailwind CSS, responsive design
- **Mobile Engineer (1):** React Native (Phase 4 only)
- **DevOps Engineer (0.5 FTE):** AWS/Azure deployment, CI/CD, monitoring
- **QA Engineer (0.5 FTE):** Automated testing, security testing
- **Product Manager (1):** Requirements, stakeholder management, UAT
- **UX Designer (0.5 FTE):** User research, wireframes, prototypes

**Total Team Size:** 5-6 FTE (varies by phase)

### Budget Estimate (Rough Order of Magnitude)

Assuming average developer rate of $100/hour (blended rate for mid-senior engineers):

- **Development Costs:** 1,440 days × 8 hours × $100 = **$1,152,000**
- **Infrastructure (18 months):** AWS hosting, databases, monitoring = **$36,000**
- **Third-Party Services:** Stripe, SendGrid, LMS licenses = **$24,000**
- **Contingency (15%):** **$181,800**

**Total Estimated Cost: $1,393,800**

### Risk Mitigation

#### High-Risk Items
1. **Integration with Legacy Systems:** Budget 20% extra time for legacy data migration
2. **FERPA Compliance:** Engage legal counsel early; security audit before launch
3. **Peak Load Handling:** Load testing with 3x expected concurrent users
4. **Waitlist Logic Complexity:** Prototype and UAT with registrar before Phase 2

#### Mitigation Strategies
- **Phased Rollout:** Pilot with one department before university-wide launch
- **Parallel Run:** Run legacy SIS alongside UniPortal for one term
- **Training:** 40 hours of registrar/admin training before go-live
- **Support Plan:** 24/7 on-call during registration periods

---

## Acceptance Criteria for MVP Go-Live

UniPortal MVP must meet all criteria below before replacing legacy SIS:

### Functional Criteria
- [ ] All Phase 1 features implemented and tested
- [ ] Zero critical bugs in production for 2 consecutive weeks
- [ ] Registrar can create a term, courses, and sections end-to-end
- [ ] Students can register for courses with shopping cart workflow
- [ ] Faculty can submit grades and system calculates GPA correctly
- [ ] Invoices generate correctly with accurate tuition calculation
- [ ] Payments process successfully via Stripe (tested with real transactions)

### Performance Criteria
- [ ] Load test: 500 concurrent users registering without errors
- [ ] Page load time < 3 seconds on 3G network
- [ ] API response time p95 < 500ms
- [ ] Database queries optimized (no N+1 queries, indexes on foreign keys)

### Security & Compliance Criteria
- [ ] FERPA compliance verified by legal counsel
- [ ] PCI DSS Level 2 compliance for payment processing
- [ ] Penetration testing completed with all high/critical issues resolved
- [ ] Audit logging enabled for all financial and grade transactions
- [ ] Role-based access control tested (students cannot access admin functions)

### Operational Readiness Criteria
- [ ] Deployment pipeline automated (CI/CD)
- [ ] Monitoring and alerting configured (Datadog/New Relic)
- [ ] Database backups automated (daily full, hourly incremental)
- [ ] Disaster recovery plan tested (RTO < 4 hours, RPO < 1 hour)
- [ ] Support team trained and runbooks documented

### User Acceptance Criteria
- [ ] UAT completed with 20 students, 5 faculty, 3 admins (95% satisfaction)
- [ ] Accessibility tested with screen reader and keyboard navigation
- [ ] Training materials created (video tutorials, user guide)
- [ ] Help desk prepared with FAQ and escalation procedures

---

## Post-Launch Support Plan

### First 30 Days (Hypercare Period)
- **24/7 On-Call:** Engineering team rotates on-call shifts
- **Daily Standups:** Review metrics, triage bugs, prioritize fixes
- **Weekly Stakeholder Updates:** Enrollment numbers, issue summary, upcoming fixes
- **Hot Fix Process:** Critical bugs deployed within 4 hours

### Ongoing Support (After 30 Days)
- **Business Hours Support:** 8am-8pm Mon-Fri during registration periods
- **Maintenance Windows:** Sunday 2am-6am for deployments
- **Quarterly Releases:** New features and enhancements
- **Annual Security Audits:** Third-party penetration testing

---

**Document Version:** 1.0
**Last Updated:** 2026-02-05
**Author:** Requirements Engineering Team
