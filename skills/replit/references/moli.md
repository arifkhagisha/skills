# Moli — persistent-session tour engine

Moli ([lexmount/moli](https://github.com/lexmount/moli)) is a
from-scratch Rust headless browser engine for AI agents: its own
DOM/layout/V8 crates, "structure first, pixels on demand". It earns
its tour seat for one reason — **session persistence**: since 1.1.21
its cookies and localStorage live in partition-specific SQLite
databases inside the server, so a logged-in session (and the
Cloudflare clearance earned with it) survives restarts instead of
being discarded per run. That is the property the fleet's
session-persistence doctrine asks for, the same one Camofox has and
fresh-profile engines lack.

## Install

`bash skills/scripts/setup.sh --machine repl --moli` — the lane
(`scripts/setup/moli.sh`) installs the pinned release binary
(MOLI_VERSION in `scripts/setup/profile-repl.sh`; size + SHA256
pins in the lane, computed from the official download — upstream
publishes no checksum asset), a `launch-moli` resident-server
launcher, and a `.env.example` in `$XDG_CONFIG_HOME/moli`.
x86_64 only: that is the verified asset; other arches die naming
the gap. The binary itself is a plain static executable — no nix
closure, no patchelf (contrast the foreign-Chromium engines).

## Driving it

- `moli serve --host 127.0.0.1 --port 9222` exposes CDP +
  WebDriver; Playwright connects with `connect_over_cdp` from the
  tour venv (the lightpanda lane's venv — Moli has no Python
  package of its own).
- `tour.sh --browser moli --machine repl` is the tour adapter
  (`_MoliEngine` in `scripts/tour.py`): config lives in
  `$XDG_CONFIG_HOME/moli/.env` (TOUR_LINKS / TOUR_COOKIES / dwell /
  gap / breaker, same as every engine). The jar seeds the session
  at tour start via `context.add_cookies`; the server's persisted
  state carries it from there. TOUR_JAR_WRITEBACK merges live
  cookies back into the jar after SHELL stops, as for the other
  Playwright engines.
- Server ownership: with `TOUR_MOLI_AUTOSTART=1` (default) the
  tour spawns `moli serve` when the endpoint is down and kills
  only the process it spawned; with `0`, a resident `launch-moli`
  workflow owns the server and the tour only connects (the
  camofox shape). `MOLI_CDP_URL` moves the endpoint.

## Gotchas (verified 2026-11-08)

- **`moli fetch --cookie-jar` does NOT apply the session** — fetch
  mode returned the logged-out page for every target in testing.
  The working path is serve + CDP with cookies planted in the
  context. Do not build on fetch mode for logged-in tours.
- The official `moli-installer.sh` does NO checksum/signature
  verification — that is why the lane pins size + SHA256 itself.
- Version probe is the `--version` FLAG (`moli 1.1.22`); there is
  no `version` subcommand.

## Evidence

2026-11-08, CDP drive from repl-0 (jafibaw jar, 13 cookies):
repl-0 / repl-1 / repl-2 / repl-3 all SHELL in 7.4–8.7s, zero
Cloudflare challenges — on par with camofox on the same wing, at a
fraction of the footprint (single ~45 MB binary).
