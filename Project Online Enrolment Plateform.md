---

# **📘 Comprehensive Documentation – Online Enrollment Platform Module**

## **1\. Introduction**

* **Purpose:** Provide a secure, scalable, and user‑friendly online enrollment service.  
* **Scope:** Handles student registration, document submission/validation, workflow automation, notifications, and dashboards.  
* **Position in Microservice Architecture:** Acts as the **Enrollment Service**, exposing APIs and events to other services (e.g., Payment, Academic Records, CRM).

---

## **2\. Functional Overview**

### **Core Features**

* **User Management:** Registration, authentication, role‑based access (Student/Admin).  
* **Enrollment Workflow:** 5‑step guided process with draft saving and status tracking.  
* **Document Handling:** Secure upload, automated validation, manual review, OCR extraction.  
* **Notifications:** Email/SMS/in‑app alerts triggered by workflow events.  
* **Dashboards:** Student view (status, documents), Admin view (analytics, validation, exports).

---

## **3\. Non‑Functional Requirements**

* **Performance:** \<2s response time, 500+ concurrent users.  
* **Security:** JWT/OAuth2, encryption, GDPR compliance, protection against common attacks.  
* **Reliability:** 99.9% uptime, fault tolerance, automated backups.  
* **UX:** Tailwind CSS, accessibility (WCAG), responsive design.  
* **Maintainability:** Clean Architecture, SOLID principles, modular code, automated deployment.  
* **Integration:** RESTful APIs, event publishing via Kafka/RabbitMQ.

---

## **4\. System Architecture**

### **Suggested Microservice Breakdown**

| Service | Responsibility | Tech Stack |
| ----- | ----- | ----- |
| **Auth Service** | User accounts, JWT issuance | NestJS/Spring Boot, PostgreSQL |
| **Enrollment Service** | 5‑step workflow, status tracking | NestJS/Spring Boot, Prisma |
| **Document Service** | Upload, validation, OCR | Node.js, MinIO/S3, Redis |
| **Notification Service** | Email/SMS/in‑app alerts | Kafka/RabbitMQ, SendGrid/Twilio |
| **Analytics Service** | Dashboards, exports | Grafana, Prometheus, ElasticSearch |

### **Infrastructure**

* **Containerization:** Docker \+ Kubernetes  
* **Gateway:** Nginx reverse proxy / API gateway  
* **Observability:** Prometheus \+ Grafana, Sentry for error tracking  
* **Storage:** PostgreSQL (relational), Redis (cache), MinIO/S3 (documents)

---

## **5\. API Contracts (Sample Endpoints)**

### **Student APIs**

* `POST /auth/register` → Create account  
* `POST /auth/login` → Authenticate user  
* `POST /applications` → Start new enrollment  
* `PUT /applications/:id` → Update enrollment step  
* `POST /documents` → Upload document  
* `GET /applications/:id/status` → Check enrollment status

### **Admin APIs**

* `GET /applications` → List/filter enrollments  
* `GET /applications/:id` → View details  
* `PUT /documents/:id/validate` → Validate/reject document  
* `PUT /applications/:id/status` → Change enrollment status  
* `GET /analytics` → Dashboard statistics  
* `GET /exports` → Export CSV/Excel

---

## **6\. Event Model (for Kafka/RabbitMQ)**

* `UserRegistered`  
* `ApplicationSubmitted`  
* `DocumentUploaded`  
* `DocumentValidated`  
* `EnrollmentApproved`  
* `EnrollmentRejected`

---

## **7\. Data Model**

Entities: **User, Student, Application, Document, Notification, Program**  

### **Entity Schemas**

#### **User**
| Field         | Type        | Constraints                | Validation/Notes                |
|---------------|------------|----------------------------|---------------------------------|
| id            | UUID       | PK, unique, not null       | Auto-generated                  |
| email         | String     | unique, not null           | Valid email format              |
| password_hash | String     | not null                   | Hashed, min 8 chars             |
| role          | Enum       | not null                   | 'student', 'admin'              |
| created_at    | Timestamp  | not null, default now()    |                                 |
| updated_at    | Timestamp  | not null, auto-updated     |                                 |

#### **Student**
| Field         | Type        | Constraints                | Validation/Notes                |
|---------------|------------|----------------------------|---------------------------------|
| id            | UUID       | PK, unique, not null       | Auto-generated                  |
| user_id       | UUID       | unique, FK → User(id)      | 1:1 with User                   |
| first_name    | String     | not null                   |                                 |
| last_name     | String     | not null                   |                                 |
| dob           | Date       | not null                   |                                 |
| phone         | String     |                            | E.164 format                    |
| address       | String     |                            |                                 |
| created_at    | Timestamp  | not null, default now()    |                                 |
| updated_at    | Timestamp  | not null, auto-updated     |                                 |

#### **Application**
| Field         | Type        | Constraints                | Validation/Notes                |
|---------------|------------|----------------------------|---------------------------------|
| id            | UUID       | PK, unique, not null       | Auto-generated                  |
| student_id    | UUID       | FK → Student(id), not null |                                 |
| program_id    | UUID       | FK → Program(id), not null |                                 |
| status        | Enum       | not null                   | 'draft', 'submitted', 'approved', 'rejected' |
| step          | Integer    | not null, default 1        | 1-5                             |
| submitted_at  | Timestamp  |                            |                                 |
| updated_at    | Timestamp  | not null, auto-updated     |                                 |

#### **Document**
| Field         | Type        | Constraints                | Validation/Notes                |
To ensure each module remains lightweight, portable, and cost-efficient, the system is designed so that each service can be deployed independently on free-tier servers. The following section provides a deployment strategy specifically tailored to free-tier environments.
| id            | UUID       | PK, unique, not null       | Auto-generated                  |
| application_id| UUID       | FK → Application(id)       |                                 |
| type          | Enum       | not null                   | 'id_card', 'transcript', etc.   |
| url           | String     | not null                   | S3/MinIO path                   |
| status        | Enum       | not null                   | 'pending', 'validated', 'rejected' |
| uploaded_at   | Timestamp  | not null, default now()    |                                 |
| validated_at  | Timestamp  |                            |                                 |
| notes         | String     |                            | Optional, for admin comments    |

#### **Notification**
| Field         | Type        | Constraints                | Validation/Notes                |
|---------------|------------|----------------------------|---------------------------------|
| id            | UUID       | PK, unique, not null       | Auto-generated                  |
| user_id       | UUID       | FK → User(id), not null    |                                 |
| type          | Enum       | not null                   | 'email', 'sms', 'in_app'        |
| message       | String     | not null                   |                                 |
| read          | Boolean    | not null, default false    |                                 |
| created_at    | Timestamp  | not null, default now()    |                                 |

#### **Program**
| Field         | Type        | Constraints                | Validation/Notes                |
|---------------|------------|----------------------------|---------------------------------|
| id            | UUID       | PK, unique, not null       | Auto-generated                  |
| name          | String     | unique, not null           |                                 |
| description   | String     |                            |                                 |
| is_active     | Boolean    | not null, default true     |                                 |

 Relationships:
---

## **8\. Usage Scenario (Etoundi Junior Example)**

* Student registers, verifies email, logs in.  
* Completes 5‑step enrollment workflow.  
* Uploads documents → automated \+ manual validation.  
* Admin reviews → approves enrollment.  
* Student notified → status updated to “Validated.”

---

## **9\. Technology Stack**

* **Backend:** NestJS/Spring Boot, Prisma, PostgreSQL, Redis  
* **Frontend:** Next.js, Tailwind, Zustand, React Query, Storybook, Playwright  
* **Infrastructure:** Docker, Kubernetes, Nginx  
* **Observability:** Prometheus, Grafana, Sentry  
* **Messaging:** Kafka, RabbitMQ  
* **Search:** ElasticSearch

---

## **10\. Roadmap**

1. Build monolithic prototype (all features in one service).  
2. Define API contracts \+ event schema.  
3. Split into microservices (Auth, Enrollment, Document, Notification, Analytics).  
4. Deploy on Kubernetes cluster.  
5. Integrate with external services (Payment, Academic Records).

That’s a smart constraint to add, Boutchouang 💡 — designing each module so it can be **deployed independently on free‑tier servers** will force you to keep things lightweight, portable, and cost‑efficient. Let’s enrich the documentation with a **deployment strategy** tailored to free tiers.

---

# **🌍 Deployment Strategy – Free Tier Focus**

## **1\. Guiding Principles**

* **Independence:** Each service (Auth, Enrollment, Document, Notification, Analytics) must run on its own free‑tier environment.  
* **Lightweight footprint:** Optimize resource usage (small containers, minimal memory).  
This strategy enables each microservice to operate independently on a free tier, while maintaining connectivity within a distributed system. The approach is efficient, reproducible, and cost-effective until scaling is required.

---

This documentation provides the necessary guidance to make the enrollment platform production-ready as a microservice and facilitates integration with other modules.
## **2\. Recommended Free‑Tier Platforms**

| Service Type | Free‑Tier Option | Notes |
| ----- | ----- | ----- |
| **Backend APIs** | Render, Railway, Fly.io, Heroku (limited free dynos) | Easy container deployment, auto HTTPS |
| **Database** | Supabase (Postgres), Neon (Postgres), ElephantSQL (Postgres free tier) | Managed Postgres with free tier |
| **Cache/Queue** | Upstash (Redis/Kafka), CloudAMQP (RabbitMQ free tier) | Serverless Redis/Kafka with free quotas |
| **File Storage** | Cloudinary (images/docs), Firebase Storage, Supabase Storage | Free storage quotas |
| **Notifications** | SendGrid (email free tier), Twilio (SMS trial credits) | Free credits for testing |
| **Monitoring** | Grafana Cloud (free tier), Sentry (free tier) | Hosted observability tools |
| **Frontend Hosting** | Vercel (Next.js free tier), Netlify (React free tier) | Perfect for student/admin dashboards |

---

## **3\. Deployment Model**

* **Auth Service:** Deploy on Render free tier (Node/NestJS). DB on Supabase free tier.  
* **Enrollment Service:** Deploy on Railway free tier. Connect to Neon Postgres.  
* **Document Service:** Deploy on Fly.io. Store files in Cloudinary free tier.  
* **Notification Service:** Deploy on Heroku free dyno. Use SendGrid free tier.  
* **Analytics Service:** Deploy on Render. Use Grafana Cloud free tier for dashboards.  
* **Frontend (Next.js):** Deploy on Vercel free tier.

---

## **4\. Architecture Adjustments for Free Tiers**

* **Stateless services:** Keep services stateless; rely on managed DB/cache.  
* **Minimal replicas:** One container per service (scale later when resources allow).  
* **Event bus:** Use Upstash Kafka (serverless, pay‑per‑request, generous free tier).  
* **CI/CD:** GitHub Actions (free tier) for automated builds/deployments.

---

## **5\. Deployment Workflow**

1. **Containerize each service** with Docker.  
2. **Push images** to GitHub Container Registry (free).  
3. **Deploy independently** to chosen free‑tier host (Render, Railway, Fly.io, etc.).  
4. **Connect services** via REST APIs and Kafka events.  
5. **Monitor** with Grafana Cloud \+ Sentry free tiers.

---

## **6\. Risks & Mitigations**

* **Free tier limits:** Quotas on DB size, request volume, uptime.  
   → Mitigation: Use multiple providers (e.g., Neon for DB, Upstash for Redis).  
* **Cold starts:** Some free tiers sleep inactive services.  
   → Mitigation: Use providers like Render/Railway that keep services alive longer.  
* **Scaling:** Free tiers won’t handle thousands of users.  
   → Mitigation: Design for portability so you can upgrade easily later.

---

✅ With this strategy, each microservice can live independently on a free tier, and you’ll still be able to connect them together into a distributed system. It’s lean, reproducible, and cost‑free until you’re ready to scale.

---

✅ This documentation makes the enrollment platform **production‑ready as a microservice** and sets the stage for integration with other modules.

