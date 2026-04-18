# Changelog

All notable changes to TheStreamDeck project.

## [0.4.0] - 2026-04-12

### Added
- Bandcamp as a streaming source (discovery/browse only — purchased FLACs remain in Plexamp path)
- NTS Radio (ntslive.co.uk) — two 24/7 live channels, web player or MPD fallback
- Worldwide.fm — curated DJ-driven internet radio, web player or MPD fallback
- §3.7 Network Layer: BBC VPN split-tunnel via Mullvad WireGuard with automatic UK-endpoint rotation on failure
- Tiered Tidal client strategy: SONE (Tauri 2/Rust, best stack fit) → tidalt (confirmed ARM64) → custom Rust client → play.tidal.com (lossy last resort)
- Source category table in §3.3 (native backend / standalone app / web-native / Chromium escalation)
- Mullvad VPN subscription added to BOM (€5/month)
- 5 new risk entries (Tidal API breakage, SONE ARM64 viability, WebKitGTK source compat, VPN rotation exhaustion, panel space for 5th button)

### Changed
- **UI architecture: Slint → Tauri 2 (Rust + WebKitGTK)** — native Rust backend for GPIO/PipeWire/SPI, WebKitGTK webview hosts per-source web UIs. Chromium available as per-source escalation fallback.
- Tidal control model: cast-from-phone (GioF71/tidal-connect) → 100% on-device control via dedicated Linux client (SONE/tidalt/custom). Lossless FLAC preserved.
- Plexamp control model: cast-only → primary control from StreamDeck touchscreen via localhost:32500 in Tauri webview. Cast-from-phone remains as bonus.
- Source switching architecture diagram updated for 7 sources + Tauri shell ownership
- VU meter on-screen pipeline: Slint widget → Tauri IPC + Canvas/WebGL in webview (physical MCP4922 path unchanged)
- Build phases restructured: Phase 2 now covers VPN + additional sources, Phase 3 covers Tauri shell + display
- GPIO button table: added Bandcamp (GPIO 25), renumbered Radio to button 5, Play/Pause to 6, Volume to 7
- UI Technology Options table: added Option E (Tauri 2), marked as selected. Slint (Option B) rejected — no native webview for per-source UIs.
- Risks table: replaced stale Tidal Connect and Slint risks with Tauri/WebKitGTK, VPN, and new-source-specific risks

## [0.3.1] - 2026-04-05

### Changed
- Project renamed from "StreamDeck Pi" to "TheStreamDeck"
- All references updated across specs, diagrams, KiCad schematic, and LaTeX sources

## [0.3.0] - 2026-04-05

### Added
- Front panel layout concept diagram (STREAMDECK-CONC-001 Rev A) — 5" display centred, source buttons flanking, available zones for VU meters
- Blank wireframe PDFs for free sketching — 70mm (x3) and 102mm (x2) form factors at 92% scale on A3
- Case wireframe PDFs with four layout options (circular + rectangular VU meters) at 70mm height

### Changed
- Display decision revised: 13.3" and 10.1" too tall for NAD-matching form factor. 5" (111 x 62mm) is the max that fits 70mm case height.
- Form factor locked to 435mm width (NAD C 3xx standard). Max height 70mm (NAD C 338 profile).
- VU meter sizing revised: 45-50mm circular or 60x40mm rectangular at 70mm case height. 85mm meters no longer viable.

## [0.2.0] - 2026-04-05

### Added
- Analogue VU meter subsystem — MCP4922 dual SPI DAC driving two 85mm panel meters (L/R)
- VU meter driver KiCad schematic (`code/streamdeck-vu-driver.kicad_sch`)
- ElectricalEngineer skill created for KiCad schematic production
- Electrical engineering language reference (IEC 61082, IEC 60617, IEC 81346, ISO 5457, ISO 7200, ISO 128)
- Printable actual-size screen templates (13.3" and 11.6") as A3 PDFs for UI sketching
- Architecture diagram Rev B with MCP4922 + VU meters + updated display

### Changed
- Display upgraded from 7" Pi Touch Display 2 (DSI, 720p) to Waveshare 13.3" HDMI LCD (H) V2 (1080p)
- UI layout updated from 1280x720 to 1920x1080 — album art area enlarged to 600x600px
- GPIO allocation updated: SPI0 (GPIO 8/10/11) added for DAC, DSI pins freed
- Performance budget updated for 1080p rendering (~9-16% CPU)
- BOM updated: $255-290 → $275-335 (display + VU meter components)

## [0.1.0] - 2026-04-04

### Added
- Project initialised with problem statement and technical specification
- Hardware architecture: Pi 4B + HiFiBerry Digi2 Pro (S/PDIF) + NAD C 3xx optical input
- First-principles analysis: Digi HAT over DAC HAT — Pi stays purely digital
- Software stack: Pi OS Lite 64-bit, PipeWire, MPD, tidal-connect (Podman), Plexamp headless
- Source switching architecture with Rust daemon + Slint UI
- VU meter visualisation pipeline: cpal + RustFFT (NEON) at 60fps
- Architecture diagram Rev A (STREAMDECK-ARCH-001)
- 758-line research file covering all integration approaches
- 5-phase build plan with weekly milestones
