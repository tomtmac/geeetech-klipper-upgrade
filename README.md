# geeetech-klipper-upgrade
Prusa i3 Pro X → Klipper Upgrade (SKR Mini E3 V3)

This repository documents the migration of a Geeetech Prusa i3 Pro X to Klipper firmware using a BTT SKR Mini E3 V3 board with TMC2209 drivers in UART mode.

The objective was to modernize motion control, increase tuning flexibility, and achieve a stable and fully documented Klipper setup.


---

Hardware

Printer: Geeetech Prusa i3 Pro X
Mainboard: BTT SKR Mini E3 V3
MCU: STM32G0B1
Drivers: TMC2209 (UART mode)
Host: Raspberry Pi 3
Firmware: Klipper
Interface: Mainsail


---

Motion Configuration

Printer Limits

max_velocity: 150

max_accel: 800

max_z_velocity: 3

max_z_accel: 60


Conservative values chosen for initial stability and mechanical safety.


---

Stepper Configuration

X Axis

rotation_distance: 85

homing_speed: 30


Y Axis

rotation_distance: 40

homing_speed: 30


Z Axis (M8 threaded rods)

rotation_distance: 0.625

homing_speed: 2


All axes were manually calibrated and verified through controlled movement tests.


---

TMC2209 Current Settings

X

run_current: 0.75

hold_current: 0.50


Y

run_current: 0.85

hold_current: 0.60


Z

run_current: 0.75

hold_current: 0.50


Extruder

run_current: 0.90

hold_current: 0.60


StealthChop enabled for X/Y/Z.
Extruder tuned for torque consistency.


---

Heating Configuration

Hotend

Sensor: EPCOS 100K B57560G104F

PID tuned

Max temp: 250°C


Heated Bed

Sensor: EPCOS 100K

PID tuned

Max temp: 110°C



---

Fan Mapping

Part Cooling Fan

Pin: PC6

Manual control


Hotend Fan

Pin: PC7

Automatically turns on above 50°C


Controller Fan

Pin: PB15

Controlled via MCU temperature

Turns on above 40°C



---

Issues Encountered and Resolved

Thermistor instability causing MCU shutdown

Z axis aggressive homing

Extruder direction inversion

Incorrect fan header mapping

Stepper current refinement

Rotation distance calibration


All issues resolved via configuration tuning and hardware verification.


---

Repository Structure

/config

printer.cfg

mainsail.cfg

timelapse.cfg


/docs

setup_notes.md



---

Current Status

Motion calibrated
Extruder calibrated
Fans correctly mapped
Stable heating
No MCU shutdowns

Printer fully operational under Klipper.


---
