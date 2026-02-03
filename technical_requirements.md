# Construction Connect Canada - Technical Requirements

## Technical Overview

This document outlines the technical specifications and implementation requirements for Construction Connect Canada, a web-based platform built with React/Next.js that connects clients with tradesmen for construction services and provides AI-powered cost estimation.

---

## Executive Summary: Key Technical Decisions

This section provides a quick reference for all major technology choices and architectural decisions.

### Platform Foundation
- **Architecture:** Modular Monolith (MVP) → Microservices (Scale)
- **Frontend Framework:** Next.js 14+ with React, TypeScript, Tailwind CSS
- **Backend:** Node.js/TypeScript with Next.js API routes
- **Database:** PostgreSQL 15+ with Prisma ORM
- **Hosting:** Amazon Web Services (AWS)
- **Primary Region:** AWS Canada (Central) for data sovereignty

### Core Service Providers

| Service Category | Provider | Rationale |
|-----------------|----------|-----------|
| **Payment Processing** | **Stripe** | Best-in-class API, Stripe Connect for marketplace, superior for subscriptions, 2.9% + $0.30 CAD |
| **Digital Signatures** | **DocuSign** | Industry standard, legally binding in Canada, comprehensive features |
| **AI Estimation** | **Anthropic Claude 3.5 Sonnet** | Superior reasoning, vision capabilities, 200K context, cost-effective |
| **Object Storage** | **AWS S3** | Reliable, scalable, cost-effective ($0.023/GB/month), integrates with CloudFront |
| **CDN** | **AWS CloudFront** | Fast global delivery, integrated with S3, included with AWS |
| **Database Hosting** | **AWS RDS PostgreSQL** | Managed, automated backups, Multi-AZ high availability |
| **Cache & Queue** | **Redis (ElastiCache)** | Session storage, API caching, job queue (BullMQ) |
| **SMS (2FA)** | **AWS SNS** | Cost-effective ($0.00645/SMS), reliable delivery |
| **Email (Transactional)** | **AWS SES** | Low cost ($0.10/1000 emails), high deliverability |
| **Maps/Location** | **Google Maps API** | Best-in-class for geocoding and location services |
| **Error Tracking** | **Sentry** | Real-time debugging, performance monitoring |
| **Analytics** | **Google Analytics 4 + Mixpanel** | User behavior, conversion tracking |

### Infrastructure Stack (AWS)

```
Application Tier:    2-10x EC2 t3.medium (Auto-scaled, Load Balanced)
Database Tier:       RDS PostgreSQL db.t3.medium (Multi-AZ, 100GB SSD)
Cache/Queue Tier:    ElastiCache Redis cache.t3.small (2 nodes)
Storage Tier:        S3 Standard (images, documents, backups)
CDN:                 CloudFront (global edge locations)
Background Jobs:     Separate EC2 instances or ECS (BullMQ workers)
Monitoring:          CloudWatch, Sentry, UptimeRobot

Estimated Cost:      $300-500/month for MVP/moderate traffic
Scaling Cost:        $1,000-2,000/month at 10,000 active users
```

### Why These Choices?

**Why AWS over Hostinger?**
- Hostinger is basic shared hosting, not suitable for SaaS platforms
- AWS provides complete ecosystem: compute, database, storage, queuing, monitoring
- Proven scalability from startup to enterprise
- 99.99% uptime SLA vs shared hosting unpredictability
- Canadian data center for compliance and performance
- Pay-as-you-grow pricing model

**Why Stripe over PayPal?**
- Stripe Connect purpose-built for marketplaces (perfect for platform commission model)
- Superior developer experience and API documentation
- Native subscription management for tradesman memberships ($50/month)
- Better webhook reliability and testing tools
- More transparent pricing with no hidden fees
- Easier to implement split payments (platform commission + tradesman payout)

**Why PostgreSQL over MySQL/MongoDB?**
- ACID compliance critical for financial transactions
- Superior relational data modeling for complex user/project/quote relationships
- Built-in JSON support for flexible metadata
- Excellent full-text search capabilities
- Better performance for complex queries and joins
- Stronger data integrity guarantees

**Why Anthropic Claude over OpenAI?**
- Superior structured reasoning for cost breakdowns
- 200K token context window (vs GPT-4's 128K)
- Excellent vision capabilities for analyzing project images
- More consistent JSON structured outputs
- Competitive pricing
- No data residency concerns for Canadian operations

**Why DocuSign?**
- Industry-standard for construction contracts
- Legal validity recognized in Canadian courts
- Professional appearance enhances platform credibility
- Comprehensive API for full automation
- Audit trail and tamper-evident seals
- Trusted brand reduces friction

---

## Technology Stack

### Frontend
- **Framework:** React with Next.js 14+ (App Router)
- **Language:** TypeScript (strongly recommended for type safety)
- **Styling:** Tailwind CSS (rapid development, consistency, performance)
- **State Management:** React Context + Zustand (lightweight, scales well)
- **Form Handling:** React Hook Form (performance optimized, great DX)
- **UI Components:** shadcn/ui (accessible, customizable components)

### Backend
- **Architecture:** Modular Monolith (initially), microservices-ready
- **API Style:** RESTful API with Next.js API routes
- **Language:** Node.js with TypeScript
- **Framework:** Next.js 14+ API Routes (unified codebase) + Express for standalone services
- **Job Queue:** BullMQ with Redis (for AI estimation processing)
- **Real-time:** WebSockets or Server-Sent Events (for notifications)

### Database
- **Primary Database:** **PostgreSQL 15+** (RECOMMENDED)
  - **Rationale:**
    - ACID compliance critical for payment transactions
    - Excellent relational data modeling for users, projects, quotes
    - Advanced JSON support for flexible metadata
    - Strong full-text search capabilities
    - Proven scalability and reliability
    - Robust transaction handling for financial operations
    - Wide hosting support on AWS RDS
  - **Schema Management:** Prisma ORM (type-safe, migrations, great DX)
- **Caching Layer:** Redis 7+ (session storage, job queue, API caching)
- **Search Enhancement:** PostgreSQL full-text search initially, Elasticsearch if needed at scale

### Infrastructure & Hosting
- **Primary Hosting:** **Amazon Web Services (AWS)** (RECOMMENDED)
  - **Rationale:**
    - **Scalability:** Easily scale from startup to enterprise
    - **Reliability:** 99.99% uptime SLA, proven infrastructure
    - **Ecosystem:** Complete suite of services (compute, storage, database, queues)
    - **Cost-effective:** Pay-as-you-grow model ideal for startup phase
    - **Canadian presence:** AWS Canada (Central) region for data sovereignty
    - **Superior to Hostinger:** Hostinger is for basic web hosting, not production SaaS platforms

  - **AWS Service Stack:**
    - **Compute:** EC2 (application servers) or ECS/Fargate (containerized)
    - **Database:** RDS for PostgreSQL (managed, automated backups)
    - **Storage:** S3 (images, documents, backups)
    - **CDN:** CloudFront (fast image/asset delivery globally)
    - **Queue:** SQS (message queuing for job processing)
    - **Notifications:** SNS (SMS for 2FA, email/SMS notifications)
    - **Secrets:** AWS Secrets Manager (API keys, database credentials)
    - **Monitoring:** CloudWatch (logs, metrics, alarms)
    - **Load Balancing:** Application Load Balancer (high availability)

- **Alternative for MVP:** Vercel (frontend) + AWS (backend services)
  - Next.js deployment optimized on Vercel
  - Backend services and database on AWS
  - Easier initial setup, migrate fully to AWS as needed

- **CDN:** CloudFront (included with AWS, or Cloudflare for additional DDoS protection)
- **Monitoring:** AWS CloudWatch + Sentry (error tracking)

---

## Server Architecture

### Recommended Architecture: Modular Monolith (Phase 1) → Microservices (Phase 2+)

**Phase 1: Modular Monolith (Launch through first 10,000 users)**

Start with a well-structured monolithic application that's designed for future separation:

```
┌─────────────────────────────────────────────────────────────┐
│                     Next.js Application                      │
├─────────────────────────────────────────────────────────────┤
│  Frontend (React)          │  API Routes (Backend)          │
│  - Client Portal           │  - Authentication              │
│  - Tradesman Portal        │  - Project Management          │
│  - Admin Dashboard         │  - Payment Processing          │
└────────────────────────────┴────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  PostgreSQL  │    │    Redis     │    │     AWS      │
│   Database   │    │  Cache/Queue │    │  S3 Storage  │
└──────────────┘    └──────────────┘    └──────────────┘
                            │
                            ▼
                  ┌──────────────────┐
                  │  Job Processor   │
                  │  (BullMQ Worker) │
                  │  - AI Estimation │
                  │  - Notifications │
                  └──────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│    Stripe    │  │   DocuSign   │  │  AWS SNS/SES │
│   Payments   │  │  Signatures  │  │ Notifications│
└──────────────┘  └──────────────┘  └──────────────┘
```

**Architecture Benefits:**
- **Simplicity:** Single codebase, easier to develop and debug
- **Shared Code:** Reuse logic between frontend and backend
- **Fast Development:** Rapid feature iteration in early stage
- **Lower Costs:** Single application server, minimal infrastructure
- **Modular Design:** Clear domain boundaries enable future separation

**Core Modules:**
1. **Authentication Module** - User auth, session management, 2FA
2. **User Module** - Profile management, role-based access
3. **Project Module** - Project CRUD, status management
4. **Estimation Module** - AI integration, job queue
5. **Marketplace Module** - Job listing, quote submission
6. **Payment Module** - Stripe integration, transaction handling
7. **Contract Module** - DocuSign integration, document generation
8. **Notification Module** - Email, SMS, in-app notifications
9. **Admin Module** - Dashboard, analytics, user management

**Deployment Configuration (AWS):**
- **Compute:** 2x EC2 t3.medium instances (load balanced)
- **Database:** RDS PostgreSQL db.t3.medium (Multi-AZ for HA)
- **Cache/Queue:** ElastiCache Redis t3.small
- **Storage:** S3 Standard for images/documents
- **CDN:** CloudFront distribution
- **Load Balancer:** Application Load Balancer
- **Estimated Cost:** $300-500/month for moderate traffic

---

### Phase 2: Microservices Architecture (Scale beyond 50,000 users)

As the platform grows, separate into independent services:

```
┌────────────────────────────────────────────────────────────────┐
│                    API Gateway (Kong/AWS ALB)                  │
└───────┬────────┬──────────┬──────────┬──────────┬──────────────┘
        │        │          │          │          │
        ▼        ▼          ▼          ▼          ▼
┌──────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────┐
│   Auth   │ │Project │ │Payment │ │AI Est. │ │Contract  │
│ Service  │ │Service │ │Service │ │Service │ │ Service  │
└──────────┘ └────────┘ └────────┘ └────────┘ └──────────┘
     │           │           │          │           │
     ▼           ▼           ▼          ▼           ▼
┌────────┐  ┌────────┐  ┌────────┐ ┌───────┐  ┌────────┐
│ User DB│  │Proj DB │  │  RDS   │ │ Redis │  │  S3    │
└────────┘  └────────┘  └────────┘ └───────┘  └────────┘
```

**When to Migrate:**
- Traffic exceeds single server capacity (>10,000 DAU)
- Need independent scaling of AI estimation service
- Team size grows beyond 10 developers
- Database queries slow despite optimization
- Feature development conflicts in monolith

**Migration Path:**
1. Extract AI Estimation Service first (highest computational load)
2. Extract Payment Service (for PCI isolation)
3. Extract Contract Service (DocuSign integration)
4. Keep core monolith for remaining features

---

### Infrastructure Components

**1. Application Servers (EC2)**
- **Purpose:** Run Next.js application
- **Configuration:**
  - Instance Type: t3.medium (2 vCPU, 4GB RAM) initially
  - Auto Scaling: 2-10 instances based on traffic
  - OS: Ubuntu 22.04 LTS or Amazon Linux 2
- **Scaling Strategy:** Horizontal scaling with Application Load Balancer

**2. Database (RDS PostgreSQL)**
- **Purpose:** Primary data store
- **Configuration:**
  - Instance: db.t3.medium (2 vCPU, 4GB RAM) initially
  - Storage: 100GB SSD (gp3), auto-scaling enabled
  - Multi-AZ: Yes (high availability)
  - Automated backups: Daily, 30-day retention
- **Connection Pooling:** PgBouncer for efficient connections

**3. Cache & Queue (ElastiCache Redis)**
- **Purpose:** Session storage, API caching, job queue
- **Configuration:**
  - Node Type: cache.t3.small (2 nodes for HA)
  - Persistence: Enabled for queue durability
- **Use Cases:**
  - Session storage (JWT refresh tokens)
  - API response caching (job listings, public profiles)
  - BullMQ job queue (AI estimation jobs)
  - Rate limiting data

**4. Object Storage (S3)**
- **Purpose:** Image and document storage
- **Buckets:**
  - `ccc-project-images-prod`: Project photos
  - `ccc-contracts-prod`: Signed contracts
  - `ccc-backups-prod`: Database backups
- **Configuration:**
  - Lifecycle policies: Move to Glacier after 90 days
  - Versioning: Enabled for contracts
  - Encryption: AES-256 at rest
- **Cost Optimization:** CloudFront CDN reduces S3 bandwidth costs

**5. Content Delivery (CloudFront)**
- **Purpose:** Fast global content delivery
- **Configuration:**
  - Origin: S3 buckets and EC2 load balancer
  - Edge locations: Global
  - Cache behavior: 1 hour for images, no cache for API
  - SSL/TLS: Certificate via AWS ACM

**6. Background Jobs (BullMQ Workers)**
- **Purpose:** Async processing (AI estimation, notifications)
- **Deployment:** Separate EC2 instances or ECS containers
- **Configuration:**
  - Separate worker processes from web servers
  - Auto-scaling based on queue depth
  - Graceful shutdown handling
- **Job Types:**
  - AI estimation requests
  - Email/SMS notifications
  - Image processing (thumbnails, compression)
  - Contract generation

**7. Monitoring & Logging**
- **CloudWatch:** Application logs, metrics, alarms
- **Sentry:** Error tracking and debugging
- **Custom Metrics:** Job queue depth, AI estimation time, payment success rate
- **Alarms:** High error rate, long queue times, database CPU >80%

---

### Scalability Considerations

**Database Scaling:**
1. **Vertical Scaling:** Upgrade RDS instance (up to db.r6g.4xlarge: 16 vCPU, 128GB)
2. **Read Replicas:** Add 1-3 read replicas for report queries
3. **Connection Pooling:** PgBouncer to handle 1000+ concurrent connections
4. **Query Optimization:** Proper indexing, query analysis with EXPLAIN
5. **Partitioning:** Partition large tables (projects, images) by date if needed

**Application Scaling:**
1. **Horizontal Scaling:** Auto-scale from 2 to 20+ EC2 instances
2. **Container Migration:** Move to ECS/Fargate for better resource utilization
3. **CDN Caching:** Offload 80%+ of static content to CloudFront
4. **API Caching:** Redis caching for frequent queries (job listings)

**Job Queue Scaling:**
1. **Multiple Workers:** Scale worker processes independently
2. **Priority Queues:** Paid members get higher priority
3. **Rate Limiting:** Respect AI service rate limits
4. **Retry Logic:** Exponential backoff for failed jobs

**Geographic Expansion:**
1. **Multi-Region:** Deploy to additional AWS regions (US West, US East for future)
2. **Data Residency:** Keep Canadian data in Canada (Central) region
3. **Global CDN:** CloudFront edge locations worldwide

---

## Authentication & User Management

### Requirements
- Client accounts with secure authentication
- Tradesman accounts with identity verification
- Profile management for both user types
- Role-based access control (RBAC)
  - Client role
  - Tradesman role
  - Admin role
- Two-factor authentication via telephone for both user types

### Implementation Considerations
- JWT-based authentication or session-based
- Password hashing with bcrypt or Argon2
- OAuth/SSO support (optional: Google, Facebook)
- Email verification on signup
- Password reset functionality
- Phone verification for 2FA (Twilio, AWS SNS, or similar)

### User Roles & Permissions

| Role | Permissions |
|------|-------------|
| Client | Create projects, upload images, view estimates, post jobs, review quotes, make payments, manage profile |
| Tradesman | Browse jobs, submit quotes, view client requirements, manage profile, update credentials, track awarded jobs |
| Admin | Full access, user management, platform configuration, analytics, dispute resolution |

---

## Data Storage Architecture

### Database Schema Overview

```
┌─────────────────┐
│     Users       │
├─────────────────┤
│ id (PK)         │
│ email           │
│ password_hash   │
│ role            │
│ phone           │
│ verified        │
│ created_at      │
└─────────────────┘
         │
         ├────────────────┐
         │                │
┌─────────────────┐  ┌─────────────────┐
│  Client Profile │  │Tradesman Profile│
├─────────────────┤  ├─────────────────┤
│ user_id (FK)    │  │ user_id (FK)    │
│ name            │  │ business_name   │
│ address         │  │ credentials     │
│ preferences     │  │ specialties[]   │
└─────────────────┘  │ membership_tier │
         │           │ rating          │
         │           └─────────────────┘
         │                    │
         ▼                    ▼
┌─────────────────┐  ┌─────────────────┐
│    Projects     │  │     Quotes      │
├─────────────────┤  ├─────────────────┤
│ id (PK)         │  │ id (PK)         │
│ client_id (FK)  │  │ project_id (FK) │
│ title           │  │ tradesman_id(FK)│
│ description     │  │ amount          │
│ category        │  │ details         │
│ location        │  │ status          │
│ status          │  │ created_at      │
│ created_at      │  └─────────────────┘
└─────────────────┘
         │
         ├────────────────┬────────────────┐
         ▼                ▼                ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│     Images      │ │  AI Estimates   │ │    Payments     │
├─────────────────┤ ├─────────────────┤ ├─────────────────┤
│ id (PK)         │ │ id (PK)         │ │ id (PK)         │
│ project_id (FK) │ │ project_id (FK) │ │ project_id (FK) │
│ url             │ │ cost_min        │ │ quote_id (FK)   │
│ filename        │ │ cost_max        │ │ amount          │
│ size            │ │ breakdown       │ │ status          │
│ uploaded_at     │ │ confidence      │ │ transaction_id  │
└─────────────────┘ │ generated_at    │ │ created_at      │
                    └─────────────────┘ └─────────────────┘
```

### Data Models

#### User
- `id`: UUID
- `email`: string (unique, indexed)
- `password_hash`: string
- `role`: enum (client, tradesman, admin)
- `phone`: string (for 2FA)
- `phone_verified`: boolean
- `email_verified`: boolean
- `created_at`: timestamp
- `updated_at`: timestamp

#### Project
- `id`: UUID
- `client_id`: UUID (foreign key)
- `title`: string
- `description`: text
- `category`: enum (kitchen, bathroom, basement, etc.)
- `location`: string/object (address, city, postal code)
- `timeline`: string
- `budget_range`: object (min, max)
- `status`: enum (draft, estimating, estimated, posted, in_progress, completed, archived)
- `metadata`: JSON
- `created_at`: timestamp
- `updated_at`: timestamp

#### Image
- `id`: UUID
- `project_id`: UUID (foreign key)
- `storage_url`: string
- `filename`: string
- `file_size`: integer
- `mime_type`: string
- `uploaded_at`: timestamp

#### AI_Estimate
- `id`: UUID
- `project_id`: UUID (foreign key)
- `cost_min`: decimal
- `cost_max`: decimal
- `breakdown`: JSON (detailed cost breakdown)
- `confidence_score`: float
- `processing_time`: integer (seconds)
- `ai_model_version`: string
- `generated_at`: timestamp

#### Quote
- `id`: UUID
- `project_id`: UUID (foreign key)
- `tradesman_id`: UUID (foreign key)
- `amount`: decimal
- `details`: text
- `timeline`: string
- `status`: enum (pending, accepted, rejected, expired)
- `created_at`: timestamp
- `expires_at`: timestamp

#### Payment
- `id`: UUID
- `project_id`: UUID (foreign key)
- `quote_id`: UUID (foreign key, optional)
- `payer_id`: UUID (foreign key to user)
- `amount`: decimal
- `payment_type`: enum (project_payment, membership_fee)
- `status`: enum (pending, completed, failed, refunded)
- `transaction_id`: string (from payment processor)
- `created_at`: timestamp

---

## Core Feature Implementation

### 1. Payment Processing

**Selected Service:** **Stripe** (RECOMMENDED for Canadian Operations)

**Why Stripe over PayPal:**
- **Superior for Canadian Market:**
  - Excellent support for Canadian businesses
  - Competitive pricing: 2.9% + $0.30 CAD per transaction
  - Full support for Canadian bank accounts and cards
  - Local CAD processing (no currency conversion required)
  - Strong presence in Ontario and across Canada

- **Technical Advantages:**
  - **Best-in-class API:** Comprehensive, well-documented, developer-friendly
  - **Stripe Connect:** Purpose-built for marketplace platforms (perfect for tradesman payouts)
  - **Webhook reliability:** Robust event system for payment tracking
  - **Subscription management:** Native recurring billing for tradesman memberships ($50/month)
  - **Payment intents:** Modern, secure payment flow with SCA compliance
  - **Fraud prevention:** Built-in Radar fraud detection

- **Business Features:**
  - **Split payments:** Easily handle commission structure (20% standard / 10% member)
  - **Payouts:** Automated payouts to tradesmen after job completion
  - **Invoicing:** Built-in invoice generation
  - **Reporting:** Comprehensive financial dashboards
  - **Refunds:** Simple refund handling via API or dashboard

- **vs PayPal:**
  - PayPal charges similar or higher fees (2.9% + $0.30, but often higher for business accounts)
  - Stripe has superior developer experience and documentation
  - Stripe Connect specifically designed for marketplaces
  - Better webhook reliability and testing tools
  - More transparent pricing, no hidden fees
  - Stripe checkout is cleaner and more customizable

**Implementation Requirements:**
- PCI DSS compliance (handled entirely by Stripe)
- Support for credit/debit cards (Visa, Mastercard, Amex)
- Support for Canadian bank accounts (EFT/ACH payments via Stripe)
- Recurring billing for tradesman memberships using Stripe Subscriptions
- One-time payments for project completion using Payment Intents
- Refund handling via Stripe API
- Invoice generation using Stripe Invoicing
- Transaction history via Stripe Dashboard and API
- Webhook handling for payment events (payment.succeeded, payment.failed, etc.)

**Stripe API Integration:**
- **Payment Intents:** For one-time project payments
- **Setup Intents:** For saving payment methods
- **Subscriptions:** For tradesman monthly memberships ($50/month)
- **Stripe Connect:** For marketplace payment splitting and tradesman payouts
  - Use "Separate Charges and Transfers" model for commission structure
  - Platform retains commission, transfers remaining amount to tradesman
- **Webhook endpoints:** For real-time payment status updates
- **Secure token handling:** Never store card data, use Stripe tokens only
- **Customer portal:** For managing subscriptions and payment methods

**Payment Flow Example:**
1. Client accepts tradesman quote for $10,000
2. Platform calculates total: $10,000 (to tradesman) + $2,000 (20% commission) = $12,000
3. Client pays $12,000 via Stripe Payment Intent
4. Platform holds funds temporarily
5. Upon project completion, Stripe transfers $10,000 to tradesman's connected account
6. Platform retains $2,000 commission automatically

**Cost Structure:**
- Per-transaction fee: 2.9% + $0.30 CAD
- Stripe Connect fee: +0.5% for marketplace transactions (worth it for functionality)
- No monthly fees, no setup fees
- No hidden costs
- Volume discounts available as platform grows

### 2. Contract Generation & Digital Signatures

**Selected Service:** **DocuSign** (Industry Standard for Digital Signatures)

**Why DocuSign:**
- **Legal Validity:** Legally binding e-signatures compliant with Canadian ESIGN and PIPEDA laws
- **Industry Standard:** Widely trusted and recognized in construction industry
- **Professional Appearance:** Enhances platform credibility
- **Comprehensive Features:** Beyond just signatures (templates, workflows, authentication)
- **Enterprise-Ready:** Scales from startup to enterprise
- **Canadian Support:** Strong presence and compliance in Canada

**Use Case in Platform:**
Once a client selects a tradesman and both parties agree on terms, the platform generates a professional construction contract and sends it to both parties for digital signature via DocuSign.

**Implementation Requirements:**

**DocuSign API Integration:**
- **API Authentication:** OAuth 2.0 for secure access
- **Envelope Creation:** Programmatically create signature requests
- **Template Management:** Store contract templates in DocuSign
- **Webhook Events:** Track signature status (sent, viewed, signed, completed)
- **Document Generation:** Populate templates with project-specific data
- **Multi-Signer Support:** Client and tradesman both sign the same document

**Contract Workflow:**
1. **Contract Generation:**
   - Client accepts tradesman's quote
   - Platform generates contract from template
   - Auto-populates: project details, pricing, timeline, terms
   - Includes platform commission breakdown
   - Adds "not-to-exceed" clauses and payment schedule

2. **DocuSign Envelope Creation:**
   - Platform creates DocuSign envelope via API
   - Uploads generated contract PDF
   - Defines signature fields for client and tradesman
   - Sets signing order (typically client first, then tradesman)
   - Adds required fields (date, initials, full signature)

3. **Signature Request:**
   - DocuSign sends email to client with link to review and sign
   - Client reviews contract, acknowledges terms, signs digitally
   - After client signs, DocuSign automatically sends to tradesman
   - Tradesman reviews and signs
   - Platform receives webhook notifications at each step

4. **Completed Contract:**
   - Both parties receive fully executed PDF copy via email
   - Platform stores signed contract in database
   - Contract accessible through user dashboards
   - Immutable record with audit trail

**Technical Implementation:**
- **DocuSign Node.js SDK:** For seamless API integration
- **Template Variables:** Dynamic field mapping (project_name, cost, timeline, etc.)
- **Webhook Endpoint:** `/api/webhooks/docusign` for status updates
- **Status Tracking:** Real-time updates in project dashboard
- **Document Storage:** Store signed PDFs in AWS S3 for long-term access
- **Notification Integration:** Email/SMS alerts when signature required or completed

**DocuSign Features to Leverage:**
- **Templates:** Pre-configured contract templates for different project types
- **Conditional Logic:** Show/hide fields based on contract type
- **Payment Tabs:** Optional integration with DocuSign Payments
- **Certificate of Completion:** Legal audit trail for all signatures
- **Mobile Signing:** Responsive signing experience on any device
- **Authentication:** SMS or email verification for signer identity

**Cost Considerations:**
- **DocuSign Pricing:** Plans start at ~$40-60 USD/month for API access
- **Per-Envelope Costs:** Typically included in plan or minimal per-document fee
- **Scaling:** Higher-tier plans as transaction volume grows
- **ROI:** Essential for legal protection and professional service delivery

**Alternative Consideration:**
- **HelloSign (Dropbox Sign):** Lower cost alternative, good API
- **PandaDoc:** Contract + e-signature + payment in one platform
- **Recommendation:** Stick with DocuSign for brand trust and comprehensive features

**Data Flow:**
```
Project Award → Contract Generation → DocuSign API Call → Envelope Created →
Email to Signers → Client Signs → Tradesman Signs → Webhook to Platform →
Completed PDF Stored → Both Parties Notified
```

**Security & Compliance:**
- All signatures legally binding under Canadian law
- Full audit trail (who signed, when, from what IP)
- Tamper-evident seal on completed documents
- Encrypted document transmission and storage
- PIPEDA compliant data handling

### 3. AI-Powered Cost Estimation

**Selected Service:** **Anthropic Claude 3.5 Sonnet** (RECOMMENDED) or OpenAI GPT-4

**Why Anthropic Claude:**
- **Superior Reasoning:** Excellent at structured analysis and cost breakdowns
- **Long Context:** 200K token window handles extensive project details and images
- **Vision Capabilities:** Analyzes uploaded project images for accurate estimates
- **Structured Output:** Reliable JSON responses for cost breakdowns
- **Cost-Effective:** Competitive pricing vs OpenAI
- **Canadian Operations:** No data residency issues

**Alternative:** OpenAI GPT-4 Vision (proven, widely adopted, slightly higher cost)

**Data Flow:**
1. Client submits project form with details and images
2. Backend validates and packages submission data
3. System creates background job for AI processing
4. Data sent to AI service with structured prompt
5. AI analyzes scope, images, and market data
6. AI returns cost estimate range with breakdown
7. Result stored in database and client notified

**Implementation Requirements:**
- Async job processing (Bull, BullMQ, or AWS SQS)
- Job queue with retry logic
- Status tracking and updates
- Timeout handling (max 10 minutes)
- Error handling and fallback options
- Notification service integration
- Rate limiting and cost management

**AI Service Integration:**
- API authentication and rate limiting
- Prompt engineering for accurate estimates
- Image analysis capability
- Structured output parsing
- Version tracking for model updates
- A/B testing for estimate accuracy

### 4. Image Upload & Storage

**Selected Service:** **AWS S3** (RECOMMENDED) + CloudFront CDN

**Requirements:**
- Drag-and-drop upload interface
- Multiple file selection
- File type validation (JPG, PNG, HEIC, WebP)
- File size limits (e.g., 10MB per image, 50MB total)
- Client-side image compression before upload
- Server-side image optimization
- Secure signed URLs for uploads
- CDN delivery for fast image loading
- Thumbnail generation
- Image deletion with cleanup

**Implementation:**
- Direct-to-cloud upload with pre-signed URLs
- Progress indicators for uploads
- Image preview functionality
- Automatic format conversion (HEIC to JPG)
- Lazy loading for image galleries
- Responsive image serving (different sizes for different devices)

### 5. Multi-Project Management

**Technical Requirements:**
- Efficient database queries with proper indexing
- Project filtering and sorting
- Pagination for large project lists
- Real-time status updates (WebSocket or polling)
- State management for project data
- Optimistic UI updates
- Data prefetching for better performance

**Database Indexing:**
- Index on `client_id` for fast project lookup
- Composite index on `client_id + status` for filtered queries
- Index on `created_at` for chronological sorting
- Full-text search index on project descriptions (optional)

### 6. Voice Dictation

**Implementation:**
- Web Speech API (browser-based, free)
- Or cloud service (Google Speech-to-Text, AWS Transcribe, Azure Speech)
- Real-time transcription to form field
- Language support (English, French for Canada)
- Microphone permission handling
- Error handling for unsupported browsers
- Fallback to text input

---

## Performance & Scalability

### Optimization Strategies
- **Database:**
  - Connection pooling
  - Query optimization with EXPLAIN
  - Proper indexing strategy
  - Read replicas for scaling reads
  - Partitioning for large tables (projects, images)

- **Caching:**
  - Redis for session storage
  - API response caching (job listings, public profiles)
  - Static asset caching with long TTL
  - CDN for images and static files

- **API:**
  - Rate limiting per user/IP
  - Request pagination (limit/offset or cursor-based)
  - GraphQL DataLoader for N+1 query prevention
  - Background jobs for heavy operations

- **Frontend:**
  - Code splitting and lazy loading
  - Image lazy loading and responsive images
  - Service worker for offline capability
  - Debouncing for search and filters

### Concurrent AI Processing
- Job queue system (Bull/BullMQ on Redis)
- Multiple worker processes for parallel processing
- Priority queue (paid memberships get priority)
- Rate limiting with AI service provider
- Graceful degradation if AI service is down

### Scalability Targets
- Support 10,000+ concurrent users
- Handle 100+ simultaneous AI estimation requests
- Store millions of projects and images
- Sub-second page load times
- 99.9% uptime SLA

---

## Security

### Application Security
- **Authentication:**
  - Secure password hashing (bcrypt with salt rounds ≥ 10)
  - JWT tokens with short expiration
  - Refresh token rotation
  - Session invalidation on logout
  - Protection against brute force (rate limiting)

- **Authorization:**
  - Role-based access control (RBAC)
  - Resource-level permissions
  - API endpoint protection
  - Admin panel security

- **Data Protection:**
  - Encryption at rest (database encryption)
  - Encryption in transit (TLS/HTTPS only)
  - Secure environment variable management
  - No sensitive data in logs
  - Regular security audits

### File Upload Security
- File type validation (MIME type checking)
- File size limits enforced
- Virus/malware scanning (ClamAV or cloud service)
- Prevent path traversal attacks
- Sanitize filenames
- Signed URLs with expiration
- CORS policies for upload endpoints

### API Security
- CORS configuration (whitelist domains)
- Rate limiting per endpoint
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection (Content Security Policy)
- CSRF tokens for state-changing operations
- API versioning for backward compatibility

### Payment Security
- PCI DSS compliance (via payment processor)
- Never store raw credit card data
- Tokenization for saved payment methods
- 3D Secure for fraud prevention
- Transaction monitoring and alerts
- Secure webhook signature verification

### Two-Factor Authentication
- SMS-based verification codes
- TOTP support (optional: Google Authenticator)
- Backup codes for account recovery
- Rate limiting on verification attempts
- Secure code generation and validation

---

## Third-Party Integrations

### Core Integrations (Required)

1. **Payment Processor: Stripe**
   - **Purpose:** Transaction processing, subscriptions, marketplace payouts
   - **Pricing:** 2.9% + $0.30 CAD per transaction, +0.5% for Connect
   - **API:** Stripe Node.js SDK v14+
   - **Features:** Payment Intents, Subscriptions, Connect (marketplace)
   - **Documentation:** https://stripe.com/docs

2. **Digital Signatures: DocuSign**
   - **Purpose:** Contract generation and e-signature workflow
   - **Pricing:** ~$40-60 USD/month for API access
   - **API:** DocuSign eSignature REST API v2.1
   - **Features:** Templates, webhooks, envelope tracking
   - **Documentation:** https://developers.docusign.com

3. **AI Service: Anthropic Claude 3.5 Sonnet**
   - **Purpose:** Cost estimation, project analysis, image analysis
   - **Pricing:** Pay-per-token (cost-effective for startup phase)
   - **API:** Anthropic API (REST)
   - **Features:** Vision, 200K context, structured outputs
   - **Documentation:** https://docs.anthropic.com
   - **Alternative:** OpenAI GPT-4 Vision

4. **Cloud Storage: AWS S3**
   - **Purpose:** Image storage, contract documents, backups
   - **Pricing:** $0.023/GB/month for Standard storage
   - **Buckets:**
     - `ccc-project-images-prod` (project photos)
     - `ccc-contracts-prod` (signed contracts)
     - `ccc-backups-prod` (database backups)
   - **Features:** Lifecycle policies, versioning, encryption

5. **SMS Service: AWS SNS**
   - **Purpose:** 2FA verification codes, payment notifications
   - **Pricing:** $0.00645 per SMS (Canada)
   - **Features:** Delivery reports, international support
   - **Alternative:** Twilio (more expensive but better deliverability)

6. **Email Service: AWS SES**
   - **Purpose:** Transactional emails (verification, notifications, receipts)
   - **Pricing:** $0.10 per 1,000 emails
   - **Features:** High deliverability, bounce handling, templates
   - **Alternative:** SendGrid (better for marketing emails)

7. **Maps/Geocoding: Google Maps API**
   - **Purpose:** Location services, address validation, distance calculation
   - **Pricing:** $5-7 per 1,000 requests
   - **Features:** Geocoding, Places API, Distance Matrix
   - **Usage:** Tradesman service area, job location display

### Recommended Integrations (Highly Beneficial)

8. **Error Tracking: Sentry**
   - **Purpose:** Real-time error monitoring and debugging
   - **Pricing:** Free tier (5,000 events/month), then $26/month
   - **Features:** Stack traces, release tracking, performance monitoring
   - **Integration:** Next.js SDK

9. **Analytics: Google Analytics 4 + Mixpanel**
   - **Purpose:** User behavior tracking, conversion funnels
   - **Pricing:** GA4 free, Mixpanel $20/month for startup plan
   - **Events:** Project submissions, quote acceptances, registrations

10. **Monitoring: AWS CloudWatch + Uptim Robot**
    - **Purpose:** Infrastructure monitoring, uptime alerts
    - **CloudWatch:** Included with AWS, logs and metrics
    - **UptimeRobot:** Free tier for basic uptime monitoring

### Optional Integrations (Future Enhancements)

- **CDN Enhancement:** Cloudflare (additional DDoS protection, analytics)
- **Search:** Elasticsearch or Algolia (if PostgreSQL full-text search insufficient)
- **Customer Support:** Intercom or Zendesk (live chat, ticketing)
- **Marketing Automation:** Mailchimp or HubSpot (email campaigns)
- **Video Calls:** Twilio Video or Daily.co (virtual consultations)
- **SMS Marketing:** Twilio or SimpleTexting (promotional messages)

---

## API Specifications

### REST API Endpoints (Draft)

#### Authentication
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `POST /api/auth/refresh` - Refresh access token
- `POST /api/auth/verify-email` - Email verification
- `POST /api/auth/verify-phone` - Phone verification (2FA)
- `POST /api/auth/forgot-password` - Password reset request
- `POST /api/auth/reset-password` - Password reset confirmation

#### Projects
- `GET /api/projects` - List user's projects
- `POST /api/projects` - Create new project
- `GET /api/projects/:id` - Get project details
- `PATCH /api/projects/:id` - Update project
- `DELETE /api/projects/:id` - Delete project
- `POST /api/projects/:id/submit` - Submit for estimation

#### Images
- `POST /api/projects/:id/images/upload-url` - Get signed upload URL
- `GET /api/projects/:id/images` - List project images
- `DELETE /api/images/:id` - Delete image

#### Estimates
- `GET /api/projects/:id/estimate` - Get AI estimate
- `GET /api/estimates/:id/status` - Check estimation status

#### Jobs (Posted Projects)
- `GET /api/jobs` - List available jobs (tradesmen)
- `GET /api/jobs/:id` - Get job details
- `POST /api/jobs/:id/quote` - Submit quote (tradesman)

#### Quotes
- `GET /api/projects/:id/quotes` - Get quotes for project (client)
- `GET /api/quotes/:id` - Get quote details
- `PATCH /api/quotes/:id/accept` - Accept quote (client)
- `PATCH /api/quotes/:id/reject` - Reject quote (client)

#### Payments
- `POST /api/payments/intent` - Create payment intent
- `POST /api/payments/confirm` - Confirm payment
- `GET /api/payments/history` - Payment history

#### Admin
- `GET /api/admin/users` - List users
- `GET /api/admin/projects` - List all projects
- `GET /api/admin/analytics` - Platform analytics

---

## DevOps & Infrastructure

### Deployment Strategy
- CI/CD pipeline (GitHub Actions, GitLab CI, or CircleCI)
- Automated testing before deployment
- Staging environment for testing
- Blue-green deployment or canary releases
- Database migration automation
- Environment variable management (Secrets Manager)

### Monitoring & Logging
- Application performance monitoring (APM)
- Error tracking and alerting
- Log aggregation (CloudWatch, ELK stack)
- Uptime monitoring (Pingdom, UptimeRobot)
- Custom metrics and dashboards
- Alert thresholds for critical issues

### Backup & Disaster Recovery
- Automated daily database backups
- Point-in-time recovery capability
- Image backup to multiple regions
- Backup retention policy (30 days minimum)
- Disaster recovery runbook
- Regular backup restoration tests

### Development Workflow
- Git branching strategy (GitFlow or trunk-based)
- Code review requirements
- Automated linting and formatting
- Unit and integration testing
- End-to-end testing (Cypress, Playwright)
- Performance testing

---

## Technical Next Steps

### Phase 1: Foundation (Weeks 1-4)
- [ ] Finalize tech stack decisions
- [ ] Set up development environment
- [ ] Configure hosting infrastructure
- [ ] Set up database (schema creation)
- [ ] Implement authentication system
- [ ] Build basic user registration/login

### Phase 2: Core Features (Weeks 5-10)
- [ ] Develop project submission forms
- [ ] Implement image upload functionality
- [ ] Integrate payment processor
- [ ] Build job queue for async processing
- [ ] Integrate AI estimation service
- [ ] Develop project management dashboard

### Phase 3: Marketplace (Weeks 11-14)
- [ ] Build tradesman portal
- [ ] Implement quote submission system
- [ ] Develop job browsing and filtering
- [ ] Build notification system
- [ ] Implement membership tiers

### Phase 4: Polish & Launch (Weeks 15-18)
- [ ] Performance optimization
- [ ] Security audit
- [ ] Load testing
- [ ] Bug fixes and refinements
- [ ] Documentation
- [ ] Beta testing
- [ ] Production launch

---

## Technical Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| AI estimation service downtime | High | Implement retry logic, queue system, fallback to manual estimation |
| Payment processor issues | High | Multiple payment options, clear error messaging, support contact |
| High image storage costs | Medium | Image compression, CDN optimization, storage lifecycle policies |
| Database performance at scale | High | Proper indexing, caching, read replicas, query optimization |
| Security vulnerabilities | Critical | Regular audits, dependency updates, penetration testing |
| API rate limiting | Medium | Caching, request throttling, upgrade plans with providers |

---

## Technical Support & Maintenance

### Support Tools Required
- Admin dashboard for user management
- Analytics dashboard for platform metrics
- Logging/debugging tools for troubleshooting
- Database query tools for investigations
- Feature flags for gradual rollouts

### Maintenance Plan
- Regular security updates (weekly)
- Dependency updates (monthly)
- Database optimization (quarterly)
- Performance audits (quarterly)
- Backup verification (monthly)
- Disaster recovery drills (bi-annually)
