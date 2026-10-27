# gdnav — GPS-denied navigation for drones

**gdnav keeps a drone flying its mission when GPS is jammed.** It works out where the drone is from its own
onboard sensors and hands that position to the autopilot (**PX4** or **ArduPilot**) in place of GPS. When the
remote control and the ground link are jammed too, gdnav decides by itself when to come home early, when to stop
and wait, and when to land.

> **Status:** simulation only (PX4 / ArduPilot SITL + Gazebo Harmonic). Not flight-tested on real hardware yet.
>
> **About this repository:** a public summary of gdnav's results. The software itself is private and proprietary.

---

## Results

1.2 km square mission at 40 m, realistic sensor errors, live closed loop (gdnav steers the autopilot). "0.3 %" =
the autopilot's worst position error, 0.3 m per 100 m flown since GPS was lost.

| Scenario | Without gdnav | With gdnav (PX4) | With gdnav (ArduPilot) |
|---|---|---|---|
| GPS jammed 30 s after start, still climbing | **288 m** off, blind landing **88 m** from home | **0.09 %**, landed 0.4 m from home | **0.21 %**, 1.7 m |
| GPS jammed at the first corner | — | 0.29 %, 0.9 m | 0.18 %, 1.7 m |
| + 30 s of camera blackout (calm / 5 m/s wind) | — | 0.24 % / 0.25 % | 0.16-0.22 % / 0.32 % |
| GPS **+ remote control + ground link** jammed | — | mission completed, 0.8 m from home | mission completed, 1.7 m |
| GPS comes back 60 s later | — | back on its own GPS, 0.1 m | stays on gdnav, 1.6 m |

**When every link is jammed and the camera is blinded on purpose**, on both autopilots:

| gdnav's confidence in its position | What happens | Measured |
|---|---|---|
| confident | the mission goes on | — |
| unsure | flies home early, to the nearest of home or a rally point | landed 4.7-6.6 m from the rally point |
| lost | holds position and waits for the camera | camera back → home 4.3-6.3 m; never back → lands, within 1.5 σ of its own estimate |

**Honest uncertainty:** gdnav's stated uncertainty (σ) is checked against the simulator's ground truth with
NEES / NIS: 0 % of samples outside the 99.9 % bound in the acceptance flights. **Small enough for a Raspberry
Pi 5** (estimate: main thread 24 % of one Pi core, camera frame p99 24 ms).

## Tests

**260 automated tests, all passing** (36.7 s). They cover the position estimate, the camera input, the PX4 and
ArduPilot interfaces, the failsafe behaviour and the safety checks. The test code is private.

## How the results were measured

Each flight starts a fresh simulator (PX4 v1.17.0 or ArduCopter 4.6.3, with Gazebo Harmonic), flies a 4 × 300 m
square at 40 m, jams GPS, finishes the mission and returns home (about 6 minutes). Errors are measured against
the simulator's ground truth.

## Coming next

- **Rangefinder:** a downward rangefinder for height above ground that keeps working in smoke, fog and dust.
- **SLAM:** building a map while flying, on a Jetson-class computer, for long flights without GPS.

Scope: navigation and safe return only — no targeting, interception or radio-spoofing features.

## Licence

gdnav: © SarthakDz. **Not open source** — proprietary, all rights reserved. No part of gdnav's own code, tests or
results may be copied, used or redistributed without written permission.
