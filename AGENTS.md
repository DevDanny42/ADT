# AGENTS.md --- PlacementOS

> **Project:** PlacementOS --- The Agentic Placement Intelligence &
> Career Readiness Platform\
> **Tagline:** The Intelligence Layer Between Education and Employment.\
> **Purpose of this file:** This is the shared project brief and
> implementation guide for human contributors and AI coding agents. Read
> it before making architectural, product, or code changes.

------------------------------------------------------------------------

## 1. Product Vision

PlacementOS is an evidence-first, multi-tenant placement intelligence
platform for colleges, universities, students, and recruiters.

The central problem is not simply that students cannot find jobs.
Placement departments often lack a reliable, current understanding of
which students are ready for which roles. Static resumes, manual
eligibility checks, disconnected coding-platform profiles, and reactive
training make it difficult to identify skill gaps and coordinate
placements.

PlacementOS turns student activity and assessment results into
explainable competency evidence. It helps students understand what to
improve, gives them practical ways to demonstrate improvement, and helps
placement teams coordinate opportunities using transparent rules and
human oversight.

**Do not build this as another generic job portal.** Build an
intelligence and workflow layer connecting education, verified
competencies, and employment opportunities.

### Product principles

1.  **Evidence before claims:** A skill should be supported by
    observable evidence wherever possible.
2.  **Explainability:** Recommendations and matches must show the
    reasons and evidence behind them.
3.  **Human oversight:** AI can assist and recommend; authorized people
    retain control over consequential placement decisions.
4.  **Deterministic eligibility:** Formal eligibility rules are
    evaluated by application code, not by an LLM.
5.  **Privacy and consent:** Integrations and processing must have a
    clear purpose, explicit consent, and revocation paths.
6.  **Fairness:** Do not use protected or sensitive characteristics to
    rank, penalize, or exclude candidates.
7.  **Multi-tenancy:** College data must remain isolated from other
    colleges and university tenants.
8.  **Useful progress:** Learning activities should create opportunities
    to produce new evidence, not just display recommendations.
9.  **Honest integrations:** Never claim to retrieve data that a
    third-party platform does not expose or that the user has not
    authorized.
10. **Working vertical slices:** Prefer a small, complete, demonstrable
    workflow over many disconnected screens.

------------------------------------------------------------------------

## 2. Users, Roles, and Portals

### Student

Students should be able to:

-   Create and maintain a profile, education details, interests, target
    roles, and career preferences.
-   Connect supported developer and learning platforms through approved
    authorization flows.
-   View a **Skill DNA** profile showing skills, proficiency estimates,
    confidence, and supporting evidence.
-   Review repositories, contributions, coding activity, certificates,
    badges, and assessment results that are legitimately accessible.
-   Select a target role or analyze a job description.
-   See skill gaps between their evidence-backed profile and a target
    role.
-   Receive a personalized learning path with practical tasks.
-   Complete aptitude, coding, and technical interview simulations.
-   Track improvement and add new evidence to their profile.
-   Discover placement drives and understand eligibility requirements
    and results.
-   Manage consent, connected accounts, and data-sharing preferences.

### Training & Placement (T&P) Officer / College Admin

Authorized college staff should be able to:

-   View a **Placement War Room** dashboard for placement readiness and
    drive activity.
-   Inspect cohort-level skill distributions and skill-gap heatmaps.
-   Compare student competencies with industry and job-role
    requirements.
-   Create and manage placement drives, job descriptions, eligibility
    rules, schedules, and communication.
-   Run mock placement drives and assessments.
-   Review explainable candidate matches and evidence.
-   Approve, reject, or modify AI-assisted recommendations where policy
    permits.
-   Monitor placement progress, interview stages, offers, and outcomes.
-   Export institutional and accreditation-oriented reports.
-   Review audit records and fairness indicators.

### Recruiter

Recruiters should be able to:

-   Maintain a company profile and create job descriptions.
-   Define role requirements and hiring stages.
-   Search candidates using approved, relevant competency evidence.
-   Review evidence-backed profiles and explainable match reasons.
-   Create shortlists and coordinate interviews.
-   Submit interview feedback and offer updates.
-   Access only the students and data shared with them under the
    platform's permissions.

### University Super Admin

University-level administrators should be able to:

-   Create and manage college tenants.
-   Configure tenant-level policies and access.
-   View permitted cross-college aggregate reports.
-   Compare institutional readiness and placement outcomes.
-   Manage university-level users and governance settings.
-   Ensure tenant isolation and policy compliance.

### Suggested role model

Use role-based access control (RBAC). A starting set of roles:

-   `STUDENT`
-   `TNP_OFFICER`
-   `COLLEGE_ADMIN`
-   `RECRUITER`
-   `UNIVERSITY_ADMIN`
-   `PLATFORM_ADMIN`

Do not rely on frontend role checks alone. Enforce permissions in the
backend for every protected operation.

------------------------------------------------------------------------

## 3. Core Product Modules

### 3.1 Student Profile and Skill DNA

Create a living competency profile rather than relying only on a
manually entered resume.

Each skill record should be distinguishable from the evidence supporting
it. A profile may include:

-   Skill name and category.
-   Estimated proficiency.
-   Confidence score or confidence band.
-   Last observed date.
-   Evidence count and evidence types.
-   Verification status.
-   Target role relevance.
-   Source and provenance.

Suggested evidence lifecycle:

1.  **Claimed** --- entered by the student but not independently
    supported.
2.  **Observed** --- detected in a connected source or activity.
3.  **Evidence-linked** --- linked to a specific repository,
    contribution, certificate, or assessment.
4.  **Verified** --- supported by an appropriate verification method,
    such as a practical assessment or reviewed work.
5.  **Expired** --- old or no longer sufficiently current evidence.

Do not label a skill "verified" merely because a keyword appears in a
resume or repository.

### 3.2 Developer Contribution Intelligence

Where supported and authorized, integrate:

-   GitHub: public repositories, languages, commits, pull requests,
    issues, and contribution summaries.
-   LeetCode: accessible public profile statistics and other data
    permitted by the platform.
-   HackerRank: profile details, badges, certificates, and other
    supported public or authorized information.

Show the source and retrieval time for imported evidence. Distinguish
the student's own contributions from repository activity by other
contributors wherever the source data allows.

**Integration limitations and safeguards:**

-   GitHub REST/GraphQL access is subject to scopes, permissions, and
    rate limits. Use OAuth with least privilege when authentication is
    required.
-   LeetCode's commonly used GraphQL endpoint is not a stable,
    documented public developer API. Treat it as an optional,
    best-effort integration. It can change or become unavailable.
-   HackerRank's official enterprise APIs are not a general-purpose API
    for every public user's full practice history. Do not promise
    complete activity access.
-   Never request a user's platform password, session cookie, or private
    token in a form.
-   If a platform does not expose data, allow the student to provide a
    certificate or use an in-platform assessment instead.
-   Make integrations independently disableable so one provider outage
    does not break the whole profile.

### 3.3 Job Description and Role Intelligence

Allow students, T&P officers, and recruiters (according to permissions)
to submit a job description.

The system should extract and structure:

-   Role title and seniority.
-   Required and preferred skills.
-   Education or experience requirements.
-   Relevant tools and technologies.
-   Assessment or hiring stages, when present.
-   Source text and extraction confidence.

Keep extracted requirements traceable to the original job description.
Let an authorized user review and edit the parsed requirements before
they become formal placement criteria.

### 3.4 Skill Gap Analysis

Compare a student's evidence-backed profile with a target role.

Example target: **Java Backend Developer**

Potential skills:

-   Java
-   Spring Boot
-   REST APIs
-   SQL and relational database design
-   Git
-   Docker
-   Kubernetes
-   Testing
-   System design fundamentals

For each requirement, show:

-   Required level or importance.
-   Student's current evidence-backed level.
-   Evidence supporting the estimate.
-   Gap severity.
-   Confidence and freshness.
-   Suggested next action.

Avoid presenting an uncertain AI estimate as an objective fact.

### 3.5 Personalized Learning Path --- Proof-to-Progress Engine

Generate a sequence of practical tasks based on skill gaps and the
student's target role.

A useful learning task should specify:

-   Skill being improved.
-   Learning objective.
-   Instructions and expected deliverable.
-   Estimated effort.
-   Evaluation rubric.
-   How completion will produce evidence.
-   Prerequisites and next task.

The Proof-to-Progress Engine should connect completed tasks to new
evidence. For example, completing a Spring Boot REST API exercise can
produce a code submission or assessment result that updates the
student's profile after validation.

Do not award verified proficiency solely because a student clicked
"complete."

### 3.6 Placement Flight Simulator

Provide realistic, practice-oriented placement preparation:

-   Aptitude and reasoning assessments.
-   Coding challenges.
-   Role-specific technical questions.
-   Adaptive technical interview sessions.
-   Optional project walkthrough and HR practice.
-   Rubric-based feedback and improvement suggestions.
-   Progress history and skill evidence, where appropriate.

Clearly label simulations as practice. Do not misrepresent generated
interview feedback as a real employer decision.

### 3.7 Placement War Room

Provide a focused dashboard for T&P teams:

-   Placement drive status and upcoming milestones.
-   Eligible, interested, applied, shortlisted, interviewed, and offered
    counts.
-   Cohort readiness overview.
-   Skill distribution and gap heatmaps.
-   Role-wise readiness.
-   Assessment participation and outcomes.
-   Pending officer approvals.
-   Interview and offer progress.
-   Exportable reports.

Use aggregate views by default. Reveal individual student data only to
users with a legitimate permission and purpose.

### 3.8 Industry Skill-Gap Profiler

Aggregate job descriptions and role requirements to identify:

-   Frequently requested skills.
-   Skills that are underrepresented in a cohort.
-   Role-specific readiness gaps.
-   Learning priorities for a department or cohort.

Show the data period, source count, and confidence. Do not imply that a
small or biased set of job descriptions represents the entire labor
market.

### 3.9 Eligibility and Placement Drive Management

Support:

-   Company and job management.
-   Placement drive creation.
-   Versioned eligibility criteria.
-   Student applications and preferences.
-   Schedule and interview-stage management.
-   Officer approvals.
-   Offers and placement outcomes.
-   Notifications and audit history.

**Eligibility must be deterministic.** Store criteria in a structured,
versioned form and evaluate them with tested Java code. An LLM may help
parse a job description into draft criteria, but an authorized human
must review those criteria before use.

For every eligibility result, retain:

-   Rule-set version.
-   Inputs used.
-   Evaluation timestamp.
-   Pass/fail result for each criterion.
-   Overall result.
-   Explanation.
-   Actor or process that initiated the evaluation.

### 3.10 Explainable Candidate Matching

Candidate matching should return a transparent explanation, not just a
score.

Possible factors include:

-   Relevant verified skills.
-   Evidence freshness and strength.
-   Practical assessment performance.
-   Role-specific requirements.
-   Student's stated preferences and consent.

Show which requirements matched, which were missing, and which could not
be assessed. Make weights configurable and documented. Do not use
protected or sensitive attributes, proxies for them, or irrelevant
personal information.

A match score is decision support, not a hiring decision.

------------------------------------------------------------------------

## 4. AI Agent Architecture

Use specialized agents for bounded tasks, coordinated by a central
orchestrator. Agents should call explicit backend tools with validated
input/output schemas. They must not receive unrestricted database or
external-system access.

### 4.1 Proposed agents

#### Skill Verification Agent

Responsibilities:

-   Inspect permitted evidence and assessment results.
-   Identify which skill claims have supporting evidence.
-   Flag weak, stale, contradictory, or missing evidence.
-   Produce an evidence-linked explanation and confidence estimate.

It must not claim that a student authored work unless the available
source supports that conclusion.

#### Skill Gap Agent

Responsibilities:

-   Compare a student's evidence-backed profile with reviewed role
    requirements.
-   Identify missing, weak, stale, and uncertain skills.
-   Explain why each gap matters for the target role.
-   Return structured gap data.

#### Learning Path Agent

Responsibilities:

-   Convert prioritized gaps into a practical sequence of learning
    tasks.
-   Respect prerequisites, estimated effort, and the student's target
    role.
-   Define measurable deliverables and rubrics.
-   Explain how each task can create new evidence.

#### Mock Interview Agent

Responsibilities:

-   Generate role-specific practice questions.
-   Conduct a practice interview.
-   Evaluate answers against a transparent rubric.
-   Provide actionable feedback and suggested follow-up practice.

Do not present the result as a certified or employer-approved
assessment.

#### Placement Orchestrator Agent

Responsibilities:

-   Coordinate placement-related workflows.
-   Summarize drive requirements and candidate evidence.
-   Request deterministic eligibility evaluation from the backend.
-   Prepare explainable shortlists for authorized review.
-   Notify the responsible officer when approval is required.

It must not bypass formal eligibility rules or independently finalize
consequential placement outcomes.

#### Trust & Fairness Agent

Responsibilities:

-   Check whether a recommendation has sufficient evidence.
-   Flag missing explanations, stale evidence, or suspiciously
    incomplete inputs.
-   Detect potentially inappropriate factors in prompts or workflow
    configurations.
-   Request human review when confidence or data quality is
    insufficient.
-   Support fairness and audit reporting.

It is a safeguard, not a substitute for legal, policy, or human review.

### 4.2 Agent execution rules

Every agent run should have:

-   A defined purpose and version.
-   A narrow input schema.
-   A narrow output schema.
-   Explicitly authorized tools.
-   Timeouts and retry limits.
-   Token/cost limits.
-   Error and fallback behavior.
-   A traceable run ID.
-   Audit records for consequential actions.

Agents must not:

-   Change formal eligibility rules.
-   Approve or reject a student for a drive.
-   Send external communications without authorization.
-   Modify a student's evidence or verification state without a defined
    validation process.
-   Access another tenant's data.
-   Expose secrets or private credentials.
-   Invent evidence or fabricate source data.

### 4.3 Orchestration workflow example

1.  A user submits a job description.
2.  The AI extracts draft role requirements.
3.  An authorized user reviews and confirms those requirements.
4.  The Skill Gap Agent compares student profiles with the confirmed
    requirements.
5.  The Learning Path Agent proposes practical tasks for gaps.
6.  Students complete tasks and submit deliverables.
7.  The verification workflow evaluates the new evidence.
8.  The Placement Orchestrator requests a deterministic eligibility
    check.
9.  The system prepares an explainable shortlist.
10. An authorized T&P officer reviews and approves the next action.
11. The platform records the action and its explanation in the audit
    trail.

------------------------------------------------------------------------

## 5. Recommended Technology Stack

Use a stack that supports reliable backend workflows, AI integration,
data integrity, and a polished user experience.

### Frontend

-   React
-   TypeScript
-   Vite
-   Tailwind CSS
-   Framer Motion
-   Recharts
-   Lucide icons

### Backend

-   Java 21
-   Spring Boot
-   Spring Security
-   JWT-based authentication (with secure refresh-token handling if
    used)
-   Spring Data JPA
-   Bean Validation
-   PostgreSQL
-   pgvector for vector search where justified
-   Redis for caching, rate limiting, and short-lived workflow state
    where appropriate

### AI

-   Gemini API
-   Spring AI or LangChain4j
-   Structured JSON outputs and schema validation
-   Embeddings and pgvector only for use cases that benefit from
    semantic retrieval

Use Python only where a separate analytical or machine-learning workload
clearly benefits from it. Do not split the system into microservices
prematurely; a modular Spring Boot application is a sensible starting
point.

### Infrastructure and delivery

-   Docker
-   Docker Compose for local development
-   AWS or another approved cloud provider
-   Object storage for uploaded documents and artifacts
-   Managed secrets storage
-   CI/CD pipeline
-   Centralized logs, metrics, and error tracking

Never commit API keys, database passwords, OAuth secrets, JWT secrets,
or production credentials.

------------------------------------------------------------------------

## 6. High-Level Architecture

``` text
React + TypeScript Portals
  ├── Student Portal
  ├── T&P / College Admin Portal
  ├── Recruiter Portal
  └── University Admin Portal
           |
           v
Spring Boot Application / REST API
  ├── Authentication and RBAC
  ├── Tenant and User Management
  ├── Student Profile and Skill DNA
  ├── Platform Integrations
  ├── Evidence and Competency Engine
  ├── Job and Role Intelligence
  ├── Skill Gap and Learning
  ├── Assessments and Interviews
  ├── Placement Drives and Eligibility
  ├── Analytics and Reporting
  ├── Consent, Fairness, and Audit
  └── AI Orchestrator and Agent Tools
           |
           +---- PostgreSQL / pgvector
           +---- Redis
           +---- Object Storage
           +---- Gemini API
           +---- GitHub API
           +---- Supported LeetCode data access
           +---- Supported HackerRank data access
```

Keep business rules in ordinary application services. AI agents should
invoke those services through controlled tools rather than
reimplementing critical rules in prompts.

------------------------------------------------------------------------

## 7. Suggested Backend Package Structure

Use a modular package structure. Adapt names to the existing repository
rather than duplicating or renaming established modules without a
reason.

``` text
com.placementos
├── auth
├── tenant
├── user
├── student
├── profile
├── integration
│   ├── github
│   ├── leetcode
│   └── hackerrank
├── evidence
├── skill
├── competency
├── skillgap
├── learning
├── assessment
├── interview
├── company
├── recruiter
├── placement
├── eligibility
├── analytics
├── ai
│   ├── orchestrator
│   ├── verification
│   ├── learning
│   ├── interview
│   └── trust
├── consent
├── fairness
├── audit
├── reporting
└── common
```

Suggested frontend feature areas:

``` text
src/
├── app/
├── components/
│   ├── ui/
│   ├── layout/
│   ├── charts/
│   └── feedback/
├── features/
│   ├── auth/
│   ├── student/
│   ├── skill-dna/
│   ├── integrations/
│   ├── evidence/
│   ├── skill-gap/
│   ├── learning-path/
│   ├── flight-simulator/
│   ├── placement/
│   ├── admin-war-room/
│   ├── recruiter/
│   └── university/
├── hooks/
├── lib/
├── services/
├── types/
└── routes/
```

------------------------------------------------------------------------

## 8. Suggested Data Model

The following is a starting domain model, not a mandate to create every
table in the first sprint.

### Core tenancy and identity

-   `tenants`
-   `users`
-   `student_profiles`
-   `student_integrations`

### Competencies and evidence

-   `skills`
-   `student_skills`
-   `skill_evidence`
-   `repositories`
-   `contributions`
-   `certificates`
-   `assessment_results`

### Roles and learning

-   `job_roles`
-   `job_descriptions`
-   `skill_requirements`
-   `learning_paths`
-   `learning_tasks`
-   `learning_task_submissions`

### Assessments

-   `assessments`
-   `assessment_attempts`
-   `interview_sessions`
-   `interview_responses`
-   `evaluation_rubrics`

### Placement and recruitment

-   `companies`
-   `placement_drives`
-   `eligibility_rules`
-   `eligibility_rule_versions`
-   `eligibility_results`
-   `applications`
-   `interviews`
-   `offers`

### Governance and AI

-   `agent_runs`
-   `audit_logs`
-   `consent_records`
-   `data_access_logs`
-   `fairness_reviews`

### Data design rules

1.  Include `tenant_id` on every college-owned record and enforce tenant
    isolation in the backend and, where appropriate, PostgreSQL
    Row-Level Security.
2.  Keep skill definitions separate from student skill claims and their
    evidence.
3.  Preserve evidence provenance: provider, source identifier, retrieval
    time, owner where known, and verification method.
4.  Version eligibility rules. Never silently change the meaning of an
    old eligibility result.
5.  Keep audit records append-only or otherwise tamper-evident.
6.  Store consent purpose, scope, timestamp, status, and revocation.
7.  Avoid storing raw third-party credentials. Store only appropriately
    protected tokens required for approved integrations.
8.  Use retention and deletion policies for personal data and uploaded
    artifacts.
9.  Add database constraints and indexes for tenant boundaries, unique
    identifiers, and common query paths.
10. Do not store a derived AI score as if it were ground truth. Store
    model/version, inputs, confidence, and explanation metadata where
    appropriate.

------------------------------------------------------------------------

## 9. Suggested REST API Surface

Use consistent response envelopes, validation, pagination,
authorization, and error handling.

### Authentication and profile

``` http
POST /api/auth/login
POST /api/auth/register
POST /api/auth/refresh
POST /api/auth/logout
GET  /api/students/me
PUT  /api/students/me
```

### Integrations and evidence

``` http
POST /api/integrations/github/connect
POST /api/integrations/leetcode/connect
POST /api/integrations/hackerrank/connect
DELETE /api/integrations/{provider}/disconnect
POST /api/integrations/{provider}/sync
GET  /api/students/me/evidence
GET  /api/students/me/skills
```

Only expose connection or sync endpoints for providers and authorization
methods that are actually implemented and supported.

### Role and learning intelligence

``` http
POST /api/jobs/analyze
GET  /api/students/me/skill-gaps
GET  /api/students/me/learning-paths
POST /api/learning-tasks/{taskId}/submissions
GET  /api/students/me/progress
```

### Assessments and interviews

``` http
GET  /api/assessments
POST /api/assessments/{assessmentId}/attempts
POST /api/interviews/start
POST /api/interviews/{sessionId}/responses
GET  /api/interviews/{sessionId}/feedback
```

### Placement

``` http
GET  /api/placement/drives
GET  /api/placement/drives/{driveId}
GET  /api/placement/drives/{driveId}/eligibility
POST /api/placement/drives/{driveId}/applications
POST /api/admin/placement/drives
POST /api/admin/placement/drives/{driveId}/evaluate-eligibility
POST /api/admin/placement/applications/{applicationId}/review
```

### Admin, recruiter, and reporting

``` http
GET  /api/admin/analytics/cohort
GET  /api/admin/analytics/skill-gaps
GET  /api/admin/audit-logs
GET  /api/recruiter/candidates
POST /api/recruiter/jobs
GET  /api/reports/placement
```

Treat these as design examples. Confirm authorization and tenant scope
for every route before implementation.

------------------------------------------------------------------------

## 10. Security, Privacy, Fairness, and Governance

These are product requirements, not optional polish.

### Security

-   Enforce backend RBAC and tenant isolation.
-   Validate every request and uploaded file.
-   Use secure password hashing and carefully managed JWT/refresh-token
    lifecycles.
-   Apply rate limits to authentication, AI, integrations, and public
    endpoints.
-   Protect against IDOR and cross-tenant access.
-   Use HTTPS in deployed environments.
-   Keep secrets in environment variables or a secrets manager, never in
    source control.
-   Use least-privilege OAuth scopes.
-   Log security-relevant events without logging passwords, access
    tokens, or unnecessary personal data.
-   Add dependency scanning and tests for authorization boundaries.

### Privacy and consent

-   Explain what data is collected, why it is needed, and who can see
    it.
-   Obtain appropriate consent before connecting external accounts or
    sharing evidence with recruiters.
-   Allow users to disconnect integrations and revoke consent.
-   Define data retention and deletion workflows.
-   Collect only data necessary for a stated purpose.
-   Restrict access to personal and assessment data by role and purpose.
-   Consider applicable Indian data-protection requirements, including
    the Digital Personal Data Protection Act, 2023 and rules in force at
    deployment. Obtain qualified legal review; do not treat this
    document as legal advice.
-   Apply additional safeguards where minors or other vulnerable users
    may be involved.

### Fairness and human oversight

-   Do not rank or exclude candidates based on protected or sensitive
    characteristics.
-   Do not use irrelevant personal data or proxies for protected traits.
-   Do not use AI-generated-code detection as a basis to penalize
    students.
-   Make the basis of a recommendation visible and contestable.
-   Provide a route for correction of inaccurate profile or evidence
    data.
-   Require authorized human approval for consequential placement
    actions.
-   Monitor whether data coverage or assessment design creates unfair
    disadvantages.
-   Keep an audit trail of recommendations, rule evaluations, approvals,
    and overrides.

------------------------------------------------------------------------

## 11. UI and Design Direction

The product should feel like a premium, modern education and enterprise
SaaS platform---not a generic AI dashboard.

### Visual direction

-   Dark-first interface.
-   Strong hierarchy and readable typography.
-   Sharp or subtly rounded edges; avoid excessive pill-shaped UI.
-   Restrained glass effects rather than blur on every surface.
-   Purposeful motion and transitions.
-   Dense but legible data visualization.
-   Clear empty, loading, success, and error states.
-   Accessible contrast, keyboard navigation, and responsive layouts.

### Suggested palette

  Token           Value                   Use
  --------------- ----------------------- ---------------------------
  Deep Obsidian   `#090A0F`               Main background
  Surface         `#12141C`               Cards and panels
  Indigo          `#5B5BF0`               Primary actions and focus
  Success         `#22C55E`               Positive status
  Text            High-contrast neutral   Primary content
  Muted           Neutral grey            Secondary content

### Component and interaction ideas

-   Bento-style dashboard sections.
-   Skill radar or competency maps where they communicate real data.
-   Cohort heatmaps.
-   Timeline-based placement stages.
-   Evidence cards with source and verification state.
-   Explainability panels for matching and eligibility.
-   Framer Motion for restrained page and state transitions.
-   Recharts for meaningful analytics.
-   Lucide for consistent iconography.
-   Variable font sizing on the landing page, with deliberate responsive
    typography.

Do not add animation that delays essential tasks or obscures data. Do
not fabricate analytics to make a screen look populated.

------------------------------------------------------------------------

## 12. Signature Differentiators

Prioritize these experiences because they communicate the product's
value:

1.  **Skill DNA Graph:** A student's evolving, evidence-backed
    competency profile.
2.  **Placement War Room:** A T&P command center for cohort readiness
    and placement operations.
3.  **Placement Flight Simulator:** A realistic practice environment for
    aptitude, coding, and technical interviews.
4.  **Proof-to-Progress Engine:** Learning tasks that create new,
    reviewable evidence.
5.  **Explainable Candidate Matching:** Clear reasons why a profile
    matches or does not match a role.
6.  **Industry Skill-Gap Profiler:** Cohort-level comparison between
    available competencies and role requirements.

------------------------------------------------------------------------

## 13. MVP Scope and Build Roadmap

Build a coherent end-to-end workflow first.

### Phase 1 --- Foundation

-   Project setup and environment configuration.
-   Authentication and RBAC.
-   Tenant and college setup.
-   Student profile.
-   Consent records.
-   Core database migrations.
-   Initial student and admin layouts.

### Phase 2 --- Developer Intelligence

-   Implement GitHub integration with least-privilege authorization.
-   Add optional LeetCode and HackerRank integrations only to the extent
    supported and stable.
-   Normalize imported data into evidence records.
-   Build Skill DNA.
-   Add sync status, provider errors, and disconnect flows.

### Phase 3 --- Role Intelligence and AI

-   Job description upload and parsing.
-   Human review of extracted requirements.
-   Skill gap analysis.
-   Personalized learning path.
-   Agent orchestration with structured schemas, bounded tools, and
    audit records.

### Phase 4 --- Placement Flight Simulator

-   Aptitude assessment.
-   Coding assessment.
-   Technical interview practice.
-   Rubric-based feedback.
-   Student progress and evidence updates.

### Phase 5 --- Admin Intelligence and Placement

-   Placement War Room.
-   Cohort skill heatmaps.
-   Industry Skill-Gap Profiler.
-   Deterministic eligibility engine.
-   Drive and application management.
-   Explainable candidate shortlist.
-   Human approval workflow.

### Phase 6 --- Governance and Deployment

-   Tenant-isolation tests.
-   Authorization and security tests.
-   Consent and revocation workflows.
-   Fairness and audit reporting.
-   Data retention and deletion.
-   Dockerized deployment and CI/CD.
-   Monitoring, logs, and production configuration.

### Recommended hackathon MVP

If time is limited, deliver this complete slice:

1.  Student and T&P officer authentication.
2.  Student profile.
3.  At least one reliable developer-platform integration (GitHub is a
    practical first choice).
4.  Evidence-backed Skill DNA.
5.  Job description analysis.
6.  Skill gap report.
7.  AI-generated practical learning path.
8.  One working assessment or interview simulation.
9.  Admin dashboard with cohort or student readiness.
10. Deterministic eligibility evaluation for a sample drive.
11. At least three coordinated, bounded agents: Skill Verification,
    Skill Gap/Learning, and Placement Orchestrator.
12. Consent, explainability, audit trail, and human approval for the
    demonstrated consequential action.

A complete, reliable vertical slice is better than a large set of mock
screens.

------------------------------------------------------------------------

## 14. Suggested Demonstration Story

Use fictional data and clearly label it as demo data.

1.  A student named Rahul signs in.
2.  Rahul connects a supported coding profile and GitHub account.
3.  PlacementOS imports available data and builds an evidence-linked
    Skill DNA profile.
4.  Rahul selects "Java Backend Developer" or uploads a sample job
    description.
5.  The role requirements are extracted and reviewed.
6.  PlacementOS identifies gaps in Spring Boot, SQL, Docker, or other
    role requirements, based on the sample evidence.
7.  The student receives a practical learning path.
8.  Rahul completes a coding or technical interview simulation.
9.  The result creates new assessment evidence and updates the profile
    where appropriate.
10. The T&P officer opens the Placement War Room and sees the student's
    updated readiness.
11. The Placement Orchestrator prepares an explainable recommendation
    for a sample drive.
12. The deterministic eligibility engine evaluates the published rules.
13. The officer reviews and approves the next step.
14. The system records the rule version, explanation, and approval in
    the audit trail.

Do not use fake external integration success, fake live data, or
unverified claims in the demo.

------------------------------------------------------------------------

## 15. Suggested Team Work Split

For a four-person team, a possible split is:

  -----------------------------------------------------------------------
  Workstream                          Primary responsibility
  ----------------------------------- -----------------------------------
  Backend                             Spring Boot APIs, PostgreSQL
                                      schema, auth, tenancy, eligibility

  AI and integrations                 Gemini, agent orchestration, GitHub
                                      and supported provider integrations

  Frontend                            Student portal, Skill DNA, learning
                                      flow, Flight Simulator

  Admin and delivery                  Placement War Room, recruiter/admin
                                      flows, Docker, CI/CD, testing, demo
                                      integration
  -----------------------------------------------------------------------

For a smaller team, combine workstreams and reduce MVP scope. Keep API
contracts and database ownership explicit to avoid integration
conflicts.

------------------------------------------------------------------------

## 16. Example Pitch

> The problem isn't that colleges don't have enough students. The
> problem is that colleges don't know which students are actually ready
> for which opportunities.
>
> Today, placement departments depend on static resumes, manual
> eligibility checks, and reactive training. We are building
> PlacementOS, an AI-powered Placement Intelligence Platform that
> transforms student evidence into verified competencies.
>
> Our platform doesn't just tell students what to learn. It identifies
> their skill gaps, creates practical learning challenges, verifies
> their improvements, and connects them with suitable opportunities.
>
> Meanwhile, specialized AI agents assist placement officers with talent
> discovery, eligibility workflows, training recommendations, and drive
> coordination. Every important recommendation is explainable,
> auditable, and subject to human oversight.
>
> We are not building another job portal. We are building the
> intelligence layer between education and employment.

------------------------------------------------------------------------

## 17. Instructions for AI Coding Agents

When working in this repository, follow these rules.

### Before changing code

1.  Inspect the existing project structure, build files, and
    conventions.
2.  Read the relevant module and tests before editing.
3.  Identify whether the requested change affects authentication, tenant
    isolation, consent, evidence, eligibility, or audit behavior.
4.  Reuse existing components and services where practical.
5.  If a requirement is ambiguous, state the assumption or ask one
    focused question rather than inventing product behavior.

### Implementation standards

-   Prefer small, cohesive changes over broad rewrites.
-   Keep controllers thin; put business logic in services.
-   Use DTOs for API boundaries. Do not expose persistence entities
    directly.
-   Validate inputs at API boundaries.
-   Use transactions for multi-step database changes that must be
    atomic.
-   Use database migrations; do not rely on manual production schema
    changes.
-   Keep tenant scope explicit in repository queries and service
    methods.
-   Enforce authorization on the server.
-   Add tests for business rules, tenant boundaries, and failure cases.
-   Use structured logging and avoid sensitive values in logs.
-   Handle external provider failures and rate limits gracefully.
-   Make AI outputs schema-validated and treat them as untrusted input.
-   Never let an LLM directly execute arbitrary SQL or change formal
    eligibility outcomes.
-   Keep secrets out of source code, test fixtures, logs, and
    documentation.
-   Document new environment variables in an example environment file
    without real credentials.
-   Keep frontend components accessible and responsive.
-   Include loading, empty, error, and success states for asynchronous
    UI.
-   Do not fabricate demo statistics in production-facing views.

### Definition of done

A feature is not complete until:

-   Its main success path works.
-   Invalid input is handled.
-   Authorization and tenant boundaries are tested.
-   Relevant error states are handled.
-   Tests are added or updated.
-   Database migrations are included if needed.
-   API and configuration changes are documented.
-   No secrets or personal data are accidentally exposed.
-   The UI is responsive and accessible where applicable.
-   Any AI recommendation includes appropriate explanation and
    uncertainty.
-   Consequential actions have the required human approval and audit
    record.

### Priority order when trade-offs are necessary

1.  Security, privacy, and tenant isolation.
2.  Correctness of business rules and evidence provenance.
3.  Complete end-to-end workflow.
4.  Explainability and auditability.
5.  Reliability and graceful failure.
6.  Usability and accessibility.
7.  Visual polish and animation.

------------------------------------------------------------------------

## 18. Non-Goals and Things to Avoid

-   Do not turn PlacementOS into a generic job listing clone.
-   Do not claim integrations can retrieve private or complete data when
    they cannot.
-   Do not scrape behind authentication or bypass provider controls.
-   Do not ask students to share third-party passwords or session
    cookies.
-   Do not let an LLM make final eligibility or hiring decisions.
-   Do not equate repository language statistics with verified skill.
-   Do not treat a model-generated score as ground truth.
-   Do not expose student-level data in aggregate dashboards
    unnecessarily.
-   Do not allow cross-tenant queries.
-   Do not build microservices before the modular application needs
    them.
-   Do not prioritize decorative dashboards over working workflows.
-   Do not fabricate placement outcomes, recruiter activity, or
    assessment results.

------------------------------------------------------------------------

## 19. First Implementation Milestone

The first demonstrable milestone should be:

**A student can sign in, connect GitHub with consent, view imported
evidence in Skill DNA, analyze a Java Backend job description, receive
an explainable skill-gap report and learning path, complete one
practical assessment, and see the updated evidence. A T&P officer can
review the student's profile and a deterministic eligibility result for
a sample drive.**

This milestone validates the central PlacementOS promise: turning
evidence into actionable career readiness and responsible placement
intelligence.
