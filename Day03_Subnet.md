# Day 3: Create Subnet

**Date:** 2026-01-12

## What

A subnet is a range of IP addresses within a VPC. Each subnet lives in exactly one **Availability Zone (AZ)**. Subnets can be **public** (route to internet gateway) or **private** (no direct internet route).

## Why

- To divide a VPC into smaller networks per AZ.
- To separate workloads — e.g., web servers in public subnets, databases in private subnets.
- For high availability — spread resources across subnets in different AZs.

## Steps (Console)

1. Go to **VPC** → left sidebar → **Subnets**.
2. Click **Create subnet**.
3. Choose:
   - **VPC ID:** the VPC you want this subnet inside
   - **Subnet name:** e.g., `public-subnet-1a`
   - **Availability Zone:** e.g., `ap-south-1a`
   - **IPv4 CIDR block:** must be within the VPC's CIDR (e.g., `10.0.1.0/24`)
4. Click **Create subnet**.
5. (For public subnet) Edit **Route Table** → add route `0.0.0.0/0` → Internet Gateway.
6. (Optional) Enable **Auto-assign public IPv4** in subnet settings.

## Steps (CLI)

```bash
aws ec2 create-subnet \
  --vpc-id vpc-0abc123 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ap-south-1a
```

## Notes

- A subnet is **public** only if it has a route to an Internet Gateway in its route table.
- AWS reserves **5 IPs in every subnet** — network, VPC router, DNS, future use, broadcast. So a `/24` subnet has 251 usable IPs, not 256.
- One subnet = one AZ. To get HA, create at least 2 subnets in 2 different AZs.
- CIDR sizing: `/28` (smallest, 16 IPs) to `/16` (largest, 65,536 IPs).
