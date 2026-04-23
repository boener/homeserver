# 📧 Email Service

_Last updated: 2026-04-22_

---

## 🧠 Overview

Custom-domain email for `ianboen.com` now works in a split model:

- **Inbound mail** is forwarded by **Cloudflare Email Routing**
- **Outbound mail** is sent through **SMTP2GO**
- **Gmail** is used as the everyday client interface

This gives normal send/receive behavior for `ian@ianboen.com` without running a mail server on the home network.

---

## ✅ Current Capability

`ian@ianboen.com` can now:

- receive mail through Gmail via forwarding
- send mail from Gmail as the custom domain address

From the user's point of view, the mailbox behaves like a normal address inside the regular Gmail account.

---

## 🧱 Delivery Model

### Inbound
```text
Internet sender → Cloudflare Email Routing → personal Gmail inbox
```

### Outbound
```text
Gmail → SMTP2GO → recipient
```

---

## 🔧 Why This Setup Exists

This approach avoids the ugly parts of self-hosted email:

- no local SMTP server to secure or maintain
- no exposed mail ports on the home network
- no need to manage full MTA reputation / spam posture from home IP space
- keeps the workflow entirely inside Gmail

This is the correct complexity level for a low-volume personal domain mailbox.

---

## 🌐 DNS Dependencies

Cloudflare DNS now contains provider-specific records required for SMTP2GO setup.

Documented change:
- added SMTP2GO-related **CNAME** records to support provider verification and/or mail authentication

Exact record values are intentionally **not duplicated here** unless there is a strong reason to preserve them in docs, because they are provider-managed details visible in Cloudflare.

---

## 📬 Gmail Configuration

Gmail is configured with a custom “Send mail as” identity for:

- `ian@ianboen.com`

Using:
- SMTP provider: **SMTP2GO**
- Authentication: SMTP2GO credentials

---

## 📈 Usage Expectations

- SMTP2GO free tier: **1000 emails/month**
- Expected usage: extremely low personal volume

This should be more than sufficient under current use, but free-tier limits should never be treated as permanent guarantees.

---

## ⚠️ Operational Notes

### This is not self-hosted email
The home server is **not** currently responsible for mail transport.

It participates only indirectly through:
- domain ownership
- DNS control
- the overall home-lab documentation model

### Future troubleshooting areas
If outbound delivery becomes unreliable, check:

- SPF
- DKIM
- DMARC
- Gmail “Send mail as” config
- SMTP2GO account status / quota

### Risk model
Primary risks here are:
- provider free-tier changes
- DNS record drift
- mail landing in spam due to authentication or reputation problems

---

## 🔐 Security Notes

- SMTP credentials live in Gmail configuration / account settings
- credentials should **not** be stored in the repo
- this setup is appropriate for light personal use, not sensitive transactional mail

---

## 📚 Related Docs

- `docs/current-state.md`
- `docs/architecture.md`
- `docs/services/cloudflare-ddns.md`
