# Task 5: AWS Architecture Diagram (draw.io)

## Architecture Overview

This architecture is designed to support a highly scalable web application capable of handling 10,000+ concurrent users with the following key characteristics:

**High Availability & Fault Tolerance**: Multi-AZ deployment across 3 availability zones ensures 99.99% uptime. Application Load Balancer distributes traffic across healthy instances, with Auto Scaling Group automatically replacing failed instances. RDS Multi-AZ provides database failover within 60 seconds.

**Scalability**: Auto Scaling Group scales from 4 to 20 instances based on CPU and request count metrics. ElastiCache Redis cluster handles session management and reduces database load by 70%. CloudFront CDN caches static content globally, reducing origin load and improving response times.

**Security**: Multi-layer security with public subnets for ALB only, private subnets for application servers and databases. Security Groups follow least privilege principle. WAF protects against common web exploits (SQL injection, XSS, DDoS). Secrets Manager handles database credentials securely. VPC Flow Logs and CloudTrail provide audit trails.

**Database & Caching**: Aurora PostgreSQL with read replicas handles up to 500,000 queries/minute. ElastiCache Redis (cluster mode) provides sub-millisecond response times for session data and frequently accessed content. Automatic backups ensure data durability.

**Observability**: CloudWatch monitors all services with custom dashboards for application metrics. CloudWatch Logs Insights enables real-time log analysis. X-Ray provides distributed tracing for performance bottlenecks. SNS sends alerts for critical events (high error rates, scaling events, database issues).

## Traffic Flow

```
User Request
    ↓
Route 53 DNS Resolution
    ↓
CloudFront CDN (if static content) → S3 (static assets)
    ↓
AWS WAF (security filtering)
    ↓
Application Load Balancer (Public Subnets, 3 AZs)
    ↓
Target Group Health Checks
    ↓
EC2 Instances (Private Subnets, Auto Scaling Group)
    ↓
ElastiCache Redis (session/cache check)
    ↓
Aurora PostgreSQL (database queries)
    ↓
Response back through ALB → CloudFront → User
```

## Scalability Considerations

1. **10,000 Concurrent Users**: With 500 requests/instance capacity, 20 instances (max ASG size) can handle 10,000 concurrent users with headroom.
2. **Database Scaling**: Aurora read replicas distribute read traffic (80% of queries). Write queries go to master instance.
3. **Caching Strategy**: Redis caches frequently accessed data (user sessions, product catalogs), reducing database load by 70%.
4. **CDN Offloading**: CloudFront serves static content (images, CSS, JS), reducing origin server load by 60%.
5. **Auto Scaling Policies**: 
   - Scale up when CPU > 70% or request count > 1000/instance
   - Scale down when CPU < 30% and request count < 300/instance


## Sample Diagram Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Internet Users                              │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Route 53      │
                    │   (DNS)         │
                    └────────┬────────┘
                             │
                    ┌────────▼─────────┐
                    │  CloudFront CDN   │
                    │  + AWS WAF        │
                    └────────┬──────────┘
                             │
┌────────────────────────────▼─────────────────────────────────────────┐
│                            VPC (10.0.0.0/16)                          │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │              Application Load Balancer (Public)                  │ │
│  │     [AZ1: 10.0.1.0/24] [AZ2: 10.0.2.0/24] [AZ3: 10.0.3.0/24]   │ │
│  └────────────┬──────────────────┬──────────────────┬──────────────┘ │
│               │                  │                  │                 │
│  ┌────────────▼─────────┬────────▼────────┬────────▼────────┐       │
│  │   EC2 Auto Scaling Group (Private Subnets)                │       │
│  │   [AZ1: 10.0.11.0/24] [AZ2: 10.0.12.0/24] [AZ3: 10.0.13.0/24] │  │
│  │   Min: 4  Max: 20  Desired: 4                              │       │
│  └────────────┬──────────────────────────────────────────────┘       │
│               │                                                        │
│  ┌────────────▼─────────────┬───────────────────────┐                │
│  │  ElastiCache Redis       │   Aurora PostgreSQL   │                │
│  │  (3 nodes, Multi-AZ)     │   (Master + 2 Replicas│                │
│  │  [Database Subnets]      │    Multi-AZ)          │                │
│  └──────────────────────────┴───────────────────────┘                │
│                                                                        │
│  Monitoring: CloudWatch + X-Ray + CloudTrail                         │
└────────────────────────────────────────────────────────────────────────┘
```

## Diagram Screenshot

![AWS Architecture Diagram](aws_architecture_diagram.pdf)

*Diagram created with draw.io showing a highly scalable AWS architecture for 10,000+ concurrent users*
