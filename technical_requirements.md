# Construction Connect Canada - Technical Requirements

## Technical Overview

This document outlines the technical specifications and implementation requirements for Construction Connect Canada, a web-based platform built with React/Next.js that connects clients with tradesmen for construction services and provides AI-powered cost estimation.

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

### 2. AI-Powered Cost Estimation

**Selected Service:** TBD (OpenAI, Anthropic, custom ML model, or specialized construction estimation API)

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

### 3. Image Upload & Storage

**Selected Service:** TBD (AWS S3, Google Cloud Storage, Cloudinary, or similar)

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

### 4. Multi-Project Management

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

### 5. Voice Dictation

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

### Required Integrations
1. **Payment Processor:** Stripe/Square/PayPal
2. **AI Service:** OpenAI/Anthropic/Custom ML
3. **Cloud Storage:** AWS S3/GCS/Cloudinary
4. **SMS Service:** Twilio/AWS SNS (for 2FA and notifications)
5. **Email Service:** SendGrid/AWS SES/Mailgun
6. **Maps API:** Google Maps (for location services)

### Optional Integrations
- Analytics: Google Analytics, Mixpanel, Amplitude
- Error Tracking: Sentry, Rollbar, Bugsnag
- Monitoring: DataDog, New Relic, Grafana
- CDN: CloudFront, Cloudflare
- Search: Algolia, Elasticsearch

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
