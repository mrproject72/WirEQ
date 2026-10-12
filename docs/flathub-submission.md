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
2. Clone the fork, create a branch named after the app ID:
   ```bash
   git clone https://github.com/mrproject72/flathub.git && cd flathub
   git checkout -b io.github.mrproject72.WirEQ
   ```
3. Create the app directory with the manifest and metainfo:
   ```bash
   mkdir io.github.mrproject72.WirEQ
   cp /path/to/WirEQ/io.github.mrproject72.WirEQ.yml io.github.mrproject72.WirEQ/
   cp /path/to/WirEQ/assets/metainfo/io.github.mrproject72.WirEQ.metainfo.xml io.github.mrproject72.WirEQ/
   ```
4. **Edit the copied manifest's main module source**: Flathub builds from
   a remote, not from the PR directory. Replace
   ```yaml
   sources:
     - type: dir
       path: .
     - cargo-sources.json
   ```
   with
   ```yaml
   sources:
     - type: git
       url: https://github.com/mrproject72/WirEQ.git
       tag: v0.12.0
       commit: <full sha of the tag>
       x-checker-data:
         type: git
         tag-pattern: ^v([\d.]+)$
     - cargo-sources.json
   ```
   (`x-checker-data` lets the Flathub bot file update PRs automatically
   when new `v*` tags land; get the sha with
   `git rev-parse v0.12.0^{commit}` in the WirEQ repo.)
5. Take fresh screenshots (see below), add them to the fork (e.g. under
   the app directory), and point the metainfo's `<screenshot>` URLs at
   them.
6. Commit, push, open a PR to `flathub/flathub` titled with the app ID.
7. In the PR description, include the justifications below verbatim —
   the reviewers ask for them when the linter errors appear.
8. The `flathub-build-bot` builds the manifest on their infra; address
   any build failure comments, then wait for a maintainer review.
9. After merge the app publishes. Later releases = a PR to
   `flathub/flathub` bumping the `tag:`/`commit:` in the same manifest.

## Screenshots must be retaken

The two screenshots the metainfo references (dated 2026-09-18) show the
**pre-rebrand UI ("Mini EQ" title, the old sidebar)** — Flathub review
checks branding consistency, so replace them first: launch the current
build, use GNOME's window-screenshot keybinding, and save two shots
(main window with a curve; optionally the Output page). Then update the
`<image>` URLs in the metainfo (both here and in the Flathub PR copy).

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
