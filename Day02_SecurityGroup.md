# Day 2: Create Security Group

**Date:** 2026-01-12

## What

A security group is a virtual firewall for an EC2 instance (or any ENI). It controls **inbound** and **outbound** traffic at the instance level. Security groups are **stateful** — if you allow inbound traffic, the response is automatically allowed out (and vice versa).

## Why

- To control who can reach your EC2 instance and on which ports.
- Acts as the first line of defense before the OS firewall.
- You can attach the same security group to multiple instances.

## Steps (Console)

1. Go to **EC2** → left sidebar → **Network & Security** → **Security Groups**.
2. Click **Create security group**.
3. Fill in:
   - **Name:** e.g., `web-server-sg`
   - **Description:** what the SG is for
   - **VPC:** select the VPC where it will be used
4. **Inbound rules** — add rules like:
   - SSH (port 22) — Source: *My IP* (recommended) or *Anywhere* (only for learning)
   - HTTP (port 80) — Source: Anywhere (0.0.0.0/0)
   - HTTPS (port 443) — Source: Anywhere (0.0.0.0/0)
5. **Outbound rules:** by default, all traffic is allowed out. Leave as is unless you need to restrict.
6. Click **Create security group**.

## Steps (CLI)

```bash
# Create the SG
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Web server SG" \
  --vpc-id vpc-0abc123

# Allow SSH from your IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp --port 22 --cidr <your-ip>/32

# Allow HTTP
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
```

## Notes

- Security groups are **stateful**. NACLs (Network ACLs) are **stateless** — that's the key difference.
- You can reference another security group as the source — useful for app-tier → DB-tier rules.
- Default SG: a new VPC always has a default SG. It allows all traffic between instances using the same SG.
- Limits: 60 inbound + 60 outbound rules per SG by default. Up to 5 SGs per ENI.
