# Sampling phase current in step with the PWM — answered

This was open item 1 through three MCU decisions and two reversals. It is the
question that decides whether an external ADC lands on the joint board, and it
has an answer that also settles why the S31 is the right part for a better
reason than price or core count.

## The mechanism is ETM

Espressif's **Event Task Matrix** routes a peripheral *event* to a peripheral
*task* in hardware, with no CPU in between. The ESP-IDF documentation says both
halves of what is needed:

- an **ADC conversion start is an ETM task** — the docs give "starting an ADC
  conversion when a pulse edge is detected" as an example of what ETM is for
- **MCPWM timers and comparators are ETM event sources** —
  `mcpwm_timer_new_etm_event()` and `mcpwm_comparator_new_etm_event()`

So:

    MCPWM comparator event  ->  ETM channel  ->  ADC conversion start

No interrupt, no software latency, no jitter from whatever else the core was
doing. And because the trigger comes from a **comparator**, the sample lands at
a chosen point inside the PWM period rather than merely at its edge — which is
exactly what FOC needs, since phase current must be read while the low side is
conducting.

This is the ESP32 family's equivalent of the timer-triggered ADC that makes
STM32G4 the reference part for motor control. [D3](decisions.md) conceded that
ground to the G4. With ETM it is much less conceded than it looked.

Sources: [ETM, ESP32-P4](https://docs.espressif.com/projects/esp-idf/en/stable/esp32p4/api-reference/peripherals/etm.html),
[MCPWM, ESP32-P4](https://docs.espressif.com/projects/esp-idf/en/stable/esp32p4/api-reference/peripherals/mcpwm.html).

## Why this decides the MCU, not just the ADC

**ETM arrived with the C6. The ESP32-S3 does not have it.**

That reframes D8, which chose the S3 on price and stock. On an S3 the choice
would have been:

- start conversions from the MCPWM interrupt — software latency, and jitter
  equal to whatever else the CPU was doing, injected directly into the current
  measurement; or
- put an external ADC on the joint board — more parts, more board area on an
  annulus that has none, and the part-count argument gone

Neither is good, and the S31 has the peripheral that makes both unnecessary.
**D9 was right for a better reason than the one given.** Price and core count
were the stated grounds; this is the real one.

## What is confirmed and what is not

**Confirmed:** ETM links MCPWM events to ADC tasks, on chips that have ETM
(C6, H2, P4), through documented ESP-IDF APIs.

**확인 필요, and it is the last piece:**

- The S31 datasheet is **preliminary v0.5**. It lists MCPWM among ETM-capable
  peripherals but does not give the ADC's sampling rate, nor confirm the ADC as
  an ETM task target on *this* chip. The Technical Reference Manual must.
- ADC maximum sample rate, and conversion time against the PWM period.
- Whether both phase currents can be sampled close enough together to be
  treated as simultaneous, or whether the two ADC units must be used in
  parallel. Two units with four differential pairs each suggests yes.

Until the TRM confirms it, BENCH v0.1 exists to measure it. That has not
changed — but the board is now being built to confirm a documented mechanism
rather than to discover whether one exists.
