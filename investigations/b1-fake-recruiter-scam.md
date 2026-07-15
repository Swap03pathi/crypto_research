# Investigation: Fake‑Recruiter Malware Scam Impersonating "B1 / Block.one"

> **Status:** Active investigation. Suspected malware sample (`Fullstack.zip`) downloaded to an
> isolated sandbox and fingerprinted; static code analysis pending. **The sample has NOT been
> executed on any personal or production system.**
>
> **Last updated:** 2026‑07‑15
>
> **Nature of document:** Defensive security investigation + runbook. This records who/what we
> researched, the indicators of compromise (IOCs), the safe analysis environment we built (a
> disposable AWS EC2 instance), and the analysis tooling (a read‑only Claude Code command). It also
> contains ready‑to‑send abuse/fraud report templates.

---

## 1. Executive summary

A job seeker was approached about a role at **B1 / Block.one** (a real blockchain company, `b1.com`).
The outreach came through **LinkedIn** and was scheduled via a **personal Calendly link**, with emails
whose reply‑to was **`contact@danielsecurity.pro`** — a domain **registered ~1 month before contact**,
**parked**, and **unrelated to Block.one**. After a pattern of deliberate no‑shows and reschedules, the
"recruiter" pivoted to a **take‑home coding assignment** delivered as **`Fullstack.zip`** via a **Google
Drive link owned by a personal Gmail** (`cool9571791@gmail.com`), **not** an `@block.one` address.

This matches the well‑documented **"Contagious Interview"** campaign (attributed to DPRK/Lazarus),
which delivers **BeaverTail**/**InvisibleFerret** malware inside fake "technical assessment" projects to
steal credentials, cloud tokens, browser sessions, and cryptocurrency wallets.

**Block.one's own published notice** confirms the modus operandi: they state that scammers
**impersonate their staff (explicitly "Brendan and Dan")**, that **no Block.one personnel recruit or
propose business via LinkedIn/Telegram/etc.**, and that people should **never download/compile/install/run
software from untrusted repositories**.

**Verdict:** High‑confidence **fake‑recruiter / malware‑delivery scam** impersonating B1/Block.one.

---

## 2. TL;DR verdict & recommended actions

- **Do NOT run the assignment** (`npm install`, build, execute, or open in an auto‑running IDE) on any
  real machine. Analyze read‑only in a disposable VM only.
- **Do NOT** share private keys, seed phrases, passwords, 2FA codes, KYC/ID documents, or pay any fee.
- **Treat all "B1" contact via LinkedIn / `danielsecurity.pro` / personal Gmail as fraudulent.** Genuine
  B1 uses `@block.one` and `b1.com` and does not recruit over LinkedIn.
- **Report** to LinkedIn, Google (Drive), Calendly, Action Fraud/IC3, and the domain registrar; send a
  courtesy heads‑up to the impersonated real person and to B1. (Templates in §12.)

---

## 3. Background — the legitimate entity being impersonated

### 3.1 B1 / Block.one
- **Website:** `b1.com` (formerly used `block.one`). The `b1.com` domain was first registered **1998‑06‑15**
  (via **MarkMonitor**) and was acquired later — the company itself dates to **2016–2017**.
- **Legal/structure:** Cayman Islands‑registered (administrator **Maples Corporate Services Limited**);
  formerly named **"Banner."**
- **Founders / leaders:** **Brendan Blumer** (CEO), **Daniel "Dan" Larimer** (CTO, departed end‑2020),
  **Brock Pierce** (listed early co‑founder).
- **Investors:** Peter Thiel (Thiel Capital / Founders Fund), Alan Howard, Louis Bacon, Bitmain, Galaxy
  Digital, Nomura, Mike Novogratz, Richard Li, Christian Angermayer.
- **Claim to fame:** created **EOSIO** and ran the **EOS ICO** (Jun 2017 – Jun 2018), raising **~$4.1–4.2B**,
  the largest token sale at the time.

### 3.2 Credibility / track record (mixed)
- **SEC settlement (2019):** $24M civil penalty for an **unregistered ICO** (≈0.58% of the raise; settled
  without admitting/denying).
- **Investor class action:** ~**$22M settlement** (finalized ~Jan 2025 for U.S. domestic transactions).
- **EOS community fallout:** widely criticized for under‑investing in EOS (allegedly reinvested <$1B of
  ~$4.2B, parked ~$2.2B in Treasuries), and for diverting focus to the **Bullish** exchange. EOS's market
  value collapsed from ~$18B peak to <$1B; the network later **rebranded to "Vaulta" (2025)**.
- **Notable success:** spun out **Bullish**, which **IPO'd on the NYSE (BLSH) in Aug 2025** (~$1.1B raised;
  intraday valuation ~$13B). Bullish also owns CoinDesk.

### 3.3 Dan Larimer (relevant because scammers "impersonate Dan")
- **Genuine technical pioneer:** invented **Delegated Proof of Stake (DPoS)** and the **Graphene** engine;
  founded **BitShares** (2013/14), co‑founded **Steem/Steemit** (2016), architected **EOSIO**.
- **Left Block.one as CTO effective 2020‑12‑31** (announced ~2021‑01‑10). Sources: Block.one's own press
  release ("Block.one Announces Departure of Daniel Larimer"), Nasdaq/CoinDesk, IQ.wiki.
- **Now:** works on **Fractally** ("fractal governance" DAOs; `fractally.com`, X `@bytemaster7` /
  `@gofractally`); has shifted much public writing toward religious/eschatological themes; **not** running
  EOS/Vaulta today.
- **Caveat:** his public LinkedIn still lists "CTO at Block.one (Current)" — **stale/outdated**.

### 3.4 B1's OFFICIAL scam warning (key corroboration)
Block.one published **"Be on High Alert for Scams"** (`b1.com/press/high-alert-for-scams/`). Direct quotes:
- *"These attempts include people impersonating b1.com personnel. They even impersonate Brendan and Dan."*
- *"No Block.one personnel will use Telegram, Skype, LinkedIn or other informal communication channels to
  propose a business relationship or partnership…"*
- *"…do not download, compile, install, or run software from untrusted sources, including repositories,
  websites and links…"*

This aligns point‑for‑point with the scam described here.

### 3.5 Verified official B1 channels (use for verification)
| Channel | Official value |
| --- | --- |
| Website | `b1.com` (formerly `block.one`) |
| General email | `support@block.one` |
| Press email | `media@block.one` |
| LinkedIn (company) | `linkedin.com/company/b1official` (~15.6k followers) |
| Careers portal | historically `careers.block.one` — **currently 503 / inactive**; no active public listings found |

---

## 4. The "Jolanta Maxwell" LinkedIn profile (the "HR" contact)

- **Profile:** `linkedin.com/in/jolanta-maxwell-fmaat-20b56511b` — headline *"Account Manager | Blockchain &
  Web3 | B2B SaaS Solutions"*, listing **"Account Manager at B1" since Aug 2021**.
- **Verified real identity (checks out):** a genuine **UK accountant/bookkeeper** in **Wymondham/Thetford,
  Norfolk**; **FMAAT** (Fellow of the AAT); owner of **Red Heels Accounting Ltd** — Companies House
  **#09617031**, incorporated **2015‑06‑01**, previously named *"JM Bookkeeping (Service) Ltd,"* renamed to
  *Red Heels Accounting Ltd* in **Jan 2023**.
- **Domain age check:** `redheelsaccounting.co.uk` registered **2023‑03‑12** (~3 years) — **consistent** with
  the documented 2023 rebrand (i.e., innocent, not a throwaway).
- **Assessment:** The verifiable parts of her identity are **genuine and innocent**. The **"B1 / Web3"
  role is unverified and incongruent** with an otherwise 100% accounting career (no Web3/tech skills). Most
  likely explanations: her profile/identity is being **used as scaffolding**, her **account is
  compromised**, or the line was embellished. **This is NOT proven identity theft** (only one genuine
  profile was found; no clone). She should be treated as a **potential victim**, contacted gently, and NOT
  publicly accused.
- **Her business contact (for a courtesy heads‑up):** `info@redheelsaccounting.co.uk` / `07707631496`.
- **To definitively check for a clone:** reverse‑image‑search her profile photo; search LinkedIn for a
  second "Jolanta Maxwell"; ask her directly whether she works for B1.

---

## 5. The scam — chronology / timeline

> Replace bracketed exact dates/times with your own records where noted.

| Date | Event |
| --- | --- |
| ~2026‑06‑01 | Domain `danielsecurity.pro` registered via Hostinger (later found parked). Infrastructure staged in advance. |
| ~late Jun 2026 | Initial contact via **LinkedIn** from the "Jolanta Maxwell" (HR) account and a recruiter alias **"Daniel"**, claiming to represent **B1/Block.one**. |
| ~2026‑06‑28 | Interview invite via **Calendly** (`calendly.com/daniel-hiring/40min`). Booking‑confirmation email reply‑to = **`contact@danielsecurity.pro`** (not `@block.one`). |
| [date] | **Round 1:** slot booked; interviewer **no‑showed**; "busy, please reschedule." |
| [date] | **Round 2:** rescheduled; **no‑show** again. |
| [date] | **Round 3:** "Daniel" joined for **~1 minute**; asked only for **resume + contact details** and **full name** (victim gave first+last, omitted middle name); asked to schedule a "technical round" for Monday. |
| 2026‑07‑13 (Mon) | **Technical round:** victim joined via the same Calendly link; **nobody attended ~30 min**; HR ("Jolanta") **unresponsive**; victim left a message and dropped. Shortly after, the **Calendly link was disabled**. |
| 2026‑07‑14 (Tue) | Message: interviewer "running very busy" → a **take‑home assignment** would be provided instead. |
| 2026‑07‑15 (Wed) AM | **`Fullstack.zip`** shared via **Google Drive** (owner `cool9571791@gmail.com`). Victim asked them to resend from an official `@block.one` account; **opened the Drive link but did NOT download/run** on their computer. |
| 2026‑07‑15 | Sample downloaded and analyzed **only inside an isolated, disposable AWS EC2 sandbox** (no credentials/keys/wallets present). **File never executed.** |

**Behavioral red flags observed:** advance‑staged throwaway domain; impersonation of a real company; only a
first name / unverifiable recruiter; personal Calendly (not company‑branded); non‑corporate email; repeated
no‑shows; a token 1‑minute "interview" collecting personal details; a link that vanished; take‑home code from
a personal Gmail; pivot to "assignment" instead of a real conversation.

---

## 6. Indicators of Compromise (IOCs)

| Type | Indicator | Notes |
| --- | --- | --- |
| Impersonated company | **B1 / Block.one** (`b1.com`) | Legitimate; being impersonated |
| Recruiter alias | **"Daniel"** | No verifiable profile found |
| Email (reply‑to) | **`contact@danielsecurity.pro`** | From the Calendly invite; not `@block.one` |
| Domain | **`danielsecurity.pro`** | Registered **2026‑06‑01** via **Hostinger**; **parked**; `clientTransferProhibited`; expires 2027‑06‑01 |
| Domain variant | **`danielsecurities.pro`** | Referenced in later correspondence |
| Scheduling | Calendly handle **`daniel-hiring`** (event "40min", title "40min - daniel") | `noindex`; **link later disabled** |
| "HR" LinkedIn | `linkedin.com/in/jolanta-maxwell-fmaat-20b56511b` | Real accountant; likely compromised/impersonated (see §4) |
| Malware lure | **`Fullstack.zip`** | Google Drive |
| Drive file ID | **`1ZyhrpzoAtWukClVpgCAyEK-KgHMvpl2J`** | Public "anyone with link" |
| Drive owner account | **`cool9571791@gmail.com`** | Personal Gmail (not corporate) |
| Sample size | **62,508 bytes** (stored/uncompressed) | Ships a full `.git/` repo (source in git objects) |
| **`Fullstack.zip` SHA‑256** | **`4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e`** | Container |
| **Malicious files** | `.git/hooks/post-checkout`, `.git/hooks/pre-commit` | Byte‑identical, 500 B each; the *only* malicious code |
| **Malicious hook SHA‑256** | `f126186fe14d02e0d07f14d1ad3b0653abaaae9e099223dbae7513796452054f` | Both hooks |
| **C2 endpoint (IP)** | **`216.126.239.166`** | Hard‑coded, plain HTTP, no domain |
| **Payload URLs** | `http://216.126.239.166/728/728m` (macOS) · `/728l` (Linux) · `/728w` (Windows) | `curl … \| sh` / `wget … \| sh` / `\| cmd` |
| **Attacker git identity** | **`wondev_mum <wondev.mum@gmail.com>`** | Author/committer on all 3 commits (2026‑07‑14, UTC‑0400) |
| Lure PDF SHA‑256 | `ed517a74a9efc7a79ed4889b7c7ce36829824704fcf672bb915d0e934f589c2d` | `Fullstack_Home_Task.pdf` (decoy brief) |
| Targeted wallets | Decided by remote second stage (not fetched) | Campaign norm: MetaMask/Phantom via BeaverTail |

> **Delivery‑method note:** unlike the classic `postinstall`/`.config.env`+`atob()` placement anticipated in
> §7, this sample hides the loader in **git hooks** (`post-checkout`/`pre-commit`) that fire on the
> `git checkout`/`git commit` steps the brief tells you to run. Shipping a full `.git/` inside the ZIP is the
> only way to plant live hooks (git never transmits hooks over clone/fetch). This is a documented evolution of
> the same campaign ("TaskJacker"). Full write‑up: [`fullstack-zip-static-analysis.md`](fullstack-zip-static-analysis.md).

---

## 7. Threat attribution — "Contagious Interview" (DPRK/Lazarus)

Corroborated by Microsoft, NVISO Labs, Silent Push, ReversingLabs, and Palo Alto Unit 42.

- **Playbook:** fake recruiter → LinkedIn → "technical take‑home" project → victim runs `npm install`/the
  app → **BeaverTail** infostealer (also a loader) + **InvisibleFerret** RAT (and sometimes **OtterCookie**)
  → steals SSH keys, cloud tokens, browser sessions, and **crypto wallets (MetaMask/Phantom)**.
- **Where the payload usually hides in these projects (inspect these first):**
  - A hidden env/config file (e.g., `server/config/.config.env`) containing a **base64 string masquerading
    as an "API key"** that is actually a payload URL, decoded via **`atob()`**.
  - An auth/`verifyToken`‑style function that does **`axios.post(api, { ...process.env })`** (exfiltrates all
    env vars) and executes the response via **`new Function(...)` / `Function.constructor`**.
  - **`.vscode/`** tasks that auto‑run `curl … | sh` on folder open.
  - **Whitespace / "scroll‑right" padding** — a tiny file that is secretly huge.
  - Payload hosts: **jsonkeeper / jsonsilo / npoint.io / vercel.app / pastebin**.
- **Mitigation baked into our procedure:** analyze read‑only; if forced to install, use
  `npm install --ignore-scripts`; run only in a disposable VM.

---

## 8. Safe analysis environment — disposable AWS EC2 sandbox

**Core principle:** *Nothing on the box (including a logged‑in Claude session) can be stolen unless the
malware actually executes.* Static analysis only reads files, so credentials are never exposed. The sandbox
is the backstop.

### 8.1 Ground rules
- Use a **separate throwaway AWS account**, not your main one.
- **Do NOT put the dummy IAM user's access keys on the instance.** Launch/terminate from a trusted machine
  or **AWS CloudShell**. The instance itself gets **no AWS credentials**.
- Put **nothing valuable** on the box: no wallets, seed phrases, SSH private keys, real logins.
- Use a **throwaway, scoped Anthropic API key** for Claude if possible; otherwise plan to **`/logout` +
  terminate** afterward (a full password change is only needed if something was executed).

### 8.2 Least‑privilege IAM policy for the dummy user
See [`artifacts/ec2-dummy-user-iam-policy.json`](artifacts/ec2-dummy-user-iam-policy.json). It allows EC2
launch/manage/terminate + describe **in one region**, and **hard‑denies** `iam:*`, `sts:AssumeRole`, and any
instance‑profile association (so no role can ever be attached to the instance). Replace the region as needed.

### 8.3 Instance hardening at launch
- **No IAM instance profile** attached.
- Enforce **IMDSv2** + **hop limit 1**: `--metadata-options HttpTokens=required,HttpPutResponseHopLimit=1`.
- Security group: **inbound SSH from your IP only**; no other inbound.
- Root EBS volume **DeleteOnTermination = true**.

### 8.4 Pre‑flight isolation checks (run BEFORE downloading) — and observed results
```bash
# 1) No IAM role should be attached -> expect 404
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/; echo
# 2) No local secrets
ls -la ~/.aws 2>/dev/null || echo "none (good)"
ls -la ~/.ssh 2>/dev/null || echo "none (good)"
env | grep -iE 'AWS|SECRET|PRIVATE|TOKEN|SEED|MNEMONIC' || echo "none (good)"
```
**Observed on the sandbox (all clear):**
- IAM role creds → **`404 - Not Found`** (no role attached).
- `~/.aws` → **none**.
- `~/.ssh` → only **`authorized_keys`** (public keys used for inbound SSH — not a secret; no private key present).
- env secrets → **none**.

### 8.5 Cleanup when done
- In Claude Code: **`/logout`**.
- **Terminate the instance** and **delete its volume**; delete the **dummy IAM user + access keys**.

---

## 9. Analysis tooling — the read‑only Claude Code command

We created a Claude Code slash command, **`/analyze-for-swapnil`**, stored at
[`artifacts/analyze-for-swapnil.md`](artifacts/analyze-for-swapnil.md) (copy it to
`~/.claude/commands/analyze-for-swapnil.md`). Behavior:

- **Fetching is allowed** (`git clone`, `npm pack`, tarball download, `tar -xf`) — none of these execute the
  target's own code.
- **NEVER executes the target:** no `npm/yarn/pnpm install`, no `pip install`, no build/postinstall, no
  running entry points.
- Produces a **VERDICT / EVIDENCE / IOCs / WHAT‑IT‑DOES** report covering: lifecycle scripts, obfuscation,
  process execution, network endpoints, dynamic remote loading, secret/credential access, and typosquatting.
- If completing analysis would require executing untrusted code, it **STOPS and recommends the sandbox.**

> Caveat noted during setup: for a **local package directory**, do **not** use `npm pack` (it can trigger
> `prepare`/`prepack` scripts). Read files directly instead.

A dependency‑free fallback prompt is included in the artifact file for when the slash command isn't available.

---

## 10. Analysis procedure (step by step)

> All read‑only. **Never `npm install` / build / run / open in an auto‑running IDE.**

**Download (Google Drive), on the sandbox only.** `gdown` via venv (Ubuntu 24.04 blocks system pip — PEP 668):
```bash
mkdir -p ~/scam-eval && cd ~/scam-eval
sudo apt-get install -y python3-venv
python3 -m venv ~/gdown-venv && ~/gdown-venv/bin/pip install --upgrade pip gdown
~/gdown-venv/bin/gdown --fuzzy "https://drive.google.com/file/d/<FILE_ID>/view" -O assignment.zip
```
**Reliable curl fallback (this is what worked here):**
```bash
cd ~/scam-eval
FID="1ZyhrpzoAtWukClVpgCAyEK-KgHMvpl2J"
curl -L -o assignment.zip "https://drive.usercontent.google.com/download?id=${FID}&export=download&confirm=t"
file assignment.zip                 # expect: Zip archive data
head -c 4 assignment.zip | xxd      # expect: 50 4b 03 04 (PK..)
sha256sum assignment.zip
```
**Scan + extract (extraction does not execute code):**
```bash
sudo apt-get install -y clamav; sudo systemctl stop clamav-freshclam 2>/dev/null || true; sudo freshclam 2>/dev/null | tail -2
clamscan assignment.zip
unzip -l assignment.zip
mkdir -p extracted && cd extracted && unzip -o ../assignment.zip
clamscan -r --infected .
ls -laR
```
**Targeted static hunt (Contagious‑Interview payload spots):**
```bash
find . -name package.json -exec echo '--- {} ---' \; -exec cat {} \;
find . -iname "*.env*" -o -iname "*.config*" -o -path "*/.vscode/*"
grep -rniE "atob\(|new Function|Function\.constructor|eval\(|process\.env|child_process|exec\(|execSync|spawn" .
grep -rniE "jsonkeeper|jsonsilo|npoint\.io|vercel\.app|pastebin|https?://[a-z0-9.-]+" . | head -60
grep -rElc '.{1000,}' .            # padded/obfuscated files (very long lines)
cat .git/config 2>/dev/null; git log --pretty="%an <%ae> %ad" 2>/dev/null | head    # identity leak
grep -rniE "/Users/|/home/|C:\\\\Users\\\\" . | head -30                            # embedded dev paths
```
**⚠️ If the sample ships a `.git/` directory (as `Fullstack.zip` did), inspect git hooks FIRST — plain
`grep` misses them, and the source lives in git objects, not on disk:**
```bash
# Non-sample hooks are the red flag (weaponized post-checkout / pre-commit)
ls -la .git/hooks | grep -v '\.sample'
grep -RniE 'curl|wget|\| ?sh|\| ?cmd|http://|https://' .git/hooks 2>/dev/null
# Read source that only exists in git objects (never checkout/commit/pull inside it!)
git -C . log --all --pretty="%an <%ae> %ad %s"
git -C . ls-tree -r --name-only --branches | head
# Check the payload can't have set global persistence (should be empty)
git config --global --get core.hooksPath; git config --global --get init.templateDir
```
**Deep read‑only analysis:** in `~/scam-eval/extracted`, run `/analyze-for-swapnil .` (decline any offer to
run/install/"test").

---

## 11. Findings so far & what remains

**Done:**
- Identified and profiled the legitimate B1/Block.one, Dan Larimer, and the "Jolanta Maxwell" identity.
- Confirmed `danielsecurity.pro` is a ~1‑month‑old parked Hostinger domain, unrelated to B1.
- Confirmed the Calendly handle `daniel-hiring` and its later disablement.
- Extracted the Drive file's public metadata: name `Fullstack.zip`, owner `cool9571791@gmail.com`.
- Downloaded `Fullstack.zip` to the isolated sandbox and computed its SHA‑256
  (`4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e`, 62,508 bytes).
- Matched the whole pattern to the documented "Contagious Interview" campaign and to B1's own scam notice.

**Static analysis — COMPLETE** (see [`fullstack-zip-static-analysis.md`](fullstack-zip-static-analysis.md)
and [`artifacts/claude-analysis.md`](artifacts/claude-analysis.md)):
- **Verdict: 🔴 MALICIOUS (high confidence)** — RCE dropper via **weaponized git hooks**
  (`.git/hooks/post-checkout` + `pre-commit`), a documented git‑hook variant of Contagious Interview
  ("TaskJacker"). The React+Express "Wallet Watchlist" app and the PDF brief are **decoys**.
- **C2:** `216.126.239.166` (plain HTTP); payload paths `/728/728m` (macOS), `/728l` (Linux), `/728w` (Win),
  each piped straight into `sh`/`cmd`.
- **Attacker identity leak:** git author **`wondev_mum <wondev.mum@gmail.com>`**; 3 commits within ~2 minutes
  on 2026‑07‑14 (UTC‑0400) — a throwaway kit.
- No `postinstall`/obfuscation/`atob`/`process.env` exfil in source (the app is clean bait); credential/wallet
  theft is in the **remote second stage** (not fetched, so no local wallet IOCs).
- IOC table (§6) and report templates (§12) updated accordingly.

**Still open (optional):** the second‑stage payload (`/728/*`) was deliberately **not** fetched, so its exact
capabilities/wallet targets are inferred from campaign norms (BeaverTail/InvisibleFerret) rather than observed.

---

## 12. Reporting templates

> IOCs below are finalized from the completed static analysis. Confirmed indicators to include in every
> report: **C2 `216.126.239.166`** (paths `/728/728m|l|w`), **attacker `wondev.mum@gmail.com`** (`wondev_mum`),
> malicious files `.git/hooks/post-checkout` + `pre-commit`, container SHA‑256 `4bec2b84…285e`.

### LinkedIn (report the profile)
```
Reason: Scam / impersonation / account may be compromised.
This account is being used in a fake-recruiter scam impersonating B1 / Block.one (b1.com).
Interviews were scheduled via a personal Calendly ("daniel-hiring", now disabled), with emails whose
reply-to was contact@danielsecurity.pro — a domain registered ~1 month ago and unrelated to Block.one.
After repeated no-shows I was sent a "take-home assignment" (Fullstack.zip) via a Google Drive link owned
by a personal Gmail (cool9571791@gmail.com), not a company address. This matches the documented
"Contagious Interview" malware campaign. Block.one's own notice states they do NOT recruit via LinkedIn and
that scammers impersonate their staff. Please review this account for impersonation/compromise.
```

### Google — report the Drive file/account (in Drive: ⋮ → Report abuse → Malware/Phishing)
```
This Google Drive file distributes malware as part of a fake-job "take-home assignment" scam impersonating
B1/Block.one.
File: Fullstack.zip
File ID: 1ZyhrpzoAtWukClVpgCAyEK-KgHMvpl2J
Owner account: cool9571791@gmail.com
SHA-256: 4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e
It is a trojanized "full-stack" project consistent with the DPRK "Contagious Interview" campaign
(BeaverTail/InvisibleFerret) designed to steal credentials and crypto wallets when installed/run.
Requesting takedown of the file and review of the owning account.
```

### Calendly Trust & Safety (trust@calendly.com)
```
The Calendly account with handle "daniel-hiring" (event "40min", link now disabled) is being used in a
fake-recruiter scam impersonating B1/Block.one to lure victims into a malware "take-home assignment."
Associated indicators: email domain danielsecurity.pro; Google Drive malware file "Fullstack.zip"
(owner cool9571791@gmail.com). Requesting review/removal.
```

### Action Fraud (UK) / FBI IC3 (US)
```
I was targeted by a fake job-recruitment scam impersonating B1/Block.one (b1.com). A "recruiter" (alias
"Daniel"; no verifiable identity) and a possibly-compromised LinkedIn "HR" account contacted me via
LinkedIn, scheduled interviews through a personal Calendly ("daniel-hiring") later disabled, and emailed
from contact@danielsecurity.pro (domain registered 2026-06-01 via Hostinger, parked). After several
deliberate no-shows, I was sent a "take-home assignment" — Fullstack.zip — via Google Drive
(file ID 1ZyhrpzoAtWukClVpgCAyEK-KgHMvpl2J; owner cool9571791@gmail.com;
SHA-256 4bec2b8443bd1600a2239cdb4d77fc9e252785fa144b00f81dd3a25e73ec285e). Analysis indicates trojanized
code matching the DPRK "Contagious Interview" campaign (BeaverTail/InvisibleFerret) intended to steal
credentials and cryptocurrency wallets. I did NOT run the file. C2: 216.126.239.166 (paths /728/728m|l|w);
attacker git identity wondev_mum <wondev.mum@gmail.com>. No financial loss.
```

### Hostinger abuse (abuse@hostinger.com)
```
Domain danielsecurity.pro (registered via Hostinger, 2026-06-01, currently parked) is being used in a
fake-recruiter malware scam impersonating B1/Block.one. Emails with reply-to contact@danielsecurity.pro
directed a victim to a malware "take-home assignment" (Google Drive file Fullstack.zip). Requesting
investigation and suspension for abuse/phishing/malware distribution.
```

### Courtesy heads‑up to the real Jolanta (info@redheelsaccounting.co.uk) — non‑accusatory
```
Subject: Possible misuse of your LinkedIn profile
Hi Jolanta, I'm reaching out as a courtesy. A LinkedIn profile under your name (listing an
"Account Manager / B1 / Web3" role) was used to contact me in what appears to be a fake-recruiter scam
impersonating B1/Block.one — involving a personal Calendly, emails from danielsecurity.pro, and a malware
"take-home" file. Your real business details (Red Heels Accounting, AAT/FMAAT) look genuine, so I suspect
your profile may have been cloned/compromised or your identity used without your knowledge. You may want to
check/secure your LinkedIn account and report any impersonation.
```

### Heads‑up to B1 (support@block.one)
```
Subject: Someone is impersonating B1/Block.one for fake recruiting
Hello, someone is impersonating B1/Block.one in a fake-recruitment scam. Indicators: recruiter alias
"Daniel"; email domain danielsecurity.pro (reply-to contact@danielsecurity.pro); Calendly "daniel-hiring";
a LinkedIn "HR" account (Jolanta Maxwell) that may be compromised; and a malware "take-home" file
"Fullstack.zip" via Google Drive (owner cool9571791@gmail.com), consistent with the "Contagious Interview"
campaign. Sharing so your team is aware. I have not run the file.
```

---

## 13. Golden safety rules

- ✅ Fetch, verify (`file`, `sha256sum`), scan (`clamscan`), extract (`unzip`), and **read** — all safe.
- ❌ **Never** `npm/yarn/pnpm install`, `pip install`, build, run, or open in an auto‑running IDE.
- ❌ Never install any "meeting/video app" they push (fake meeting apps are a common malware vector).
- ❌ Never share private keys, seed phrases, passwords, 2FA codes, ID/KYC, or pay any fee.
- ✅ Do everything in the disposable VM; keep nothing valuable on it; `/logout` + terminate when done.
- ✅ Verify any "B1" contact only via `b1.com` / `@block.one` / `linkedin.com/company/b1official`.

---

## 14. Sources / references
- Block.one — "Be on High Alert for Scams" (`b1.com/press/high-alert-for-scams/`).
- Block.one — "Announces Departure of Daniel Larimer"; Nasdaq/CoinDesk (Jan 2021); IQ.wiki.
- SEC press release 2019‑202 ($24M ICO penalty); Reuters (class‑action settlement).
- Microsoft Security Blog; NVISO Labs; Silent Push; ReversingLabs; Palo Alto Unit 42 — "Contagious
  Interview" / BeaverTail / InvisibleFerret / OtterCookie.
- Companies House — Red Heels Accounting Ltd (#09617031). Nominet/Verisign RDAP — domain registration dates.

---

*Prepared as a defensive security investigation. Do not execute the referenced sample outside an isolated,
disposable environment.*
