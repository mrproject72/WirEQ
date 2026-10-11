# WirEQ

Compact PipeWire system-wide parametric equalizer for Linux desktops.

**WirEQ** is a native Rust/GTK4 parametric EQ that routes desktop playback
through a PipeWire filter-chain. It grew out of a from-scratch rewrite of
[mini-eq](https://github.com/bhack/mini-eq) (Python/GTK/PipeWire), built to
eliminate the ~30% CPU cost of the Python GIL under real-time audio
processing — same core feature set, native speed, plus multi-device
per-output chains, a fully editable response graph, and a sandbox-proof
flatpak.

**Repository:** <https://github.com/mrproject72/WirEQ>

## Features

- System-wide parametric EQ for PipeWire desktop playback
- GTK4/Libadwaita interface with a compact 10-band fader workflow (up to 32 bands)
- PipeWire routing and default-output tracking
- PipeWire filter-chain DSP using builtin biquad filters
- Optional spectrum analyzer and LUFS loudness readout
- Auto preset links that follow the detected PipeWire port
- Background mode keeps the EQ active after closing the window
- Optional GNOME Shell extension for quick panel access
- Search and import headphone correction presets from AutoEq
- Import Equalizer APO-style text presets

## Why native?

The original Python implementation used the Python GIL for real-time audio
processing, costing ~30% of a CPU core. WirEQ compiles to native code and
processes audio at native speed with minimal overhead: ~0.4% of a core at
idle with the spectrum running.

## Tech Stack

- **Rust 1.98.1** — compiled native binary, no GIL
- **GTK4 + Libadwaita** — modern GNOME UI
- **PipeWire 0.10** — audio routing and filter-chain DSP
- **SPA biquad filters** — native DSP processing

## Install

```bash
# Build from source
cargo build --release
./target/release/wireq

# Or install the flatpak bundle from a CI run
flatpak install --bundle wireq.flatpak
```

## Development

See [AGENTS.md](AGENTS.md) for project structure, architecture, and development workflow.

## Origins

The original project: [bhack/mini-eq](https://github.com/bhack/mini-eq) by
[bhack](https://github.com/bhack). WirEQ began as an independent rewrite of
it and is **not** affiliated with or endorsed by the upstream author. The
biquad coefficients are asserted against upstream reference values in the
test suite so the audio processing matches.

See `docs/BUGS.md` for the deliberate divergences from upstream
(Auto-Safe, Smooth band editing, the LED indicator and the widened
headroom budget are all additions with no upstream equivalent).

## License

GPL-3.0-or-later
