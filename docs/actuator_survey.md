# What shipping humanoids actually put in a leg

The brief asked for 80–150 Nm from a 1:6–1:9 planetary. [docs/joint_sizing.md](joint_sizing.md)
argued from first principles that this does not close. MyActuator's 2025 general
catalogue settles it with two complete robots they sell the actuators for.

Source: MyActuator *2025 General Catalog*, pages 113–116.

## 1.6 m, 80 kg humanoid — their large machine

| joint | part | ratio | rated | **peak** | power | mass |
|---|---|---|---|---|---|---|
| **leg** | RH-32 harmonic | **100:1** | 150 Nm | **229 Nm** | 282 W | 4.32 kg |
| waist | RH-25 harmonic | 100:1 | 108 Nm | 157 Nm | 282 W | 2.42 kg |
| upper arm | RH-17 harmonic | 100:1 | 35 Nm | 54 Nm | 91 W | 1.11 kg |
| lower arm | RH-14 harmonic | 100:1 | 11 Nm | 28 Nm | 28 W | 0.78 kg |
| drive wheels | X10-40 planetary | 7:1 | 15 Nm | 40 Nm | 265 W | 1.15 kg |

An 80 kg humanoid's leg is **100:1 harmonic**, not quasi-direct drive at all.
The only 7:1 part in the machine drives the wheels it stands on, and it makes
40 Nm.

## 1.3 m, 35 kg humanoid — their small machine

| joint | part | ratio | rated | **peak** | power | V | mass | Kt |
|---|---|---|---|---|---|---|---|---|
| **hip front, knee** | X8-120 planetary | **19.6:1** | 43 Nm | **120 Nm** | 574 W | 48 | 1.40 kg | 2.4 Nm/A |
| hip side/rot, waist | X6-60 planetary | 19.6:1 | 20 Nm | 60 Nm | 320 W | 48 | 0.82 kg | 2.1 Nm/A |
| arm, elbow, ankle | X4-36 planetary | 36:1 | 10.5 Nm | 34 Nm | 100 W | 24 | 0.36 kg | 1.9 Nm/A |
| wrist | X4-10 planetary | 12.5:1 | 4 Nm | 10 Nm | 100 W | 24 | 0.26 kg | 0.8 Nm/A |

This is the closest thing in the catalogue to what the brief describes, and the
knee that reaches 120 Nm does it at **19.6:1**, not 9:1.

## The finding

**Nothing in a 100-model catalogue makes 80–150 Nm at 1:6–1:9.** The parts that
reach that torque get there with 20:1 or 100:1. The parts that run at 7–12:1
make 10–40 Nm. These are two separate regions of the catalogue and the brief
asked for a point between them.

So the choice is not between vendors, it is between properties:

| | keep 1:6–1:9 | reach 80–150 Nm |
|---|---|---|
| torque | 20–40 Nm | yes |
| backdrivable | yes | much less |
| reflected inertia | low | ×(ratio²) higher |
| impact tolerance | high | needs a clutch or series elasticity |
| off-the-shelf | yes | yes |

Both are buildable. **They are not the same joint and the brief asked for both.**

## The current this implies, which is the number the board needs

X8-120, the 120 Nm knee, at 48 V:

    rated   574 W / 48 V              ≈ 12 A
    rated   43 Nm / 2.4 Nm/A          ≈ 18 A
    peak    120 Nm / 2.4 Nm/A         ≈ 50 A

**확인 필요:** whether 2.4 Nm/A is referred to the output shaft or the rotor.
Taken at the output the peak is 50 A; taken at the rotor it is 50 A × 19.6,
which is not credible, so the output reading is assumed. The datasheet must
confirm this before any component is chosen, because every number below scales
with it.

## What that does to the two-conductor requirement

The brief requires both power and data on one pair through the joint, PoDL
style. Against the figures above:

| | current |
|---|---|
| Silicognition ManT1S, the one shipping T1S power-over-pair board | **0.7 A** |
| X6-60 hip, rated | ~7 A |
| X8-120 knee, rated | ~12–18 A |
| X8-120 knee, peak | **~50 A** |

A 50 A peak through two slip-ring contacts, with 10 Mbps riding on the same two
contacts, is seventy times what the only commercial example of this does. The
coupling inductors have to carry that DC without saturating and still present a
high impedance at 10 MHz; the slip-ring contact resistance turns into both heat
and noise on the pair.

**This is not a reason to abandon two conductors. It is a reason to decide the
joint class first.** Three ways out, and they lead to different boards:

1. **Arm-class only on two wires.** 20–40 Nm, 5–10 A. The coupling network is
   ordinary, the slip ring is ordinary, and the architecture is as described in
   the brief. Legs then use something else.
2. **Two wires for data and logic, separate rings for motor current.** A slip
   ring with more than two contacts is normal; the brief's "two conductors"
   becomes "two signal conductors". Everything else in the brief survives.
3. **Raise the bus voltage.** 50 A at 48 V is 25 A at 96 V. This moves the
   problem rather than removing it, and 48 V was a stated requirement.

## 확인 필요

- X8-120 torque constant reference point (output vs rotor), as above.
- X8-120 and X6-60 peak *phase* current and how long peak may be held.
- Whether a Ø20 hollow-bore slip ring exists that passes 50 A on two contacts.
  No candidate found yet.
- Hollow bore: the brief wants Ø20 through the joint. The catalogue's Hollow
  Direct Drive series (p65) has not been read yet.
