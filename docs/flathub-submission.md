# Flathub submission — prep and checklist

State as of 2026-10-12: everything local is submission-ready; the
submission itself needs the GitHub-side steps that only the account
owner can do.

## Verified locally

- `flatpak-builder-lint manifest` — two errors remain, both deliberate
  policy exceptions (see Justifications below). Everything else clean.
- `appstreamcli validate --pedantic` on the metainfo — clean.
- The full gold path passes against the installed bundle
  (`tests-live/twodev_gold_path.sh --flatpak`, 10/10) with the exact
  manifest that would be submitted.
- Icons ship at all hicolor sizes + symbolic; screenshots are the next
  optional improvement (see below).

## Steps only the account owner can do

1. Fork `flathub/flathub` on GitHub (the same account as
   `mrproject72/WirEQ`).
2. Create a new branch `io.github.mrproject72.WirEQ` in the fork.
3. Add:
   - `io.github.mrproject72.WirEQ.yml` (verbatim from this repo),
   - `io.github.mrproject72.WirEQ.metainfo.xml` (from
     `assets/metainfo/`), plus any screenshots the metainfo references
     (`<screenshot>` URLs must be reachable; hosting them in the PR's
     repo or on the GitHub release page both work).
4. Open a PR to `flathub/flathub` titled with the app ID.
5. In the PR description, include the justifications below verbatim —
   the reviewers ask for them when the linter errors appear.
6. A Flathub bot builds the manifest on their infra; address any build
   failure comments, then wait for a maintainer review.

## Justifications for the remaining linter errors

- `finish-args-flatpak-spawn-access` /
  `--talk-name=org.freedesktop.Flatpak`:
  the app has a launcher button that opens **pavucontrol** (per-stream
  volume and app routing, which an EQ must not duplicate) when it is
  installed on the host. That requires `flatpak-spawn --host`. The
  button is insensitive unless a probe finds pavucontrol on the host.
- `finish-args-unnecessary-xdg-config-wireq-rw-access`:
  **intentional config sharing with non-flatpak installs.** Presets,
  per-device curve memory and the AutoEq cache live in
  `~/.config/wireq` and are shared between the flatpak and the
  deb/rpm builds; a one-shot migration copies the legacy `mini-eq`
  directory on first run. The sandbox-private
  `~/.var/app/<app-id>/config/wireq` would fork the user's presets
  across install methods.

## Optional polish before or during review

- Screenshots: the metainfo currently has none. Upstream ships
  `docs/screenshots/mini-eq{,-dark}.png`; retake WirEQ screenshots
  (the GNOME Shell Screenshot D-Bus is restricted on this machine —
  use the shell's own screenshot keybinding) and add
  `<screenshot>` entries with `<caption>`s.
- Runtime: `org.gnome.Platform 51` is available (lint warning only);
  50 remains supported. Revisit when bumping, and bump the CI container
  (`ghcr.io/flathub-infra/flatpak-github-actions:gnome-51`) together
  with it.
