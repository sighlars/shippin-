# OpenLotus + vibe-to-ship — First-User Customer Review

**Role:** first external user (no prior account, no pairing, no MCP configured)
**Date:** 2026-09-17 (UTC)
**Tester environment:** Windows 11, Node v24.17.0, npm 11.13.0, git 2.54.0.windows.1, PowerShell 5.1 + `cmd /c`, POSIX sh via `C:\Program Files\Git\bin\sh.exe`
**Sources used (no assumptions):** skill README + raw `SKILL.md` at `https://github.com/sighlars/vibe-to-ship`, live pages `openlotus.io`, `/docs`, `/vibe-to-ship`, `/vibe-to-ship/loop`, `/vibe-to-ship/patterns`, `/vibe-to-ship/starters`, `/vibe-to-ship/security`, `/llms.txt`, `/map`, `/dashboard`, `/workspace`, `/pair`, `/new-project`, `/pricing`
**Local verification:** cloned `https://github.com/sighlars/vibe-to-ship` to temp dir, ran `scripts/loop.sh boot|triage|act|verify|learn`, `scripts/doctor.sh`, `scripts/install.sh` in an empty project

**Goal given by founder:** "check openlotus.io and its pages like a real user trying to check the docs, descriptions, tools etc and be using it and let me know the flaws, bugs etc, don't assume, use what's written in the skill readme and the openlotus.io pages"

**Verdict in one line:** The offline skill loop works and is well-written, but the online onboarding path is broken — `npx openlotus pair` 404s, MCP wiring disagrees with itself, `install.sh` pollutes `AGENTS.md`, and all three core app surfaces are auth-gated with no preview.

---

## 1. Customer journey (what I actually did)

### 1.1 Landing → docs → skill
1. Opened `https://openlotus.io`. Read hero: "One platform for the work that actually matters", "Install the skill / Connect your agent", "Progress Map / Dashboard / Agent Workspace", "Wire your agent in. Three steps, one time."
2. Opened `https://github.com/sighlars/vibe-to-ship`. Read README: 5-beat loop (boot, triage, act, verify, learn), install via `git clone ... ~/.claude/skills/vibe-to-ship`, then `bash .../scripts/install.sh` + `bash .../scripts/loop.sh boot`. Optional pairing via `npx openlotus pair` for 5 MCP tools.
3. Fetched raw `SKILL.md`. Noted it says "6 Beats of the Loop" (Beat 0 Setup + Beats 1–5), full command sequences, path convention `VTS=$HOME/.claude/skills/vibe-to-ship` / `.opencode/skills/vibe-to-ship`, run from project root.
4. Opened `/docs`. Noted "Two ways to get set up: With the skill: say 'set up OpenLotus' / Without it: copy the three blocks". Noted 3 manual steps: `mcp.json` + `npx openlotus pair` + `AGENTS.md/CLAUDE.md` rules. Noted `llms.txt` paste block.
5. Opened `/vibe-to-ship` + subpages `/loop`, `/patterns`, `/starters`, `/security`. Compared install commands and loop description against README/SKILL.md.
6. Opened `/map`, `/dashboard`, `/workspace`, `/pair`, `/new-project`, `/pricing`, `/llms.txt`.

### 1.2 Attempted pairing (the documented happy path)
1. Ran `npx -y openlotus --help` → `npm error 404 Not Found - GET https://registry.npmjs.org/openlotus`.
2. Ran `npm search openlotus` → "No matches found".
3. Ran `npm view @openlotus/cli version` and `npm view openlotus-cli version` → both 404.
4. Result: could not proceed with any browser pairing, MCP setup, `get_memory/get_reality/get_drift/record_decision`, hooks (`npx openlotus status`), or dashboard verification.

### 1.3 Attempted skill-only path (offline)
1. `git clone https://github.com/sighlars/vibe-to-ship.git` → success.
2. `sh scripts/loop.sh boot` in cloned repo → `READY (0 high, 3 watch)`.
3. `sh scripts/loop.sh triage` → correct `quiet 0d · 0 dirty · 4 TODOs` + Noise lines.
4. `sh scripts/loop.sh act` → prints TASK/SCOPE/DONE/STOP template.
5. `sh scripts/loop.sh verify` on clean tree → `VERDICT: PASSED` with 3 WARNs (no scope, no build, no tests).
6. `sh scripts/loop.sh learn` → stdout template (no MEMORY.md), asks to fill Decisions/Next.
7. Created empty `first-user-proj/`, ran `install.sh` → created `AGENTS.md` (78 lines), printed "no mcp.json found... not paired yet — run: npx openlotus pair", then `doctor.sh` → READY.

---

## 2. Issues (ordered by severity)

### [B-01] P0 — `npx openlotus pair` package does not exist on npm — onboarding fully blocked
- **Area:** onboarding / CLI / docs / skill / homepage
- **Where it is written:**
  - Homepage "Connect your agent" Step 1: `npx openlotus pair`
  - `/docs` → "Pairing a project": `npx openlotus pair`
  - `SKILL.md` Beat 0: `npx openlotus pair (no flags)`
  - `llms.txt`: Setup step 1
  - `scripts/install.sh:86`: `not paired yet — run: npx openlotus pair`
  - `references/agent-rules-snippet.md:55`: hooks `npx openlotus status`
- **Steps to reproduce (fresh machine, verified 2026-09-17):**
  ```cmd
  node --version  :: v24.17.0
  npm --version   :: 11.13.0
  npx -y openlotus --help
  :: npm error 404 Not Found - GET https://registry.npmjs.org/openlotus
  npm search openlotus
  :: No matches found
  npm view @openlotus/cli version  :: 404
  npm view openlotus-cli version    :: 404
  ```
- **Expected:** package resolves, browser opens to `/pair`, CLI writes `.openlotus/config.json`.
- **Actual:** 404, no pairing file can be created, no MCP server path exists locally, all downstream steps (mcp.json pointing at `cli/mcp.mjs`, `get_*` tools, dashboard) unverifiable.
- **Impact:** First user cannot become a paired user. Every "3 steps, one time" / "Pair in one command" claim fails.
- **Suggested fix (pick one, update everywhere):** publish `openlotus` to npm (preferred), OR rename docs to real package name, OR ship CLI from the skill repo / GitHub release with install instructions + version pin. Add `npm view` CI check so docs never drift from registry again.

### [B-02] P0 — `install.sh` appends the whole contributor doc, not just the rules block
- **Area:** skill script → `AGENTS.md`
- **File:** `scripts/install.sh:37-55` does `cat "$RULES_SRC"` where `RULES_SRC=references/agent-rules-snippet.md`
- **Source file:** `references/agent-rules-snippet.md:1-76` contains wrapper prose + snippet + hooks + explainer
- **Steps to reproduce:**
  ```sh
  mkdir first-user-proj && cd first-user-proj
  sh /tmp/vibe-to-ship-test/scripts/install.sh
  # → "[install] appended rules block to AGENTS.md"
  cat AGENTS.md  # 78 lines
  ```
- **Expected:** `AGENTS.md` gets only the fenced `## OpenLotus loop (standing rules)` block (lines 22–37).
- **Actual:** `AGENTS.md` contains "Copy the block below into your repo's CLAUDE.md or AGENTS.md...", "Where the file goes" table, "What happens after you commit it", "Optional enforcement", "Why this works". Meta-instructions leak into the agent's auto-loaded rules.
- **Impact:** Every new project starts with noisy, self-referential rules; agents will read instructions-about-instructions every session.
- **Suggested fix:** split `agent-rules-snippet.md` into `snippet.md` (exact block only) vs `README` prose, OR make `install.sh` extract only between ```` ```markdown ```` fences. Add idempotency test: run twice, diff `AGENTS.md`, assert only one standing-rules block and no "Copy the block below" string.

### [B-03] P1 — Beat count and memory filename disagree
- **Where:**
  - README: "5-beat operating loop — boot, triage, act, verify, learn" + "The 5-beat loop" table
  - `SKILL.md` §3 title: "The 6 Beats of the Loop" (Beat 0 Setup + Beats 1–5)
  - `/docs` "The vibe-to-ship skill": "A 5-step routine" (Boot/Triage/Act/Verify/Learn, no Setup)
  - `/vibe-to-ship/loop`: "The 5-beat loop" (1 Boot … 5 Learn)
  - Memory file: `SKILL.md` §2 + §5 + rules snippet step 4 say `PROGRESS.md`; `scripts/loop.sh:72-91` + SKILL.md §6 table say `MEMORY.md` (`learn` appends to `MEMORY.md`)
- **Reproduce:** `sh scripts/loop.sh learn` in repo without `MEMORY.md` → "No MEMORY.md found — printing ... instead. (Create MEMORY.md...)". No mention of `PROGRESS.md`.
- **Expected:** one canonical count and one canonical filename across README/SKILL/site/scripts.
- **Actual:** new user must guess whether Setup counts, and whether to create `PROGRESS.md` or `MEMORY.md` (pairing import depends on this per SKILL.md §2: "so a later pairing imports cleanly").
- **Suggested fix:** decide: either "6 beats (0–5)" everywhere OR "5 beats + 1-time setup" everywhere; decide `PROGRESS.md` vs `MEMORY.md` (recommend one file, one name, with migration note). Update README table, SKILL.md §3 title, `/docs`, `/loop`, `loop.sh` strings together.

### [B-04] P1 — MCP server path, invocation, and tool count disagree
- **Where:**
  - Homepage `mcp.json`: `{ "command": "node", "args": ["/absolute/path/to/openlotus/cli/mcp.mjs"], "env": { "OPENLOTUS_BASE_URL": "https://www.openlotus.io" } }`
  - `/docs` `mcp.json`: `{ "command": "node", "args": ["…/cli/mcp.mjs"] }` (placeholder, no env)
  - `references/openlotus-engine.md` §2: `node cli/openlotus.mjs agent` (different filename); §2b: `cli/mcp.mjs`
  - `install.sh:74-76` hint: `{ "mcpServers": { "openlotus": { "command": "node", "args": ["/abs/path/to/cli/mcp.mjs"] } } }`
  - Tool counts: README "five MCP tools — `get_memory`, `get_reality`, `get_drift`, `record_decision`, `create_project`"; engine lists 6 (+ `switch_project`); `/docs` "Makes 4 tools appear in every session"
  - Docs also mention `openlotus sync` / `openlotus status` CLI commands with no installable CLI (see B-01)
- **Expected:** copy-paste `mcp.json` works after pairing; tool list matches `cli/*.mjs` exports.
- **Actual:** no verifiable path; user cannot know which file to point at or how many tools should appear.
- **Suggested fix:** single canonical snippet (with `OPENLOTUS_BASE_URL` decision made explicit), single CLI filename, single tool table shared by README/SKILL/engine/docs/llms.txt. Include `node --check <path>` validation step already hinted in SKILL.md Beat 0.

### [B-05] P1 — `/map`, `/dashboard`, `/workspace` require auth with no logged-out preview; `/pair` and `/new-project` are JS-only
- **URLs:** `https://www.openlotus.io/map`, `/dashboard`, `/workspace`, `/pair`, `/new-project`
- **Actual (fetched as logged-out user):**
  - `/map`, `/dashboard`, `/workspace` → "Sign in | OpenLotus / Welcome back to OpenLotus / Email + Password / Don't have an account? Create account". No feature preview.
  - `/pair` → only "Pair your repo | OpenLotus OPENLOTUS" (no steps in HTML).
  - `/new-project` → "Loading... This should only take a moment" (never resolves in fetch).
- **Expected (per homepage promise):** "Progress Map — The shared tree, visualized", "Dashboard — Your week, summarized", "Agent Workspace — Talk with the map loaded" evaluable before signup; `/pair` shows the 3 copy-pastes (docs says "code is shown on ... /pair page").
- **Impact:** Cannot evaluate the core value prop; pairing flow cannot be audited without an account.
- **Suggested fix:** SSR/static fallback for `/pair` (steps + mcp.json + rules), public read-only demo map/dashboard (homepage already shows mock trees — link them), skeleton + timeout message for `/new-project` loader.

### [B-06] P1 — Install commands disagree (clone vs cp vs cd-into-skill)
- **Where:**
  - README: `git clone https://github.com/sighlars/vibe-to-ship.git ~/.claude/skills/vibe-to-ship` (Claude) / `.opencode/skills/vibe-to-ship` (opencode), then `bash $VTS/scripts/install.sh` from project root
  - `/vibe-to-ship` Quick install: `cp -r vibe-to-ship ~/.claude/skills/` / `cp -r vibe-to-ship .opencode/skills/` (no clone step, implies local dir)
  - `/vibe-to-ship/loop` top: `git clone https://github.com/sighlars/vibe-to-ship.git` then `cd vibe-to-ship && ./scripts/install.sh && ./scripts/doctor.sh` (runs inside skill — writes rules into the skill repo itself, violating SKILL.md path convention: "scripts read/write the project they run in, never the skill")
  - Homepage Beat 0 note: "From `openlotus-frontend`: opens your browser to pick a project. ... After `pnpm link`, just `openlotus pair`." — `openlotus-frontend` never introduced to new user
- **Suggested fix:** one canonical install block per agent (clone URL + destination + `bash $VTS/scripts/install.sh` from project root + `loop.sh boot`). Remove `cd vibe-to-ship && ./scripts/install.sh` or label it "contributor-only". Explain `openlotus-frontend` or remove the reference.

### [B-07] P2 — Windows / POSIX assumptions undocumented
- **Files:** all `scripts/*.sh` (`#!/bin/sh`, `set -eu`)
- **Reproduce on stock Windows:** `sh` not on PATH in `cmd`; `npx`/`npm.ps1` blocked by ExecutionPolicy (`PSSecurityException`). Had to use `cmd /c` + full path `C:\Program Files\Git\bin\sh.exe`.
- **Expected:** docs note "Windows: use Git Bash" + PATH tip, or `.ps1`/`.cmd` wrappers.
- **Actual:** no Windows notes in README/docs/skill; first Windows user stalls at step 1.
- **Suggested fix:** add 3-line Windows callout + test matrix (`shellcheck` already required by CONTRIBUTING — add Windows smoke).

### [B-08] P2 — `verify.sh` PASSES on empty verification
- **File:** `scripts/verify.sh:93-143`
- **Reproduce:** clean tree, no `package.json`/`go.mod`, no `--scope` → output: `WARN working tree clean`, `WARN no --scope`, `WARN no build`, `WARN no test`, `OK no new drift markers`, `VERDICT: PASSED`.
- **Expected:** `PASSED` means build+tests+scope checked; empty check should be `WARN`-only / `SKIPPED` / non-zero or explicit "nothing to verify".
- **Impact:** agent can claim "verified" when nothing ran — exactly what the skill claims to prevent.
- **Suggested fix:** if no build AND no tests AND no scope AND clean tree, exit 0 but verdict `NO-OP — nothing verified` (or exit 2), and forbid "tested/passing" language downstream.

### [B-09] P2 — README "What gets installed" tree does not match repo
- **README tree lists:** `boot.sh → doctor.sh`, `docs/tapes/`, `patterns/`, `examples/`
- **Actual clone (verified):** `scripts/{doctor.sh,install.sh,loop.sh,triage.sh,verify.sh}` (no `boot.sh`), root has `docs/`, `references/`, `scripts/`, `assets/`, no `patterns/` dir in listing
- **Also:** README "The 5-beat loop" table headers repeat (`Beat | Command | What it does | Fails when` then again `Beat | Command | What it checks | Fails when`) — copy-paste duplication.
- **Suggested fix:** generate tree from repo or remove it; dedupe table headers.

---

## 3. What worked (keep these)

- Skill clones cleanly; scripts are readable POSIX `sh`, `set -eu`, no dependencies beyond `git`/`node`.
- Offline loop is genuinely useful: `boot` correctly reported `0 high, 3 watch`; `triage` counts matched `git status/log/grep`; `act` contract is clear; `learn` template has branch/dirty/last-commit.
- `/llms.txt` is the best onboarding artifact — concise primitives, tool table, setup, pricing, trust notes. Link it more prominently.
- Pricing is consistent between homepage and `/pricing` (Free $0/1 product/500 events, Pro $39/3 products/5k, Studio $129/unlimited/3 seats).
- `/vibe-to-ship/{loop,patterns,starters,security}` content is strong (diamond pattern, fresh-context rule, 3-attempt cap, denylist) and matches SKILL.md intent.

---

## 4. Suggested update checklist (for founder)

- [ ] B-01: publish or rename `openlotus` CLI; update homepage + `/docs` + SKILL.md Beat 0 + `install.sh:86` + hooks snippet + `llms.txt` in one PR; add registry-existence CI
- [ ] B-02: fix `install.sh` to write snippet-only; add test asserting `AGENTS.md` contains exactly one `OpenLotus loop (standing rules)` and zero `"Copy the block below"`
- [ ] B-03: unify beats (5 vs 6) + `PROGRESS.md` vs `MEMORY.md` across README/SKILL/site/scripts
- [ ] B-04: single `mcp.json` snippet + single CLI filename + single tool table (5 vs 6 vs 4)
- [ ] B-05: SSR/static `/pair`, public demo map/dashboard, loader timeout for `/new-project`
- [ ] B-06: single install block per agent; remove `cd vibe-to-ship && ./scripts/install.sh` or mark contributor-only; explain or drop `openlotus-frontend`
- [ ] B-07: Windows/Git-Bash callout
- [ ] B-08: `verify.sh` NO-OP verdict when nothing ran
- [ ] B-09: fix install tree + dedupe README table

---

## Appendix A — Evidence log (commands + outputs)

```text
node --version → v24.17.0
npm --version → 11.13.0
git --version → git version 2.54.0.windows.1

npx -y openlotus --help
→ npm error 404 Not Found - GET https://registry.npmjs.org/openlotus

npm search openlotus → No matches found
npm view @openlotus/cli version → 404
npm view openlotus-cli version → 404

git clone https://github.com/sighlars/vibe-to-ship.git → success
sh scripts/loop.sh boot → "OK: git repo on branch main, 0 dirty files / OK: node v24.17.0 / 3 watch / Result: READY"
sh scripts/loop.sh triage → "quiet 0d · 0 dirty files · 4 TODOs / Noise: 4 TODO markers"
sh scripts/loop.sh act → TASK/SCOPE/DONE/STOP template
sh scripts/loop.sh verify (clean, no package.json) → 3 WARNs + "VERDICT: PASSED"
sh scripts/loop.sh learn (no MEMORY.md) → stdout template with <fill in> placeholders

sh scripts/install.sh (empty dir) → "[install] mode: opencode / appended rules block to AGENTS.md / no mcp.json found / not paired yet"
cat AGENTS.md → 78 lines, starts with "<!-- vibe-to-ship ... -->" then full contributor doc
sh scripts/doctor.sh (empty, unp-paired, no git) → "not inside a git repo / node OK / no mcp.json / not paired / standing rules present / READY"
```

Pages fetched 2026-09-17: `/`, `/docs`, `/vibe-to-ship`, `/vibe-to-ship/loop`, `/vibe-to-ship/patterns`, `/vibe-to-ship/starters`, `/vibe-to-ship/security`, `/llms.txt`, `/map`, `/dashboard`, `/workspace`, `/pair`, `/new-project`, `/pricing`, GitHub repo + raw `SKILL.md`.

---

*Prepared as a first-user review for the founder to action. All claims above are tied to quoted docs or reproduced commands; no paired/MCP behavior could be tested because pairing is blocked by B-01.*
