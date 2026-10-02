# T1S Robot Arm — working notes for Claude

A modular robot built from one joint design, wired with two conductors per
joint. The long-term target is an electric Atlas-class machine; the near-term
target is one joint module that is honest about its numbers.

## The constraints that shape everything

**Every joint turns forever.** Slip ring for power and data, absolute encoder
on the output shaft, turn counting in firmware. No cable wrap, no end stops,
no "unwind before you continue". This is why the bus is two wires and not a
harness.

**Two conductors through the joint.** One 10BASE-T1S pair carries data and 48 V
at the same time (PoDL: inductors lift the DC off the pair, series capacitors
keep it out of the PHY). Four-wire is a debug option on a jumper, never the
shipping configuration.

**Torque comes from current, not from gearing.** QDD: a large-diameter BLDC
through a 1:6–1:9 metal planetary. Backdrivable, survives impact, no harmonic
drive. Leg-class joints are the design point; arm-class is the same module
turned down.

**ESP32 outside, T1S inside.** No Raspberry Pi anywhere. Wi-Fi is for
diagnostics, OTA and as an emergency-stop fallback — never in the control loop.
IK, vision and learning run on a PC; the 1 kHz loop runs on the joint MCU.

## Numbers that are assumptions, not facts

These came from the project brief and have **not** been derived yet. Recompute
them in step 3 and replace this section with the calculation:

- leg joint peak torque 80–150 Nm
- arm joint peak torque 20–40 Nm

Anything else that a datasheet has not confirmed goes in the relevant README
under **확인 필요**, with the specific question written out. Do not round a
guess into a specification.

## House rules

- Commit at each step, with the reasoning in the message rather than the diff.
- Re-check part price and stock immediately before ordering; both move.
- Never commit tokens, keys, or anything else secret.
- Generated artefacts (gerbers, STEP, reports) are committed, because the
  point of generating them is that the result can be diffed.
- A board is not done until `run_all.sh` reports **NETLIST MATCH**, **unrouted
  0** and **DRC error 0**. Those three lines go in the commit message.

## Not in scope here, and must not be treated as done

Structural strength, brakes, emergency stop behaviour, and pinch/collision
safety need their own review before this machine operates near a person.
Nothing in this repository has had that review.
