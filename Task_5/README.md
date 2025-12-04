# Task 5: AWS Architecture Diagram (draw.io)

## Architecture Overview

This architecture is designed to support a highly scalable web application capable of handling 10,000+ concurrent users with the following key characteristics:

**High Availability & Fault Tolerance**: Multi-AZ deployment across 3 availability zones ensures 99.99% uptime. Application Load Balancer distributes traffic across healthy instances, with Auto Scaling Group automatically replacing failed instances. RDS Multi-AZ provides database failover within 60 seconds.

**Scalability**: Auto Scaling Group scales from 4 to 20 instances based on CPU and request count metrics. ElastiCache Redis cluster handles session management and reduces database load by 70%. CloudFront CDN caches static content globally, reducing origin load and improving response times.

**Security**: Multi-layer security with public subnets for ALB only, private subnets for application servers and databases. Security Groups follow least privilege principle. WAF protects against common web exploits (SQL injection, XSS, DDoS). Secrets Manager handles database credentials securely. VPC Flow Logs and CloudTrail provide audit trails.

**Database & Caching**: Aurora PostgreSQL with read replicas handles up to 500,000 queries/minute. ElastiCache Redis (cluster mode) provides sub-millisecond response times for session data and frequently accessed content. Automatic backups ensure data durability.

**Observability**: CloudWatch monitors all services with custom dashboards for application metrics. CloudWatch Logs Insights enables real-time log analysis. X-Ray provides distributed tracing for performance bottlenecks. SNS sends alerts for critical events (high error rates, scaling events, database issues).

## Architecture Components

### 1. **Edge & Security Layer**
- **Route 53**: DNS management with health checks and failover routing
- **CloudFront**: Global CDN for static asset delivery and DDoS protection
- **AWS WAF**: Web Application Firewall protecting against OWASP Top 10 vulnerabilities

### 2. **Load Balancing & Compute**
- **Application Load Balancer (ALB)**: Internet-facing, spans 3 public subnets
- **Auto Scaling Group**: 4-20 EC2 instances (t3.medium) across 3 private subnets
- **Launch Template**: Immutable infrastructure with automated deployments

### 3. **Data Layer**
- **Amazon Aurora PostgreSQL**: Multi-AZ deployment with 2 read replicas
  - Master in private subnet AZ1
  - Read replicas in AZ2 and AZ3
  - Automated backups and point-in-time recovery
- **ElastiCache Redis**: Cluster mode enabled, 3 nodes across AZs
  - Session management
  - Application-level caching
  - Reduces database queries by 70%

### 4. **Networking**
- **VPC**: 10.0.0.0/16 CIDR block
- **Public Subnets**: 3 subnets (10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24) for ALB
- **Private Subnets**: 3 subnets (10.0.11.0/24, 10.0.12.0/24, 10.0.13.0/24) for app servers
- **Database Subnets**: 3 subnets (10.0.21.0/24, 10.0.22.0/24, 10.0.23.0/24) for RDS
- **NAT Gateways**: 3 NAT Gateways (one per AZ) for high availability
- **Internet Gateway**: Single IGW for public subnet internet access

### 5. **Security Components**
- **Security Groups**: 
  - ALB SG: Inbound 80/443 from internet
  - App SG: Inbound 80 from ALB SG only
  - DB SG: Inbound 5432 from App SG only
  - Cache SG: Inbound 6379 from App SG only
- **NACLs**: Network-level access control lists for subnet isolation
- **AWS Secrets Manager**: Secure storage for database credentials
- **IAM Roles**: EC2 instance roles with least privilege access

### 6. **Observability & Monitoring**
- **CloudWatch Metrics**: Custom application metrics, infrastructure monitoring
- **CloudWatch Logs**: Centralized logging with log groups for ALB, EC2, RDS
- **CloudWatch Alarms**: Alerts for CPU, memory, disk, error rates, latency
- **X-Ray**: Distributed tracing for performance analysis
- **SNS Topics**: Alert notifications to operations team

### 7. **Additional Services**
- **S3 Buckets**: 
  - Static asset storage (images, CSS, JS)
  - Application logs archival
  - Database backup storage
- **CloudTrail**: Audit logging for compliance
- **Systems Manager**: Parameter Store for configuration management

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

## Cost Optimization

- Reserved Instances for baseline capacity (4 instances)
- Spot Instances for burst capacity (beyond 4 instances)
- S3 Lifecycle policies to move old logs to Glacier
- CloudFront reduces data transfer costs
- Aurora Serverless v2 option for variable workloads

## Disaster Recovery

- **RPO**: 5 minutes (Aurora continuous backup)
- **RTO**: 15 minutes (automated failover + ASG replacement)
- **Multi-Region**: Can extend to multi-region active-passive setup for global resilience

## Draw.io Diagram Instructions

### Creating the Diagram

1. **Access draw.io**: Visit https://app.diagrams.net/
2. **Select Storage**: Choose "Device" to save locally or "Google Drive"/"OneDrive" for cloud storage
3. **Import Template** (Option 1): 
   - Download AWS architecture icons: https://aws.amazon.com/architecture/icons/
   - Or use built-in AWS shapes in draw.io (More Shapes → AWS19)
4. **Create New Diagram** (Option 2):
   - Start with blank diagram
   - Enable AWS shape library: More Shapes → Search "AWS" → Enable AWS19

### Diagram Layout Recommendations

**Left to Right Flow** (recommended for web applications):
```
[Users/Internet] → [Route 53] → [CloudFront/WAF] → [ALB] → [EC2 Auto Scaling] → [Database/Cache]
```

**Top to Bottom Layers**:
- Layer 1 (Top): Edge services (Route 53, CloudFront, WAF)
- Layer 2: Load Balancers (ALB) in public subnets
- Layer 3: Application servers (EC2 ASG) in private subnets
- Layer 4: Data layer (RDS, ElastiCache) in database subnets
- Layer 5 (Bottom): Monitoring and security services

### Key Elements to Include

1. **VPC Boundary**: Large rectangle encompassing all AWS resources
2. **Availability Zones**: 3 distinct columns/sections within VPC
3. **Subnet Types**: Color-coded (blue for public, green for private, orange for database)
4. **Security Groups**: Small icons or dotted lines showing security boundaries
5. **Arrows**: Show traffic flow direction (use colors: blue for user traffic, red for admin/monitoring)
6. **Icons**: Use official AWS icons for authenticity
7. **Labels**: Add text labels for CIDRs, instance types, and capacities

### Exporting the Diagram

1. **PNG Export** (for documentation):
   - File → Export as → PNG
   - Check "Transparent Background" (optional)
   - Check "Selection Only" (if you want to export part)
   - Choose resolution (300 DPI recommended for print quality)
   - Click "Export"

2. **PDF Export** (for high quality):
   - File → Export as → PDF
   - Check "Crop to content"
   - Click "Export"

3. **SVG Export** (for scalability):
   - File → Export as → SVG
   - Best for web use and future editing

4. **Save Source File**:
   - File → Save as → Choose .drawio format
   - Keep the source file for future edits

### Tips for Professional Diagrams

- **Use a Grid**: Enable grid snapping (View → Grid) for alignment
- **Consistent Sizing**: Keep similar components the same size
- **Color Coding**: Use consistent colors for component types
- **Legends**: Add a legend explaining colors and symbols
- **Title & Date**: Include diagram title, version, and creation date
- **Annotations**: Add notes for non-obvious design decisions

## Uploading to GitHub

1. Export diagram as PNG: `architecture_diagram.png`
2. Export diagram as PDF (optional): `architecture_diagram.pdf`
3. Save source file: `architecture_diagram.drawio`
4. Place all files in the `Task_5` folder
5. Update README.md with embedded image:
   ```markdown
   ![Architecture Diagram](architecture_diagram.png)
   ```

## AWS Screenshots to Include

While this task is primarily about creating a diagram, you may want to include screenshots of:
- Existing multi-AZ deployments (if you've implemented parts of this)
- CloudWatch dashboard showing metrics
- Auto Scaling Group configuration
- RDS Multi-AZ setup

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

## Additional Resources

- AWS Architecture Center: https://aws.amazon.com/architecture/
- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- AWS Reference Architectures: https://github.com/aws-samples/aws-refarch-webapp
- Draw.io AWS Examples: https://www.diagrams.net/blog/aws-diagrams

---

**Note**: After creating your diagram, add the screenshot below and update this README with any specific design decisions you made.

## Diagram Screenshot

![AWS Architecture Diagram](architecture_diagram.png)

*Diagram created with draw.io showing a highly scalable AWS architecture for 10,000+ concurrent users*
