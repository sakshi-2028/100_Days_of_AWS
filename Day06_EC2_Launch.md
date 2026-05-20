# Day 6: Launch EC2 Instance

**Date:** 2026-04-16

## What

EC2 (Elastic Compute Cloud) gives you virtual servers in AWS. "Launching" an instance means starting one VM with a chosen OS image, size, network, storage, and security configuration.

## Why

- To host applications, websites, databases, or any compute workload.
- Pay-per-use (per second for most Linux AMIs).
- Quickly scalable — launch, stop, or terminate as needed.

## Steps (Console)

1. Go to **EC2** → click **Launch instances**.
2. **Name:** e.g., `my-first-server`.
3. **AMI (image):** pick OS (e.g., Amazon Linux 2023, Ubuntu 22.04). Free tier eligible AMIs are marked.
4. **Instance type:** e.g., `t2.micro` or `t3.micro` (free tier).
5. **Key pair:** select the one created on Day 1 (or create new).
6. **Network settings:**
   - VPC + Subnet (pick the public subnet from Day 3 if you want internet access)
   - Auto-assign public IP: **Enable**
   - Security group: select the one from Day 2 (or create new with SSH/HTTP allowed)
7. **Storage:** default 8 GiB GP3 is fine, or attach the GP3 volume from Day 5.
8. Click **Launch instance**.
9. Wait until **Instance state** = `running` and **Status checks** = `2/2 passed`.

## Connect (SSH)

```bash
chmod 400 my-first-key.pem
ssh -i my-first-key.pem ec2-user@<public-ip>   # Amazon Linux
ssh -i my-first-key.pem ubuntu@<public-ip>     # Ubuntu
```

## Steps (CLI)

```bash
aws ec2 run-instances \
  --image-id ami-0abc123 \
  --instance-type t3.micro \
  --key-name my-first-key \
  --security-group-ids sg-0abc123 \
  --subnet-id subnet-0abc123 \
  --associate-public-ip-address \
  --count 1
```

## Notes

- Free tier: 750 hours/month of `t2.micro` or `t3.micro` for the first 12 months.
- Stopped instance = no compute charge, but EBS volumes still cost money.
- Terminated instance = gone forever (root volume deleted by default).
- If SSH fails: check SG inbound port 22, key permissions (400), and the right username for the AMI.
