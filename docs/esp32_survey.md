# The ESP32 line, as it stands in October 2026

Written because D3 was made, and D8 reversed it, without either having looked at
the current line-up. The part the brief kept pointing at — **S31** — was misread
twice here as "S3" and was never surveyed.

## The field

| | cores | Ethernet MAC | radio | notes |
|---|---|---|---|---|
| **ESP32-S3** | 2 × Xtensa LX7 @ 240 | none | Wi-Fi 4, BLE | what D8 chose; $1.99, in stock |
| **ESP32-S31** | 1 × RISC-V HP @ 320 (FPU, SIMD) + 1 LP | **gigabit, RGMII** | Wi-Fi 6, BT 5.4, 802.15.4 | mass production 2026-07-27 |
| **ESP32-P4** | 2 × RISC-V @ 400 + 1 LP | 10/100 RMII | **none** | $4.67, out of stock at LCSC |
| ESP32-C5 | 1 × RISC-V @ 240 | none | Wi-Fi 6 dual band | single core |
| ESP32-C61 | 1 × RISC-V @ 160 | none | Wi-Fi | budget part |
| ESP32-H4 | 2 × RISC-V @ 96 | none | 802.15.4, no Wi-Fi | too slow for the loop |

Sources: [Espressif ESP32-S31](https://www.espressif.com/en/products/socs/esp32-s31),
[CNX on the S31](https://www.cnx-software.com/2026/03/24/esp32-s31-dual-core-risc-v-mcu-offers-gigabit-ethernet-wifi-bluetooth-and-802-15-4-connectivity/),
[Adafruit, mass production](https://blog.adafruit.com/2026/07/27/esp32-s31-now-in-mass-production-and-available-for-purchase/).

Only the S31 and the P4 carry an Ethernet MAC at all. **Everything else in the
family must reach the bus through an SPI MAC-PHY**, which is why D8's LAN8651
choice follows from picking an S3.

## ESP32-S31, in detail

- RISC-V HP core RV32IMAFCP at 320 MHz, **with FPU and SIMD** — directly useful
  for the Park/Clarke transforms and the current PI loop
- plus one low-power RISC-V core
- 512 KB SRAM; 250 MHz 8-bit DDR PSRAM with concurrent flash and PSRAM access
- 62 GPIO
- **gigabit Ethernet MAC exposed over RGMII**
- Wi-Fi 6, Bluetooth 5.4, 802.15.4
- ESP32-S31NRV16 around **$3.40–$5.32** at DigiKey depending on quantity

## What is genuinely attractive here

A joint MCU with an Ethernet MAC means the PHY hangs off the MAC instead of off
an SPI link — the same argument D3 made for the P4, now attached to a part that
is actually in production. The FPU and SIMD are a better fit for FOC maths than
anything else in the family.

## What is not settled, and why this is not a decision yet

**The interface is RGMII, and the T1S PHY is MII/RMII.** Gigabit MACs commonly
support RMII as well for 10/100, but *commonly* is not *this part*. If the S31
MAC is RGMII-only, **LAN8670 does not connect to it** and the whole reason to
prefer the S31 evaporates. This is the single question that decides it.

**The core topology is asymmetric.** The plan has been FOC on one core and the
bus on the other. The S3 has two equal 240 MHz cores, which suits that exactly.
The S31 has one fast core and one *low-power* core, and an LP core is not where
a network stack goes. So the S31 may be a faster chip that is a worse fit for
this particular split — or the 320 MHz HP core with SIMD may simply absorb both.
Unknown.

**Price and stock at our distributor.** $3.40–5.32 is DigiKey; LCSC was not
checked. The S3 is $1.99 with stock confirmed.

## 확인 필요 — in the order that decides things

1. **Does the ESP32-S31 EMAC support RMII, or RGMII only?** Datasheet. If RMII
   is absent, the S31 is out for this use and D8 stands unchanged.
2. S31 ADC: resolution, channels, sample rate, and synchronous sampling with
   MCPWM. Same question that was never answered for the P4, and it still gates
   the joint board for every candidate.
3. Whether the LP core can carry the bus task, or whether both tasks land on the
   HP core.
4. LCSC price and stock for S31 in a package we can hand-place or have assembled.
5. ESP-IDF support maturity for S31 Ethernet plus the `lan867x` driver together.

Until item 1 is answered from the datasheet, **D8 stands: ESP32-S3 with
LAN8651.** It is cheap, in stock, and already running on hardware in this lab.
