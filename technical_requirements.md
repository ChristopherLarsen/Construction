# Construction Connect Canada - Technical Requirements

## Technical Overview

This document outlines the technical specifications and implementation requirements for Construction Connect Canada, a web-based platform built with React/Next.js that connects clients with tradesmen for construction services and provides AI-powered cost estimation.

---

## Technology Stack

### Frontend
- **Framework:** React/Next.js
- **Language:** TypeScript (recommended)
- **Styling:** CSS-in-JS or Tailwind CSS
- **State Management:** TBD (Redux, Zustand, or React Context)
- **Form Handling:** React Hook Form or Formik

### Backend
- **API:** RESTful or GraphQL
- **Language:** Node.js/TypeScript or alternative
- **Framework:** TBD (Express, NestJS, Next.js API routes)

### Database
- **Primary Database:** TBD (PostgreSQL, MySQL, or MongoDB)
- **Caching Layer:** Redis (recommended)
- **Search:** Elasticsearch or similar (for job matching)

### Infrastructure
- **Hosting:** TBD (AWS, Google Cloud, Azure, or Vercel)
- **CDN:** CloudFront, Cloudflare, or similar
- **Image Storage:** S3, Google Cloud Storage, or Cloudinary
- **Monitoring:** TBD (DataDog, New Relic, or similar)

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

**Selected Service:** TBD (Stripe, Square, PayPal, or Canadian provider)

**Requirements:**
- PCI DSS compliance (handled by payment processor)
- Support for credit/debit cards
- Support for ACH/bank transfers (optional)
- Recurring billing for tradesman memberships
- One-time payments for project completion
- Refund handling
- Invoice generation
- Transaction history
- Webhook handling for payment events

**API Integration:**
- Payment intent creation
- Payment confirmation
- Subscription management
- Webhook endpoints for status updates
- Secure token handling (never store card data)

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
