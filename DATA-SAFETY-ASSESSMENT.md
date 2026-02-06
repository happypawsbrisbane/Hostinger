# Data Safety Assessment — ASH & HOUND Website

**Date:** 2026-02-06
**Scope:** Full codebase audit of the static website hosted on Vercel

---

## Overview

This is a **static HTML website** (no backend/database) for a pet care business. Data handling is limited to:

- Two client-facing forms (enquiry + onboarding) submitted to **Formspree** (third-party)
- No server-side code, no database, no user accounts
- No analytics or tracking scripts detected

**Overall risk level: Low-to-Medium** — the site collects sensitive client data through forms but has good baseline security headers.

---

## Positive Findings

| Area | Status |
|------|--------|
| Security headers (X-Frame-Options, X-Content-Type-Options, X-XSS-Protection) | Configured in vercel.json |
| Permissions-Policy (camera, microphone, geolocation disabled) | Configured |
| Referrer-Policy | strict-origin-when-cross-origin |
| External links use `rel="noopener noreferrer"` | Yes |
| All URLs use HTTPS | Yes |
| No analytics/tracking code | Confirmed |
| Error messages are generic (no stack traces) | Confirmed |
| No sensitive data in console logs | Confirmed |
| HTML caching disabled (must-revalidate) | Configured |
| No user accounts or stored credentials | Confirmed |

---

## Issues Found

### HIGH — Sensitive Data Sent to Third-Party Without Encryption

**Files:** `enquiry.html:172`, `client-onboarding.html:998`

The client onboarding form collects highly sensitive data and sends it directly to Formspree:

- Client names, phone numbers, emails, addresses
- Emergency contact information
- Pet medical conditions and medications
- Home security system details and access codes

While HTTPS protects data in transit, all information is stored unencrypted on Formspree's servers. There is no privacy disclosure informing clients that their data is processed by a third party.

**Recommendation:** Add a privacy notice about Formspree data handling. Consider moving to a self-hosted form backend for sensitive fields. Evaluate client-side encryption before transmission.

---

### MEDIUM — innerHTML Usage

**File:** `client-onboarding.html:662`

Dynamic pet sections are created using `innerHTML` with template literals. Although current inputs are controlled, this pattern is vulnerable to XSS if input sources change.

**Recommendation:** Replace with `createElement()` and `textContent`.

---

### MEDIUM — Deprecated Clipboard API

**File:** `client-onboarding.html:973`

Uses `document.execCommand('copy')` which is deprecated and less reliable.

**Recommendation:** Replace with `navigator.clipboard.writeText()`.

---

### LOW — Missing Content Security Policy (CSP)

No CSP header is configured. Adding one would provide an additional layer of XSS protection.

**Recommendation:** Add a CSP header in `vercel.json`.

---

### LOW — Missing HSTS Header

HTTP Strict-Transport-Security is not configured, meaning browsers are not forced to use HTTPS on repeat visits.

**Recommendation:** Add `Strict-Transport-Security: max-age=63072000; includeSubDomains` to `vercel.json` headers.

---

### LOW — No Subresource Integrity (SRI)

Google Fonts are loaded from external CDN without integrity hashes.

**Recommendation:** Add SRI attributes to external resource links.

---

## Summary

| Severity | Count | Key Concern |
|----------|-------|-------------|
| High | 1 | Sensitive client data sent to third-party unencrypted |
| Medium | 2 | innerHTML usage, deprecated API |
| Low | 3 | Missing CSP, HSTS, SRI |

The site has a solid security baseline with proper headers, HTTPS, and no tracking. The primary concern is the handling of sensitive client data (medical info, home security details) through a third-party form service without client-side encryption or privacy disclosure.

---

## Recommended Priority Actions

1. Add a privacy notice on the onboarding form disclosing Formspree as a data processor
2. Add Content-Security-Policy and Strict-Transport-Security headers to `vercel.json`
3. Replace `innerHTML` with safe DOM methods in `client-onboarding.html`
4. Replace deprecated `execCommand('copy')` with modern Clipboard API
5. Evaluate migrating form handling to a self-hosted solution for sensitive data
