# Task 1: Networking & Subnetting (AWS VPC Setup)

## Approach

This task implements a foundational AWS network architecture using Terraform. The design follows AWS best practices for a multi-tier network:

1. **VPC Design**: Created a VPC with CIDR block `10.0.0.0/16`, providing 65,536 IP addresses for scalability.
2. **Public Subnets**: Two public subnets (`10.0.1.0/24` and `10.0.2.0/24`) across different AZs for high availability, each supporting 256 hosts for internet-facing resources.
3. **Private Subnets**: Two private subnets (`10.0.11.0/24` and `10.0.12.0/24`) isolated from direct internet access for backend services and databases.
4. **Internet Gateway**: Attached to VPC to enable public subnet internet connectivity.
5. **NAT Gateway**: Deployed in the first public subnet to allow private subnet resources to access the internet for updates while remaining protected from inbound traffic.

## CIDR Range Explanation

- **VPC CIDR**: `10.0.0.0/16` - Provides a large address space (65,536 IPs) for future growth
- **Public Subnet 1**: `10.0.1.0/24` - 256 IPs in AZ 1
- **Public Subnet 2**: `10.0.2.0/24` - 256 IPs in AZ 2
- **Private Subnet 1**: `10.0.11.0/24` - 256 IPs in AZ 1
- **Private Subnet 2**: `10.0.12.0/24` - 256 IPs in AZ 2

The `/24` subnets provide sufficient capacity for typical use cases while leaving room for additional subnets. The spacing (1.x for public, 11.x for private) allows easy identification and future expansion.

## Deployment Instructions


2. **Initialize Terraform**:
   ```bash
   terraform init
   ```
3. **Review the plan**:
   ```bash
   terraform plan
   ```
4. **Apply the configuration**:
   ```bash
   terraform apply
   ```
5. **Take screenshots** of:
   - VPC dashboard
   - Subnets list
   - Route tables
   - NAT Gateway and Internet Gateway

6. **Add screenshots below** (after taking them in AWS Console)

## AWS Screenshots

### VPC
![VPC Screenshot](screenshots/vpc.png)

### Subnets
![Subnets Screenshot](screenshots/subnets.png)

### Route Tables
![Route Tables Screenshot](screenshots/route_tables.png)

### NAT Gateway & Internet Gateway
![NAT and IGW Screenshot](screenshots/nat_igw.png)

