# GPG Subkey Rotation Runbook

This runbook describes the **authoritative procedure** for rotating OpenPGP subkeys (signing + encryption) while preserving a long-lived offline primary key.

This document is designed to be stored **with key backups** and followed during calm maintenance windows *or* under time pressure.

---

## Purpose

- Keep the **primary key** long-lived and offline
- Use **short-lived, replaceable subkeys** for daily signing and encryption
- Ensure laptop rebuilds are fully automated via chezmoi
- Make expiration a routine event, not an emergency

---

## When to Run This

- **Proactively:** ~30 days before subkey expiration
- **Reactively:** immediately after expiration (safe; only causes temporary signing/encryption failures)

Expiration is not a failure state. It is a reminder.

---

## Preconditions (Do Not Skip)

Before starting, confirm:

- You have access to the **primary secret key backup**
- You have the **revocation certificate** stored safely
- Daily machines currently run in **subkeys-only mode** (`sec#` present, `ssb` present)

---

## Phase A — Key Surgery (Primary Key Environment)

Perform this phase in a **controlled environment**:

- Offline laptop, *or*
- Temporary isolated `GNUPGHOME`, *or*
- Hardware-backed environment

### 1. Load the Primary Key

```bash
export GNUPGHOME="$(mktemp -d)"
chmod 700 "$GNUPGHOME"

gpg --import primary-secret-key.asc
```

Verify:

```bash
gpg -K
```

You should see the primary secret key and existing subkeys.

---

### 2. Add New Subkeys

```bash
gpg --edit-key <PRIMARY_FPR>
```

Inside the `gpg>` prompt:

#### Add a Signing Subkey

```
addkey
```

- Type: **ECC (sign only)**
- Curve: **Ed25519**
- Expiration: **1y** (or your policy)

#### Add an Encryption Subkey

```
addkey
```

- Type: **ECC (encrypt only)**
- Curve: **cv25519**
- Expiration: **1y**

Do **not** delete old subkeys yet.

```
save
```

---

### 3. (Optional but Recommended) Revoke Old Subkeys

Only do this **after** new subkeys exist.

```bash
gpg --edit-key <PRIMARY_FPR>
```

For each expired or superseded subkey:

```
key N
revkey
key N
```

- Reason: *Key superseded*

Then:

```
save
```

This preserves history while preventing ambiguity.

---

## Phase B — Publish Updated Public Key

Export the updated public key:

```bash
gpg --armor --export <PRIMARY_FPR> > publickey.asc
```

Update **everywhere that matters**:

- **GitHub GPG keys**
- Any profile, website, or documentation where you publish a fingerprint

You do **not** need to publish to public keyservers unless you are participating in distro-level workflows.

---

## Phase C — Regenerate Backups (Critical)

### 1. New Primary Secret Snapshot

```bash
gpg --armor --export-secret-key <PRIMARY_FPR> > primary-secret-key.asc
```

- Store encrypted in 1Password (or equivalent)
- ~~Archive the previous snapshot with a clear date label~~

---

### 2. New Subkeys-Only Backup

```bash
gpg --armor --export-secret-subkeys <PRIMARY_FPR> > subkeys-secret.asc
```

- Store as the canonical `gpg-privatekey-subkeys` document
- This is what daily machines restore automatically

---

### 3. Revocation Certificate

**Do nothing.**

Existing revocation certificates remain valid across subkey rotations.

---

## Phase D — Roll Forward Daily Machines

On each laptop or daily machine:

```bash
chezmoi apply
```

Expected behavior:

- New subkeys imported
- Old subkeys replaced or superseded
- Primary secret key remains absent
- Automation enforces invariants

Verify:

```bash
gpg -K

git commit --allow-empty -m "subkey rotation test"
git log -1 --show-signature
```

The signature **must** use the new signing subkey.

---

## Phase E — Cleanup (Optional)

After all machines are updated and verification succeeds:

- Old revoked subkeys may remain (safe)
- Or be deleted from the primary during a later maintenance window

No urgency.

---

## Failure Modes & Recovery

- **Forgot to rotate before expiration:** rotate anyway; service resumes immediately
- **Primary key lost:** revoke entire key and migrate identity
- **ChezMoi fails:** expected — fix invariant, do not bypass automation

---

## Invariant (The Rule That Matters)

> The primary key is long-lived and offline.  
> Subkeys are short-lived, replaceable, and automated.

If this invariant holds, everything else is recoverable.
