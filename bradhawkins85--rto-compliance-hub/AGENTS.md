# Compliance Management and Governance Platform for RTOs

## Objective
Design and implement a comprehensive, modular compliance management system for Registered Training Organizations (RTOs). The system centralizes governance, training, feedback, compliance tracking, and resource management into one platform.

---

## 1. Core Modules & Functional Requirements

### 1.1 Governance
- Manage and store policies, procedures, and compliance documentation.
- Track financial compliance, child safety, leadership fit and proper tests, workforce adequacy, and insurance documentation.
- Support continuous improvement workflows with version control and audit history.
- Role-based permissions for editing and reviewing policies.
- Integration with Google Drive for document storage and linking.

### 1.2 Training Product Management
- Link training products to associated SOPs, validation reports, and instructional materials.
- Map assessment strategies and schedules for each training product.
- Support both accredited and non-accredited training.
- AI analysis to identify misalignments or outdated materials.

### 1.3 Feedback Management
- **Learner Feedback:** Collect via surveys (JotForm) by trainer, course, or learner type.
- **Employer Feedback:** Capture outcomes-based metrics and anecdotal input.
- **Industry Feedback:** Gather surveys and audio anecdotes from stakeholders (employers, safety officers, training coordinators).
- Integrate with AI sentiment and trend analysis for feedback aggregation.
- Export summaries and generate compliance insights.

### 1.4 Professional Development
- Manage staff PD cycles from planning to completion.
- Track vocational and industry currency, including work placements and skill maintenance.
- Store credentials, certificates, and expiration reminders.
- Link PD activities to compliance standards and staff records.

### 1.5 Resource Management
- **Assets:** Track cranes, plant, tablets, laptops, lifting equipment, and maintenance schedules.
- **Infrastructure:** Manage classrooms, offices, exclusion zones, and training yards.
- **Learning Resources:** Central library of guides, videos, and podcasts linked to training modules.
- **Systems Integration:** Connect to third-party systems (SMS, LMS, CRM, Xero).

### 1.6 Complaints & Appeals
- Workflow-driven process for complaint logging, tracking, and resolution.
- Capture student demographics and incident details.
- Link complaint outcomes to policy and continuous improvement actions.
- Automated reporting to meet compliance obligations.

### 1.7 Compliance Tracking & Mapping
- Compliance map linking all 29+ RTO standards to related policies, procedures, training, and evidence.
- Dashboard to visualize current compliance status.
- Automated reminders for renewals, audits, or missing documentation.

### 1.8 HR & Onboarding
- Centralized staff database syncing with Xero for payroll and positions.
- Onboarding workflows integrated with compliance documents and PD requirements.
- Department-level access control (training, admin, management).
- API connections to Accelerate for trainer and student details.

---

## 2. Data Architecture & Integrations
- **Database:** Relational (PostgreSQL / Airtable equivalent).
  - Tables: Staff, Training, Policies, Assets, Feedback, Standards, Complaints.
- **APIs & Integrations:**
  - JotForm (webhooks for data ingestion).
  - Airtable or internal SQL backend.
  - Accelerate API for student/trainer data.
  - Xero API for payroll sync.
  - n8n for automation workflows.
  - Grafana for dashboards.
  - Google Drive API for document links and storage.

---

## 3. Automation & AI Features
- **AI Analysis:** Process student and employer feedback to detect trends and improvement opportunities.
- **Auto-Reminders:** Schedule compliance renewals, PD expirations, and document reviews.
- **Smart Workflows:** Conditional logic for forms (e.g., training type triggers relevant SOP).
- **Predictive Insights:** Forecast compliance gaps or risks based on feedback and training trends.

---

## 4. User Experience
- Portal interface for staff with personalized dashboards.
- Dynamic navigation by role (trainer, admin, leadership).
- Search and filter across all modules (resources, policies, training).
- Embedded JotForm and internal forms for feedback and submissions.
- Consistent branding and responsive design for desktop and mobile.

---

## 5. Security & Permissions
- Role-based access control (RBAC) for users and departments.
- Secure API authentication (OAuth2 / JWT).
- Audit logs for all document changes and user actions.
- Encrypted data storage for PII and compliance evidence.

---

## 6. Future Features (Scalable Roadmap)
- AI compliance auditor for proactive alerts.
- Predictive staff training recommendations.
- Self-service employer and auditor portals.
- ChatGPT integration for document Q&A (policy search assistant).
- Data visualizations for compliance trends and performance metrics.

---

## 7. Technical Stack Recommendations
- **Frontend:** React (Next.js) + TailwindCSS
- **Backend:** Node.js / FastAPI
- **Database:** PostgreSQL (with Prisma ORM or Supabase)
- **Automation:** n8n or Temporal for workflows
- **Analytics:** Grafana + Prometheus
- **AI Layer:** OpenAI API or Azure Cognitive Services
- **Hosting:** AWS / Vercel / GCP (depending on integration needs)

---

## 8. Example User Stories
- As an **admin**, I want to see which compliance standards are not yet mapped to any policy so I can fill the gaps.
- As a **trainer**, I need to complete and sign SOP training via JotForm and see my progress dashboard.
- As a **manager**, I want to monitor PD completion rates and compliance metrics across departments.
- As a **compliance officer**, I need automated alerts when any credential or insurance is about to expire.

---

## 9. Suggested Repo Structure
```
/compliance-platform
├── frontend/ (React app)
├── backend/ (API services)
│   ├── auth/
│   ├── integrations/ (Accelerate, Xero, JotForm)
│   ├── ai/
│   ├── compliance/
│   ├── feedback/
│   ├── training/
│   ├── hr/
├── database/
│   ├── schema.prisma
│   ├── migrations/
├── workflows/ (n8n or Temporal scripts)
├── docs/
│   ├── PRD.md
│   ├── API.md
│   ├── Integration_Specs.md
└── README.md
```

---

This PRD is designed to guide GitHub Copilot and GitHub Spark in generating architecture, models, workflows, and integration code for each module of the Compliance Management Platform.


---

## 10. Acceptance Criteria (per Module)

### 10.1 Governance
- **Create/Update Policy**: Given a user with `ComplianceAdmin` role, when they upload or link a policy (PDF/Doc), then the system stores metadata (title, version, owner, review date) and version history, and the policy is searchable within 5 seconds.
- **Version Control**: When a policy is superseded, the system preserves prior versions and marks only one version as `Current` at any time.
- **Standards Mapping**: A policy can be mapped to ≥1 compliance standards; dashboard immediately reflects counts of mapped/unmapped items.
- **Access Control**: Users without `ComplianceAdmin` can view `Published` policies but cannot edit; attempts are logged.
- **Review Workflow**: Policies with review dates in ≤30 days appear in a `Due for Review` queue and notify owners daily until resolved.

### 10.2 Training Product Management
- **Linking**: A training product record must include links to SOPs, validation docs, and assessment strategy. Missing links flag the product as `Incomplete`.
- **Schedule**: Trainers assigned to a product see due training tasks on their dashboard within 60 seconds of assignment.
- **Audit Trail**: All edits (who/when/what) are visible in the product’s history tab.

### 10.3 Feedback Management
- **Ingestion**: Completing a JotForm survey triggers a webhook; data is persisted within 3 seconds and visible in the `Feedback` table.
- **Anonymity**: If the submitter opts out of identification, PII fields are not stored; analytics still aggregates results.
- **AI Insights**: Daily job produces sentiment score (–1..1) and top 5 themes for the last 30/90 days with precision/recall ≥0.7 on test set.
- **Filtering**: Users can filter feedback by course, trainer, employer type, date range; results return in <2 seconds for up to 10k records.

### 10.4 Professional Development (PD)
- **Credential Tracking**: Each credential has an issue date and expiry; system calculates next due date and shows `Due` (≤30 days) and `Overdue` states.
- **Completion Evidence**: Submitting PD via form attaches proof (file/link); record becomes `Verified` only after manager approval.
- **Notifications**: Daily digest email/WhatsApp-compatible webhook to each staff member and manager for items due in the next 7/30 days.

### 10.5 Resource Management
- **Asset Lifecycle**: Create, assign, service, retire states must be supported; servicing schedule creates tasks automatically by time or usage metric.
- **Evidence**: Upload service reports and invoices; assets show `Compliant` only if last service ≤ scheduled interval.

### 10.6 Complaints & Appeals
- **Workflow**: Statuses = `New` → `In Review` → `Actioned` → `Closed`; each transition requires notes. SLA breach flags if `New` not moved within 2 business days.
- **Linkage**: Complaint records can be linked to policies, staff, and training sessions; closure requires root cause and corrective action.

### 10.7 Compliance Tracking & Mapping
- **Coverage**: Standards catalog maintained; dashboard shows % mapped to policies, SOPs, training, and evidence.
- **Gaps Report**: Export CSV/PDF within 10 seconds for up to 500 standards-to-evidence links.

### 10.8 HR & Onboarding
- **Sync**: Xero/Accelerate sync runs daily (and on demand) and creates/updates staff records; duplicates resolved via email or external ID.
- **Onboarding Pack**: New staff automatically receive assigned SOP/PD forms based on department/role; completion tracked.
- **Permissions**: Department membership determines visible forms and modules; changes reflect on next login.

### 10.9 Performance & Security (Cross-cutting)
- **Latency**: 95th percentile API response <500 ms for typical queries; background jobs can exceed.
- **Availability**: 99.5% monthly uptime (excluding planned maintenance).
- **Security**: RBAC enforced on every endpoint; PII at rest encrypted; all changes audit-logged.

---

## 11. API Endpoint Specifications (v1)

> Base URL: `/api/v1`
> Auth: Bearer JWT (OAuth2). All endpoints return RFC 7807 error objects on failure.

### 11.1 Auth
- **POST** `/auth/login`
  - **Body**: `{ "email": string, "password": string }`
  - **200**: `{ "access_token": string, "expires_in": number, "role": string }`

### 11.2 Users & HR
- **GET** `/users` — list (filter by `department`, `role`, `q`).
- **POST** `/users` — create user.
- **GET** `/users/{id}` — retrieve.
- **PATCH** `/users/{id}` — update (department, role, status).
- **POST** `/users/{id}/credentials` — add credential.
- **GET** `/users/{id}/pd` — list PD records.
- **POST** `/sync/xero` — trigger Xero sync (admin only).
- **POST** `/sync/accelerate` — trigger Accelerate sync (admin only).

**User Schema**
```json
{
  "id": "uuid",
  "name": "string",
  "email": "string",
  "department": "Training|Admin|Management|Support",
  "roles": ["Trainer", "SystemAdmin", "ComplianceAdmin"],
  "status": "Active|Inactive"
}
```

### 11.3 Policies & Governance
- **GET** `/policies` — list (filter by `standardId`, `status`).
- **POST** `/policies` — create (metadata + file link).
- **GET** `/policies/{id}` — details + version history.
- **PATCH** `/policies/{id}` — update metadata.
- **POST** `/policies/{id}/publish` — publish new version.
- **POST** `/policies/{id}/map` — map to standards: `{ standardIds: string[] }`.

### 11.4 Standards
- **GET** `/standards` — list all standards & clauses.
- **GET** `/standards/{id}/mappings` — policies, SOPs, evidence linked to a standard.

### 11.5 Training Products & SOPs
- **GET** `/training-products` — list.
- **POST** `/training-products` — create.
- **GET** `/training-products/{id}` — retrieve, including linked SOPs, assessments.
- **POST** `/training-products/{id}/sops` — link SOPs: `{ sopIds: string[] }`.
- **GET** `/sops/{id}` — SOP detail (policy link, latest version).

### 11.6 Feedback & Webhooks
- **POST** `/webhooks/jotform` — JotForm ingestion endpoint.
  - **Body**: opaque JotForm payload; server validates signature and maps fields.
  - **202**: enqueued for processing, returns `{ submissionId }`.
- **GET** `/feedback` — list with filters (`type=learner|employer|industry`, `courseId`, `trainerId`, `dateFrom`, `dateTo`, `anonymous`).
- **GET** `/feedback/insights` — aggregated metrics and AI themes.

**Feedback Record**
```json
{
  "id": "uuid",
  "type": "learner|employer|industry",
  "courseId": "string|null",
  "trainerId": "string|null",
  "rating": 0,
  "comments": "string|null",
  "anonymous": true,
  "createdAt": "iso8601"
}
```

### 11.7 PD & Credentials
- **GET** `/pd` — query PD items (filters: `userId`, `status`, `dueBefore`).
- **POST** `/pd` — create PD item: `{ userId, title, evidenceUrl?, hours?, dueAt }`.
- **POST** `/pd/{id}/complete` — mark as complete with evidence: `{ evidenceUrl, completedAt }`.
- **GET** `/credentials` — list credentials (filters: `userId`, `expiresBefore`).

### 11.8 Assets & Resources
- **GET** `/assets` — list (filters: `type`, `status`, `location`).
- **POST** `/assets` — create asset.
- **PATCH** `/assets/{id}` — update.
- **POST** `/assets/{id}/service` — log service: `{ date, notes, documents[] }`.
- **POST** `/assets/{id}/state` — transition lifecycle `{ state: "Assigned|Servicing|Retired" }`.

### 11.9 Complaints & Appeals
- **POST** `/complaints` — create: `{ source, description, studentId?, trainerId?, courseId? }`.
- **GET** `/complaints` — list (filters: `status`, `dateFrom`, `dateTo`).
- **PATCH** `/complaints/{id}` — update status/notes/rootCause/correctiveAction.
- **POST** `/complaints/{id}/close` — close with required fields.

### 11.10 Compliance Dashboard
- **GET** `/compliance/coverage` — % mapping coverage by standard and artifact type.
- **GET** `/compliance/gaps` — exportable report (CSV/PDF selection via `Accept` header).

### 11.11 Files & Evidence
- **POST** `/files/sign-url` — get pre-signed upload URL for evidence.
- **POST** `/evidence` — attach evidence to any entity `{ entityType, entityId, url, description }`.

### 11.12 Notifications & Jobs
- **GET** `/jobs` — list scheduled/background jobs and next run time (admin).
- **POST** `/jobs/run` — trigger named job: `{ name: "syncXero|syncAccelerate|pdReminders|policyReviews" }`.

---

## 12. Non-Functional Requirements
- **Observability**: Structured logging (trace IDs), metrics for API latency, job durations, webhook failures; alerting thresholds defined.
- **Internationalization**: All user-visible strings externalized; AU date/time formats default.
- **Data Retention**: Complaint and feedback data retained ≥7 years; right-to-erasure flow for PII where lawful.

---

## 13. OpenAPI Stub
A future task will generate an OpenAPI 3.1 spec from these endpoints. Acceptance: `npm run build:openapi` produces a valid spec, and CI validates with `spectral`.


---

## 14. OpenAPI 3.1 Spec (Draft)

```yaml
openapi: 3.1.0
info:
  title: Compliance Platform API
  version: 1.0.0
  description: REST API for RTO Compliance Management & Governance Platform
servers:
  - url: https://api.example.com/api/v1
security:
  - bearerAuth: []
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    ProblemDetails:
      type: object
      properties:
        type: { type: string }
        title: { type: string }
        status: { type: integer }
        detail: { type: string }
        instance: { type: string }
    User:
      type: object
      required: [id, name, email, department, roles, status]
      properties:
        id: { type: string, format: uuid }
        name: { type: string }
        email: { type: string, format: email }
        department:
          type: string
          enum: [Training, Admin, Management, Support]
        roles:
          type: array
          items:
            type: string
            enum: [Trainer, SystemAdmin, ComplianceAdmin]
        status:
          type: string
          enum: [Active, Inactive]
    Credential:
      type: object
      properties:
        id: { type: string, format: uuid }
        userId: { type: string, format: uuid }
        name: { type: string }
        issuedAt: { type: string, format: date }
        expiresAt: { type: string, format: date }
        evidenceUrl: { type: string, format: uri }
    Policy:
      type: object
      properties:
        id: { type: string, format: uuid }
        title: { type: string }
        version: { type: string }
        ownerId: { type: string, format: uuid }
        status:
          type: string
          enum: [Draft, Published, Archived]
        reviewDate: { type: string, format: date }
        fileUrl: { type: string, format: uri }
        mappedStandardIds:
          type: array
          items: { type: string }
    Standard:
      type: object
      properties:
        id: { type: string }
        title: { type: string }
        clause: { type: string }
    TrainingProduct:
      type: object
      properties:
        id: { type: string, format: uuid }
        code: { type: string }
        name: { type: string }
        status: { type: string, enum: [Active, Inactive] }
        sopIds: { type: array, items: { type: string, format: uuid } }
        assessmentStrategyUrl: { type: string, format: uri }
    SOP:
      type: object
      properties:
        id: { type: string, format: uuid }
        title: { type: string }
        policyId: { type: string, format: uuid }
        version: { type: string }
        fileUrl: { type: string, format: uri }
    Feedback:
      type: object
      properties:
        id: { type: string, format: uuid }
        type: { type: string, enum: [learner, employer, industry] }
        courseId: { type: string, nullable: true }
        trainerId: { type: string, nullable: true }
        rating: { type: number, minimum: 0 }
        comments: { type: string, nullable: true }
        anonymous: { type: boolean }
        createdAt: { type: string, format: date-time }
    PDItem:
      type: object
      properties:
        id: { type: string, format: uuid }
        userId: { type: string, format: uuid }
        title: { type: string }
        hours: { type: number, nullable: true }
        dueAt: { type: string, format: date-time }
        status: { type: string, enum: [Planned, Due, Overdue, Completed, Verified] }
        evidenceUrl: { type: string, format: uri, nullable: true }
    Asset:
      type: object
      properties:
        id: { type: string, format: uuid }
        type: { type: string }
        name: { type: string }
        location: { type: string }
        status: { type: string, enum: [Available, Assigned, Servicing, Retired] }
        lastServiceAt: { type: string, format: date }
        documents: { type: array, items: { type: string, format: uri } }
    Complaint:
      type: object
      properties:
        id: { type: string, format: uuid }
        source: { type: string }
        description: { type: string }
        status: { type: string, enum: [New, InReview, Actioned, Closed] }
        studentId: { type: string, nullable: true }
        trainerId: { type: string, nullable: true }
        courseId: { type: string, nullable: true }
        rootCause: { type: string, nullable: true }
        correctiveAction: { type: string, nullable: true }
        createdAt: { type: string, format: date-time }
    Evidence:
      type: object
      properties:
        id: { type: string, format: uuid }
        entityType: { type: string }
        entityId: { type: string }
        url: { type: string, format: uri }
        description: { type: string, nullable: true }

paths:
  /auth/login:
    post:
      summary: Login and receive JWT
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email: { type: string, format: email }
                password: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: object
                properties:
                  access_token: { type: string }
                  expires_in: { type: integer }
                  role: { type: string }
        default:
          description: Error
          content:
            application/json:
              schema: { $ref: '#/components/schemas/ProblemDetails' }

  /users:
    get:
      summary: List users
      parameters:
        - in: query
          name: department
          schema: { type: string }
        - in: query
          name: role
          schema: { type: string }
        - in: query
          name: q
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/User' }
    post:
      summary: Create user
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/User' }
      responses:
        '201': { description: Created }

  /users/{id}:
    get:
      summary: Get user
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema: { $ref: '#/components/schemas/User' }
    patch:
      summary: Update user
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
      responses:
        '200': { description: Updated }

  /users/{id}/credentials:
    post:
      summary: Add credential to user
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Credential' }
      responses:
        '201': { description: Created }

  /users/{id}/pd:
    get:
      summary: List PD for user
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/PDItem' }

  /sync/xero:
    post:
      summary: Trigger Xero sync
      responses:
        '202': { description: Accepted }

  /sync/accelerate:
    post:
      summary: Trigger Accelerate sync
      responses:
        '202': { description: Accepted }

  /policies:
    get:
      summary: List policies
      parameters:
        - in: query
          name: standardId
          schema: { type: string }
        - in: query
          name: status
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/Policy' }
    post:
      summary: Create policy
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Policy' }
      responses:
        '201': { description: Created }

  /policies/{id}:
    get:
      summary: Get policy with versions
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Policy' }
    patch:
      summary: Update policy
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
      responses:
        '200': { description: Updated }

  /policies/{id}/publish:
    post:
      summary: Publish a new version of a policy
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200': { description: Published }

  /policies/{id}/map:
    post:
      summary: Map policy to standards
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                standardIds:
                  type: array
                  items: { type: string }
      responses:
        '200': { description: Mapped }

  /standards:
    get:
      summary: List standards
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/Standard' }

  /standards/{id}/mappings:
    get:
      summary: Get mappings for a standard
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200': { description: OK }

  /training-products:
    get:
      summary: List training products
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/TrainingProduct' }
    post:
      summary: Create training product
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/TrainingProduct' }
      responses:
        '201': { description: Created }

  /training-products/{id}:
    get:
      summary: Get training product
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema: { $ref: '#/components/schemas/TrainingProduct' }

  /training-products/{id}/sops:
    post:
      summary: Link SOPs to training product
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                sopIds:
                  type: array
                  items: { type: string }
      responses:
        '200': { description: Linked }

  /sops/{id}:
    get:
      summary: Get SOP detail
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200': { description: OK }

  /webhooks/jotform:
    post:
      summary: Receive JotForm submission
      responses:
        '202':
          description: Accepted for processing

  /feedback:
    get:
      summary: List feedback
      parameters:
        - in: query
          name: type
          schema: { type: string, enum: [learner, employer, industry] }
        - in: query
          name: courseId
          schema: { type: string }
        - in: query
          name: trainerId
          schema: { type: string }
        - in: query
          name: dateFrom
          schema: { type: string, format: date }
        - in: query
          name: dateTo
          schema: { type: string, format: date }
        - in: query
          name: anonymous
          schema: { type: boolean }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/Feedback' }

  /feedback/insights:
    get:
      summary: Aggregated metrics and AI themes
      responses:
        '200': { description: OK }

  /pd:
    get:
      summary: Query PD items
      parameters:
        - in: query
          name: userId
          schema: { type: string }
        - in: query
          name: status
          schema: { type: string }
        - in: query
          name: dueBefore
          schema: { type: string, format: date }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/PDItem' }
    post:
      summary: Create PD item
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/PDItem' }
      responses:
        '201': { description: Created }

  /pd/{id}/complete:
    post:
      summary: Complete PD item
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                evidenceUrl: { type: string, format: uri }
                completedAt: { type: string, format: date-time }
      responses:
        '200': { description: Completed }

  /credentials:
    get:
      summary: List credentials
      parameters:
        - in: query
          name: userId
          schema: { type: string }
        - in: query
          name: expiresBefore
          schema: { type: string, format: date }
      responses:
        '200': { description: OK }

  /assets:
    get:
      summary: List assets
      parameters:
        - in: query
          name: type
          schema: { type: string }
        - in: query
          name: status
          schema: { type: string }
        - in: query
          name: location
          schema: { type: string }
      responses:
        '200': { description: OK }
    post:
      summary: Create asset
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Asset' }
      responses:
        '201': { description: Created }

  /assets/{id}:
    patch:
      summary: Update asset
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
      responses:
        '200': { description: Updated }

  /assets/{id}/service:
    post:
      summary: Log service event for asset
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                date: { type: string, format: date }
                notes: { type: string }
                documents:
                  type: array
                  items: { type: string, format: uri }
      responses:
        '201': { description: Created }

  /assets/{id}/state:
    post:
      summary: Transition asset state
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                state:
                  type: string
                  enum: [Assigned, Servicing, Retired]
      responses:
        '200': { description: Updated }

  /complaints:
    post:
      summary: Create complaint
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Complaint' }
      responses:
        '201': { description: Created }
    get:
      summary: List complaints
      parameters:
        - in: query
          name: status
          schema: { type: string }
        - in: query
          name: dateFrom
          schema: { type: string, format: date }
        - in: query
          name: dateTo
          schema: { type: string, format: date }
      responses:
        '200': { description: OK }

  /complaints/{id}:
    patch:
      summary: Update complaint
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
      responses:
        '200': { description: Updated }

  /complaints/{id}/close:
    post:
      summary: Close complaint
      parameters:
        - in: path
          name: id
          required: true
          schema: { type: string }
      responses:
        '200': { description: Closed }

  /compliance/coverage:
    get:
      summary: Coverage by standard and artifact
      responses:
        '200': { description: OK }

  /compliance/gaps:
    get:
      summary: Export gaps report
      responses:
        '200': { description: OK }

  /files/sign-url:
    post:
      summary: Get pre-signed upload URL
      responses:
        '200': { description: OK }

  /evidence:
    post:
      summary: Attach evidence to an entity
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Evidence' }
      responses:
        '201': { description: Created }

  /jobs:
    get:
      summary: List scheduled/background jobs
      responses:
        '200': { description: OK }

  /jobs/run:
    post:
      summary: Trigger named job
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                  enum: [syncXero, syncAccelerate, pdReminders, policyReviews]
      responses:
        '202': { description: Accepted }
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bradhawkins85) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
