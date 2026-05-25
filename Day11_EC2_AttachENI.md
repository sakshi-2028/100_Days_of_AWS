# Day 11: Attach an Elastic Network Interface (ENI) to an EC2 Instance

**Date:** 2026-05-25

## What

An **Elastic Network Interface (ENI)** is a virtual network card that lives inside a VPC subnet. It carries network attributes such as a primary private IPv4 address, optional secondary private IPs, one or more security groups, a MAC address, and (optionally) a public/Elastic IP. An EC2 instance always has a **primary ENI** (device index `0`), and you can **attach additional (secondary) ENIs** to give the instance more network interfaces.

In this task we attach an existing interface (`devops-eni`) to an existing instance (`devops-ec2`) in **us-east-1**, and confirm the attachment status is **attached**.

## Why

- **Multi-homing** — put one instance on multiple subnets (e.g., a public-facing interface + a private management interface).
- **Fast failover / HA** — an ENI keeps its private IP, MAC, and security groups when moved, so you can detach it from a failed instance and attach it to a standby — clients keep reaching the same IP.
- **Separation of traffic** — isolate management, application, or monitoring traffic on dedicated interfaces.
- **Licensing/appliances** — some software is licensed to a MAC address; an ENI's MAC stays constant across instances.

## Steps (Console)

1. Go to **EC2** → **Network & Security** → **Network Interfaces** (region: **us-east-1**).
2. Select the interface **`devops-eni`**.
3. Click **Actions** → **Attach**.
4. In the dialog, choose the **Instance:** `devops-ec2`.
5. Click **Attach**.
6. Refresh and confirm the interface **Status** shows **`in-use`** and the **Attachment status** is **`attached`**.
7. Make sure the instance has finished initialising (Instance state **running**, status checks **2/2 passed**) before submitting.

## Steps (CLI)

```bash
# 1. Find the ENI ID by its Name tag
aws ec2 describe-network-interfaces \
  --filters "Name=tag:Name,Values=devops-eni" \
  --query "NetworkInterfaces[].NetworkInterfaceId" \
  --output text \
  --region us-east-1

# 2. Find the instance ID by its Name tag
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1

# 3. Attach the ENI (device-index 1 = secondary; index 0 is the primary ENI)
aws ec2 attach-network-interface \
  --network-interface-id eni-0abc123 \
  --instance-id i-0abc123 \
  --device-index 1 \
  --region us-east-1

# 4. Verify — Status should be "in-use" and Attachment.Status "attached"
aws ec2 describe-network-interfaces \
  --network-interface-ids eni-0abc123 \
  --query "NetworkInterfaces[].{Status:Status,Attachment:Attachment.Status,Device:Attachment.DeviceIndex}" \
  --output table \
  --region us-east-1
```

To detach later (cleanup), you need the **attachment ID**:

```bash
# Get the attachment ID
aws ec2 describe-network-interfaces \
  --network-interface-ids eni-0abc123 \
  --query "NetworkInterfaces[].Attachment.AttachmentId" \
  --output text \
  --region us-east-1

# Detach using that attachment ID
aws ec2 detach-network-interface \
  --attachment-id eni-attach-0abc123 \
  --region us-east-1
```

## Notes

- **Device index 0 is the primary ENI** — it's created with the instance and **cannot be detached**. Secondary ENIs use device index `1`, `2`, …
- **Same Availability Zone required** — an ENI is subnet-scoped, so it can only attach to an instance in the **same AZ** as the ENI's subnet.
- **Instance type limits** the number of ENIs and IPs per ENI (e.g., a `t2.micro` supports 2 ENIs, a `t3.large` more). Check the limit before attaching many interfaces.
- **OS-level config may be needed** — a secondary ENI shows up in AWS as `attached`, but the guest OS may not auto-configure it; you sometimes need to bring up the second interface manually (routing/`ip` config).
- **Lifecycle** — by default a **secondary** ENI is *not* deleted when the instance terminates (its `DeleteOnTermination` is `false`), unlike the primary ENI. So the interface survives and can be reused.
- Always confirm the **region** (`us-east-1` here) and **AZ** before running CLI commands — interfaces and instances are region/AZ-scoped.
