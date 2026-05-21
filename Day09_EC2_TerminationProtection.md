# Day 9: Enable Termination Protection for EC2 Instance

**Date:** 2026-05-21

## What

Termination protection prevents an EC2 instance from being accidentally **terminated** via the console, CLI, or API. While stop protection only blocks the *Stop* action, termination protection blocks the irreversible *Terminate* action — once an instance is terminated, the instance itself (and its root EBS volume, by default) is gone.

## Why

- Termination is **irreversible** — no "undo" button after the instance is gone.
- Protects critical production servers from a misclick or a wrong `aws ec2 terminate-instances` call.
- Adds a mandatory two-step process: disable protection first, then terminate.
- Required as a compliance/safety control in most production AWS environments.

## Steps (Console)

1. Go to **EC2** → **Instances** (region: **us-east-1**).
2. Select the instance (e.g., `xfusion-ec2`).
3. Click **Actions** → **Instance settings** → **Change termination protection**.
4. Check the box **Enable**.
5. Click **Save**.

## Steps (CLI)

```bash
# Enable termination protection
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123 \
  --disable-api-termination "{\"Value\": true}" \
  --region us-east-1

# Verify it's enabled
aws ec2 describe-instance-attribute \
  --instance-id i-0abc123 \
  --attribute disableApiTermination \
  --region us-east-1

# Disable later (only when you actually want to terminate)
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123 \
  --disable-api-termination "{\"Value\": false}" \
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

## Notes

- **Termination protection** ≠ **Stop protection** — they are two separate flags. Enable both on critical production servers.
  - *Termination protection* → blocks `TerminateInstances`.
  - *Stop protection* → blocks `StopInstances`.
- Termination protection does **not** prevent termination triggered by:
  - **Auto Scaling** scale-in events
  - **Spot instance** interruptions
  - **OS-level shutdown** with `shutdown-behavior=terminate` (set on the instance itself)
- The setting is per-instance — it does not propagate to instances launched from an AMI of this instance.
- Always confirm the **region** (`us-east-1` here) before running CLI commands — instances are region-scoped.
