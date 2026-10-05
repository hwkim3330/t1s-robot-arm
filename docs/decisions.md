# Decisions

Each entry says what was chosen, what it rules out, and what would reverse it.

---

## D1 — Two signal conductors, separate rings for motor current

The brief wanted everything on one pair. [actuator_survey.md](actuator_survey.md)
found a 120 Nm knee draws order 50 A at peak, against the 0.7 A that the one
shipping T1S power-over-pair product distributes. Seventy times is not a margin.

**Chosen:** the slip ring carries a T1S pair with PoDL for data and logic power,
**and separate high-current rings for the motor bus**. A slip ring with more
than two contacts is ordinary hardware; nothing else in the brief changes.

Infinite rotation survives, the module stays a module, the real-time control
stays on T1S. The only thing given up is the literal reading of "two
conductors", and it was given up because 50 A through two contacts with 10 Mbps
on the same contacts is not a thing that works.

**Reversed by:** an arm-only build. At 20–40 Nm the current is 5–10 A and the
original two-wire form is buildable. Kept as the **J40-2W** variant.

---

## D2 — Two sizes of one design, both ours

ManT1S is a communications board, not an actuator; there is nothing to buy
there. Off-the-shelf actuators do not cover the requirement either — the
catalogue has nothing at 80–150 Nm with 1:6–1:9.

**Chosen:** build the actuator, in two sizes that share everything but the
rotor, ring gear and housing diameter.

| | **J40** (arm) | **J120** (leg) |
|---|---|---|
| peak torque | 40 Nm | 120 Nm |
| reduction | ~1:9 | ~1:20 |
| backdrivable | yes | reduced, accepted |
| bus current, rated | ~5 A | ~15 A |
| bus current, peak | ~15 A | ~50 A |

Shared: joint PCB, FOC firmware, T1S bus and PLCA addressing, slip ring,
absolute output encoder with multi-turn counting, Ø20 hollow bore, and the
110 × 110 / 4 × M8 port face.

J120's ~1:20 is the honest cost of the torque. The brief chose 1:6–1:9 to keep
backdrivability and impact tolerance; at 1:20 reflected inertia is roughly five
times worse than at 1:9, and that is a real loss. It is taken because 120 Nm
from a 1:9 planetary needs a rotor that does not fit the port face.

---

## D3 — ESP32-P4 with LAN8670 over RMII, not S3 with LAN8651 over SPI

**Both parts are Microchip.** The change is which Microchip device and where
the MAC lives, not an escape from a single supplier — see the supply note
below.

| | LAN8651 | LAN8670 |
|---|---|---|
| what it is | MAC **and** PHY | PHY only |
| host side | SPI | MII / **RMII** |
| needs | nothing from the MCU | an Ethernet MAC in the MCU |

### The SPI argument is weaker than it first looks

An earlier draft of this entry said RMII "removes the SPI bottleneck". At
10 Mbit/s that is not true and the number says so:

    control frame, 64 bytes = 512 bits
      on the wire at 10 Mbit/s      51 us
      across SPI at 20 MHz          26 us      half the wire time
    control period at 1 kHz       1000 us

SPI adds roughly 26 us plus interrupt overhead to a 1000 us period — about 3 %.
Real, but not a bottleneck, and 20 MHz carries 10 Mbit/s with headroom to spare.
What RMII actually buys is **determinism and CPU**: DMA moves frames instead of
an interrupt handler moving bytes, so jitter drops and the data path leaves the
CPU. That is worth having in a 1 kHz loop. It is not worth overstating.

### The real reason is compute, not the bus

The joint runs FOC and the network on the same die. The P4's two 400 MHz RISC-V
cores leave room to put the control loop on one core and everything else on the
other; an S3 doing both is tighter. RMII comes along with the P4 because the P4
has a MAC — it is a consequence of the choice, not the argument for it.

### What this costs

| | S3 + LAN8651 (SPI) | P4 + LAN8670 (RMII) |
|---|---|---|
| on hardware we already have | **works today** (elite-t1s-hat) | never built here |
| host signals | 6 | 9 |
| maturity | proven in this lab | newer silicon, newer driver |
| headroom for FOC | tight | ample |

Walking away from a combination that already runs on our own board is a real
cost and is counted here rather than left out.

### Against STM32G4

The G4 is the better motor-control die — HRTIM at 184 ps, CORDIC and FMAC, dual
and triple ADCs built for sampling phase currents in step with the PWM, and it
is what SimpleFOC targets first. The brief required the ESP32 family, so the
comparison is closed by constraint. This entry does not claim the P4 is the
better motor controller.

### 확인 필요

- **P4 ADC**: resolution, sample rate, and whether two phase currents can be
  sampled synchronously with the PWM. If not, an external ADC goes on the board
  and the part-count argument weakens. **Nothing is ordered until the datasheet
  answers this.**
- **Second source**: 10BASE-T1S silicon is close to a Microchip monopoly.
  onsemi's NCN26010 is the one alternative known here, and it is a SPI MAC-PHY,
  so it does not fit the RMII path. Whether anyone else shipped a T1S PHY by
  2026 has not been surveyed. Single-source is a design risk and should be
  checked before committing to a footprint.

**Reversed by:** a bad ADC answer, or a second source that only exists in the
SPI form. The fallback is the proven S3 + LAN8651 path, or G4 for the loop with
a small ESP32 at the network edge.

## D4 — No custom silicon

The idea of one chip that is "ESP32 plus T1S" was raised. As an ASIC it is out
of reach: a mixed-signal die with a compliant Ethernet PHY means PHY IP
licensing, a mask set, and a verification effort measured in millions of dollars
and years, for a part that would be bought in hundreds.

**Chosen:** a **module** instead — P4, LAN8670, the SPE front end and the
supplies on one small board with a defined footprint, reused unchanged in every
joint. From the rest of the design's point of view that *is* one component: one
part number, one firmware image, one set of pads.

This is the achievable form of the same idea, and it is where "our own" actually
pays — not in owning a die, but in owning the module every joint is built from.

---

## D5 — No radio in the joint. Wi-Fi lives at the body, the watchdog lives in the joint

**The ESP32-P4 has no Wi-Fi and no Bluetooth.** Espressif's answer is a
companion chip — a C-series part over SDIO or SPI using ESP-Hosted. Confirmed
on [Espressif's own P4 page](https://www.espressif.com/en/products/socs/esp32-p4).

The brief asked for Wi-Fi as diagnostics, OTA, and an **emergency-stop
fallback**. Taken literally that means a second radio chip inside every joint.

**Chosen:** no radio in the joint at all.

- **Wi-Fi exists once, at the body gateway**, which is also where the battery,
  the eFuses and the main switch are. Diagnostics and OTA reach a joint over
  T1S, through the gateway. The brief already said the inside of the machine is
  T1S and the outside is Wi-Fi; this simply puts the boundary at the body
  instead of repeating it in every joint.
- **The emergency stop does not depend on a radio.** A joint that stops hearing
  the bus cuts torque by itself. The watchdog is local, it is the thing the
  brief already asked for, and it is strictly better than a Wi-Fi fallback:
  a radio link is least likely to work in exactly the situations where the stop
  matters. A fallback that shares a failure mode with the fault is not a
  fallback.

### What this buys on the joint board

A part with no radio needs no RF layout, no antenna, no shield can, and **no
radio certification**. That is the whole reason pre-certified modules like
ESP32-WROOM exist — under the tin there is only the SoC, a flash die, sometimes
PSRAM, a 40 MHz crystal, the RF matching network and decoupling, with the
antenna as a trace outside the can. You buy the module to skip RF design and
the certification that goes with it.

So the two boards in this machine want opposite things:

| | joint board | body gateway |
|---|---|---|
| wireless | none | Wi-Fi |
| part form | **bare P4 chip** | **pre-certified module** |
| RF layout | none needed | done by the module vendor |
| radio certification | not applicable | carried by the module |

Using a bare chip in the joint is cheap and small precisely because there is
nothing to certify. Using a module at the gateway avoids the one piece of RF
work the machine actually needs.

### 확인 필요

- P4 external flash and PSRAM: which are mandatory, and what the minimum
  configuration for this firmware is. The product page does not say; the
  datasheet must.
- P4 ADC resolution, channel count and sample rate, and synchronous sampling
  with MCPWM. Still unanswered — the product page omits it, and this is the
  item that decides whether an external ADC lands on the joint board.

---

## D6 — Zenoh at the gateway, raw frames in the joint

Zenoh is already in use in `esp32-t1s-bridge`, and zenoh-pico runs on MCUs, so
the temptation is to run it everywhere. It does not belong in the 1 kHz loop.
Publish/subscribe middleware carries routing, serialisation and buffering, and
all three show up as jitter. At a 1 ms period jitter is torque ripple.

    PC  (IK, vision, learning)
      |  Zenoh                      <- correct layer for this
    body gateway
      |  T1S, fixed-size frames     <- must stay bare
    joint MCU  (1 kHz FOC)

**Chosen:** the joint exchanges fixed-size frames in its PLCA slot, with no
TCP/IP stack above them — raw Ethernet is enough and it is what makes the period
predictable. The gateway translates between that and Zenoh. zenoh-pico runs on
the gateway. Whether it also runs on the joint for configuration and telemetry
is left open; it must never sit between the bus and the current loop.

The joint closes its own current loop. That is what makes a comms failure safe
and keeps a slow PC from reaching the torque.

---

## D7 — Bare chip in the joint; de-risk on a separate bench board

An earlier draft argued for a module on the first spin to reduce unknowns. For
an integrated servo that is wrong, and the reason is mechanical.

The joint PCB is an annulus: it has to clear a Ø20 hollow bore and fit inside a
Ø98–130 housing. Against that, a module is expensive in exactly the dimension
there is none of:

- its fixed rectangular footprint dictates the layout of a ring-shaped board
- the shield can plus the module's own substrate costs more than a millimetre
  of height, which is a lot inside a joint
- nothing can be placed under it, so one side of a two-sided board is lost
- a keep-out has to be left for an antenna the P4 does not have

With no radio there is no certification to buy from a module either. **The joint
board uses the bare chip.**

**The first-spin risk is real and is handled by splitting the boards, not by
putting a module in the joint:**

| | bench board | joint board |
|---|---|---|
| shape | whatever is convenient | annulus around the bore |
| purpose | prove FOC, current sensing, the bus, the firmware | carry the proven circuit |
| probing | test points everywhere | none |
| part form | module or chip, does not matter | bare chip |

The T1S half of the bench already exists — `elite-t1s-hat` runs on a
T-ETH-Elite today. What has never been built here is P4 with FOC and phase
current sensing, and that is what the bench board is for.

---

## D8 — Reverses D3: ESP32-S3 with LAN8651, on price, stock and prior art

D3 chose the P4 for compute headroom, having never checked what either part
costs or whether it can be bought. Checked (LCSC, 2026-10-02):

| part | price | stock |
|---|---|---|
| **ESP32-S3**, bare chip | **$1.99** | 1,462 |
| ESP32-S3-WROOM-1-N4, module | $2.93 | 3,728 |
| ESP32-S3-MINI-1U-N8, module | $3.31 | 1,459 |
| **ESP32-P4NRW32** | $4.67 | **out of stock** |

**A part that cannot be bought is not a design option.** That alone would settle
it; the price is 2.3× as well, against an estimate of ~$6 in cost.md that was
simply wrong.

### The compute argument does not survive contact either

D3's real reason for the P4 was room to run FOC and the network on one die. The
S3 is also dual-core, at 240 MHz instead of 400, and the same split applies —
one core for the loop, one for the bus. A 1 kHz current loop is not a hard
target for either; SimpleFOC runs FOC on S3-class parts routinely at PWM
frequencies well above that.

And the SPI cost was already measured in D3: about 26 µs of a 1000 µs period,
roughly 3 %. That was the number that made the RMII argument weak. It makes the
SPI path acceptable for the same reason.

### Consequences

The S3 has **no Ethernet MAC**, so the PHY-over-RMII arrangement goes with it.
The bus part returns to **LAN8651**, the SPI MAC-PHY — which is exactly what
already runs on `elite-t1s-hat` here. The SPI pins from the original brief
(SCLK IO39, MOSI IO38, MISO IO41, CS IO40, IRQ IO42, RST IO2) become the joint
board's assignment, free of the TF-card sharing and the IO0 strapping compromise
the HAT had to live with.

**Chosen:** ESP32-S3 (bare chip, per D7) + LAN8651 over SPI.

What is given up: the hardware MAC, DMA framing, and 400 MHz cores. What is
gained: a part that exists, at $2, on a combination already proven in this lab.

**Reversed by:** P4 stock returning *and* the loop proving tight on an S3. Both
would have to be true. Measure the loop on the bench board before revisiting.

### Note on the earlier estimate

cost.md put the MCU at ~$6 and the electronics subtotal at $50–80. The MCU line
is now $2. That does not change the conclusion of cost.md — electronics were
15–25 % of a joint and machining still dominates — but it is a reminder that
every figure on that page is an estimate until it is a quote, and this one was
out by 3×.

---

## D9 — Reverses D8: ESP32-S31 with LAN8670 over RMII

[esp32_survey.md](esp32_survey.md) left the S31 blocked on one datasheet line:
RGMII-only would mean LAN8670 cannot attach. The datasheet answers it, and two
other things the trade press got wrong come out with it.

| | reported in articles | **ESP32-S31 datasheet** |
|---|---|---|
| Ethernet interface | RGMII | **MII and RMII** (alongside a 1000 Mbps MAC) |
| cores | 1 HP + 1 low-power | **HP subsystem is dual-core RISC-V to 320 MHz**, plus a separate LP core at 40 MHz |
| ADC | not stated | **2 × 12-bit SAR, up to 16 channels, 4 differential pairs per unit** |

Source: [ESP32-S31 datasheet](https://documentation.espressif.com/esp32-s31_datasheet_en.html).

### Both objections in the survey dissolve

**RMII is supported**, so the T1S PHY attaches to a hardware MAC. (RMII tops out
at 100 Mbps, so the gigabit figure must be RGMII; irrelevant here — 10BASE-T1S
is 10 Mbps and RMII carries it with room to spare.)

**The HP subsystem is dual-core.** The asymmetry worry came from journalism, not
the datasheet. FOC on one HP core and the bus on the other works exactly as
planned, at 320 MHz instead of the S3's 240, with FPU and SIMD on top.

**The ADC has differential pairs**, which is what a shunt measurement wants —
four per unit, two units.

**Chosen:** ESP32-S31 + LAN8670 over RMII.

### What it costs against D8

| | ESP32-S3 | **ESP32-S31** |
|---|---|---|
| price | $1.99 (LCSC) | $3.40–5.32 (DigiKey) |
| HP cores | 2 × 240 MHz Xtensa | 2 × 320 MHz RISC-V, FPU + SIMD |
| Ethernet MAC | none → SPI MAC-PHY | **MII/RMII → PHY only** |
| ADC | 12-bit SAR | 12-bit SAR ×2, 16 ch, differential |
| proven here | **yes, today** | no |

Roughly $2 more per joint, against electronics that are 15–25 % of a joint whose
machining alone is $100–300. The price difference is not what decides this; the
hardware MAC and the differential ADC are.

What is still given up is the thing D8 valued most: `elite-t1s-hat` runs today
and this does not. That is what the bench board in D7 is for.

### 확인 필요 — before any footprint is committed

- **Confirm MII/RMII from the datasheet's own interface table.** The reading
  above came through a summariser, and it reverses a decision. Read the table.
- ADC maximum sample rate, and whether conversions can be triggered by MCPWM or
  a timer. Still unanswered for every candidate, and still the item that decides
  whether an external ADC lands on the joint board.
- LCSC price and stock for the S31 in an assemblable package. Only DigiKey
  pricing is known.
- ESP-IDF maturity for S31 Ethernet together with the `lan867x` driver.

**Reversed by:** the interface table not saying RMII, or the ADC proving unable
to sample in step with the PWM. The fallback is D8 unchanged.
