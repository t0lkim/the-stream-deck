# TheStreamDeck — Unified Streaming Audio Appliance

**Version:** 0.2.0 (Design Spec)  
**Author:** Mike & Maia  
**Date:** 2026-04-04  
**Status:** Draft  

---

## 1. Overview

A Raspberry Pi 4-based appliance that consolidates BBC 6 Music, Tidal HiFi Plus, Plexamp, and internet radio into a single device with a 13.3" touchscreen, physical source-selection buttons, real-time VU meter visualisation, and S/PDIF digital output to a NAD C 3xx integrated amplifier.

One device. One amp input. Instant switching. Album art on screen.

---

## 2. Hardware Architecture

### 2.1 Core Components

| Component | Selection | Rationale |
|-----------|-----------|-----------|
| **SBC** | Raspberry Pi 4 Model B (4GB) | Already owned. Quad A72 @ 1.5GHz, VideoCore VI GPU. |
| **Transport HAT** | HiFiBerry Digi2 Pro | S/PDIF output (optical + coax), dual oscillator for low jitter, 192kHz/24-bit. NOT a DAC — pure digital transport. |
| **Display** | Waveshare 13.3" HDMI LCD (H) V2 with Case | 1920x1080 IPS, HDMI + USB touch (no GPIO conflict), 10-point capacitive, toughened glass, built-in speaker, tilt stand. ~$160-190. |
| **Amplifier** | NAD C 3xx (existing) | Optical digital input, Cirrus Logic internal DAC. Handles all D/A conversion. |
| **VU Meter DAC** | MCP4922 (DIP-16) | Dual 12-bit SPI DAC. Drives L/R analogue panel meters. Zero jitter, 4096 steps. SPI0 (GPIO 8/10/11) — no HiFiBerry conflict. ~$3-4. |
| **VU Panel Meters** | 2x 85mm analogue VU meters with backlight | Moving-coil needle meters, 200uA FSD. Driven by MCP4922 via series resistor (15k + 5k trim pot per channel). ~$8-40/pair depending on quality tier. |
| **Storage** | 64GB A2 microSD | Fast random I/O for OS + applications. |
| **Network** | Ethernet (wired) | Reliability for streaming. Wi-Fi available as fallback. |

### 2.2 Audio Signal Path

```
Streaming Source (Tidal / Plexamp / MPD)
    |
    v
PipeWire (audio server / multiplexer)
    |
    +---> HiFiBerry Digi2 Pro (I2S → S/PDIF framing)
    |         |
    |         +---> TOSLINK optical cable
    |                   |
    |                   v
    |              NAD C 3xx optical input
    |                   |
    |                   v
    |              NAD internal DAC (Cirrus Logic, D/A conversion)
    |                   |
    |                   v
    |              NAD amplifier stage → Speakers
    |
    +---> PipeWire monitor source (audio tap for visualisation)
              |
              v
         VU Meter / Spectrum Analyser (cpal + RustFFT → display)
```

**Why Digi HAT, not DAC HAT:**
- The Pi stays purely digital — zero analogue stages on the Pi side
- The NAD's Cirrus Logic DAC is purpose-built with dedicated power regulation and shielding — objectively superior to any ~$30 Pi HAT DAC
- Using a DAC HAT to feed the NAD's analogue input = unnecessary double conversion (digital → HAT DAC → analogue → NAD → amp)
- TOSLINK optical provides galvanic isolation, eliminating ground loop hum from the Pi's switching PSU
- Full 24-bit/192kHz support end-to-end

### 2.3 Physical Controls

Source switching buttons connected via GPIO (pins not used by Digi HAT — HDMI display uses no GPIO):

| Button | Function | GPIO Pin (suggested) |
|--------|----------|---------------------|
| 1 | BBC 6 Music | GPIO 17 |
| 2 | Tidal | GPIO 27 |
| 3 | Plexamp | GPIO 22 |
| 4 | Radio (cycle presets) | GPIO 23 |
| 5 | Play/Pause | GPIO 24 |
| 6 | Volume +/- (rotary encoder) | GPIO 5/6 |

The touchscreen also provides full source selection and transport controls.

### 2.4 GPIO Pin Allocation

| Pin Range | Used By |
|-----------|---------|
| GPIO 18, 19, 20, 21 | HiFiBerry Digi2 Pro (I2S: BCK, LRCK, DIN, DOUT) |
| GPIO 4 | HiFiBerry Digi2 Pro (clock select) |
| GPIO 8 (SPI0 CE0) | MCP4922 DAC chip select |
| GPIO 10 (SPI0 MOSI) | MCP4922 DAC data in |
| GPIO 11 (SPI0 SCLK) | MCP4922 DAC clock |
| GPIO 2-3 (I2C) | Available |
| GPIO 17, 22-24, 27 | Physical buttons |
| GPIO 5, 6 | Rotary encoder (volume) |

Display connects via HDMI (video) + USB (touch) — zero GPIO usage. No conflicts between display, Digi HAT, DAC, and buttons.

### 2.5 Analogue VU Meter Subsystem

Physical needle VU meters driven digitally from the audio analysis pipeline via SPI DAC.

**Signal chain:**
```
PipeWire monitor tap → RMS per channel (Rust)
    → VU ballistics filter (300ms IIR)
    → MCP4922 SPI write (12-bit, dual channel)
    → Series resistor (15k + 5k trim pot)
    → 85mm analogue panel meter (200uA FSD)
```

**Wiring:**
```
Pi 3.3V ──── MCP4922 VDD, VRefA, VRefB, SHDN
Pi GND  ──── MCP4922 VSS, LDAC
Pi GPIO8 ─── MCP4922 CS
Pi GPIO10 ── MCP4922 SDI
Pi GPIO11 ── MCP4922 SCK

MCP4922 VOutA ──[15k + 5k trim]── VU Left+  ── VU Left-  ── GND
MCP4922 VOutB ──[15k + 5k trim]── VU Right+ ── VU Right- ── GND
```

**Why not PWM?** Hardware PWM on GPIO 18 conflicts with HiFiBerry PCM_CLK. Software PWM produces visible needle jitter (~13 usable steps). The MCP4922 gives 4096 steps with zero jitter — proven superior in head-to-head testing (Hackaday.io reference project).

**Power:** Both meters draw ~0.4mA total from the Pi's 3.3V rail. Negligible.

**Ballistics:** Implement a VU-standard IIR filter in software (300ms time constant, 1-1.5% overshoot). Cheap meters need software damping due to poor mechanical damping. Quality meters (Flash Star, Hoyt) can rely on their own mechanical inertia with only a gentle 50ms software smoothing pass.

**Meter quality tiers:**

| Tier | Source | Price/pair | Damping | Notes |
|------|--------|-----------|---------|-------|
| Budget | AliExpress "85mm VU meter" | $8-15 | Poor — needs software compensation | Adequate for decorative/indicative use |
| Mid-range | Flash Star / Sifam | $30-40 each | Good — proper VU behaviour | Recommended for this build |
| Professional | Hoyt AL19R/AL20 | $50-150 each | Excellent — ANSI-standard | Buy-once quality |

---

## 3. Software Architecture

### 3.1 OS Layer

| Component | Choice | Notes |
|-----------|--------|-------|
| **OS** | Raspberry Pi OS Lite 64-bit (Bookworm) | Minimal, ARM64, PipeWire default |
| **Audio Server** | PipeWire | Default on Bookworm. Multiplexes all sources. Monitor sources for visualisation. |
| **Container Runtime** | Podman (rootless) | For Tidal Connect. No Docker daemon. |
| **Init** | systemd | User services for Plexamp, system services for MPD |

### 3.2 Audio Sources

#### BBC 6 Music (and other BBC/internet radio)
- **Player:** MPD (Music Player Daemon)
- **Protocol:** HLS (AAC-LC)
- **Quality:** 320kbps AAC (UK), 96kbps (worldwide)
- **Stream URL:** `http://lstn.lv/bbc.m3u8?station=bbc_6music&bitrate=320000`
- **Config:** MPD playlist entries for each station
- **Switching:** MPD client commands (`mpc play`, `mpc stop`)
- **Metadata:** MPD provides ICY metadata (artist/title) from stream

#### Tidal HiFi Plus
- **Player:** GioF71/tidal-connect (Podman container)
- **Protocol:** Tidal Connect (proprietary, runs inside container)
- **Quality:** Lossless FLAC (16/44.1 or 24/96 depending on source)
- **Auth:** OAuth via Tidal mobile app (cast to this device)
- **Control model:** Cast-based — control from Tidal app on phone/tablet, Pi is the renderer
- **Metadata API:** OpenTidal (TypeScript) or rstidal (Rust) for album art, track info
- **Container config:**
  ```bash
  podman run -d --name tidal-connect \
    --device /dev/snd \
    -e FRIENDLY_NAME="StreamDeck" \
    -e FORCE_PLAYER_BACKEND="PipeWire" \
    giof71/tidal-connect:latest
  ```

#### Plexamp (Local Plex Library)
- **Player:** Plexamp headless (Node.js 20)
- **Requirement:** Plex Pass (mandatory)
- **Setup:** systemd user service via odinb/bash-plexamp-installer
- **Control model:** Cast-based — control from any Plexamp client on network
- **Web UI:** `http://localhost:32500` — can display in Chromium kiosk on touchscreen
- **Quality:** Depends on Plex library encoding (FLAC preferred)
- **Metadata:** Plex API provides full album art, artist info, lyrics

#### Internet Radio (Generic)
- **Player:** MPD
- **Protocol:** HLS, Shoutcast/Icecast (varies per station)
- **Config:** MPD playlist with station URLs
- **Presets:** Configurable list of favourite stations

### 3.3 Source Switching Architecture

```
                    ┌─────────────────────┐
                    │   Source Controller  │
                    │   (Rust daemon)      │
                    └─────────┬───────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              v               v               v
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │   MPD    │   │  Tidal   │   │ Plexamp  │
        │ (radio)  │   │ Connect  │   │(headless)│
        └────┬─────┘   └────┬─────┘   └────┬─��───┘
             │               │               │
             └───────────────┼───────────────┘
                             │
                             v
                    ┌─────────────────────┐
                    │     PipeWire        │
                    │  (session manager)  │
                    └─────────┬───────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    v                   v
           ┌──────────────┐   ┌──────────────┐
           │ Digi2 Pro    │   │ Monitor Src  │
           │ (S/PDIF out) │   │ (VU meter)   │
           └──────────────┘   └──────────────┘
```

**Source Controller** is a custom Rust daemon that:
1. Listens for GPIO button presses and touchscreen source-select events
2. Mutes/stops the current source (e.g., `mpc stop` for MPD)
3. Activates the new source
4. Updates the UI to reflect active source
5. PipeWire handles audio routing — no manual ALSA device management

**Switching latency target:** < 3 seconds from button press to audio output.

### 3.4 Display & UI

#### Layout (1920x1080, landscape — Waveshare 13.3")

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  ┌───────────────────────────┐  ┌──────────────────────────────────────┐ │
│  │                           │  │                                      │ │
│  │      Album Artwork        │  │        VU Meter / Spectrum           │ │
│  │       (600x600)           │  │         Visualisation                │ │
│  │                           │  │          (L + R)                     │ │
│  │                           │  │                                      │ │
│  │                           │  │                                      │ │
│  └───────────────────────────┘  └──────────────────────────────────────┘ │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │  Track: Song Title                                    Artist Name   │ │
│  │  Album: Album Title                                   FLAC 24/96   │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    ┌────┐ ┌────────────┐ ┌────┐  │
│  │ BBC  │ │ TDAL │ │ PLEX │ │ RDIO │    │ ◀◀ │ │   ▶ / ⏸   │ │ ▶▶ │  │
│  └──────┘ └──────┘ └──────┘ └──────┘    └────┘ └────────────┘ └────┘  │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

- **Album art:** Prominent, high-resolution (600x600px display area — 1080p gives room to breathe)
- **VU meter:** Real-time L/R needle-style or bar-style, user-switchable to spectrum analyser. Full HD canvas for detailed needle rendering.
- **Now playing:** Track title, artist, album, format/bitrate — larger text at 1080p
- **Source buttons:** Touch targets for source switching (mirror physical buttons)
- **Transport:** Play/pause, skip (where applicable)

#### UI Technology Options

| Option | Stack | Pros | Cons |
|--------|-------|------|------|
| **A. Rust + wgpu** | Native framebuffer, GPU-accelerated | Fastest, lowest overhead, single language with audio engine | Most dev effort, custom widget toolkit |
| **B. Rust + Slint** | Native UI toolkit for embedded | Declarative UI, GPU-accelerated, designed for embedded touchscreens | Newer ecosystem, licensing (free for open source) |
| **C. TypeScript + Electron** | Chromium-based | Rapid development, web tech, rich ecosystem | Higher memory/CPU overhead (~200MB RAM) |
| **D. Chromium kiosk + web app** | Lightweight web app | Simplest for Plexamp web UI integration | Limited VU meter performance |

**Recommended: Option B (Rust + Slint)** — Purpose-built for embedded touchscreen UIs, GPU-accelerated, works with framebuffer (no X11/Wayland needed), and keeps the entire codebase in Rust. The VU meter can render at 60fps via Slint's OpenGL ES backend on VideoCore VI.

### 3.5 VU Meter & Visualisation

**Audio Analysis Pipeline:**
```
PipeWire monitor source
    │
    v
cpal (Rust) — captures audio frames from monitor
    │
    v
RustFFT (1024-point, NEON-accelerated on aarch64)
    │
    ├── RMS per channel → VU ballistics IIR filter (300ms)
    │       │
    │       ├── MCP4922 SPI write → Physical analogue VU meters (L/R)
    │       │
    │       └── Slint VU widget → On-screen VU visualisation
    │
    └── FFT bins → Spectrum analyser (optional view)
    │
    v
Slint UI renderer → HDMI Display @ 60fps
```

**Performance budget (Pi 4):**

| Component | CPU | Latency |
|-----------|-----|---------|
| PipeWire monitor tap | <0.5% | ~5ms |
| 1024-point FFT (RustFFT + NEON) | <0.5% | ~23ms (window) |
| VU ballistics + MCP4922 SPI write | <0.1% | ~1ms |
| 1080p 60fps render (OpenGL ES) | ~8-15% | <16ms |
| **Total** | **~9-16%** | **~44ms** |

84-91% CPU remains for audio playback. 44ms visual latency is imperceptible for VU meters. 1080p rendering costs slightly more GPU than 720p but Pi 4's VideoCore VI handles it comfortably.

**VU Meter Modes (user-switchable via touch):**
1. **Classic VU:** Dual analogue needle meters (L/R), ballistic response matching real VU meter standards (300ms integration, 1% overshoot)
2. **LED Bar:** Horizontal or vertical LED-style bars (L/R), peak hold with decay
3. **Spectrum:** Frequency spectrum analyser, 32 or 64 bands

### 3.6 Volume Control

| Strategy | Method | Notes |
|----------|--------|-------|
| **Software (PipeWire)** | PipeWire volume control via `wpctl` | Simple, works for all sources. Minor quality loss at very low volumes (bit depth reduction). |
| **Pass-through** | No Pi volume control, use NAD remote | Cleanest signal path. Requires NAD remote or amp panel. |

**Recommended:** Software volume via PipeWire as default, with an option to disable for pass-through mode. The rotary encoder on GPIO 5/6 controls PipeWire master volume.

---

## 4. Build Phases

### Phase 1: Bare Metal Audio (Week 1-2)
- Flash Pi OS Lite 64-bit
- Install and configure HiFiBerry Digi2 Pro
- Verify S/PDIF output to NAD via TOSLINK
- Configure PipeWire
- Install MPD, add BBC 6 Music stream, verify playback
- Install Plexamp headless, verify playback
- Install Tidal Connect (Podman), verify playback
- Verify source switching works (manual CLI commands)

### Phase 2: Source Controller Daemon (Week 3)
- Rust daemon: GPIO button listener
- Source state machine (stop current, start new)
- PipeWire integration for audio routing
- CLI interface for testing
- Verify < 3 second switching

### Phase 3: Display & UI (Week 4-5)
- Connect Waveshare 13.3" HDMI LCD (H) V2 (HDMI + USB touch)
- Slint UI: source selection, now-playing display (1920x1080)
- Album art retrieval (Plex API, OpenTidal API, MPD ICY metadata)
- Touch input handling (source buttons, transport controls)

### Phase 4: VU Meter (Week 6)
- cpal audio capture from PipeWire monitor
- RustFFT analysis pipeline
- Slint VU meter widget (classic needle + LED bar modes)
- Spectrum analyser mode
- 60fps render validation

### Phase 5: Enclosure & Polish (Week 7-8)
- Physical enclosure design (Pi mounts behind 13.3" display via VESA/bracket, button panel alongside)
- Button panel integration
- Boot-to-UI (systemd auto-start, no desktop environment)
- Error handling (network loss, service crashes, auto-restart)
- Final integration testing

---

## 5. Bill of Materials

| Item | Estimated Cost | Status |
|------|---------------|--------|
| Raspberry Pi 4 Model B (4GB) | -- | Already owned |
| HiFiBerry Digi2 Pro | ~$35 | To purchase |
| Waveshare 13.3" HDMI LCD (H) V2 with Case | ~$175 | To purchase |
| TOSLINK optical cable (1-2m) | ~$8 | To purchase |
| 64GB A2 microSD | ~$10 | Likely owned |
| Tactile buttons (6x) + rotary encoder | ~$10 | To purchase |
| Stacking GPIO header (for button access) | ~$3 | To purchase |
| MCP4922 dual SPI DAC (DIP-16) | ~$4 | To purchase |
| 2x 85mm analogue VU panel meters + backlight | ~$15-40 | To purchase (tier dependent) |
| 2x 15k resistor + 2x 5k trim pot + 100nF cap | ~$2 | To purchase |
| Enclosure / mounting bracket | ~$15-30 | To design/purchase |
| **Total (excl. owned items)** | **~$275-335** | |

---

## 6. Risks & Mitigations

| Risk | Severity | Mitigation |
|------|----------|------------|
| Tidal kills unofficial Connect protocol | High | Fall back to Tidal via phone cast (UPnP/DLNA). Monitor GioF71 repo for updates. |
| Plexamp headless drops ARM64 support | Low | Plex is invested in Pi ecosystem. Worst case: MPD + Plex DLNA. |
| BBC changes stream URLs/format | Low | Community-maintained URL lists. MPD handles format changes. |
| PipeWire source switching causes audio pops | Medium | Implement soft mute (fade out 50ms → switch → fade in 50ms) in source controller. |
| Slint licensing changes | Low | Open source use is free. Fallback: wgpu + custom widgets. |
| Pi 4 thermal throttling during sustained VU + playback | Low | Passive heatsink case. VU analysis is <16% CPU — well within thermal budget. |
| Cheap VU meter overshoot/ringing | Medium | Software VU ballistics filter (300ms IIR) compensates for poor mechanical damping. Use 400-500ms tau for budget meters. |

---

## 7. Future Enhancements (Out of Scope)

- AirPlay 2 support (shairport-sync)
- Spotify Connect
- Bluetooth receiver mode
- Room correction DSP (CamillaDSP)
- Remote web UI for control from phone/laptop
- Multiple DAC/amp output support
- IR remote control integration
