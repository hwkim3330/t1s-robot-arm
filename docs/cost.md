# What a joint costs, and why that is not the reason to build it

**Every figure on this page is a rough estimate made without a quote.** Prices
and stock move; re-check before ordering, as CLAUDE.md requires. The slip ring
and the machining are the two that could move the total by a factor of two, and
both need a real quote.

## Per joint, small quantity

| electronics | est. |
|---|---|
| ESP32-P4 | ~$6 |
| LAN8670 PHY | ~$4 |
| gate drivers + 6 MOSFETs | ~$9 |
| current sense amps and shunts | ~$3 |
| absolute output encoder | $10–30 |
| PCB, 4–6 layer annulus | $10–30 |
| **subtotal** | **$50–80** |

| mechanics | est. |
|---|---|
| BLDC stator and rotor | $40–80 |
| metal planetary set | $50–150 |
| cross-roller bearing | $30–60 |
| **slip ring** (signal pair + high-current rings) | **$30–100+** |
| **housing machining, small quantity** | **$100–300** |
| **subtotal** | **$250–600** |

**Electronics are 15–25 % of the joint. Machining alone is often more than the
whole electronics bill.**

## The uncomfortable part

A comparable off-the-shelf actuator — MyActuator X8-120 class, 120 Nm peak —
sits in the same range or below once small-quantity machining is counted.

**Building this is not cheaper than buying it.** At the quantities in view it is
probably more expensive.

## So the reason to build is what cannot be bought

Nothing in the catalogue offers:

- **a 10BASE-T1S bus in the joint**, with PLCA giving the control loop a
  guaranteed slot — commercial actuators ship CAN
- **unlimited rotation**: slip ring, absolute output encoder, multi-turn count.
  Catalogue actuators have cable limits
- **power and data on the same conductors** through that slip ring
- **our firmware in the loop**, which is what makes the bus, the watchdog and
  the torque behaviour ours to change

That is the case for building, and it is worth stating plainly because the
wrong reason — "it will be cheaper" — collapses on the first quote and takes
the project's judgement with it.

## Where the money would actually be saved

If cost becomes the point rather than capability:

- **Machining dominates.** Printed housings for the bench, machining only for
  the ones that carry load, or a design that uses stock tube and plate instead
  of billet.
- **Quantity.** Machining per unit falls hard between 1 and 10; the electronics
  barely move.
- **Buy the gearbox, build around it.** The planetary and the bearing are the
  parts with the least to gain from being ours.
- **One size, not two.** J40 and J120 double the mechanical tooling and share
  only the electronics.

## 확인 필요

- Slip ring quote: signal pair plus high-current rings, Ø20 hollow bore. No
  candidate part found yet, and this is the component with no obvious supplier.
- Machining quote for the housing, at 1 off and at 10 off.
- X8-120 actual purchase price, to make the build-versus-buy comparison real
  rather than estimated.
