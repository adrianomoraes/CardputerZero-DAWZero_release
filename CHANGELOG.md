# Changelog

All notable changes to **CardputerZero DAW Zero** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html):
- **MAJOR (`X.0.0`)**: Breaking session format or `DAWZERO_PLUGIN_API_VERSION` ABI changes.
- **MINOR (`0.X.0`)**: Backwards-compatible new features, UI tabs/workflows, or controller enhancements.
- **PATCH (`0.0.X`)**: Bug fixes, DSP/stability improvements, and documentation updates.

---

## [0.5.0] - 2026-11-27

### Added
- **Multi-Format Audio Export UI (`SES → 3:EXP`) & Upgraded Offline Mixdown Engine**:
  - Added dedicated **`3:EXP`** subtab under **`SES`** (`1:PROJ | 2:MON | 3:EXP`) with 11 configurable rows: **Format** (`WAV`, `FLAC`, `MP3`, `OGG`, `OPUS`), **Encoding** (`16/24/32-bit Float` WAV, `16/24-bit Lossless` FLAC, `128–320 kbps` MP3/OGG/OPUS), **Sample Rate** (`48 kHz` / `44.1 kHz`), **Channels** (`Stereo` / `Mono`), **Range** (`Full Song` or `Loop [start–end]`), **Loop Repeats** (`1x`–`8x`), **FX Tail** (`0s` / `2s` / `4s`), **Apply FX** (fresh offline Track + Master FX clones), **Normalize** (`OFF` / `-1.0 dB` / `-0.3 dB` two-pass zero-RAM peak scan), and **Export Mode** (`Mixdown Only` or `Mix + Stems`).
  - Native C++ RIFF/WAVE writer for native-rate WAV (`16/24/32f`) and zero-temp-file `f32le` piping into `ffmpeg` for compressed/resampled formats.
- **Dual Hosted Digital Instruments (`MID → 1:ROUT | 2:MID1 | 3:MID2`)**:
  - Each MIDI track (`MID 1` and `MID 2`) now hosts its own independent headless instrument (`PianoZero` or `DXZero`) with its own `DAWZERO_CTRL_SOCK` control socket, BPM sync, and strict ALSA Sequencer client ID binding (supporting **`PianoZero + DXZero`** or **`2× DXZero`**, with a RAM guardrail preventing `2× PianoZero` on 414 MB RAM).
  - Dedicated **`2:MID1`** and **`3:MID2`** subtabs put `Host Inst` launch/stop right on Row 0 followed directly by all live exported parameters (`Bank`, `Instrument`, `FX Wet Mix`, `Filter Cutoff`, etc.).
- **Dual Loop Strategies (Per-Session Selectable in `SES → 1:PROJ`)**:
  - **`Regions (A/B/C)` (Default)**: 3 switchable loop regions (`A`, `B`, `C`) with independent `[start–end]` markers (`Alt+U`/`Alt+I` on `TRK → TIME`) and quantized bar-boundary section switching via **`N`** (`LP A>B`).
  - **`Per-Track Poly`**: Independent per-track polyrhythmic loop lengths (`OFF, 1B, 2B, 3B, 4B, 6B, 8B`) on every Audio & MIDI track, wrapping organically during playback and `SES → 3:EXP` offline export.
- **Far-Right `MST` Master Volume Fader & Live 30 FPS VU Meters on `MIX → 1:BOARD`**:
  - Compacted channel strips to fit all **9 strips (`A1–A6`, `M1–M2`, `MST`)** with continuous live peak VU meters (`teal → amber → red`) and direct Master Volume + Master FX access.
- **Non-Destructive MIDI `Quantize` Slider on `MIX → 2:STRIP`**:
  - Per-MIDI-track real-time quantization slider (`OFF`, `1/32`, `1/16`, `1/8T`, `1/8`, `1/4`, `1/2`, saved in `session.txt` as `t<i>_mquant`), adjustable with `Z`/`C`, `ENTER`, or V61 Knob 3.
- **Zoom-Proportional Subtrack Tool Precision & Double-Tap Play/Stop Jump**:
  - `SPLIT`, `MOVE`, `TRM-L`, `TRM-R` (and `WAVE` scrubbing) step proportionally with horizontal zoom (`Z:ALL = 1 beat` down to `Z:4B = 1/16 beat`).
  - Double-tapping **`SPACE`** or **V61 Button 2** within `450 ms` stops and jumps the playhead to the start of the active subtrack / zoomed window (or Bar `1.1`).
- **Full Session State Persistence & `Demo_NeonPulse` Showcase Session**:
  - Sessions now persist and auto-restore Track & Master FX chains, per-subtrack Mute/Solo, Loop Mode & `A/B/C` / Poly loop lengths, MIDI Quantize, and running hosted instruments (`inst0_type`/`inst1_type`) with their saved patches (`inst0_init`/`inst1_init`), auto-saving on session switch and exit.
  - Added [`examples/Demo_NeonPulse/`](examples/Demo_NeonPulse/) and [`scripts/generate_demo_session.py`](scripts/generate_demo_session.py): an 8-bar, 120 BPM showcase session with all 6 Audio tracks, 2 MIDI tracks driving 2× `DXZero`, Track & Master FX, `A/B/C` regions, and polyrhythmic lengths.

### Changed
- **MIDI Channel Volume & Pan Control**: `MID 1` / `MID 2` mixer Volume & Pan now send MIDI **CC 7 / CC 10** and scale Note-On velocities independently per channel.
- **Wider `AUD` Device Labels**: Expanded `In Dev` / `Out Dev` text width to `31` characters so full USB/built-in device names display clearly.

### Fixed
- **Zero Hanging MIDI Notes on Mute / Solo**: Added `send_midi_all_notes_off()` (CC 64=0, CC 123=0, CC 120=0) on track Mute, Solo silencing, and subtrack Mute/Solo, and always pass pending Note-Off events through during playback.
- **Fixed UI Freeze When Switching `AUD` Input/Output Devices**: Made device/buffer switching non-blocking via async audio-thread restart and `snd_pcm_drop()` before `snd_pcm_close()`.
- **Fixed Double `shutdown()` Wiping Hosted Instruments on Exit**: Guarded `DawEngine::shutdown()` with `shutdown_done_` so `~DawEngine()` never overwrites `session.txt` after instruments stop, and initialized ALSA MIDI before session restore so hosted instruments bind immediately on reopen.

---

## [0.4.2] - 2026-11-27

### Added
- **3-Subtab `MIX` Console (`1:BOARD` | `2:STRIP` | `3:FX`)**:
  - Consolidated `FX` into `MIX` as subtab **`3:FX`**, streamlining the top ribbon to 6 tabs (`TRK | MIX | REC | MID | AUD | SES`).
  - Added **`MIX → 1:BOARD`** 8-channel vertical mixer console displaying all channels (`A1–A6`, `M1–M2`) side-by-side with top rotary Pan dials, vertical Volume sliders + live peak bars, and `M` / `S` / `FX` badges (`Z`/`C` or Knob 2 selects channel, `F`/`X` or Knob 3 adjusts volume, `U`/`I` adjusts pan, `M`/`S`/`A` toggles, `ENTER` opens `3:FX`).
- **`TRK → TIME` Loop Markers & Multi-Take Loop Recording (`C1→C4`)**:
  - Set Loop Start (`Alt+U` `[`) and Loop End (`Alt+I` `]`) markers on `TRK → TIME` and toggle loop playback/recording with **`L`**.
  - Recording with Loop enabled automatically rotates into the next empty subtrack (`C1→C4`) on every loop wrap, notifying `"No subtrack available (4/4 full)"` when all 4 slots are filled.
- **V61 Knob 4 Seamless Subtab + Tool Sweep on `TRK`**:
  - Turning Alesis V61 Knob 4 in `NAV` mode now sweeps continuously through `CTRL → TIME → WAVE → CLIP` and all 7 edit tools (`SPLIT / MOVE / TRM-L / TRM-R / DUP / JOIN / DEL`) while keeping `T` working as before.

### Changed
- **Selective `JOIN` Tool (`TRK → CLIP`)**:
  - `JOIN` now merges only **soloed** subtracks (if any subtrack on the track is soloed) or only **non-muted** subtracks (otherwise), leaving excluded subtracks untouched.
- **Clamped Linear Track Selection on `MIX`**:
  - Track selection on `MIX` (`1:BOARD`, `2:STRIP`, `3:FX`) now moves linearly from **`AUD 1` to `MID 2`** without wrapping around, for both keyboard (`Z`/`C`) and V61 **Knob 3**.

### Fixed
- **Loop Recording 4th Subtrack (`C4`) Handling**:
  - Fixed multi-take loop wrap checking to evaluate each armed track independently so a full 4/4 track (e.g. `MID 1`) no longer aborts recording on another armed track that still has `C4` open.

---

## [0.4.1] - 2026-11-26

### Added
- **Subtrack Clipboard (`Ctrl+C` / `Ctrl+V`) & Duplicate Tool (`DUP`)**:
  - Copy and paste Audio or MIDI subtracks (`C1–C4`) at the playhead across slots or same-type tracks with full 16-step `Ctrl+Z` / `Ctrl+Y` Undo & Redo.
  - Added **`DUP`** tool in `TRK → CLIP` to duplicate the selected subtrack directly into the next free slot right after it.

### Changed
- **`SPLIT` Promoted to First Tool in `TRK → CLIP`**:
  - Tool cycle order is now `SPLIT / MOVE / TRM-L / TRM-R / DUP / JOIN / DEL`.
- **Two-Column Yellow/White Help Overlay (`Alt+M` `?` / `Fn+H` / `H`)**:
  - Upgraded the context-sensitive Help overlay to an aligned 2-column layout with commands highlighted in yellow (`#FFE066`) and descriptions in white (`#F8FAFC`).

### Fixed
- **Edge-to-Edge Waveform Bin Scaling Under Zoom (`TRK → CLIP` & `TRK → WAVE`)**:
  - Replaced floored integer bin widths with proportional `(b * inner_w) / nbins` mapping (up to 80 bins in `CLIP`, 64 bins in `WAVE`), so inner amplitude waveforms always align pixel-exact with clip borders at every zoom level.

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
