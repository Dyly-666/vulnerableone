---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/web-application
---

# 🕸️ Web Application

A personal reference for web application security testing — organized by phase, from **identifying** a vulnerability to **exploiting** it.

Each topic page follows the same structure: how to spot it, how it works, how to exploit it, how to fix it.

## Recon & Enumeration

* [Basic Recon](basic-recon.md) — assessment methodology, fingerprinting, directory brute-forcing, common tools (Gobuster, Nikto, Nmap scripts, WPScan)

## Injection

* [SQL Injection](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/sqli.md) — UNION-based, error-based, blind boolean-based, time-based
* [Command Injection](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/command-injection.md) — basic blind, out-of-band exfiltration

## Authentication & Session

* [Authentication Bypass](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/auth-bypass.md) — logic flaws, password reset, 2FA bypass, session manipulation
* [JWT Attacks](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/jwt-attacks.md) — alg=none, RS→HS confusion, weak secret, JWK/JKU injection

## File Handling

* [LFI / RFI](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/lfi-rfi.md) — local/remote file inclusion, PHP wrappers, log poisoning
* [File Upload](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/file-upload.md) — unrestricted upload, extension bypass techniques

## Server-Side Request Issues

* [SSRF](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/ssrf.md) — internal port scan, cloud metadata, blind SSRF

## Business Logic & Misc

* [IDOR](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/idor.md) — horizontal and vertical IDOR, mass assignment
* [Race Conditions](https://github.com/Dyly-666/vulnerableone/blob/main/offensive-treasure/web-application/race-conditions.md) — TOCTOU, concurrent request race, symlink race
