# Analysis output

Sanitized, read‑only **static‑analysis results** for the investigation in
[`../../b1-fake-recruiter-scam.md`](../../b1-fake-recruiter-scam.md).

## What goes here
Text/markdown evidence extracted from the suspected sample without executing it, e.g.:

- `file-hashes.txt`, `file-sizes.txt` — inventory + SHA‑256 of each file
- `package-json.txt`, `lockfiles.txt` — manifests + lifecycle scripts
- `exec-patterns.txt` — `atob` / `new Function` / `eval` / `process.env` / `child_process` hits (file:line)
- `urls.txt`, `payload-hosts.txt` — network endpoints / C2 / second‑stage hosts
- `git-identity.txt` — `.git` author name/email, commit timestamps (attacker identity leak)
- `dev-paths.txt` — embedded `/Users/<name>` or `/home/<user>` paths
- `claude-analysis.md` — the full VERDICT / EVIDENCE / IOCs / WHAT‑IT‑DOES write‑up

## Rules (read before adding files)
1. **Never commit the malware** — no `Fullstack.zip`, no `extracted/` tree, no binaries, and no
   decoded second‑stage payloads as runnable files. Decoded payloads may only appear as **inert
   quoted snippets inside markdown**. (The repo `.gitignore` blocks the common cases as a safety net.)
2. **Defang live indicators** in committed text: `hxxps://`, `evil[.]com`, `1.2.3[.]4`.
3. **Do not commit/push from the burner analysis VM** using a real credential. Move the sanitized
   text to a clean machine (or paste it into the chat) and commit from there.
4. **Mind repo visibility** — this folder will contain attacker IOCs and possibly their identity.
