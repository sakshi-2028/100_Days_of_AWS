# Day 7: Change EC2 Instance Type

**Date:** 2026-05-15

## What

Changing the instance type (a.k.a. "resizing") lets you move an existing EC2 instance to a different size — more or less CPU, RAM, and network performance — without recreating it.

## Why

- App needs more resources (scale up).
- Cost optimization — move from oversized to right-sized.
- Switching instance family (e.g., `t3` → `m5`) for better performance.

## Steps (Console)

1. Go to **EC2** → **Instances** → select the instance.
2. Click **Instance state** → **Stop instance**. Wait until state = `stopped`.
3. With the instance still selected → **Actions** → **Instance settings** → **Change instance type**.
4. Pick the new instance type (e.g., `t3.small`).
5. Click **Apply**.
6. **Instance state** → **Start instance**.

## Steps (CLI)

```bash
# Stop
aws ec2 stop-instances --instance-ids i-0abc123

# Wait until stopped, then change type
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123 \
  --instance-type "{\"Value\": \"t3.small\"}"

# Start again
aws ec2 start-instances --instance-ids i-0abc123
```

## Notes

- The instance **must be stopped** before changing type. Cannot be done while running.
- **Public IP changes** after stop/start unless an Elastic IP is attached.
- Architecture must match: cannot move from x86 (`t3`) to ARM/Graviton (`t4g`) directly — different AMI needed.
- Root volume and data are preserved across resize.
- Some instance types are not compatible with older HVM/PV AMIs — check before switching.
