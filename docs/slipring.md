# The slip ring — the premise holds

Open item 3 in [versions.md](versions.md) had no candidate part from any
supplier, and without one the infinite-rotation requirement — the thing this
whole architecture is built around — had no foundation. It has one.

## What exists off the shelf

**Ø20 through-bore, standard catalogue parts:**

| | |
|---|---|
| H2042 | ID 20 mm, OD 42 mm |
| H2056 | ID 20 mm, OD 56 mm |
| circuits | 6–24 wires |
| current | **5 A or 10 A per ring** |
| mixing | power and signal in the same body is standard |

**Larger and higher current:** hollow-shaft slip rings run from Ø3 to Ø300 bore
at **2 A to 500 A per ring**, so the current ceiling is a question of how much
ring you are willing to pay for, not of whether it can be done.

**Data:** the rotarX CER/CEQ series carries power, signal and **real-time
Ethernet at up to 1 Gbps** through the rotation, in one body. Our bus is
10 Mbps. **Ethernet through a slip ring is ordinary commercial practice**, and
T1S asks two orders of magnitude less than what is already sold.

Sources: [Senring through-hole](https://www.senring.com/through-hole-slip-ring/),
[rotarX through-bore](https://www.rotarx.com/en/slip-rings/through-bore-slip-rings/).

## What this means per variant

**J40 (arm, ~15 A peak)** — solved with a catalogue part. An H2056 at 10 A per
ring covers 15 A on two paralleled rings, leaves circuits for the T1S pair, and
clears the Ø20 bore. Nothing custom.

**J40-2W (strict two conductors)** — the brief's original form. Two rings total,
both carrying the pair with 48 V on it. Within a standard part's ratings.

**J120 (leg, ~50 A peak)** — not a stock Ø20 part. Standard Ø20 bodies top out
around 10–20 A. Two ways on, both ordinary engineering:

1. **Parallel the rings.** Five or six 10 A rings make 50 A. This is how
   high-current slip rings are normally built, and an OD 56 body has the
   circuit count for it with the pair still separate.
2. **Larger body.** Bore stays Ø20; outer diameter grows until the rings are
   rated for the current. Costs housing diameter, which the joint has to give.

**확인 필요:** contact resistance per ring, and therefore the heat at 50 A, and
what paralleling does to current sharing between rings. That is the number that
decides between (1) and (2), and no quote has been requested yet.

## The risk that is actually left

Not existence — **noise**. The T1S pair runs through sliding contacts a few
millimetres from rings switching tens of amps at the inverter's frequency. A
1 Gbps product proves it can be done, but those products are engineered for it:
separated rings, shielding, grounded guard rings between power and data.

So the pair's rings must be specified as **shielded and physically separated
from the power rings**, and that is a requirement on the slip ring order, not
something the PCB can fix afterwards.

This moves the infinite-rotation premise from *unfounded* to *an ordinary
procurement problem with one specification to get right*.
