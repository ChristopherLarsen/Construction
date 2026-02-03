# Tech Stack Cost Estimate
## Construction Connect Canada

**Assumptions:**
- 10,000 user sessions per month
- 100 completed contracts per month
- Average project value: $15,000 (tradesman quote)
- Average platform commission: 15% = $2,250
- Average total transaction: $17,250
- Estimated AI estimate requests: 2,000/month (20% of sessions)
- Average 5 images per project

---

## Infrastructure Costs (AWS)

### Core Infrastructure
| Service | Configuration | Monthly Cost |
|---------|--------------|--------------|
| **EC2 (Application Servers)** | 2x t3.medium instances (load balanced) | $120 |
| **RDS PostgreSQL** | db.t3.medium (Multi-AZ, 100GB SSD) | $180 |
| **ElastiCache Redis** | cache.t3.small (2 nodes for HA) | $85 |
| **S3 Storage** | 10GB images + 5GB contracts + backups | $0.35 |
| **CloudFront CDN** | Data transfer and caching | $50 |
| **Application Load Balancer** | High availability traffic distribution | $25 |
| **CloudWatch** | Logs, metrics, monitoring | $15 |
| **Secrets Manager** | API keys, credentials storage | $5 |
| **Background Workers (EC2)** | 1x t3.small for BullMQ workers | $25 |
| **Data Transfer** | Outbound data transfer | $20 |
| | **Infrastructure Subtotal** | **$525/month** |

**Note:** This aligns with the $300-500/month estimate for moderate traffic in the technical requirements. At 10,000 sessions, we're toward the higher end.

---

## Third-Party Service Costs

### Payment Processing
| Service | Usage | Calculation | Monthly Cost |
|---------|-------|-------------|--------------|
| **Stripe** | 100 contracts @ avg $17,250 | 3.4% + $0.30 per transaction | **$58,680** |

**Note:** This is a **pass-through cost** paid by customers. Platform revenue (commission) is $225,000/month (100 contracts × $2,250 avg commission), so Stripe fees represent 26% of the transaction value or 7.7% of the platform's gross revenue.

**Stripe fees breakdown:**
- Base transaction fee: 2.9% + $0.30
- Stripe Connect marketplace fee: +0.5%
- Total per transaction: ($17,250 × 0.034) + $0.30 = $586.80
- 100 transactions: $58,680/month

---

### Core Services
| Service | Usage | Calculation | Monthly Cost |
|---------|-------|-------------|--------------|
| **DocuSign** | API access + 100 envelopes | ~$50 USD/month | $70 CAD |
| **Anthropic Claude AI** | 2,000 estimates/month | $0.03 per estimate | $60 |
| **AWS SNS (SMS)** | 1,200 SMS (2FA + notifications) | $0.00645 per SMS | $8 |
| **AWS SES (Email)** | 2,500 emails/month | $0.10 per 1,000 | $0.25 |
| **Google Maps API** | 2,000 requests/month | $6 per 1,000 requests | $12 |
| | **Core Services Subtotal** | | **$150/month** |

---

### Monitoring & Analytics
| Service | Usage | Monthly Cost |
|---------|-------|--------------|
| **Sentry** | Error tracking, performance monitoring | $26 |
| **Mixpanel** | User analytics, conversion funnels | $20 |
| **UptimeRobot** | Uptime monitoring | $0 (free tier) |
| | **Monitoring Subtotal** | **$46/month** |

---

## Total Tech Stack Costs

### Monthly Operating Costs
| Category | Cost | % of Total |
|----------|------|------------|
| **AWS Infrastructure** | $525 | 42% |
| **Stripe Processing Fees** | $58,680 | — (pass-through) |
| **Core Services** | $150 | 12% |
| **Monitoring & Analytics** | $46 | 4% |
| **Help Desk Software** | $300 | 24% |
| | | |
| **Total Tech Stack (excluding Stripe)** | **$1,021/month** | |
| **Total Including Stripe Fees** | **$59,701/month** | |

---

## Cost Analysis

### Platform Economics (100 contracts/month)

**Revenue:**
- Gross Merchandise Value (GMV): $1,725,000 (100 × $17,250)
- Platform Commission Revenue: $225,000 (avg 15% commission)

**Tech Stack Costs:**
- Core Platform Operations: $1,021/month
- Stripe Processing Fees: $58,680/month (passed to customers)
- **Total Tech Costs: $59,701/month**

**Tech Stack as % of Revenue:**
- Core platform costs: $1,021 / $225,000 = **0.45% of revenue**
- Including Stripe fees: $59,701 / $225,000 = **26.5% of revenue**

**Important Note:** Stripe fees are typically added to the customer's total, not deducted from platform revenue. So the actual tech stack cost burden on the platform is just **$1,021/month or 0.45% of revenue**.

---

## Cost Breakdown by Service Priority

### Essential Services (Cannot operate without)
| Service | Monthly Cost |
|---------|--------------|
| AWS Infrastructure (compute, database, storage) | $525 |
| Stripe Payment Processing | $58,680 (pass-through) |
| DocuSign Contracts | $70 |
| AWS SNS/SES (notifications) | $8 |
| | **$59,283** |

### High-Value Services (Core features)
| Service | Monthly Cost |
|---------|--------------|
| Anthropic Claude AI Estimation | $60 |
| Google Maps API | $12 |
| Sentry Error Tracking | $26 |
| | **$98** |

### Growth & Optimization Services
| Service | Monthly Cost |
|---------|--------------|
| Mixpanel Analytics | $20 |
| Help Desk Software | $300 |
| | **$320** |

---

## Scaling Projections

### At 50,000 Sessions/Month (500 contracts)

| Category | 10K Sessions | 50K Sessions | Increase |
|----------|--------------|--------------|----------|
| **AWS Infrastructure** | $525 | $1,200 | +129% |
| **Stripe Fees** | $58,680 | $293,400 | +400% |
| **AI Estimation** | $60 | $300 | +400% |
| **SMS/Email** | $8 | $40 | +400% |
| **Google Maps** | $12 | $60 | +400% |
| **Other Services** | $416 | $516 | +24% |
| | | | |
| **Total (excl. Stripe)** | $1,021 | $2,116 | +107% |
| **Total (incl. Stripe)** | $59,701 | $295,516 | +395% |

**Key Insights:**
- Core platform costs scale sub-linearly with traffic (2x users = 2x cost)
- Variable costs (AI, SMS, Maps) scale linearly with usage
- Stripe fees scale with transaction volume and value
- Infrastructure shows good economies of scale (10GB → 50GB storage is minimal cost increase)

---

## Cost Optimization Opportunities

### Short-Term Savings
1. **Reserved Instances:** EC2 and RDS reserved instances could save 30-40% ($210/month savings)
2. **S3 Lifecycle Policies:** Move old images to Glacier ($0.004/GB) after 90 days
3. **CloudFront Optimization:** Aggressive caching could reduce S3 requests by 60%
4. **SMS Optimization:** Use email where possible, SMS only for critical notifications

**Potential Savings:** $100-150/month (10-15% reduction)

### Medium-Term Optimizations
1. **AI Model Optimization:** Fine-tune prompts to reduce token usage by 20% ($12/month savings at current scale)
2. **Database Read Replicas:** Add read replicas instead of scaling primary (+$90/month) to avoid more expensive vertical scaling
3. **CDN Edge Caching:** Implement aggressive edge caching for image variants

### Long-Term Considerations
1. **Stripe Negotiation:** At $1M+/month volume, negotiate lower transaction fees (possible 0.2-0.4% reduction)
2. **AWS Enterprise Support:** At $5K+/month AWS spend, consider Enterprise Support for better pricing
3. **Multi-Region Deployment:** Future Canada-wide coverage may require US East region addition

---

## Budget Recommendations

### Conservative Monthly Budget (10K sessions, 100 contracts)
- **Tech Stack:** $1,200/month (buffer for overages)
- **Stripe Fees:** Passed through to customers
- **Emergency Fund:** $300/month (25% buffer for spikes)
- **Total Tech Budget:** $1,500/month

### Growth Budget (50K sessions, 500 contracts)
- **Tech Stack:** $2,500/month
- **Stripe Fees:** Passed through to customers
- **Emergency Fund:** $500/month
- **Total Tech Budget:** $3,000/month

---

## Risk Factors & Contingencies

### Potential Cost Overruns
1. **AI Estimation Costs:** If estimate requests exceed 20% of sessions or token usage is higher
2. **SMS Costs:** 2FA fraud or notification spam could increase SMS volume
3. **S3 Storage:** If users upload more than 5 images per project
4. **Data Transfer:** High-resolution images and document downloads

### Mitigation Strategies
1. **Rate Limiting:** Prevent abuse of AI estimation and API calls
2. **Image Compression:** Aggressive client-side compression before upload
3. **Usage Monitoring:** CloudWatch alarms for unusual cost spikes
4. **Budget Alerts:** AWS Budget alerts at 80% and 100% of expected spend

---

## Summary

**For 10,000 user sessions and 100 completed contracts per month:**

- **Core Tech Stack Cost:** $1,021/month
- **As % of Platform Revenue:** 0.45%
- **Per Contract:** $10.21
- **Per User Session:** $0.10

**This represents excellent unit economics** for a marketplace platform. The tech stack cost is minimal compared to revenue, leaving room for:
- Marketing and customer acquisition
- Customer support and operations
- Product development
- Profit margins

**Stripe processing fees** ($58,680/month) are substantial but passed through to customers, so they don't impact platform profitability directly.

---

*Last Updated: February 2026*
*Based on: technical_requirements.md and business_overview.md*
