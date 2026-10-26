---
topic: atx-power-hat-i2c
category: knowledge
tags: [knowledge, atx-power-hat-i2c]
updated_at: 2026-10-26T06:01:06.807550+00:00
confidence: 0.95
---

# Knowledge: Atx-Power-Hat-I2C

- The Pod Bay 3 ATX Power Management HAT for Raspberry Pi 5 listens at I2C
address `0x5a` on bus 1 and persists settings in onboard EEPROM.
- Register `0x34` (`PWR_MGT`) controls AC/ATX power restoration behavior: `0x00`
stays off (default), `0x01` restores last power state, and `0x03` always boots
when power is applied (`i2cset -y 1 0x5a 0x34 0x03`). Bit 7 (`0x80`) optionally
enables low-voltage shutdown if the 12V rail drops below 9V.
- Register `0x35` controls the hardware watchdog (`i2cset -y 1 0x5a 0x35 0x80`
to enable).
