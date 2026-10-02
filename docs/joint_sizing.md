# Joint sizing — what the numbers actually allow

The brief asked for one joint module shared between arms and legs, built from
an 8108–10015 class BLDC through a 1:6–1:9 metal planetary, making 80–150 Nm
at the leg and 20–40 Nm at the arm. It also said those torque figures were
estimates to be recomputed. This is that recomputation, and it does not come
out the way the brief assumed.

## The reference point

CubeMars **AK80-9** is exactly the machine the brief describes: an 8108-class
stator with a 9:1 planetary, built as a robot joint.

| | AK80-9 V3.0 |
|---|---|
| reduction | 9:1, 9 arcmin backlash |
| rated torque | **9 Nm** |
| peak torque | **22 Nm** |
| rated speed | 390 rpm at 48 V |
| size / mass | Ø98 × 38.5 mm, 485 g |

Source: [CubeMars AK80-9 V3.0](https://www.cubemars.com/product/ak80-9-v3-0-robotic-actuator.html).
Peak current is **확인 필요** — the product pages quote torque and voltage but
not a peak phase current, and the coupling network cannot be designed without
it.

## Where the brief breaks

**22 Nm is the peak of that whole architecture**, not of the motor. Behind the
9:1 the rotor itself is making about 2.4 Nm.

To reach 150 Nm at the output while keeping 1:9, the motor has to make
**16.7 Nm** — seven times what an 8108 produces. Torque scales roughly with
rotor diameter squared times stack length, so seven times the torque is not a
larger winding or a hotter drive; it is a rotor on the order of Ø150–180 mm.
That is no longer an 8108, and no longer 485 g.

The other direction does not work either. Reaching 150 Nm from a 2.4 Nm rotor
takes about **1:62**, and at that ratio the joint is not quasi-direct drive any
more: it stops being backdrivable, reflected inertia goes up with the square of
the ratio, and the impact tolerance the brief asked for is gone. The whole
reason for choosing 1:6–1:9 was to keep those properties.

So **80–150 Nm and "8108 through 1:9" are not the same joint**, and no choice
of winding or controller reconciles them.

## What is consistent

| class | torque | what it takes |
|---|---|---|
| **arm** | 20–40 Nm | AK80-9 sits at 22 Nm peak — the bottom of the band. A slightly larger rotor, or 1:12, covers 40 Nm comfortably. Off-the-shelf. |
| **leg** | 80–150 Nm | Ø130–150 mm rotor at 1:8–1:10, roughly 15–19 Nm at the rotor. This is MIT Cheetah 3 territory (230 Nm at 1:7.7) and is a custom or large-frameless build, not a drone motor. |

Both are the same *architecture* — large rotor, low metal planetary,
backdrivable, current-controlled. They are not the same *part*.

## The proposal

**One design, two sizes.** Everything that makes this a module stays shared:

- the same joint PCB, FOC firmware and control loop
- the same two-conductor bus, PoDL coupling and PLCA addressing
- the same slip ring, absolute output encoder and multi-turn counting
- the same 110 × 110, 4 × M8 port face
- the same housing geometry, scaled

What differs is the rotor, the ring gear and the housing diameter. A leg-sized
module on an arm would be several kilograms of dead weight carried at the end
of a lever — the opposite of what modularity is for.

## Why this has to be settled before the node board

The node board's coupling network is sized by the current it passes, and that
current comes from this page.

A leg joint at 150 Nm will draw, order of magnitude:

    150 Nm at ~5 rad/s        = 750 W mechanical
    at ~0.7 electrical-to-mechanical
                              ≈ 1.1 kW electrical
    at 48 V                   ≈ 22 A

Against that, the one commercial T1S board doing power over the pair —
Silicognition **ManT1S** — distributes **60 V at 700 mA, 42 W**
([Crowd Supply](https://www.crowdsupply.com/silicognition/mant1s)). That is a
factor of twenty-five below a leg joint.

This does not mean the two-wire requirement fails. It means the coupling
inductors have to carry roughly 22 A of DC without saturating, and the slip
ring has to pass it through two contacts, and both of those are a different
class of part from anything in a 42 W design. **Choosing them is step one, not
a detail at the end.**

## 확인 필요

- AK80-9 peak phase current at 48 V. Needed for the coupling network and for
  the eFuse trip point.
- Peak joint speed. 5 rad/s above is an assumption; the real figure comes from
  the gait, and the power scales with it directly.
- Duty cycle. Peak torque for how long decides the thermal design and whether
  22 A is a transient or a rating.
- Slip ring: two contacts at 22 A continuous, with 10 Mbps riding on the same
  contacts. Contact resistance becomes heat and noise. No part selected yet.
- Whether the arm module can be a lower-current variant of the same board, or
  whether one board must cover both.
