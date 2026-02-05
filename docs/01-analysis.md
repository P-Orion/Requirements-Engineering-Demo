# UniPortal Competitive Analysis: OSIS Study

## Executive Summary

This document analyzes **OSIS (Open Student Information System)**, a production university SIS developed by Université catholique de Louvain (UCLouvain) in Belgium. OSIS is a Django-based (Python 3.x) open-source system managing the complete lifecycle of higher education administration. This analysis extracts architectural patterns, data models, and feature sets to inform UniPortal's design for a modern US university context.

---

## 1. OSIS System Overview

**Technology Stack:**
- **Backend:** Python 3.4+, Django 1.11+ (some deployments use Django 2.2+)
- **Frontend:** HTML5, JavaScript (traditional Django templates, not React SPA)
- **Database:** PostgreSQL (assumed from Django conventions)
- **License:** GPL v3 (open source)
- **Architecture:** Monolithic Django application with pluggable modules
- **Deployment:** European university context (Belgian/French academic calendar and regulations)

**Modular Structure:**
OSIS uses a microservice-like module approach within Django:
- `osis-admission` — Student admission and application processing
- `osis-dissertation` — Thesis/dissertation management for graduate students
- `osis-internship` — Clinical/professional internship tracking (medicine, education)
- `osis-assistant` — Teaching assistant management and allocation
- `osis-document` — Central document management system with file upload handling
- `osis-assessments` — Examination and evaluation management
- `osis-program-management` — Curriculum and program structure definition

---

## 2. Entity & Data Model Analysis

### Core Entities Identified (from OSIS codebase inspection)

| Entity | Purpose | Key Relationships |
|--------|---------|-------------------|
| **Person** | Base entity for all users (students, faculty, staff) | → Student, → Tutor, → Staff |
| **Student** | Enrolled learners | → Admissions, → Enrollments, → Assessments |
| **AcademicYear** | Time period container (e.g., 2023-2024) | → Terms, → Programs |
| **EducationGroup** | Programs, majors, minors, certificates | → Courses, → Requirements |
| **LearningUnit** | Individual course/module | → LearningUnitYear (course offering per year) |
| **LearningUnitYear** | Specific offering of a course in an academic year | → LearningComponentYear (lecture, lab, recitation) |
| **Enrollment** | Student registration in a LearningUnitYear | → Student, → LearningUnitYear |
| **Offer** | Program offering (degree program available to students) | → EducationGroup, → AcademicYear |
| **Admission** | Application and acceptance tracking | → Student, → Offer |
| **Dissertation** | Thesis/dissertation projects | → Student, → Advisors, → Defense |
| **Internship** | Clinical/field placement | → Student, → Organization, → Supervisor |
| **Assessment** | Exam scores and evaluations | → Student, → LearningUnitYear |
| **EntityVersion** | Organization hierarchy (faculties, departments) | Self-referential tree structure |

### Authentication & Authorization Model
- **Role-based access control (RBAC)** via Django groups/permissions
- Roles: Student, Faculty, Program Manager, Dean, Registrar, Administrator
- Permissions attached to Django models (e.g., `can_edit_admission`, `can_submit_grades`)

---

## 3. Feature Inventory by Module

### OSIS Core Features

#### **Admission Module**
- Online application portal
- Document upload and verification
- Multi-stage workflow (submitted → reviewed → accepted/rejected)
- International student handling (visa requirements, equivalency evaluation)

#### **Program Management**
- Hierarchical curriculum builder (programs → tracks → courses)
- Prerequisite and corequisite definition
- Credit requirements and distribution rules
- Degree audit functionality

#### **Enrollment/Registration**
- Course search and filtering
- Section selection with time conflict detection
- Waitlist management
- Drop/add period enforcement

#### **Assessment/Grading**
- Grade entry by instructors
- Multiple assessment types (exam, project, continuous assessment)
- Grade scale configuration (0-20 European scale, customizable)
- Transcript generation

#### **Dissertation Management**
- Proposal submission workflow
- Advisor assignment and approval
- Defense scheduling
- Final document submission

#### **Internship Management**
- Placement matching (student ↔ organization)
- Evaluation rubrics
- Site visit tracking
- Compliance verification (hours, certifications)

---

## 4. Feature Comparison: OSIS vs. Modern US University SIS

| Feature Category | OSIS (European Context) | UniPortal (US University Need) | Gap Analysis |
|------------------|-------------------------|--------------------------------|--------------|
| **Registration** | Basic enrollment, manual section selection | Real-time seat availability, shopping cart, schedule optimizer | Need: Cart UI, conflict detection, real-time sync |
| **Billing** | ❌ Not included (European free tuition) | ✅ **Critical**: Tuition calculation, payment plans, 1098-T tax forms | **Major gap** — must build from scratch |
| **Financial Aid** | ❌ Not applicable | ✅ **Critical**: FAFSA integration, award letters, disbursement | **Major gap** — US-specific compliance |
| **Academic Standing** | Basic progression rules | GPA calculation, probation, dean's list, SAP (financial aid) | Need: Automated policy enforcement |
| **Waitlists** | ❌ Not implemented | ✅ Required: Auto-enroll when seat opens, priority rules | Must build |
| **Holds** | ❌ Not implemented | ✅ Required: Registration holds (financial, advising, immunization) | Must build |
| **Transcript Formats** | European Diploma Supplement | US-style transcripts, PDF generation, electronic delivery | Customization needed |
| **Course Prerequisites** | Manual definition | Automated validation during registration | OSIS has basic model, need runtime enforcement |
| **Mobile Access** | Desktop-only Django templates | Responsive web app, mobile-first design | **Architecture gap** — need React SPA |
| **Third-Party Integrations** | Minimal | LMS (Canvas/Blackboard), payment gateway, identity provider (SAML/OAuth) | Must build integration layer |
| **Reporting** | Basic Django admin queries | Business intelligence dashboards, IPEDS reports, retention analytics | Need: Reporting engine |
| **Self-Service** | Limited student portal | Full self-service: registration, payment, grades, degree audit | Need: Modern UX |

---

## 5. Architecture Patterns Observed

### OSIS Architectural Patterns
1. **Monolithic Django App with Pluggable Modules**: Each functional area is a separate Django app that can be enabled/disabled
2. **Model-Driven Development**: Heavy use of Django ORM models with business logic in model methods
3. **Traditional MVC**: Django views + templates (server-side rendering)
4. **Permissions at Model Level**: Fine-grained permissions using Django's built-in auth framework
5. **Document Centralization**: `osis-document` provides a reusable file handling service for all modules

### UniPortal Architectural Improvements
1. **Decoupled Frontend**: React SPA for better UX and mobile responsiveness
2. **RESTful API Layer**: Django REST Framework (or FastAPI) to expose backend as API
3. **Microservices-Ready**: Design modules as bounded contexts (registration, billing, grading) that could be split later
4. **Caching Strategy**: Redis for session management and real-time seat counts
5. **Message Queue**: RabbitMQ/Celery for async tasks (enrollment processing, notification emails)
6. **Audit Logging**: Immutable event log for compliance (who registered for what, when)

---

## 6. Five Key Takeaways for UniPortal Design

### 1. **Billing is Non-Negotiable and Complex**
OSIS omits billing entirely due to European free tuition. For US universities, billing is as critical as registration:
- Tuition varies by residency (in-state/out-of-state), program, credit hours, fees
- Payment plans, refunds (prorated by drop date), late fees
- Integration with payment gateways (Stripe, Nelnet)
- 1098-T tax form generation
- **Action**: Billing must be a first-class module with its own data model (Invoice, Payment, RefundSchedule)

### 2. **Real-Time State Management is Essential**
OSIS's batch-oriented enrollment process won't meet expectations for US students who expect instant feedback:
- Seat availability must update in real-time during registration
- Schedule conflicts must be detected immediately before registration
- Waitlist auto-enrollment must happen within seconds of a drop
- **Action**: Use WebSocket or short polling for seat counts; Redis for shared state

### 3. **Prerequisites and Academic Rules Need Runtime Enforcement**
OSIS defines prerequisites in the data model but relies on manual checking:
- UniPortal must block registration attempts that violate prerequisites
- Must enforce credit hour limits (e.g., 18 credits max without dean approval)
- Must prevent enrollment in courses for which student lacks standing (e.g., seniors-only seminar)
- **Action**: Build a rules engine that evaluates `StudentEligibility(student, section) → boolean + reason`

### 4. **Integration is a Product Feature, Not an Afterthought**
Modern universities expect SIS to integrate with 5-10 external systems:
- LMS (Canvas, Blackboard) for roster sync
- Identity provider (SAML/LDAP) for SSO
- Payment gateway (Stripe, Nelnet CASHNet)
- IPEDS reporting (federal compliance)
- Degree audit tools (u.achieve, DegreeWorks)
- **Action**: Design an integration layer with adapters for each system; expose webhooks for real-time sync

### 5. **User Experience is a Competitive Differentiator**
OSIS uses traditional Django templates. Modern students expect consumer-grade UX:
- Mobile-responsive design (50%+ of traffic will be mobile)
- Instant search with autocomplete
- Visual schedule builder (drag-and-drop)
- One-click registration with shopping cart
- **Action**: Invest in React SPA with Tailwind CSS; user-test with students before launch

---

## 7. Recommendations for UniPortal MVP

### Must-Have for MVP (Phase 1)
1. Student registration with course search, cart, and conflict detection
2. Billing with tuition calculation and payment processing
3. Grading portal for faculty
4. Student self-service dashboard (schedule, grades, balance)
5. Hold management (financial, advising)

### Can Defer to Post-MVP
1. Waitlist auto-enrollment (start with manual waitlist)
2. Degree audit (students can track manually via spreadsheet initially)
3. Advanced reporting/BI dashboards (use Django admin for initial queries)
4. Mobile native apps (responsive web is sufficient for MVP)
5. LMS integration (manual roster export for first semester)

### Never Compromise
1. Data integrity (financial transactions must be ACID-compliant)
2. Security (PII protection, PCI DSS for payments, FERPA compliance)
3. Audit logging (immutable record of all registration and grade changes)

---

## Sources

Research for this analysis based on:
- [OSIS Official Documentation](https://uclouvain.github.io/osis/)
- [FOSDEM 2017 - OSIS Presentation](https://archive.fosdem.org/2017/schedule/event/open_student_info_system/)
- OSIS GitHub repositories: [osis-admission](https://github.com/uclouvain/osis-admission), [osis-dissertation](https://github.com/uclouvain/osis-dissertation), [osis-internship](https://github.com/uclouvain/osis-internship), [osis-document](https://github.com/uclouvain/osis-document)
- Django SIS architecture patterns and best practices

---

**Document Version:** 1.0
**Last Updated:** 2026-02-05
**Author:** Requirements Engineering Team
