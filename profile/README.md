# Sigil Auth

PKI-based strong authentication. Hardware-backed ECDSA keypairs. No passwords, no shared secrets, no database.

Sigil Auth uses challenge-response signing between devices and servers. The private key lives in Secure Enclave (iOS), StrongBox (Android), or a hardware key (YubiKey). It never leaves — no export, no backup, no transmission.

## How it works

1. Device generates a P-256 keypair in secure hardware. The SHA-256 fingerprint of the public key becomes the device's universal identifier.
2. Your app stores user→fingerprint mappings. Sigil never holds user data.
3. To authenticate: push a challenge to the device, device signs it, you verify the signature against the registered fingerprint.
4. For critical actions: request M-of-N group approval. Groups are people (one person, multiple devices — any device in their group can approve).

```
Your App → Sigil Service → Push Relay → Device
                ↑                          ↓
                └──── signed response ─────┘
```

The service is stateless — identity derived from a BIP39 mnemonic, no database required. The push relay (APNs/FCM) maps fingerprints to push tokens. Everything else lives in your system.

## Quick start

```bash
docker run -d -p 8400:8400 sigilauth/server
```

```
POST /v1/auth/challenge    # push challenge to a device fingerprint
POST /v1/auth/respond      # device returns public key + signature
POST /v1/mpa/request       # request M-of-N group approval
GET  /v1/mpa/status/:id    # check approval progress
```

Docs: [sigilauth.com/docs](https://sigilauth.com/docs)

## What's here

| Repo | What it is |
|------|------------|
| [app](https://github.com/sigilauth/app) | iOS (Swift) + Android (Kotlin) native apps |
| [desktop](https://github.com/sigilauth/desktop) | macOS, Windows, Linux native apps |
| [server](https://github.com/sigilauth/server) | Auth service (Docker) + embeddable SDKs |
| [relay](https://github.com/sigilauth/relay) | Push notification relay (APNs/FCM) |
| [web](https://github.com/sigilauth/web) | Website and docs |

Licensed AGPL-3.0. Embeddable client libraries are MIT.

## Design choices

**Stateless service.** Mnemonic-derived identity means you can run multiple instances with zero coordination. No database to secure, replicate, or back up.

**Visual verification.** Devices show a 5-emoji pictogram derived from their fingerprint. Users can verify a device identity at a glance without comparing hex strings.

**Standard crypto.** P-256, AES-256-GCM, HKDF. No proprietary formats. Any compatible implementation works.

---

Built by [Wagmi Labs](https://wagmi.bio). No commercial plans — we built this because we needed it.
