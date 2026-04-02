# Ori Garmin Watch Face — Project Brief

## What We're Doing

Forking the [Segment34mkII](https://github.com/ludw/Segment34mkII) Garmin watch face (MIT licensed) as a starting point, then evolving it into an Ori-branded watch face for backcountry skiing.

## Why

Phase 4 of Ori includes Garmin integration. A custom watch face is the first step — glanceable crew status, mesh connection, and SOS right on the wrist.

## Source Repo: Segment34mkII

- **Platform:** Garmin Connect IQ
- **Language:** Monkey C
- **License:** MIT
- **Supports:** 80+ Garmin devices (fenix 6/7/8, epix, venu, FR series, MARQ, Enduro, etc.)
- **Repo:** https://github.com/ludw/Segment34mkII

### Architecture (2 source files)

- `Segment34App.mc` — Tiny app shell, creates view + delegate, forwards settings changes
- `Segment34View.mc` — ~3580 lines, the entire watch face. Contains 3 classes:
  - `Segment34View extends WatchUi.WatchFace` — main rendering
  - `Segment34Delegate extends WatchUi.WatchFaceDelegate` — tap zone handling (5 zones)
  - `StoredWeather` — cached weather data class

### Key Patterns to Know

**34-segment display** — Custom bitmap fonts, not algorithmic. Each digit glyph has all 34 segments. The `#` character renders all segments lit (dim backdrop). Time is drawn in two passes: dim background segments, then bright active segments on top, with optional outline layer.

**Multi-device support via build annotations:**
- Screen size: `:Round240`, `:Round260`, `:Round280`, `:Round360`, `:Round390`, `:Round416`, `:Round454`, `:Square`
- Display type: `:MIP`, `:AMOLED`
- Memory: `:HighMem`, `:LowMem`
- The compiler strips code not matching the target device — so `loadResources()`, `calculateLayout()`, etc. each have multiple complete implementations

**13-color theme system** — 24 presets + custom. Colors: bg, clock, clockBg, outline, dataVal, fieldBg, fieldLbl, date, dateDim, notif, stress, bodybatt, moon. Parsed from compact hex strings.

**Weather caching** — Stores to `Application.Storage` so weather persists when BLE disconnects. Hysteresis pattern for memory management (clear cache below 15KB free, re-enable above 17KB).

**AOD (Always-On Display)** — Burn-in protection via pixel shifting each minute + dot pattern overlay. Partial updates clip to just the seconds region.

**Data sourcing priority (high-mem devices):**
1. Complications API (preferred, most current)
2. SensorHistory API (fallback)
3. ActivityMonitor/Activity API (last resort)

**Moon phase** — Julian Day calculation, 8 phases mapped to a custom bitmap font. Hemisphere-aware. Death Star Easter egg on May 4th.

**Settings** — 65+ configurable properties. All read into cached instance vars in `updateProperties()` to avoid repeated lookups during rendering.

**Tap zones** — Screen divided into 5 zones (top, mid, bottom-left, bottom-center, bottom-right). Each configurable to cycle themes or open a Garmin complication.

### File Structure

```
Segment34mkII/
  manifest.xml              — App type, min API 3.2.0, permissions, device list
  monkey.jungle             — Build config, per-device annotations + resource paths
  source/
    Segment34App.mc         — App entry point
    Segment34View.mc        — All rendering, data, and logic
  resources/
    fonts/                  — 58 files: segment fonts (6 sizes × variants), moon, icons
    settings/
      properties.xml        — 65+ property defaults
      settings.xml          — User-facing settings UI
    strings/                — English strings
    drawables/
  resources-deu/fre/ita/pol/spa/swe/  — Localized strings
  disp-resources/
    amoled/                 — AMOLED display resources
    mip/                    — MIP display resources
```

## Future: Ori Watch Face Vision

Eventually replace/augment data fields with Ori-specific info:

- **Crew count** — "3 connected" from mesh
- **Mesh status** — green dot = active, pulsing = reconnecting
- **SOS indicator** — red alert when crew member triggers SOS
- **Last heard** — time since last position from each crew member
- **Tour/Descent mode** — current mode indicator
- **Signal strength** — LoRa mesh quality

This would require a Connect IQ companion app or widget that communicates with the Ori phone app via Garmin's BLE messaging API.

## Dev Setup

1. Download [Connect IQ SDK Manager](https://developer.garmin.com/connect-iq/sdk/)
2. Install VS Code + **Monkey C** extension (by Garmin)
3. Java 8+ required (`java -version`)
4. Verify: Cmd+Shift+P → "Monkey C: Verify Installation"
5. Clone: `git clone https://github.com/ludw/Segment34mkII.git`
6. Open in VS Code, select target device, build + run in simulator

## Status

- [ ] SDK installed and verified
- [ ] Repo cloned and building
- [ ] Identified target Garmin device
- [ ] First successful simulator run
- [ ] First deploy to physical watch
- [ ] Begin Ori customization
