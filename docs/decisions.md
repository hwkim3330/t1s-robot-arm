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

The brief fixed the ESP32 family and asked whether to compare STM32G4. It also
specified SPI pins for a LAN8651 MAC-PHY. There is a better arrangement.

**ESP32-P4** carries a 10/100 Ethernet MAC whose ESP-IDF driver uses **RMII**.
**LAN8670** is a 10BASE-T1S PHY with an **MII/RMII** host interface, and
Espressif publishes an official **`lan867x`** driver component for it.

So the PHY can hang off the hardware MAC instead of off a 20 MHz SPI link to a
MAC-PHY. That removes the SPI bottleneck and the SPI interrupt latency from the
control path, and it is a combination both vendors support rather than one we
invent.

**Chosen:** ESP32-P4 + LAN8670 (RMII). One chip runs the 1 kHz FOC loop on one
core and the network on the other.

**What this gives up:** STM32G4 is the better motor-control die — HRTIM at
184 ps, CORDIC and FMAC accelerators, dual/triple ADCs built for synchronous
phase-current sampling, and it is what SimpleFOC targets first. The P4 has
MCPWM and general ADCs. The brief ruled the comparison out by requiring ESP32,
and the P4's two 400 MHz RISC-V cores have the compute to do in software what
the G4 does in hardware — but this is a choice to meet a constraint, not a
claim that the P4 is the better motor controller.

**확인 필요:** the P4's ADC resolution, sample rate, and whether it can sample
two phase currents synchronously with the PWM. If it cannot, an external ADC
goes on the joint board and the part count argument weakens. **Nothing is
ordered until this is answered from the datasheet.**

**Reversed by:** that ADC answer coming back badly. The fallback is G4 for FOC
and a small ESP32 for the bus — two chips, one inter-chip link, and the brief's
ESP32 requirement satisfied at the network edge rather than in the loop.

---

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
