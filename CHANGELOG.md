# Changelog

All notable changes to **CardputerZero DAW Zero** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html):
- **MAJOR (`X.0.0`)**: Breaking session format or `DAWZERO_PLUGIN_API_VERSION` ABI changes.
- **MINOR (`0.X.0`)**: Backwards-compatible new features, UI tabs/workflows, or controller enhancements.
- **PATCH (`0.0.X`)**: Bug fixes, DSP/stability improvements, and documentation updates.

---

## [0.4.0] - 2026-11-25

### Added
- **Hosted Instrument Sequencer Auto-BPM Sync (`BPM|<bpm>`)**:
  - DAW Zero now pushes project BPM over `DAWZERO_CTRL_SOCK` on instrument launch (`READY`) and whenever BPM changes, allowing sequencer-equipped instruments like `DXZero` to auto-align to the DAW project tempo (`Sync DAW BPM`, ON by default).
- **Mini Waveform & MIDI Activity Previews + Per-Subtrack `M`/`S` in `TRK → CLIP`**:
  - Each `C1–C4` clip bar now displays centered mini waveform amplitude bars (Audio) or note ticks (MIDI) over a baseline so audio start/peaks are immediately visible when moving, trimming, or splitting.
  - Added per-subtrack Mute (`M`) and Solo (`S`) badges and real-time audio/MIDI playback support inside `TRK → CLIP`.
- **Vertical Amplitude Zoom (`Alt+U` / `Alt+I` = `[` / `]`) & Taller `WAVE` View**:
  - Added 5 vertical amplitude gain levels (`1x`, `2x`, `4x`, `8x`, `16x`) across `TRK → WAVE` (expanded to `86 px` tall) and `TRK → CLIP` mini waveforms.
- **Footer-Free Workspace & Context-Sensitive Help Overlay (`Alt+M` `?` / `Fn+H` / `H`)**:
  - Removed the static bottom tips bar across all views (reclaiming 17 px of vertical space; transient toasts and delete confirmations still appear briefly when needed).
  - Press and hold (> 250 ms) or tap **`Alt+M` (`?`)**, **`Fn+H` (`HELP`)**, or **`H`** to view context-sensitive commands for the active tab/subtab.
- **Global `K` Key Metronome Click Toggle**:
  - Press **`K`** on any tab to toggle the metronome click (`ON` / `OFF`) with toast confirmation.

### Fixed
- **Click-Free `INST BUS` Adaptive Micro-Resampling**:
  - Replaced hard ring-buffer drops with gentle 1–4 frame linear-interpolated clock-drift absorption and smooth underrun decay, eliminating cracks when playing sampled `DXZero` drums.
- **Multi-Track `Bounce To` (`AUD 1–6`) & `MID 1/2` Routing**:
  - Selecting `Bounce To` (`AUD 1–6`) or launching a hosted instrument now automatically routes `INST BUS`, arms/monitors the target audio track, disarms the previous `INST BUS` track, wires both `MID 1` and `MID 2`, and isolates `bounce_midi_trk_` during Live Bounce.

---

## [0.3.0] - 2026-11-25

### Added
- **Universal Hosted Instrument Control Protocol (`DAWZERO_CTRL_SOCK`) & `MID → 2:INST` Subtab**:
  - Any hosted CardputerZero digital instrument can now export `rotary`, `numeric`, `toggle`, and `trigger` controls over a lightweight Unix domain socket (`HELLO`, `DEF`, `VAL`, `SET`, `TRIG`, `GET`).
  - Split the **`MID`** tab into **`1:ROUT`** (routing & Live Bounce) and **`2:INST`** (dynamic instrument controls, switchable via `V`, `Q`/`E`, or V61 Knob 4, with smooth V61 Knob 3 sweeping).
  - **`DXZero` exported controls:** `Bank`, `Instrument` (Patch), `Seq Track`, `Seq Pattern`, `Seq Play/Stop`, `Seq Rewind`, `FX Wet Mix`, and `Filter Cutoff`.
  - **`PianoZero` exported controls:** `Sound Bank`, `Touch Curve`, `Reverb FX`, `Reverb Send`, `Tone Tilt EQ`, `Pedal/Mech` toggle, and `Pedal Level`.

---

## [0.2.0] - 2026-11-24

### Added
- **Hosted Headless Digital Instruments (`PianoZero` / `DXZero`) via Virtual `INST BUS`**:
  - Launch `PianoZero` or `DXZero` headlessly from **`MID → Host Inst`** with automatic PipeWire null-sink (`dawzero_inst_bus`) routing and ALSA Sequencer MIDI wiring.
  - Supports MIDI-only playback directly through `INST BUS` (checking both `MID 1` and `MID 2`), Audio-only live recording from the hosted instrument without requiring a MIDI track, and simultaneous MIDI + Audio Live Bounce.
- **4 Stacked Subtracks (`C1–C4`) for Both Audio & MIDI (`TRK → CLIP`)**:
  - Extended 4-slot stacked subtrack recording and editing to `MID 1–2` alongside `AUD 1–6`.
  - Added playhead-driven **`SPLIT`** (move playhead to exact cut point with Knob 3 or `Z`/`C`, press `ENTER` to slice), **`JOIN`** into `C1`, non-destructive **`MOVE` / `TRM-L` / `TRM-R`**, and fast double-tap **`D`** delete hotkey for tracks and subtracks.
- **16-Step Undo / Redo (`Ctrl+Z` / `Ctrl+Y` / `Ctrl+Shift+Z`)**:
  - Lightweight metadata-snapshot undo stack with 800 ms drag coalescing and zero PCM memory duplication.
- **Timeline Zoom (`U`/`I`) & Dual-Layer Alesis V61 Knobs**:
  - Added 5 zoom levels (`ALL`, `32B`, `16B`, `8B`, `4B`) with numbered bar ruler and bar/beat gridlines across `TIME`, `WAVE`, and `CLIP`.
  - Added dual-layer Alesis V61 rotary mode: **Btn 1 ON (`NAV`)** for Tab/Row/Value/Subtab navigation, **Btn 1 OFF (`ZOOM / MIX`)** for Zoom (`K1`), Window Pan (`K2`), Track Vol / FX P1 (`K3`), and Master Vol (`K4`).
- **Portable Session Persistence & Real-Time Health Monitor (`SES → 1:PROJ | 2:MON`)**:
  - Auto-save after every recording and on clean exit (`Hold ESC`), auto-restore on startup, relative filenames in `session.txt`, custom in-app session renaming, and live DSP CPU / `XRUN` / `INST BUS` underrun / RAM / CPU telemetry.
- **Zero-Series Neon Launcher Icon (`share/images/daw-zero.png`)**:
  - Replaced placeholder icon with a 100 × 100 RGBA Electric Blue & Neon Amber-Orange rounded neon icon matching `PianoZero` and `DXZero`, plus a 256 × 256 README header asset (`docs/daw-zero-icon.png`).

### Fixed
- **Crack-Free Real-Time Recording & `INST BUS` Capture**:
  - Replaced buffered `stdio` capture with unbuffered raw `read()` on `parec --latency-msec=5 --process-time-msec=5` feeding a 2-period jitter-buffered `StereoRingBuffer`, added dedicated `midi_mtx_` (removing `try_to_lock` silence drops), forced 48 kHz in hosted `PianoZero`, and installed `/etc/security/limits.d/99-audio-rt.conf` + `SCHED_FIFO` / `nice -11`.
- **Zero-Bleed Subtrack Overdubbing**:
  - Isolated live capture into dedicated `rec_l_` / `rec_r_` buffers so recording `C2–C4` never bleeds existing subtracks, while keeping track FX non-destructive on playback.
- **Full-Song MIDI Piano-Roll Rendering**:
  - Replaced the 120-note hard cutoff with even stride-sampling across all visible note-on events so recordings past Bar 5 draw completely.
- **Non-Blocking ALSA Duplex & Device Switching**:
  - Switched ALSA playback/capture to `SND_PCM_NONBLOCK` + 40 ms `snd_pcm_wait` and ensured `stop()` runs before `wpctl set-default`, preventing stalls or deadlocks when switching audio devices.

---

## [0.1.0] - 2026-11-23

### Added
- **Initial Release of CardputerZero DAW Zero**:
  - 8-track multitrack Audio (`AUD 1–6`) & MIDI (`MID 1–2`) engine at 48 kHz full-duplex (PipeWire + ALSA PCM + ALSA Sequencer).
  - Stable C ABI Modular DSP Plugin Host (`DAWZERO_PLUGIN_API_VERSION = 1`) with 4 built-in effects (`SVF Filter`, `Soft Overdrive`, `Stereo Delay`, `Gain & Utility`) and `dlopen` external `.so` plugin support (`plugins/example_lpf/example_lpf.cpp`).
  - 7-tab 320 × 170 LVGL 9.5 Neon UI (`TRK | MIX | FX | REC | MID | AUD | SES`) with dynamic framebuffer sizing for HDMI readiness.
  - CardputerZero `evdev` keyboard mapping (`F/X/Z/C`, `TAB`, `1–7`, `SPACE`, `R`, `A/M/S`, `L`, `Hold ESC`) and Alesis V61 USB-MIDI controller integration with LED sync (`CC 48–51`).
  - Ardour-compatible WAV + SMF Type 0 `.mid` + `mixdown.wav` session storage under `~/DAWZero-Sessions/`.
  - Automated native build & deploy script (`deploy_cardputerzero.sh`) reusing the CardputerZero LVGL 9.5 build cache.
