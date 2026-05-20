# Day 5: Create GP3 Volume

**Date:** 2026-03-17

## What

GP3 is the newer generation of **general-purpose SSD** EBS volume. Unlike GP2, you can scale **IOPS** and **throughput independently** of the volume size — you don't have to grow the disk just to get more performance.

## Why

- Cheaper than GP2 for the same performance baseline (~20% less).
- Provision performance separately: up to 16,000 IOPS and 1,000 MB/s throughput.
- Good default for most workloads — boot volumes, app servers, dev/test, small DBs.

## Steps (Console)

1. Go to **EC2** → left sidebar → **Elastic Block Store** → **Volumes**.
2. Click **Create volume**.
3. Set:
   - **Volume type:** `gp3`
   - **Size:** e.g., 30 GiB
   - **IOPS:** 3,000 (free baseline; scale up if needed)
   - **Throughput:** 125 MB/s (free baseline)
   - **Availability Zone:** must match the EC2 instance's AZ
4. Click **Create volume**.
5. (Optional) Select the volume → **Actions** → **Attach volume** → choose the instance.

## Steps (CLI)

```bash
aws ec2 create-volume \
  --volume-type gp3 \
  --size 30 \
  --iops 3000 \
  --throughput 125 \
  --availability-zone ap-south-1a
```

## Notes

- GP3 baseline (free) = **3,000 IOPS + 125 MB/s** at any size. You pay extra only above this.
- EBS volumes are **AZ-locked** — you can only attach to an instance in the same AZ.
- To move data across AZs: snapshot the volume → create new volume from snapshot in the target AZ.
- GP2 vs GP3: GP2 ties IOPS to size (3 IOPS/GiB). GP3 decouples them. Migrate when possible.
