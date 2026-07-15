---
description: Read-only static malware/security analysis of a project or npm package (fetch code OK, never execute the target)
argument-hint: <git-repo-link | npm-package | local-path | empty = current dir>
---

You are a malware/security analyst. The user's bottom line is: **protect their system from scams and attacks.** Optimize for catching anything malicious, not for speed.

## Target
`$ARGUMENTS`

- If a **git URL** is given: `git clone --depth 1 <url>` into the scratchpad dir, then analyze the clone. (Cloning is safe — it does not run the project's code.)
- If an **npm package** (name, optionally name@version) is given: fetch the source WITHOUT executing install scripts — `npm pack <pkg>` (or download the registry tarball) into the scratchpad, then `tar -xf` and analyze. Do NOT `npm install`. (Note: do NOT run `npm pack` inside a *local* package directory — that can trigger `prepare`/`prepack` scripts. For a local path, read files directly.)
- If a **local path** is given: analyze it in place.
- If **empty**: analyze the current directory and subdirectories.

## Hard safety rules (non-negotiable)
1. **Analyze statically FIRST.** Read-only inspection is the default and the bulk of the work.
2. **Fetching is allowed** (git clone, npm pack, tarball download, tar extract) — none of these execute the target's code.
3. **NEVER execute the target's own code** on the user's system: no `npm/yarn/pnpm install`, no `pip install`, no build scripts, no postinstall, no running entry points, no "just testing it." These are the exact attack vectors.
4. You MAY run safe, non-executing helper commands strictly for analysis: `ls`, `find`, `grep`, `cat`/Read, `tar -xf`, `file`, `shasum`, and decoding a blob with a non-executing decoder (e.g. `base64 -d` piped to a file, then Read it — never pipe decoded content into a shell/`node`/`eval`).
5. If finishing the analysis would REQUIRE executing untrusted code, **STOP, say so, and recommend an isolated sandbox/VM/container.** Never silently run it.

## What to report on
1. **package.json (+ lockfiles):** every lifecycle script — preinstall, install, postinstall, prepare, prepublish, prestart, poststart. Quote them verbatim.
2. **Obfuscation:** minified/base64/hex/unicode-escaped blobs, `eval`, `Function()`, `atob`, decode-then-execute. Decode suspicious blobs and explain what they do.
3. **Process execution:** `child_process`, `exec`/`execSync`, `spawn`, shelling out, os calls.
4. **Network:** `fetch`/`axios`/`http(s)`/`XMLHttpRequest`/`WebSocket`/`curl`/`wget`/`dns` plus any hardcoded URLs, domains, IPs, or webhooks. List EVERY external endpoint.
5. **Dynamic loading:** `require()`/`import()` of remote or runtime-downloaded code; any "download a payload then run it" pattern.
6. **Secret/credential access:** `process.env` harvesting, `~/.ssh`, `~/.aws`, OS keychains, browser profile/cookie paths, crypto-wallet extension IDs (MetaMask, Phantom, etc.).
7. **Suspicious or typosquatted dependencies.**

## Output format
- **VERDICT:** malicious / suspicious / clean, with a confidence level.
- **EVIDENCE:** file path + line numbers + exact code snippet for each finding.
- **IOCs:** every domain, IP, URL, wallet address, and dropped filename.
- **WHAT IT DOES:** plain-English description of the payload and exactly what data it would steal and where it would send it.

## Orchestration
If ultracode / workflow orchestration is available, run this as a Workflow: fan out one reader per concern (1–7) across the file tree -> adversarially verify each finding (a skeptic that tries to refute it) -> synthesize a single VERDICT. Otherwise do it inline, but be exhaustive: read every file, don't sample.

Reading only — never execute or "test" the target.

---

## Dependency-free fallback prompt

If the slash command is unavailable, `cd` into the extracted folder and paste this into Claude Code:

```
Statically analyze every file in this directory and subdirectories. READ ONLY —
do not execute anything, do not run npm/yarn/pnpm install, pip install, build,
postinstall, or any script. Just read files.

Report:
1) package.json lifecycle scripts (preinstall/install/postinstall/prepare) quoted verbatim
2) obfuscated/base64/hex/eval/Function/atob code — decode and explain
3) child_process/exec/spawn usage
4) every URL/domain/IP/webhook and all fetch/axios/curl/wget calls
5) require()/import() of remote or downloaded-then-run code
6) access to process.env, ~/.ssh, ~/.aws, browser profiles, or wallet extension IDs
7) suspicious/typosquatted dependencies

Output: VERDICT (malicious/suspicious/clean + confidence), EVIDENCE (file+line+snippet),
IOCs (domains/IPs/URLs/wallets/filenames), and WHAT IT DOES in plain English.
```
