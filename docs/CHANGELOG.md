# Changelog

All notable changes to TheStreamDeck project.

## [0.3.1] - 2026-04-05

### Changed
- Project renamed from "StreamDeck Pi" to "TheStreamDeck"
- Project directory moved from `PROJECTS/private/streaming-appliance/` to `PROJECTS/public/TheStreamDeck/`
- All references updated across specs, diagrams, KiCad schematic, LaTeX sources, and PAI memory

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
