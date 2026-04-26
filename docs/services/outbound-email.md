# 📧 Outbound Email / SMTP Alerts

_Last updated: 2026-04-26_

---

## Purpose

The primary server (`ubuntu`) can send outbound email through SMTP2Go.

This is intended for lightweight server notifications and alerts, such as:

- backup failures
- cron job output
- service health alerts
- honeypot / Cowrie notifications
- disk space warnings

This is **not** intended to run a public mail server or receive inbound email.

---

## Architecture

```text
Local script / cron / system tool
        ↓
mail command (mailutils)
        ↓
Postfix on ubuntu
        ↓
SMTP2Go relay over authenticated TLS
        ↓
External recipient inbox
```

---

## Components

### Mail Client

Installed package:

```bash
mailutils
```

This provides the `mail` command for simple command-line email sending.

Example:

```bash
echo "Message body" | mail -s "Subject" recipient@example.com
```

---

### Local MTA

Installed/configured service:

```text
Postfix
```

Postfix is used only as a local outbound mail transfer agent.

It accepts mail locally from scripts and relays it through SMTP2Go.

---

## SMTP Relay

SMTP relay provider:

```text
SMTP2Go
```

Relay host:

```text
[mail.smtp2go.com]:587
```

Port `587` uses STARTTLS with authentication.

---

## Important Postfix Configuration

Primary config file:

```text
/etc/postfix/main.cf
```

Relevant settings:

```text
relayhost = [mail.smtp2go.com]:587

smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous

smtp_tls_security_level = encrypt
smtp_tls_note_starttls_offer = yes
smtp_use_tls = yes

sender_canonical_maps = regexp:/etc/postfix/sender_canonical
```

Notes:

- The relay host must use brackets so it exactly matches the SASL password map key.
- Duplicate `relayhost` or `smtp_tls_security_level` entries should be avoided.
- `smtp_tls_security_level = encrypt` forces encrypted SMTP relay.

---

## Credentials

Credential source file:

```text
/etc/postfix/sasl_passwd
```

Expected format:

```text
[mail.smtp2go.com]:587 SMTP2GO_USERNAME:SMTP2GO_PASSWORD
```

Compiled Postfix map:

```text
/etc/postfix/sasl_passwd.db
```

Security permissions:

```text
-rw------- root root /etc/postfix/sasl_passwd
-rw------- root root /etc/postfix/sasl_passwd.db
```

The plaintext credential file should remain readable only by root.

After editing credentials, rebuild the map:

```bash
sudo postmap /etc/postfix/sasl_passwd
sudo systemctl restart postfix
```

---

## Default Sender Rewrite

SMTP2Go rejected unauthenticated-looking local sender domains such as:

```text
ian@ubuntu
```

because `ubuntu` is not a verified sender domain.

A sender canonical map now rewrites outgoing sender addresses to:

```text
ian@ianboen.com
```

Rewrite file:

```text
/etc/postfix/sender_canonical
```

Current rule:

```text
/.*/ ian@ianboen.com
```

This means all outgoing mail from this server is rewritten to use the verified sender address.

This is intentionally broad because this server is expected to send simple alerts rather than user-specific mail.

---

## Test Command

Send a test email:

```bash
echo "Default sender rewrite test from Ubuntu server." | mail -s "SMTP2Go default sender test" recipient@example.com
```

Expected result:

- email arrives successfully
- From address shows as `ian@ianboen.com`

---

## Troubleshooting Notes

### Authentication mismatch

Symptom:

```text
550 relay access denied - please authenticate
```

Cause found:

- `relayhost` did not match the SASL password map host key exactly.

Fix:

```text
relayhost = [mail.smtp2go.com]:587
```

matching:

```text
[mail.smtp2go.com]:587 USERNAME:PASSWORD
```

---

### Sender domain not verified

Symptom:

```text
550 From header sender domain not verified (ubuntu)
```

Cause:

- Postfix was sending mail as `ian@ubuntu`.
- SMTP2Go requires the From domain/email to be verified.

Fix:

- Added sender canonical rewrite to `ian@ianboen.com`.

---

## Operational Role

This email path should be treated as the server's basic notification channel.

Future alerting work should prefer this local pattern:

```bash
mail -s "Alert subject" recipient@example.com
```

rather than embedding SMTP credentials directly into each script.
