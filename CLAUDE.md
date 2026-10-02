# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this repository is

This is **not a software project** — it is a documentation and configuration
repository for a hardware conversion. It captures the migration of a **Geeetech
Prusa i3 Pro X** 3D printer from its stock firmware to **Klipper**, running on a
**BTT SKR Mini E3 V3** control board.

There is no build step, no test suite, no dependencies, and no application to
run. The "code" here is Klipper configuration (INI-style `.cfg` syntax) plus a
Markdown writeup. Changes are validated on the physical printer, not by CI.

### Hardware context (needed to reason about the configs)

| Item | Value |
|------|-------|
| Printer | Geeetech Prusa i3 Pro X (cartesian) |
| Mainboard | BTT SKR Mini E3 V3 |
| MCU | STM32G0B1 |
| Stepper drivers | TMC2209, UART mode (shared UART on `PC11`/`PC10`, addresses 0–3) |
| Host | Raspberry Pi 3 |
| Firmware / UI | Klipper + Mainsail |
| Z axis | M8 threaded rods (fine `rotation_distance`) |

## File layout

Everything lives at the repository root. Note the `.txt` suffixes — on the
running printer these are the real Klipper `.cfg` files; here they carry `.txt`
so they render as plain text on GitHub.

| File | Role |
|------|------|
| `README.md` | Human-facing narrative of the upgrade: hardware, motion/heating/fan settings, issues resolved, current status. |
| `printer.cfg.txt` | **The primary, hand-authored config.** MCU, printer limits, extruder, heater_bed, steppers, fans, TMC2209 drivers, bed-leveling macro. `[include mainsail.cfg]` pulls in the client macros. |
| `mainsail.cfg.txt` | Standard **Mainsail** client-macro bundle (PAUSE/RESUME/CANCEL_PRINT, park/extrude/retract helpers). Vendor-generated — treat as boilerplate. |
| `timelapse.cfg.txt` | Standard **moonraker-timelapse** plugin macros. Vendor-generated — treat as boilerplate. |

The `/config` and `/docs` directory structure described in `README.md` is
aspirational — it does **not** exist in the repo yet. The configs are flat at
the root.

## Working conventions

### Source of truth
When `README.md` and `printer.cfg.txt` disagree, **`printer.cfg.txt` is
authoritative** — it is the file that actually runs the printer. The README has
drifted from the config in several places (e.g. X `rotation_distance`, Z
`rotation_distance`, Z `hold_current`, several `homing_speed` values). If you
change a value in the config, update the corresponding prose in `README.md` in
the same change so the two stay in sync; if you only notice drift, flag it
rather than silently "fixing" one side.

### Hand-authored vs. generated
- **`printer.cfg.txt`** is the only file a human tunes by hand. Focus edits here.
- **`mainsail.cfg.txt`** and **`timelapse.cfg.txt`** are third-party generated
  bundles. Do not refactor or "improve" them; only touch them if replacing them
  wholesale with a newer vendor version.

### The SAVE_CONFIG block
`printer.cfg.txt` contains an auto-generated block:
```
#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
```
Klipper itself writes this section (it stores calibrated PID values here). **Never
hand-edit anything from that marker down**, and keep it as the last block in the
file. New sections go *above* it.

### Comments and language
Comments in `printer.cfg.txt` are in **Portuguese** (e.g. `NIVELAR_MESA` =
"level the bed", `# apenas para validar sensores`). Preserve the existing
language of comments and macro names when editing nearby code; match the
surrounding style rather than translating.

### Safety-critical values — change deliberately
This config drives real hardware. Be conservative and explicit about:
- **`max_temp` / `min_temp`** on `[extruder]` and `[heater_bed]` — thermal safety.
- **TMC `run_current` / `hold_current`** — too high overheats motors/drivers.
- **`position_max`, `position_endstop`, `homing_speed`** — wrong values crash the
  toolhead or bed. Z `homing_speed` is intentionally very low (`2`) for the M8 rods.
- **`stealthchop_threshold`** — `999999` enables StealthChop on X/Y/Z; `0` (extruder)
  keeps it in SpreadCycle for torque.
- Pin assignments (`PC6` part fan, `PC7` hotend fan, `PB15` controller fan, etc.)
  are board-specific to the SKR Mini E3 V3 — do not change them without a reason
  tied to the physical wiring.

Never silently loosen a safety limit. If a request implies raising temps,
currents, or speeds, call out the risk.

## Git workflow

- Active development branch for this work: **`claude/claude-md-docs-yghxg2`**.
- Commit with clear, descriptive messages; push with `git push -u origin <branch>`.
- Do **not** open a pull request unless explicitly asked.
- There is nothing to lint, build, or test in this repo — validation happens by
  loading the config on the printer and running `RESTART` / `FIRMWARE_RESTART` in
  Klipper. Do not claim a change is "verified" beyond confirming the config
  syntax is well-formed.

## Quick reference: what to edit for common requests

| Request | Where |
|---------|-------|
| Change motion speeds/accel | `[printer]` in `printer.cfg.txt` |
| Retune a stepper's distance/current | `[stepper_*]` and matching `[tmc2209 stepper_*]` |
| Adjust temps or PID | `[extruder]` / `[heater_bed]` (live values) — do not touch the SAVE_CONFIG copy |
| Fan behavior | `[fan]`, `[heater_fan hotend_fan]`, `[temperature_fan controller_fan]` |
| Bed leveling points | `[bed_screws]` / `[gcode_macro NIVELAR_MESA]` |
| Client/UI macros | `mainsail.cfg.txt` (generally leave alone) |
| Update the human writeup | `README.md` |
