# Problem Statement: Streaming Audio Appliance

**Problem Statement:**
An audiophile with multiple streaming subscriptions (Tidal HiFi Plus, Plexamp/Plex, BBC 6 Music, internet radio) experiences frustrating, slow source switching when listening through a NAD C 3xx integrated amplifier, because each service runs on a different device requiring manual amp input selection. This fragmented workflow disrupts listening sessions, adds unnecessary physical interaction, and prevents enjoying visual elements like album artwork during playback.

**Components:**
- **Stakeholder:** Home audiophile with NAD pre/power amp, multiple streaming subscriptions, and Raspberry Pi 4 hardware
- **Problem:** Switching between audio sources requires changing physical devices and amp inputs — slow, disruptive, and frustrating
- **Context:** Daily listening sessions across BBC 6 Music, Tidal, Plexamp (local library), and internet radio stations
- **Impact:** Broken listening flow, reluctance to switch sources, underuse of paid subscriptions, no visual engagement (album art, now-playing info)
- **Constraints:**
  - Must use existing Raspberry Pi 4 Model B
  - Must connect to NAD C 3xx via single input (digital optical preferred)
  - PiOS as operating system
  - No Python in implementation stack (Rust/TypeScript preferred)
  - No cloud dependencies for core playback
  - Plex Pass required for Plexamp (already owned)
  - Tidal HiFi Plus subscription (already owned)
  - BBC 320kbps streams geo-restricted to UK
- **Measurable:**
  - Source switching completes in under 3 seconds
  - All four sources (BBC 6 Music, Tidal, Plexamp, internet radio) playable from one device
  - Single amp input used for all sources
  - Album artwork displayed during playback
  - VU meter visualisation responds accurately to L/R audio channels
  - Zero audible pops or clicks during source transitions

**First Principles Check:**
- **Root Problem:** CONFIRMED — the root issue is multiple independent audio devices competing for a single listening position. The amp's input selector is a symptom of the real problem: no unified audio transport layer.
- **Key Assumptions:**
  - Tidal Connect (unofficial Docker) will remain functional (risk: Tidal could break it)
  - PipeWire can multiplex all sources without audible degradation
  - Pi 4 has sufficient CPU for simultaneous playback + VU meter rendering
  - The NAD's internal DAC is superior to any Pi HAT DAC (validated by spec analysis)
- **Constraints Status:**
  - Pi 4 hardware: verified immutable (owned)
  - NAD amp: verified immutable (owned, has optical digital input)
  - No Python: negotiable but strongly preferred
  - Plex Pass: verified immutable (Plexamp requirement)

**Priority:** High — daily quality-of-life improvement
**Status:** Active
**Created:** 2026-04-04
