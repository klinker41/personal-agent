---
topic: growatt-modbus-rtu
category: knowledge
tags: [knowledge, growatt-modbus-rtu]
updated_at: 2026-11-01T06:01:07.445119+00:00
confidence: 0.95
---

# Knowledge: Growatt-Modbus-Rtu

- On a Growatt SPF 3500 inverter in Lithium mode via Modbus RTU (/dev/ttyUSB0),
SoC thresholds are stored in Holding Registers scaled in 0.1% units (e.g., 500 =
50.0%) and written using Modbus Function Code 0x06.
- Growatt SPF 3500 key Modbus registers: Holding Register 37 is Program 12 (Back
to Utility SoC threshold, bat_low_to_uti), Holding Register 95 is Program 13
(Back to Battery SoC threshold), Holding Register 82 is Program 21 (Low DC
Cut-off SoC threshold), and Input Registers 17 and 18 report battery voltage and
SoC.
