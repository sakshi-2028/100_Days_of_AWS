# Day 1: Create Key Pair

**Date:** 2026-01-12

## What

A key pair is a set of cryptographic keys (public + private) used by AWS to authenticate SSH access to EC2 instances. AWS stores the public key on the instance; you keep the private key file (`.pem` or `.ppk`).

## Why

- Required to log in to a Linux EC2 instance over SSH.
- Without the private key, you cannot connect to the instance.
- One key pair can be reused across many instances.

## Steps (Console)

1. Go to **EC2** → left sidebar → **Network & Security** → **Key Pairs**.
2. Click **Create key pair**.
3. Enter a **Name** (e.g., `my-first-key`).
4. **Key pair type:** RSA (default) or ED25519.
5. **Private key file format:**
   - `.pem` → for OpenSSH / Mac / Linux.
   - `.ppk` → for PuTTY on Windows.
6. Click **Create key pair**. The private key file downloads automatically — **save it safely**, AWS will not show it again.

## Steps (CLI)

```bash
aws ec2 create-key-pair \
  --key-name my-first-key \
  --query 'KeyMaterial' \
  --output text > my-first-key.pem

chmod 400 my-first-key.pem
```

## Notes

- Set file permissions to `400` on Mac/Linux — SSH will refuse the key otherwise.
- If you lose the `.pem` file, you cannot recover it. You will have to detach the volume or use SSM to regain access.
- Key pairs are region-specific. A key created in `ap-south-1` won't appear in `us-east-1`.
