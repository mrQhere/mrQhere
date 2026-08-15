### Hi, I'm Q 👋
B.Tech ECE (Data Science) @ SRM KTR, class of 2028.
Building toward a security-focused SWE role — AppSec / VAPT.

### 🔧 Main Projects
- **[SecurityManagementPlatform](https://github.com/mrQhere/SecurityManagementPlatform)** —
  local-first, zero-egress VAPT orchestration tool. EPSS/GreyNoise/CISA KEV
  correlation stack — not another tool-count wrapper.
- **[eye-scan](https://github.com/mrQhere/eye-scan)** — offline-first edge AI
  diagnostic PWA. ONNX/WASM on-device inference, Grad-CAM explainability,
  federated learning hooks — zero patient data leaves the device.
- **[stock](https://github.com/mrQhere/stock)** — Indian stock-market analysis
  dashboard (Streamlit/XGBoost/PyTorch).

### 🛠️ Stack
Burp Suite · Nmap · FFUF · Amass · Nuclei · SQLMap · Python · ONNX · Kali/Parrot

### 📌 Currently
Grinding CTFs (picoCTF → HTB/THM) · VAPT dev internship @ CountAI 

# SMP — Fix Brief (Re-verified against live code, 2026-08-15, commit `3949b6c`)

Real progress since the last pass — five items below are genuinely fixed and
well-implemented, not just relabeled. Two others are new findings from this
pull: one is a case of a fix moving the leak rather than closing it, the
other is a real regression introduced by the Docker networking fix. Read
those two carefully before assuming "was in the last brief" means "already
handled."

No pushing to `main`. Branch, then stop for review.

---

## Fixed — verified against real behavior, do not touch
- `smp-dev` hardcoded password, `_get_current_user` `"dev"` fallback,
  hardcoded JWT secret — all still gone.
- **Encryption fail-closed** — both `_encrypt_and_compress_data` and
  `_decrypt_and_decompress_data` in `tools/db_manager.py` now raise
  `SMPDatabaseError` when no active key is available, instead of silently
  writing/reading unencrypted data. The Fernet exception handling was also
  corrected to catch the right exception type. Real fix.
- **Archive path traversal** — `tools/tool_installer.py` now resolves every
  zip/tar member against the destination dir with `is_relative_to()` before
  extracting, and explicitly skips symlinks/hardlinks in tar archives. Real
  fix, well implemented.
- **Installer integrity** — all 9 downloaded tools (was 3) now have SHA-256
  checksums verified before extraction, and every `/master.zip` mutable-branch
  URL is gone. Real fix.
- **Dependency pinning** — `requirements.txt` is now 21/21 pinned with `==`,
  0 unpinned. Real fix.
- **Unit tests** — `tests/test_security.py` and `tests/test_new_architecture.py`
  now exist, and they target exactly the right things:
  `test_smp9999_error_sanitization`, `test_encryption_fail_closed`,
  `test_archive_path_traversal`. Confirm these are wired into CI as a
  required job if they aren't already.

---

## P0 — Still open

### 1. Error leak moved, not closed
**Problem:** `to_dict()` in `tools/errors.py` now sanitizes the message for
`SMP-9xxx` (unclassified) codes only — good, that closes the original
`get_version`/`risk_scores` leaks as literally reported. But
`risk_scores()` in `api/server.py` now does:
```python
raise SMPDatabaseError(f"Failed to fetch risk scores: {e}")
```
`SMPDatabaseError` is `SMP-3xxx`, which `to_dict()` does **not** sanitize —
so the raw `str(e)` still reaches the client, just wrapped in a
better-looking exception class instead of a bare dict. The taxonomy is
being used correctly at the call site and incorrectly at the sanitization
boundary.
**Fix — pick one, don't patch this one call site:**
- (a) Sanitize at the boundary for *every* code, not just `SMP-9xxx`: keep
  a separate `debug_message` field that's logged but never serialized, and
  make `to_dict()`'s public `message` always the safe, pre-written
  per-class description (e.g. `SMPDatabaseError`'s default message, not an
  f-string with `{e}` baked in).
- (b) Or, if some classified errors are genuinely meant to carry
  caller-safe detail (e.g. `SMPInvalidTargetError("URL must start with
  http://")` — that's fine to show), audit every `SMPError(...f"...{e}"...)`
  call site individually and remove the raw exception interpolation from
  ones that aren't.
Don't just re-add a special case for `SMP-3xxx` — grep for the pattern
`f".*\{e\}"` inside any `raise SMP*Error(...)` call across the codebase and
fix all of them the same way, since this is clearly a pattern the team
reaches for by habit.
**Verify:** trigger a real DB failure (rename the db file mid-request),
confirm the API response `message` field contains no path, no raw
exception text — only the class's safe default message. Confirm the real
detail is still in the server log.

### 2. Docker networking fix introduced a new exposure regression
**What changed:** `Dockerfile`/`docker-compose.yml` now set
`SMP_API_HOST=0.0.0.0`, and `api/server.py`'s `start_server()` reads that
env var correctly — that part is right, and it fixes the original
"Docker port unreachable" bug.
**New problem:** `main.py`'s `start_api_mode()` — the actual entry point
used by `python3 main.py --api`, including inside the Docker container via
the `CMD` — has its own **hardcoded** call:
```python
uvicorn.run(app, host="0.0.0.0", port=8000)
```
This bypasses `api/server.py`'s `start_server()` entirely, so the env var
never gets consulted here. Net effect: running `python3 main.py --api`
**natively, outside Docker**, now unconditionally binds to all network
interfaces regardless of `SMP_API_HOST` — the opposite problem from before.
A local install now exposes its API to the whole LAN by default with no way
to opt out short of a firewall.
**Fix:** `start_api_mode()` in `main.py` should call `api.server.start_server()`
(the function that already does this correctly) instead of hand-rolling its
own `uvicorn.run()`, or at minimum read `SMP_API_HOST` with the same
`127.0.0.1` default before calling `uvicorn.run()` directly.
**Verify:** run `python3 main.py --api` natively (not in Docker) with no
`SMP_API_HOST` set, then from another machine on the same network attempt
`curl http://<host-LAN-IP>:8000/api/v6/health` — this must fail/refuse.
Then confirm the Docker path (`docker compose up -d`, curl from the host)
still works as before.

### 3. Windows compatibility still broken
**File:** `main.py`, `import fcntl` — still unconditional, top-level.
**Fix:** unchanged from before —
```python
if os.name == "nt":
    # Windows-appropriate lock implementation
else:
    import fcntl
```
**Verify:** confirm the import is conditional and the Unix locking path is
unchanged on Linux/macOS.

---

## P1 — Real progress, needs finishing

### 4. Authorization gate: schema built and tested, never enforced
**What exists now:** `core/authorization.py` defines `AuthorizationSchema`,
`AuthorizationTracker`, and `is_valid()` (checks status + expiry) — this is
a genuine, reasonably complete implementation of the consent/engagement
tracking this brief asked for. `core/scan_policy.py` (rate limits, scanner
allow/deny lists, time windows) is **actually wired in** —
`scanners/scan_planner.py` calls `self.policy.is_scanner_allowed(...)`
before adding scanners to a plan. That half is real and working.
**The gap:** `AuthorizationTracker`/`is_valid()` from `core/authorization.py`
is not imported or called anywhere outside its own test file. Nothing in
`scan_planner.py` or the scan-execution path checks whether a target has a
valid, non-expired, non-revoked authorization before running intrusive
tests. The consent model exists on paper (and passes its own tests, which
only test the class in isolation) but doesn't gate anything yet — this is
the same "looks done because it has tests" shape as the error-taxonomy
issue from the last pass; a unit test on an unwired class proves the class
works, not that the feature works.
**Fix:** in `scan_planner.py` (or wherever a scan actually gets dispatched
to scanners), require a valid `auth_id` for the target's engagement and
call `AuthorizationTracker.is_valid(auth_id)` before allowing any scanner
above a defined activity-level threshold to run. Refuse/block with a clear
`SMPAuthError` if invalid or missing.
**Verify:** create an expired or revoked authorization for a target,
attempt to run an intrusive scan against it, confirm it's blocked — not
just that `is_valid()` returns `False` in isolation, but that a real scan
attempt is actually stopped by it.

### 5. Scope engine and authorization tracker are stubs, not wired to plans
**File:** `core/scope_engine.py`, `_load_scope_rules()`.
**Problem:** returns `[]` unconditionally — `# Stub: Normally this would
query the database`. `is_allowed()` correctly defaults to deny when no
rules exist, so this fails closed (not a security bug) — but it means
`ScanPlanner.create_plan()` currently either blocks everything or runs
against whatever default happens upstream; scope enforcement isn't real
yet. This is the same gap as item 4 above (`AuthorizationTracker`): a
well-built class that nothing has connected to real data.
**Fix:** back `_load_scope_rules()` with actual per-engagement scope rules
from the database (reuse whatever schema `AuthorizationSchema.scope` in
`core/authorization.py` was meant to feed). Do this at the same time as
wiring in `AuthorizationTracker` (item 4) — they're the same missing layer.
**Verify:** define real scope rules for a test engagement, confirm
in-scope targets plan successfully and out-of-scope targets are rejected
with a clear reason, not just an empty-rules deny.

---

## P2 — Structural, still not scoped for this pass
### 5. No real user/authorization model
Unchanged from before — `/api/v6/auth/token` still mints a JWT with
whatever `sub` the caller supplies, no user table, no roles. Fine for
single-user local use; needs a real design pass before multi-tenant.

---

## Explicitly out of scope for this pass
Scan payload/detection logic, report content/generation, and anything under
`intelligence/` beyond what's named above.
