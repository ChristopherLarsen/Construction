---
layout: default
title: Technical Requirements - Construction Connect Canada
---

# 🔧 Technical Requirements

**[← Back to Home](index.md)**

Construction Connect Canada - Platform Architecture & Implementation Guide

## Technical Overview

This document outlines the technical specifications and implementation requirements for Construction Connect Canada, a web-based platform built with React/Next.js that connects clients with tradesmen for construction services and provides AI-powered cost estimation.

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
| **Payment Processing** | **Stripe** | Best-in-class API, Stripe Connect for marketplace, 2.9% + $0.30 CAD |
| **Digital Signatures** | **DocuSign** | Industry standard, legally binding in Canada, comprehensive features |
| **AI Estimation** | **Anthropic Claude 3.5 Sonnet** | Superior reasoning, vision capabilities, 200K context, cost-effective |
| **Object Storage** | **AWS S3** | Reliable, scalable, cost-effective ($0.023/GB/month) |
| **CDN** | **AWS CloudFront** | Fast global delivery, integrated with S3 |
| **Cache & Queue** | **Redis (ElastiCache)** | Session storage, API caching, job queue (BullMQ) |
| **SMS (2FA)** | **AWS SNS** | Cost-effective ($0.00645/SMS), reliable delivery |
| **Email** | **AWS SES** | Low cost ($0.10/1000 emails), high deliverability |
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
- Stripe Connect purpose-built for marketplaces
- Superior developer experience and API documentation
- Native subscription management for tradesman memberships
- Better webhook reliability and testing tools
- More transparent pricing with no hidden fees
- Easier to implement split payments

**Why PostgreSQL over MySQL/MongoDB?**
- ACID compliance critical for financial transactions
- Superior relational data modeling
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
    - Scalability: Easily scale from startup to enterprise
    - Reliability: 99.99% uptime SLA, proven infrastructure
    - Ecosystem: Complete suite of services (compute, storage, database, queues)
    - Cost-effective: Pay-as-you-grow model ideal for startup phase
    - Canadian presence: AWS Canada (Central) region for data sovereignty
    - Superior to Hostinger: Hostinger is for basic web hosting, not production SaaS platforms

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

## Server Architecture

### Recommended Architecture: Modular Monolith (Phase 1) → Microservices (Phase 2+)

**Phase 1: Modular Monolith (Launch through first 10,000 users)**

Start with a well-structured monolithic application that's designed for future separation with clear domain boundaries for easy extraction.

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

### Phase 2: Microservices Architecture (Scale beyond 50,000 users)

As the platform grows, separate into independent services with API Gateway coordination.

**When to Migrate:**
- Traffic exceeds single server capacity (>10,000 DAU)
- Need independent scaling of AI estimation service
- Team size grows beyond 10 developers
- Database queries slow despite optimization
- Feature development conflicts in monolith

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

### User Roles & Permissions

| Role | Permissions |
|------|-------------|
| Client | Create projects, upload images, view estimates, post jobs, review quotes, make payments, manage profile |
| Tradesman | Browse jobs, submit quotes, view client requirements, manage profile, update credentials, track awarded jobs |
| Admin | Full access, user management, platform configuration, analytics, dispute resolution |

## Core Feature Implementation

### 1. Payment Processing - Stripe

**Why Stripe over PayPal:**
- Best-in-class API with comprehensive, well-documented developer experience
- Stripe Connect purpose-built for marketplace platforms
- Superior webhook reliability and testing tools
- Native subscription management for tradesman memberships
- More transparent pricing with no hidden fees
- Easier to implement split payments (platform commission + tradesman payout)

**Payment Flow:** Client pays total (project cost + platform commission) via Stripe Payment Intent. Upon completion, Stripe automatically transfers project cost to tradesman's account and platform retains commission.

### 2. Digital Signatures - DocuSign

**Why DocuSign:**
- Industry standard for construction contracts
- Legal validity recognized in Canadian courts
- Professional appearance enhances platform credibility
- Comprehensive API for full automation
- Audit trail and tamper-evident seals

### 3. AI-Powered Cost Estimation

**Selected Service: Anthropic Claude 3.5 Sonnet**

**Why Anthropic Claude:**
- Superior reasoning for structured cost analysis and breakdowns
- 200K token context window (vs GPT-4's 128K)
- Excellent vision capabilities for analyzing project images
- Reliable JSON structured outputs for cost breakdowns
- Competitive pricing, cost-effective for startup phase
- No data residency issues for Canadian operations

## Performance & Scalability

### Optimization Strategies

- **Database:** Connection pooling, query optimization, proper indexing, read replicas
- **Caching:** Redis for sessions, API response caching, CloudFront CDN
- **API:** Rate limiting, pagination, request optimization
- **Frontend:** Code splitting, lazy loading, service workers, image optimization

### Scalability Targets

- Support 10,000+ concurrent users
- Handle 100+ simultaneous AI estimation requests
- Store millions of projects and images
- Sub-second page load times
- 99.9% uptime SLA

## Security

### Application Security

- **Authentication:** Secure password hashing (bcrypt), JWT tokens, refresh token rotation
- **Authorization:** Role-based access control, resource-level permissions, API endpoint protection
- **Data Protection:** Encryption at rest and in transit (TLS/HTTPS), secure environment management

### Payment Security

- PCI DSS compliance (via Stripe)
- Never store raw credit card data
- Tokenization for saved payment methods
- 3D Secure for fraud prevention
- Secure webhook signature verification

### Two-Factor Authentication

- SMS-based verification codes
- TOTP support (optional: Google Authenticator)
- Backup codes for account recovery
- Rate limiting on verification attempts

## Third-Party Integrations

| Service | Purpose | Pricing |
|---------|---------|---------|
| **Stripe** | Payment processing, subscriptions, marketplace | 2.9% + $0.30 CAD per transaction |
| **DocuSign** | Contract generation and e-signatures | ~$40-60 USD/month |
| **Anthropic Claude** | AI-powered cost estimation | Pay-per-token (cost-effective) |
| **AWS S3** | Image and document storage | $0.023/GB/month |
| **AWS SNS** | SMS for 2FA and notifications | $0.00645 per SMS |
| **AWS SES** | Transactional emails | $0.10 per 1,000 emails |
| **Google Maps API** | Location services and geocoding | $5-7 per 1,000 requests |
| **Sentry** | Error tracking and monitoring | Free tier or $26/month |

## Deployment & Operations

### Recommended Deployment Strategy

- CI/CD pipeline (GitHub Actions, GitLab CI, or CircleCI)
- Automated testing before deployment
- Staging environment for testing
- Blue-green deployment or canary releases
- Database migration automation
- Environment variable management via AWS Secrets Manager

### Monitoring & Logging

- Application performance monitoring (APM)
- Error tracking and alerting (Sentry)
- Log aggregation (CloudWatch)
- Uptime monitoring (UptimeRobot)
- Custom metrics and dashboards

## Development Roadmap

### Phase 1: Foundation (Weeks 1)

- Finalize tech stack decisions
- Set up development environment
- Configure hosting infrastructure
- Set up database and schema
- Implement authentication system
- Build basic user registration/login

### Phase 2: Core Features (Weeks 2)

- Develop project submission forms
- Implement image upload functionality
- Integrate payment processor
- Build job queue for async processing
- Integrate AI estimation service
- Develop project management dashboard

### Phase 3: Marketplace (Week 3)

- Build tradesman portal
- Implement quote submission system
- Develop job browsing and filtering
- Build notification system
- Implement membership tiers

### Phase 4: Polish & Launch (Week 4)

- Performance optimization
- Security audit
- Load testing
- Bug fixes and refinements
- Documentation and beta testing
- Production launch

---

**[← Back to Home](index.md)**
