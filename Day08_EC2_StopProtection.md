# Day 8: Enable Stop Protection for EC2 Instance

**Date:** 2026-05-20

## What

Stop protection prevents an EC2 instance from being accidentally **stopped** via the console, CLI, or API. The user has to explicitly disable the protection before they can stop the instance.

## Why

- Avoid downtime caused by an accidental "Stop" click.
- Helpful for production servers where stopping breaks something (e.g., loses ephemeral data on instance store volumes).
- Adds a second confirmation step before a destructive action.

## Steps (Console)

1. Go to **EC2** → **Instances** → select the instance.
2. Click **Actions** → **Instance settings** → **Change stop protection**.
3. Check the box **Enable**.
4. Click **Save**.

## Steps (CLI)

```bash
# Enable stop protection
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123 \
  --disable-api-stop "{\"Value\": true}"

# Disable later (before you actually want to stop)
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123 \
  --disable-api-stop "{\"Value\": false}"
```

## Notes

- **Stop protection** ≠ **Termination protection** — they are two separate flags.
  - *Stop protection* → prevents `StopInstances`.
  - *Termination protection* → prevents `TerminateInstances`.
  - Enable both on critical production servers.
- Stop protection does **not** prevent the OS itself from shutting down (e.g., `sudo shutdown -h now` from inside the instance still works).
- Auto Scaling actions and Spot interruptions can still stop the instance — stop protection only blocks API stop calls.
- For instance-store-backed instances, the stop action is not supported at all — stop protection doesn't apply.
