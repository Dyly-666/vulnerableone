# Web AppSec Cheatsheet Repo — Maintenance Instructions

This repo is a personal web application security testing cheatsheet, synced to GitBook.
Your job is to take raw source material I give you (pasted text, a writeup, or a URL)
and turn it into a properly placed, well-formatted cheatsheet entry.

## Repo structure
- Flat structure: each topic is a single `.md` file at repo root (e.g. `basic-recon.md`, `sqli.md`, `xss.md`).
- `README.md` = table of contents / landing page. NOT a content file — never put cheatsheet content here.
- `SUMMARY.md` = GitBook navigation index. Every content file MUST be listed here or it won't appear in GitBook.
- `CLAUDE.md` (this file) = instructions only. Never edit this unless I explicitly ask you to change the workflow.

## Workflow for every new piece of content

1. **Identify the topic.** Read the source material and determine which vulnerability
   class / testing category it belongs to (e.g. SQLi, XSS, LFI/RFI, SSRF, IDOR, auth bypass,
   deserialization, recon, subdomain enum, API testing, etc).

2. **Decide: existing file or new file.**
   - List the current `.md` files in repo root first.
   - If a file for that topic already exists, APPEND a new section to it (don't overwrite existing content).
   - If no matching file exists, create a new one: `topic-name.md` (lowercase, hyphen-separated, matching the style of `basic-recon.md`).

3. **Convert the source material into this template** for every technique/entry:

   ```markdown
   ## <Short, descriptive technique name>

   **Category:** <e.g. LFI, RCE, Auth Bypass>
   **Severity context:** <when this matters / typical impact>

   ### How to identify it
   <What to look for during recon/testing — request/response patterns, error messages,
   parameter names, code patterns, tool output signatures that suggest this vuln class exists>

   ### How it works
   <Plain-language explanation of the root cause/mechanism, 2-4 sentences>

   ### Exploitation
   <code block: commands, payloads, scripts as relevant>

   ### Example
   <Real or sanitized example if source material has one — redact actual target
   domains/IPs/credentials from real engagements unless they're already public CTF/HTB targets>

   ### Mitigation
   <How this gets fixed/prevented>

   ### References
   - <links if available>
   ```

   - Every entry must follow Identify → How it works → Exploit → Mitigate, in that order.
     This repo's whole point is mapping "how do I spot this" to "how do I exploit this."
   - Rewrite everything in your own words — do not copy-paste large verbatim blocks from
     source writeups, even if I paste them to you. Summarize and restructure.
   - Keep code/commands verbatim (that's fine — it's functional content, not prose).
   - If the source material is messy (like a blog writeup with mixed narrative and commands,
     or raw unstructured notes), extract just the technique and reformat it cleanly —
     drop narrative fluff, fix garbled/typo'd lines, deduplicate repeated tips.

4. **Update `SUMMARY.md`.**
   - If you created a new file, add a new entry in the correct logical position (group by
     phase: recon → injection → auth/session → file handling → SSRF/business logic → misc).
   - If you only appended to an existing file, no SUMMARY.md change needed (file already listed).
   - Keep SUMMARY.md's existing structure/grouping style — inspect it first, match its conventions.

5. **Update `README.md`'s table of contents** if a new file was added — one-line description
   per topic file, same grouping as SUMMARY.md.

6. **Show me a diff-style summary** of what changed (which file(s) touched, what section(s) added)
   before I need to ask — just a short confirmation, not a full file dump.

## Source material handling
- I'll sometimes paste raw text/writeups directly in chat.
- I'll sometimes give you a URL — fetch it, extract the technique, ignore site chrome/ads/unrelated content.
- Either way, apply the same template and placement workflow above.

## Things to never do
- Never invent commands/payloads that weren't in the source material or aren't accurate —
  if something is unclear, flag it for me instead of guessing.
- Never commit/push automatically — stage changes and let me review before I commit myself,
  unless I explicitly say "commit and push."
- Never restructure the whole repo (e.g. converting flat structure to folders) without me asking first.
- Never put maintenance/instruction content into README.md, SUMMARY.md, or any topic file.
