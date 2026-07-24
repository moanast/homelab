# Hardware Reference

> ⚠️ **This document describes intended wiring for when GPIO output is implemented. As of the current code, no GPIO pin is actually driven — `dry_run: false` only changes a console log message, not electrical output. Do not wire a valve, relay, or water line expecting the current build to switch it. See [Current GPIO Status](#current-gpio-status) below and [ROADMAP.md](ROADMAP.md) for when real output lands.**

## Current GPIO Status

`main.cpp` includes `<gpiod.hpp>` and each `Valve` instance holds a `gpiod::line` member, but no code path currently:
- opens a GPIO chip (e.g. `/dev/gpiochip0`),
- requests a line for output,
- or sets a line's value.

Both the `dry_run: true` and `dry_run: false` branches in `Valve::open()`/`Valve::close()` only `std::cout` a message. This is a deliberate, staged approach — logic first, hardware output second — not an oversight, but it means **this document is a wiring reference for the planned next step, not instructions for today's binary.**

## Target Platform

Two platforms are relevant to this project, at different stages:

| Stage | Platform | GPIO API | Status |
|---|---|---|---|
| Current | Linux (dev machine or SBC, e.g. Raspberry Pi) | `libgpiod` (`gpiod.hpp`) | Logic implemented; GPIO output not yet wired (see above) |
| Planned | ESP32 (Arduino framework) | `pinMode`/`digitalWrite` | Not started — see [ROADMAP.md](ROADMAP.md) |

The pin numbers below (`pin_gpio` in `config.yaml`) are Linux GPIO offsets as they'd apply to a `libgpiod`-compatible board (e.g. Raspberry Pi BCM numbering). **These will very likely need to be renumbered for ESP32** — ESP32 GPIO numbering and pin capabilities (which pins support output, which are input-only, which are reserved for flash/PSRAM) are different from a Raspberry Pi's, and not every pin number valid on one board is valid or safe to use on the other. Treat the config values below as placeholders from the Linux prototyping stage, not as verified ESP32 pin assignments.

## Configured Zones (from `config.yaml`)

| Zone ID | GPIO Pin (Linux/libgpiod numbering) | Moisture Threshold | Intended Watering Duration | Relay Polarity |
|---|---|---|---|---|
| `sector_1` | 17 | 35 (units not yet specified — see note below) | 1200 sec (20 min) — not yet enforced by code | Not set in `config.yaml`; defaults to active-high |
| `sector_2` | 27 | 40 (units not yet specified) | 900 sec (15 min) — not yet enforced by code | Not set in `config.yaml`; defaults to active-high |

> **Moisture threshold units are undefined.** `prag_umiditate` is currently an unspecified numeric scale (percentage? raw ADC/analog reading? sensor-specific units?). This should be pinned down before moisture logic is implemented — see [ROADMAP.md](ROADMAP.md) — since "35" means something very different for a capacitive soil sensor's raw ADC output (often 0–4095 or 0–1023 depending on ADC resolution) versus a calibrated percentage (0–100).

## Relay Polarity

`main.cpp` reads an optional `tip_releu` field per zone:
- `"active_high"` (default if the field is absent): the GPIO line should be driven **high** to energize/open the valve relay.
- `"active_low"`: the GPIO line should be driven **low** to energize/open the valve relay.

Currently, only `sector_1.yaml` (a draft file, not loaded by the running code — see [README.md](README.md#project-structure)) sets `tip_releu: "active_low"` for `sector_1`. `config.yaml`, the file actually loaded, does not set `tip_releu` for either zone, so **both zones currently default to active-high** when the code runs.

**Before wiring any relay:** confirm your specific relay module's polarity. Many common relay boards (especially low-cost "active-low" modules used with microcontrollers) energize when the control pin is pulled **low**, which is the opposite of a naive assumption. Wiring an active-low relay module while the code treats the zone as active-high (the current `config.yaml` default) means the valve state will be inverted — open when you expect closed, and vice versa. If you're using an active-low relay board, add `tip_releu: "active_low"` to that zone's entry in `config.yaml` (not just in `sector_1.yaml`, which the code doesn't read).

## Planned Wiring Reference (Once GPIO Output Is Implemented)

This section is intentionally general, since it describes a not-yet-implemented stage. Fill in specifics once the actual relay/valve hardware is selected and GPIO output logic lands.

**General solenoid valve + relay circuit (typical setup for this class of project):**

1. **Microcontroller/SBC GPIO pin** → relay module's control/signal input (through the relay board's own opto-isolation, if present — do not connect a GPIO pin directly to a solenoid's coil).
2. **Relay module** switches a separate, appropriately-rated power circuit for the actual solenoid valve — solenoid valves typically draw more current (often 200mA–1A+ depending on the valve) and may run at a different voltage (commonly 12V or 24V DC/AC for irrigation solenoids) than a GPIO pin can safely source (typically 3.3V, low mA).
3. **Solenoid valve** is wired into that separately-powered circuit, not directly to the microcontroller.
4. **Flyback/snubber diode** across the solenoid coil (if not already integrated into your relay module) to protect against inductive voltage spikes when the coil de-energizes.

**Before connecting real hardware, confirm:**
- Your specific relay module's control voltage/current requirements match what your board's GPIO can source (ESP32 GPIO: 3.3V logic, limited current — check your specific relay module's datasheet).
- Your solenoid valve's voltage and current draw, and that your power supply for that circuit is rated for it.
- Polarity (active-high vs. active-low) matches your `tip_releu` config, as described above.
- Water-side plumbing (valve orientation, pressure rating) separately from the electrical side — this document only covers electrical/GPIO wiring, not plumbing.

## Soil Moisture Sensors (Not Yet Integrated)

No moisture sensor reading exists in the current code — `prag_umiditate` is config-only, unused at runtime (see [README.md](README.md#current-behavior)). When this is implemented (see [ROADMAP.md](ROADMAP.md)), relevant hardware decisions to make and document here will include:
- Sensor type (capacitive vs. resistive — capacitive is generally preferred for longevity, as resistive probes corrode over time when powered continuously)
- ADC pin assignment per zone (on ESP32, note that some ADC2 pins conflict with Wi-Fi usage — this should inform which ADC pins are chosen)
- Calibration procedure (dry-air reading and fully-submerged/saturated-soil reading, to convert raw ADC values to the `prag_umiditate` scale)

This section will be filled in as part of the moisture-integration work tracked in [ROADMAP.md](ROADMAP.md).

## Safety Notes

- **Do not connect line-voltage (mains) power to this system without appropriate isolation and, if you're not experienced with mains wiring, a qualified electrician's input.** Many relay modules and irrigation solenoids run on low-voltage DC/AC, but always verify the rating of your specific components before assuming.
- **Test with `dry_run: true` first**, even after GPIO output is implemented, to confirm the logic (which zones trigger, in what order, for how long) matches expectations before any physical valve is connected.
- **Fail-safe default matters for irrigation:** a bug that leaves a valve open indefinitely wastes water; a bug that leaves it stuck closed just skips a watering cycle. When GPIO logic is implemented, consider what state the code defaults to on crash/restart (this isn't yet decided — see [ROADMAP.md](ROADMAP.md)), and prefer defaulting to closed/off where the relay hardware allows it.
