# Flathub submission — current-process guide

Written 2026-10-12 against https://docs.flathub.org (Submission +
Requirements + Linter). The flathub/flathub repo no longer hosts app
manifests on master; submissions PR against the `new-pr` branch and are
later moved to their own repository under the Flathub org.

## Read first: the generative-AI policy

https://docs.flathub.org/docs/for-app-authors/requirements#generative-ai-policy:

- **Flathub manifests must not contain AI-generated or AI-assisted
  content.** The submission manifest has to be authored by you. Use
  this repo's `io.github.mrproject72.WirEQ.yml` as the reference to
  work from — read every module, retype/adjust what you keep, and make
  sure you can explain each part. Do not paste agent-generated manifest
  text.
- **AI agents must not open or automate the submission PR, nor write
  its commit messages, description, or review replies.** The PR text,
  justifications and commit messages must be your own words.
- **App code**: disclose that the application (a Rust port of the
  Python upstream) was developed with heavy AI assistance, and roughly
  what that covers. Reviewers evaluate disclosed AI material at their
  discretion; undisclosed material is grounds for rejection.

Local docs that predate this version of the process (and any
agent-generated copy-paste manifest under docs/flathub/) are
references for study, not submission content.

## Verified facts (as of 2026-10-12)

- Runtime: **stay on org.gnome.Platform 50**. GNOME 51 (fd 26.08) has
  no LLVM Sdk extension (`llvm21` stops at 25.08, and there is no
  26.08 build), so bindgen (pipewire-sys) cannot run — the 51 build
  fails on `Unable to find libclang`. Actively maintained Rust apps on
  Flathub (e.g. net.nokyan.Resources) are also on 50 with the same
  rust-stable + llvm21 pair. The linter's runtime-update message is a
  warning only.
- `flatpak-builder-lint manifest` on the repo manifest: two errors,
  both deliberate and both documented below (flatpak-spawn for the
  pavucontrol launcher; host `~/.config/wireq` share). Everything else
  clean. The metainfo passes `appstreamcli validate --pedantic` and its
  screenshots show the current branding.
- The gold path passes against the exact manifest to be submitted
  (`tests-live/twodev_gold_path.sh --flatpak`, 10/10).
- The local build quirk: `x-checker-data` with `type: git` inside a git
  source trips the linter's deep JSON validation
  (`manifest-json-warnings`) when the manifest file is named after the
  app id. If the linter flags it for you, drop the `x-checker-data`
  block from the app's git source — automatic update PRs are a
  convenience, not a requirement.

## Steps only the account owner can do

Per https://docs.flathub.org/docs/for-app-authors/submission:

1. Fork `flathub/flathub` with **"Copy the master branch only"
   unchecked** (the `new-pr` branch must come along).
2. Clone the fork and branch:
   ```bash
   git clone --branch=new-pr https://github.com/mrproject72/flathub.git
   cd flathub && git checkout -b wireq-submission
   ```
3. Add at the **top level** of the repo (not a subdirectory):
   - `io.github.mrproject72.WirEQ.yml` — authored by you (see the AI
     policy above). Technical facts it must carry:
     - app id / runtime / sdk / runtime-version "50" as in the repo
       manifest,
     - sdk-extensions `rust-stable` + `llvm21`,
     - the finish-args from the repo manifest (two of them are the
       documented exceptions),
     - the three dependency modules (libebur128, pipewire
       filter-chain, wireq) with the wireq module's source being a
       **git source pinned to the release tag** (`v0.12.0`,
       commit `dd5df5c0a17e60f172afb60fbf891b79392ef722`) — Flathub
       builds from a remote; a `type: dir` source only works inside
       this repository,
     - `cargo-sources.json` copied next to the manifest (the vendor
       lock for the offline cargo build).
   - `LICENSE` handling: flatpak-builder auto-installs top-level
     license files per module; verify the built app contains
     `share/licenses/io.github.mrproject72.WirEQ/` and add an explicit
     install step if it does not.
4. Commit (your own commit message), push to your fork.
5. Open the PR **against the `new-pr` base branch** (never master),
   titled `Add io.github.mrproject72.WirEQ`. Write the description
   yourself: what the app does, that it is the Rust rewrite of
   bhack/mini-eq with AI-assisted development (disclosed per policy),
   and the two permission justifications in your own words:
   - `flatpak-spawn --host`: the pavucontrol launcher button opens the
     host's pavucontrol (per-stream volume/app routing the EQ must not
     duplicate); the button is insensitive unless a probe finds it.
   - `--filesystem=xdg-config/wireq`: presets, per-device curve memory
     and the AutoEq cache live in the host `~/.config/wireq` and are
     shared between the flatpak and the deb/rpm installs; the
     sandbox-private path would fork the user's presets.
6. Comment `bot, build` on the PR when you want a test build. Fix
   failures, answer reviewer questions. No need to close/reopen the PR
   for changes.
7. After approval the reviewers create `flathub/io.github.mrproject72.WirEQ`
   and invite you (2FA must be enabled; accept within a week).
   Maintenance afterwards: bump the git tag in your repo's manifest and
   push — the bot builds it.

## Optional before submission

- `flathub.json` only if you must restrict architectures (default is
  x86_64 + aarch64; the manifest's cargo build is arch-agnostic).
- Retake screenshots any time — the current three
  (`screenshot/wireq-main|output|presets.png`) are referenced by the
  metainfo and pass validation.
