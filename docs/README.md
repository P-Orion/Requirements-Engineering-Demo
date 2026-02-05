# UniPortal Requirements Engineering Demo

**A comprehensive demonstration of requirements engineering best practices for a university student information system.**

---

## Project Overview

**UniPortal** is a modern web-based Student Information System (SIS) designed for mid-sized universities (~8,000 students). This project demonstrates professional requirements engineering practices including competitive analysis, requirements documentation, data modeling, architecture design, traceability matrices, and interactive prototyping.

### Key Features
- 📚 **Course Registration:** Real-time seat availability, shopping cart, prerequisite validation, conflict detection
- 💵 **Billing & Payments:** Tuition calculation, payment processing, refund schedules, financial holds
- 📊 **Grading:** Faculty grade entry, GPA calculation, transcript generation
- 👤 **Student Portal:** Dashboard, weekly schedule, account balance, degree progress

### Technology Stack
- **Frontend:** React 18, Tailwind CSS (single-page application)
- **Backend:** Django + Django REST Framework (Python)
- **Database:** PostgreSQL (relational data model)
- **Cache:** Redis (real-time seat counts, sessions)
- **Queue:** RabbitMQ + Celery (async tasks, waitlist processing)
- **Integrations:** Canvas LMS, Stripe payments, LDAP authentication

---

## Documentation Index

### 📄 [01 - Competitive Analysis](./01-analysis.md)
**Analysis of OSIS (Open Student Information System)**

Study of a production university SIS to inform UniPortal's design:
- Technology stack and architecture patterns from UCLouvain's OSIS
- Entity/data model extraction (12+ core entities)
- Feature inventory across 7 modules (admission, grading, dissertation, internship, etc.)
- **Comparison table:** OSIS (European context) vs. UniPortal (US university needs)
- **5 strategic takeaways:**
  1. Billing is non-negotiable for US universities
  2. Real-time state management is essential
  3. Prerequisites need runtime enforcement
  4. Integration is a product feature
  5. User experience is a competitive differentiator

**Read this to understand:** How competitive analysis informs product strategy and requirements.

---

### 📄 [02 - Requirements & Data Model](./02-requirements-and-data-model.md)
**Business Rules, Functional Requirements, ER Diagram, and Architecture**

Complete requirements specification with:

#### Business Rules (15 rules, ID'd BR-001 to BR-015)
- Prerequisite enforcement, credit hour limits, time conflict prevention
- Enrollment capacity and waitlist priority logic
- Add/drop deadlines and tuition refund schedules
- Hold system (financial, advising, immunization, conduct)
- GPA calculation, academic standing, and dean's list criteria
- Grade submission deadlines and change authorization
- Concurrent enrollment limits and course repeat policies

#### Functional Requirements (30 requirements, MoSCoW prioritized)
- **Registration Module (10 FRs):** Course search, shopping cart, prerequisite validation, conflict detection, waitlist
- **Grading Module (5 FRs):** Grade entry, submission workflow, GPA calculation, grade changes
- **Billing Module (7 FRs):** Tuition calculation, payment processing, refunds, holds, 1098-T tax forms
- **Student Portal (5 FRs):** Dashboard, schedule view, transcripts, degree audit, hold resolution
- **Administration (5 FRs):** Term setup, catalog management, section creation, reports, user roles

#### Entity-Relationship Diagram
Mermaid ER diagram with **15+ entities:**
- Core: Student, Faculty, Admin, User
- Academic: Course, Section, Term, Prerequisite, Enrollment, WaitlistEntry
- Financial: Invoice, Payment, Hold
- Requirements: DegreeRequirement, RequirementCompletion

**Also available as:** [er-diagram.mermaid](./er-diagram.mermaid)

#### System Architecture Diagram
Microservices-oriented architecture:
- **Client Layer:** React SPA, mobile browsers
- **API Gateway:** NGINX, JWT authentication
- **Application Services:** Registration, Billing, Grading, Portal, Admin (Django)
- **Data Layer:** PostgreSQL, Redis cache
- **Background Processing:** RabbitMQ queue, Celery workers
- **External Integrations:** Canvas LMS, Stripe payments, LDAP, SendGrid email, S3 storage

**Also available as:** [architecture.mermaid](./architecture.mermaid)

**Read this to understand:** How business rules translate to functional requirements and data models.

---

### 📄 [03 - Traceability & Roadmap](./03-traceability-and-roadmap.md)
**Traceability Matrix, Release Plan, and Effort Estimates**

#### Traceability Matrix
**14 key traceability chains** mapping:
- Functional Requirement → Business Rule → API Endpoint → Data Entities → UI Component

**Example chain (FR-REG-004):**
- **Business Rule:** BR-001 (Prerequisite Enforcement)
- **Requirement:** Prevent registration without completed prerequisites
- **API:** `POST /api/enrollment/validate-prerequisites`
- **Entities:** Student, Course, Prerequisite, Enrollment
- **UI:** CartValidation.jsx (error modal with missing prerequisites)

Includes detailed example with JSON API contract and SQL query.

#### 4-Phase Release Roadmap (18 months)

| Phase | Duration | Features | Effort |
|-------|----------|----------|--------|
| **Phase 1: MVP** | 6 months | Core registration, grading, billing, portal | 480 dev-days |
| **Phase 2: v1.1** | 4 months | Waitlist, payment plans, reports, analytics | 240 dev-days |
| **Phase 3: v1.2** | 4 months | Degree audit, transcripts, LMS integration | 320 dev-days |
| **Phase 4: v2.0** | 4 months | Mobile apps (iOS/Android), BI dashboards, ML | 400 dev-days |

**Total:** 1,440 developer-days, estimated **$1,393,800** (includes infrastructure, services, contingency)

#### Go-Live Acceptance Criteria
Functional, performance, security, and operational readiness checklists ensuring MVP quality before launch.

#### Post-Launch Support Plan
- 30-day hypercare period (24/7 on-call)
- Ongoing support strategy
- Maintenance windows and release cadence

**Read this to understand:** How to trace requirements to implementation and plan iterative delivery.

---

## 🎨 Interactive Prototype

### [prototype/index.html](../prototype/index.html)
**Single-file React application demonstrating UniPortal UI**

Open `prototype/index.html` in any modern browser (no build step required) to explore:

#### Dashboard Tab
- **Weekly Schedule:** Calendar grid view with color-coded courses
- **Quick Stats Cards:** GPA (3.67), Account Balance ($2,450), Enrolled Courses (5), Active Holds (1)
- **Active Holds Panel:** Advising hold with resolution guidance
- **Upcoming Deadlines:** Drop deadline, registration dates, payment due dates
- **Current Courses List:** All enrolled courses with titles

#### Registration Tab
- **Course Search:** Search by code, title, or instructor
- **Filters:** Department (CS, MATH, PHIL) and Level (300, 400)
- **Course Catalog (8 courses):**
  - Real-time seat availability
  - Waitlist counts
  - Prerequisite badges
  - Full course details (CRN, instructor, schedule, location)
- **Shopping Cart:**
  - Add/remove courses
  - Total credit hours calculation
  - Estimated tuition preview
  - "Register" button with hold warning
- **Smart Validation:**
  - ✅ **Prerequisite checking:** Try adding CS 450 (Machine Learning) → blocked if missing CS 341/MATH 310
  - ✅ **Time conflict detection:** Try adding courses with overlapping times → error with conflicting course name
  - ✅ **Hold enforcement:** Registration blocked if advising hold active

#### Billing Tab
- **Account Summary:** Total charges, payments made, current balance
- **Payment History Table:** 5 transactions with dates, descriptions, amounts, running balance
- **Make a Payment Form:**
  - Custom amount or quick buttons ($500, full balance)
  - Payment method selector (Credit, Debit, ACH, Payment Plan)
  - Payment plan promotion banner
  - "Process Payment" action with success notification

#### Features
- 🎨 Professional UI with Tailwind CSS and custom gradients
- 🔔 Toast notifications for actions (success, warning, error)
- 📱 Responsive design (works on mobile, tablet, desktop)
- 🎯 Realistic mock data for CS junior student (Sarah Chen)
- ⚡ Instant interactions (no backend required)

**How to use:**
```bash
# Option 1: Open directly in browser
open prototype/index.html

# Option 2: Serve with Python (for CORS/CSP strict environments)
cd prototype
python3 -m http.server 8000
# Navigate to http://localhost:8000
```

**Read this to understand:** How requirements translate to user experience and interface design.

---

## Document Relationships

```
01-analysis.md (Competitive Analysis)
    ↓
    Informs product strategy and gap analysis
    ↓
02-requirements-and-data-model.md (Requirements & Design)
    ↓
    Defines business rules, functional requirements, data model, architecture
    ↓
03-traceability-and-roadmap.md (Traceability & Planning)
    ↓
    Maps requirements to implementation and plans phased delivery
    ↓
prototype/index.html (Interactive Demo)
    ↓
    Demonstrates user experience and validates requirements
```

---

## Requirements Engineering Best Practices Demonstrated

### 1. **Competitive Analysis**
- Studied production system (OSIS) to understand proven patterns
- Identified gaps between European and US university needs
- Extracted architectural patterns and anti-patterns
- Derived strategic recommendations from analysis

### 2. **Business Rules Documentation**
- 15 clearly defined rules with IDs (BR-001 to BR-015)
- Each rule includes rationale and exceptions
- Rules ground functional requirements in business policy
- Traceable from requirement through implementation

### 3. **Functional Requirements Specification**
- 30 requirements organized by module
- Each requirement has:
  - Unique ID (FR-REG-001, FR-GRD-002, etc.)
  - MoSCoW priority (Must/Should/Could/Won't Have)
  - Clear description and acceptance criterion
- Organized by bounded context (Registration, Grading, Billing, Portal, Admin)

### 4. **Data Modeling**
- Entity-relationship diagram with 15+ entities
- All relationships show cardinality (one-to-many, many-to-many)
- Attributes typed (int, string, date, enum, etc.)
- Normalized design (3NF) to prevent data anomalies

### 5. **Architecture Design**
- Multi-layer architecture (client, gateway, services, data)
- Separation of concerns (each service has clear responsibility)
- External integration points identified
- Scalability patterns (cache, queue, async processing)

### 6. **Traceability**
- Requirements mapped to business rules
- Requirements mapped to API endpoints
- Requirements mapped to data entities
- Requirements mapped to UI components
- End-to-end traceability for compliance and change impact analysis

### 7. **Iterative Planning**
- Phased delivery (MVP → v1.1 → v1.2 → v2.0)
- Features prioritized by business value
- Effort estimates grounded in team capacity
- Risk mitigation strategies identified

### 8. **Prototyping**
- Interactive prototype validates requirements with stakeholders
- Realistic data demonstrates system behavior
- UI/UX testing before development begins
- Requirements ambiguities surfaced early through prototyping

---

## How to Use This Demo

### For Requirements Engineers
1. Start with `01-analysis.md` to see competitive analysis methodology
2. Study `02-requirements-and-data-model.md` for requirements documentation patterns
3. Review `03-traceability-and-roadmap.md` for traceability and planning techniques
4. Explore `prototype/index.html` to see how requirements translate to UI

### For Product Managers
1. Review `01-analysis.md` for market positioning and feature prioritization
2. Use `03-traceability-and-roadmap.md` roadmap as template for release planning
3. Reference `02-requirements-and-data-model.md` business rules for policy discussions
4. Demo `prototype/index.html` to stakeholders for early feedback

### For Software Architects
1. Study `02-requirements-and-data-model.md` ER and architecture diagrams
2. Review `03-traceability-and-roadmap.md` for API design patterns
3. Use data model as foundation for database schema design
4. Reference architecture diagram for deployment planning

### For Developers
1. Start with `03-traceability-and-roadmap.md` to understand requirement → implementation mapping
2. Use `02-requirements-and-data-model.md` as API specification reference
3. Reference ER diagram for database schema implementation
4. Study `prototype/index.html` React code for frontend patterns

### For Students Learning Requirements Engineering
This demo provides a complete requirements engineering case study:
- **Analysis Phase:** Competitive analysis, stakeholder research
- **Specification Phase:** Business rules, functional requirements, data models
- **Validation Phase:** Prototyping, traceability verification
- **Planning Phase:** Roadmap, effort estimation, risk mitigation

Use this as a template for your own projects or assignments.

---

## Project Metrics

| Metric | Value |
|--------|-------|
| **Business Rules** | 15 |
| **Functional Requirements** | 30 |
| **Data Entities** | 15+ |
| **Traceability Chains** | 14 detailed examples |
| **Release Phases** | 4 (18 months) |
| **Effort Estimate** | 1,440 developer-days |
| **Budget Estimate** | $1,393,800 |
| **Lines of Documentation** | ~1,500 |
| **Prototype Components** | 3 major tabs (Dashboard, Registration, Billing) |
| **Mock Data Records** | 50+ (courses, students, payments, etc.) |

---

## Document Standards

All documentation follows these standards:
- **Markdown format** for readability and version control
- **Consistent ID schemes:** BR-XXX (Business Rules), FR-XXX-YYY (Functional Requirements)
- **Mermaid diagrams** for data models and architecture (renderable in GitHub, GitLab, VS Code)
- **Versioning:** All documents include version number and last updated date
- **Hyperlinks:** Cross-references between documents for easy navigation
- **Examples:** Each concept illustrated with concrete examples

---

## Credits

**Author:** Requirements Engineering Team
**Project:** UniPortal — University Student Information System
**Date:** February 2026
**Purpose:** Demonstration of requirements engineering best practices

**Inspired by:**
- OSIS (Open Student Information System) by UCLouvain
- Modern SaaS university systems (Ellucian Banner, Workday Student, Oracle PeopleSoft Campus Solutions)
- Requirements engineering standards (IEEE 830, ISO/IEC/IEEE 29148)

---

## License

This requirements engineering demonstration is provided for educational purposes. All content is original work created for this demo project.

---

**Next Steps:**
1. Review all documentation files in order (01 → 02 → 03)
2. Open prototype in browser to see requirements come to life
3. Use this demo as template for your own requirements engineering projects

**Questions or feedback?** This demo was created to showcase professional requirements engineering practices for portfolio and educational purposes.
