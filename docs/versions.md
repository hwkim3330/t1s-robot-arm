# Versions — what gets built, in what order, and what each one proves

Three boards, not one. The joint board is the product; the other two exist so
that the joint board is not the place where something is tried for the first
time.

Every board is done only when `run_all.sh` reports NETLIST MATCH, unrouted 0 and
DRC error 0 — and that is the floor, not verification. See CLAUDE.md.

---

## BENCH v0.1 — prove the parts, shaped for probing

**Purpose:** answer the questions that are still open, on hardware, before
anything is committed to an annulus.

| | |
|---|---|
| form | rectangular, generous, test points everywhere |
| MCU | **ESP32-S31**, module or chip — does not matter here |
| bus | **LAN8670** on RMII |
| power | bench supply at 48 V, no PoDL coupling yet |
| motor | one half-bridge set driving a small BLDC on the bench |

**What it has to answer:**

1. Can the S31 ADC sample two phase currents **in step with the MCPWM**? This
   is the open item that decides whether an external ADC lands on the joint
   board, and it has been unanswered through three MCU decisions.
2. Does ESP-IDF drive S31 Ethernet and `lan867x` together, today?
3. Does the FOC loop hold 1 kHz on one HP core with the bus on the other, and
   what is the jitter?
4. What does a PLCA round actually cost, with the number of nodes planned?

**Not on this board:** PoDL coupling, slip ring, the annular form, 48 V from a
battery. Those are separate risks and they get their own board.

---

## NODE v0.2 — the coupling network, on its own

**Purpose:** prove 48 V and 10 Mbps sharing one pair, before that circuit is
buried inside a joint.

| | |
|---|---|
| form | rectangular, still probeable |
| input | **48 V**, 60 V-rated buck, TVS, fuse |
| coupling | power-extraction inductors, DC-blocking capacitors, local bulk |
| options | 2-wire / 4-wire jumper, common-mode choke footprint |
| pads | slip-ring wiring pads |

Sized for the **J40** current band (5–15 A), because D1 keeps the literal
two-wire form only for the arm. The J120 motor bus does not pass through this
network at all — it gets its own slip-ring contacts.

**What it has to answer:** what the PHY sees with 48 V on the pair, what the DC
path sees at hot-plug, and whether the inductors stay out of saturation at the
arm's peak.

---

## JOINT v0.3 — the annulus

**Purpose:** the product. Only proven circuits go on it.

| | |
|---|---|
| form | **annulus**, clearing a Ø20 bore, inside a Ø98–130 housing |
| MCU | **bare ESP32-S31** (D7 — no module in the joint) |
| bus | LAN8670, RMII |
| power | coupling network from NODE v0.2, scaled to the variant |
| drive | 3-phase, current sense per phase |
| sensing | absolute encoder on the **output shaft**, multi-turn count |
| safety | local watchdog: bus goes quiet, torque goes to zero |
| radio | **none** (D5) |

---

## GATEWAY v0.1 — the body

| | |
|---|---|
| MCU | ESP32 **module**, pre-certified (D5 — the only RF work in the machine) |
| radio | Wi-Fi: diagnostics, OTA, PC link |
| bus | T1S master end, PLCA coordinator |
| software | **zenoh-pico**, bridging T1S frames to Zenoh for the PC (D6) |
| power | 48 V battery box, per-port eFuse, main switch, emergency stop |

---

## The joint variants

Same board, same firmware, same port face. Different motor, gear and housing
diameter.

| | **J40** (arm) | **J120** (leg) | **J40-2W** (arm, strict) |
|---|---|---|---|
| peak torque | 40 Nm | 120 Nm | 40 Nm |
| reduction | ~1:9 | ~1:20 | ~1:9 |
| bus current, peak | ~15 A | ~50 A | ~15 A |
| conductors through the joint | 2 signal + motor rings | 2 signal + motor rings | **2, total** |
| backdrivable | yes | reduced | yes |

**J40-2W** is the brief's original architecture, kept because at arm currents it
is buildable. **J120** is why D1 split the slip ring: 50 A through two contacts
carrying 10 Mbps is not a thing that works.

---

## Order of work

    BENCH v0.1  ──answers the ADC question──┐
                                            ├──> JOINT v0.3
    NODE  v0.2  ──answers the coupling──────┘
    GATEWAY v0.1 can proceed in parallel; it shares no unknowns with the joint.

**Nothing is ordered for JOINT v0.3 until BENCH v0.1 has answered question 1.**
Three MCU decisions have now been made and reversed without it, which is
exactly the cost of leaving it open.

---

## The open items, gathered

| # | question | blocks |
|---|---|---|
| 1 | S31 ADC: sample rate, and MCPWM-synchronised conversion | BENCH, then everything |
| 2 | Confirm MII/RMII from the datasheet's own interface table | D9 itself |
| 3 | Slip ring: Ø20 bore, signal pair + high-current rings. **No candidate part found at all** | J120, and the infinite-rotation premise |
| 4 | Housing machining quote, 1 off and 10 off | whether this is affordable |
| 5 | X8-120 purchase price | build-versus-buy, honestly |
| 6 | Second source for a T1S PHY | committing to a footprint |
| 7 | Peak joint speed and duty from the gait | every current figure |

Item 3 is the one with no supplier at all. If no such slip ring exists, the
infinite-rotation requirement needs rethinking before the mechanics are drawn.
