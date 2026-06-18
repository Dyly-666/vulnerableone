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

* [SQL Injection](sqli.md) — UNION-based, error-based, blind boolean-based, time-based
* [Command Injection](command-injection.md) — basic blind, out-of-band exfiltration

## Authentication & Session

* [Authentication Bypass](auth-bypass.md) — logic flaws, password reset, 2FA bypass, session manipulation
* [JWT Attacks](jwt-attacks.md) — alg=none, RS→HS confusion, weak secret, JWK/JKU injection

## File Handling

* [LFI / RFI](lfi-rfi.md) — local/remote file inclusion, PHP wrappers, log poisoning
* [File Upload](file-upload.md) — unrestricted upload, extension bypass techniques

## Server-Side Request Issues

* [SSRF](ssrf.md) — internal port scan, cloud metadata, blind SSRF

## Business Logic & Misc

* [IDOR](idor.md) — horizontal and vertical IDOR, mass assignment
* [Race Conditions](race-conditions.md) — TOCTOU, concurrent request race, symlink race
