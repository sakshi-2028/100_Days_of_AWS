# Day 4: Enable Versioning for S3 Bucket

**Date:** 2026-02-16

## What

S3 versioning keeps **multiple versions of the same object** in a bucket. Every PUT to an existing key creates a new version instead of overwriting. A delete creates a special "delete marker" — the older versions are still there.

## Why

- Protects against accidental deletes and overwrites.
- Lets you roll back to a previous version of a file.
- Required for features like **MFA Delete** and **Cross-Region Replication**.

## Steps (Console)

1. Go to **S3** → click your bucket.
2. Open the **Properties** tab.
3. Find **Bucket Versioning** → click **Edit**.
4. Select **Enable** → click **Save changes**.

## Steps (CLI)

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Enabled

# Check current status
aws s3api get-bucket-versioning --bucket my-bucket-name
```

## Notes

- Once **Enabled**, versioning cannot be turned back off — only **Suspended**. Suspending stops new versions but keeps existing ones.
- Older versions still **cost storage**. Use a lifecycle rule to expire non-current versions if you don't need full history.
- A delete marker is just a new version — to actually recover, delete the marker (or restore the previous version).
- Versioning is at the **bucket level**, not per-object.
