# Day 10: Attach an Elastic IP to an EC2 Instance

**Date:** 2026-05-22

## What

An **Elastic IP (EIP)** is a static, public IPv4 address that you allocate to your AWS account and can attach to (or move between) EC2 instances. Unlike the default auto-assigned public IP — which changes every time an instance is stopped and started — an Elastic IP **stays the same** until you explicitly release it.

## Why

- **Stable public address** — the IP does not change across instance stop/start cycles.
- **DNS-friendly** — point an `A` record at the EIP without updating it every reboot.
- **Fast failover** — remap the same IP to a healthy/replacement instance during an outage, so clients keep reaching the same address.
- **Whitelisting** — give partners or firewalls one fixed IP to allow.

## Steps (Console)

1. Go to **EC2** → **Network & Security** → **Elastic IPs** (region: **us-east-1**).
2. Click **Allocate Elastic IP address** → keep **Amazon's pool of IPv4 addresses** → **Allocate**.
3. Select the new EIP → **Actions** → **Associate Elastic IP address**.
4. Choose:
   - **Resource type:** Instance
   - **Instance:** select your instance (e.g., `xfusion-ec2`)
   - **Private IP address:** select the instance's private IP
5. Click **Associate**.

## Steps (CLI)

```bash
# 1. Allocate a new Elastic IP (note the AllocationId in the output)
aws ec2 allocate-address \
  --domain vpc \
  --region us-east-1

# 2. Associate it with an instance
aws ec2 associate-address \
  --instance-id i-0abc123 \
  --allocation-id eipalloc-0abc123 \
  --region us-east-1

# 3. Verify the association
aws ec2 describe-addresses \
  --allocation-ids eipalloc-0abc123 \
  --region us-east-1
```

To look up the instance ID by Name tag:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1
```

To clean up later (disassociate, then release so you stop being billed):

```bash
# Disassociate from the instance
aws ec2 disassociate-address \
  --association-id eipassoc-0abc123 \
  --region us-east-1

# Release the Elastic IP back to AWS
aws ec2 release-address \
  --allocation-id eipalloc-0abc123 \
  --region us-east-1
```

## Notes

- **Billing gotcha:** An Elastic IP is **free only while it is associated with a running instance**. AWS charges a small hourly rate for EIPs that are *allocated but idle* (not attached, or attached to a stopped instance). Always **release** EIPs you no longer need.
- Associating an EIP **replaces** the instance's existing auto-assigned public IP — the old public IP is released and does not come back.
- Each EIP is **region-scoped** — you cannot move an EIP across regions, only between instances/ENIs in the same region.
- Default soft limit is **5 Elastic IPs per region** per account; request a quota increase if you need more.
- For high availability you can associate an EIP with a **network interface (ENI)** instead of an instance directly, then move the ENI between instances.
- As of 2024 AWS also charges for **all public IPv4 addresses** (including in-use ones) — consider IPv6 or a NAT/ALB pattern for cost-sensitive workloads.
