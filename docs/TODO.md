## Session 2026-10-10 backlog (prioritised)

The flatpak EQ now works end-to-end (native-biquad graph, sandbox push,
drag editing, exit reliability). Ranked next steps:

1. ~~**Graph parity polish (steps 6-9)**~~ — DONE 2026-10-11: the
   gradient fill, the glow, the contribution curve, the bypass styling,
   the focus row + the N Bands chip, solo dimming, fader-drag
   selection, and the LUFS meter bar.
2. ~~**Preset lifecycle**~~ — DONE 2026-10-11: audited — add/update/
   rename/delete/import/export were all already wired; added the
   replace confirmation (upstream confirm_preset_replacement) and the
   delete confirmation with the curve-kept relabel.
   EXTENDED 2026-10-11 (commit `c0793b7`): profiles wider than 10 bands
   now show all their faders — the row builds all 32 widgets and the
   visible count follows `fader_band_count_for_profile` on preset
   apply/reset/restore, with a horizontal scrollbar beyond 10 bands
   (upstream `set_visible_band_count` parity; the 648px window minimum
   and the compact ≤10-band row are unchanged).
   EXTENDED 2026-10-11: a +/− band-count stepper next to the "N Bands"
   chip activates the next log-spaced slot as a flat Bell / deactivates
   the last visible band, so the profile width is editable in the UI
   without loading a profile. The tick payload now always covers all 32
   faders (332 controls), which also fixes deactivated nodes keeping
   their last wet controls after a shrink (docs/BUGS.md).
3. ~~**Controller-pattern UX**~~ — DONE 2026-10-11: while a graph dot or
   a fader is dragged, the band payload pushes ride the 33 ms tick
   (upstream's 16 ms class) so the curve follows the mouse live; outside
   a drag the 400 ms debounce stands. The full state migration (moving
   the bands out of the fader widgets into one authoritative list) is
   DEFERRED: the mouse-absolute drag fix removed the copy-lag bug class,
   and migrating every edit path risks the just-fixed behaviours for no
   further user-visible gain. Revisit only if a new bug lands in the
   fader/graph state sync.
4. ~~**Unit-test coverage for the backend**~~ — DONE 2026-10-11.
   Part 1: target_is_stale, peak_hold_step, and the Sin/Bell graph-label
   regression guard are extracted and tested. Part 2 (commit `92091c9`):
   the pod wire format is pinned by a round-trip test —
   `build_props_controls_pod_bytes` ↔ `parse_props_controls_pod` with
   exact f32 equality (the wire carries f32 per the upstream cast), the
   112-control push-set contract at 10 active bands (preamp b0..a2 both
   sides + per-band Freq/Q/Gain and mixer Gain 1/2 both sides), and the
   bypass-neutrality push (every mixer dry, filters still addressed).
   The deeper routing paths that need a live engine are covered by the
   containerised PipeWire smoke test below and the gold path.
5. **Flathub submission** — the rebrand is done (app ID
   `io.github.mrproject72.WirEQ`, the manifest/metainfo/icons renamed);
   the submission itself needs the Flathub repo setup + maintainer
   review on the GitHub account. The GNOME Shell extension from
   upstream is a separate follow-up.
6. ~~**AGENTS.md phase status refresh**~~ — DONE 2026-10-11 (part of the
   WirEQ rebrand commit: all phases ✅, the stale graph claims removed).

# TODO

Low-priority backlog. Nothing here blocks a release.

## CI / multi-distro pipeline

CI is currently green on `ubuntu-24.04` after dropping the libadwaita
`v1_7` requirement (WrapBox -> GTK4 FlowBox), so the app builds against
libadwaita 1.5 as shipped by Ubuntu 24.04 LTS.

Ranked by value, all via `container:` jobs (GitHub-hosted runners are
Ubuntu-only):

1. **Containerised PipeWire smoke test — highest value.** All 90 tests are
   unit tests; `pipewire_backend.rs` and `routing.rs` have zero automated
   coverage and are only verified on a dev machine. PipeWire can run in a
   container against a dummy/null sink, which would let CI:
   - create the filter-chain and assert `wireq_sink` appears
   - exercise route/unroute and assert streams are handed back to the
     default output (regression test for the "audio stops when the app
     closes" bug)
   - confirm biquad coefficients actually reach the node
2. ~~**Flatpak build job**~~ — DONE 2026-10-06: `io.github.mrproject72.WirEQ.yml`
   + `flatpak` job in `ci.yml`; appstream/desktop validated.
3. **Fedora** (`fedora:latest`) — largest PipeWire + GNOME desktop overlap;
   exercises the RPM dependency path.
4. **Arch** (`archlinux:latest`) — rolling, so it surfaces libadwaita /
   GTK API drift early, before it reaches users.
5. **Debian stable** (`debian:trixie`) — worthwhile but conservative
   packages make the signal slow-moving.
6. **CachyOS** (`ghcr.io/cachyos/docker`) — Arch-based, so build coverage
   duplicates Arch. Its real differentiator is x86-64-v3/v4 optimised
   packages and a tuned kernel, so the useful job there is a **CPU
   benchmark** (measured %CPU under load vs a stock target) rather than a
   compile check — actual evidence for the project's low-CPU premise.

Skip: Alpine/musl — not a realistic PipeWire desktop target.

## CI / release flow

The current `.github/workflows/ci.yml` runs `fmt`, `clippy -D warnings`,
`check --release`, `test --lib` and `build --release` on push and PR to
`main`. Gaps worth closing eventually:

- **Branch protection on `main`** — require the CI check to pass and at
  least one review before merge. Right now nothing enforces it.
- **Release artifacts** — CI now produces the flatpak repo via `flatpak-builder`;
  still add `actions/upload-artifact` for the release binary and a `.deb`.
- **Tag-triggered releases** — no workflow on `push: tags: [v*]`, so
  tagging does nothing.
- **No integration test on live PipeWire** — all 90 tests are unit tests.
  The backend (`pipewire_backend.rs`, `routing.rs`) is only verified on a
  developer machine. A containerised PipeWire smoke test would catch real
  regressions.
- **No coverage reporting.**

## Version numbering and changelog

The crate has sat at `0.9.0` for the whole rewrite with no changelog
discipline. Needs:

- a real version scheme from a numbered first release
- `CHANGELOG.md` (Keep a Changelog format), back-filled from
  `docs/*-updates.md`
- README updated with the version history

## Filter-chain output re-target (needs live validation)

The monitor now follows a system default-output change, but the
filter-chain's own output does not. Implementing it means removing the
existing `wireq_sink_output -> old_sink` link and creating a new one.

Do NOT ship this without testing on real hardware: a wrong link produces
either silence or a feedback loop. Needs a machine with at least two
output devices and a live sink switch to validate against.

Building blocks already present in `routing.rs`: `find_node_id_by_name`,
`find_node_target`, `get_links` (RouteInfo has source/target/link ids),
`remove_link`, `link_nodes`.

## Feature gaps

- **Preset lifecycle** — no save/save-as, import/export/delete, fallback
  presets. (Revert/reapply, file monitoring, and output-preset linking
  are implemented.)
- **Fader bottom residual clip** at some window sizes (see Known Issues).
- **Flatpak packaging** and **GNOME Shell extension**.
- **Config migration** — the app ID and config directory were renamed to
  `io.github.mrproject72.WirEQ` / `~/.config/wireq`. Existing
  settings under the old `~/.config/wireq` are **not** picked up. If
  that matters, add a one-shot migration on first run.

## Session 2026-10-11 follow-up (user-verified fixes)

- **Peak chip flicker** — root cause: two writers to the peak label
  (apply_live_peak at 10 Hz vs update_curve_peak's estimate every 33 ms
  tick, different formats). Fixed: the live monitor peak owns the label
  while active; the estimate only updates the internal value. Also the
  dBFS display is now a peak-HOLD meter (1 s hold, 6 dB/s fall) at a
  10 Hz update cadence (upstream emit parity).
- **Ctrl+C / exit reliability** — the D-Bus teardown runs before the
  window closes; the shutdown re-adoption is guarded.
- **Stale targets from a killed instance** no longer hide streams from
  the enable.
- **The window minimum width** raised to 648 (the toolbar needs 644).
- **The app icon** restored in the flatpak (explicit installs + cache).
