# Hardware Test Plan

How to verify switches, coils, and lights against the real machine, a few at a time,
using MPF's built-in Service CLI. No game code required for any of this.

## Setup

Two terminal windows, both opened in this machine folder (where `config/config.yaml` lives).

**Terminal 1** — run MPF normally:
```
mpf
```

**Terminal 2** — connect the service tool to that running instance:
```
mpf service
```
This connects over BCP and puts the machine into service mode. Commands and names tab-complete.

Type `exit` or `quit` in Terminal 2 when you're done with a session.

## Testing switches

```
list_switches        # see every switch name MPF knows about
monitor_switches      # live view — now go press/trip switches by hand
```

`monitor_switches` updates in real time as you trigger inputs. Confirm the name that
lights up on screen matches the physical switch you just pressed.

Suggested order (a few at a time, report what doesn't match before moving on):

1. Cabinet switches you already know work (volume up/down, start, flippers) —
   confirms the tool itself is working.
2. `0804` board switches — gangway, trapdoor, Rudy's hideout, wind tunnel, dummy eject.
   These are the least-verified group (wiring sheet never got a "2026" revision for them).
3. `1616` board switches — locks, steps, ramp switches, gangway rollunder.
4. `3208` board switches — targets, bumpers, inlanes/outlanes, EOS switches.

## Testing coils

```
list_coils
coil_pulse c_knocker     # fires only that one coil
```

Go through a few at a time, e.g.:
```
coil_pulse c_bumper_up_l
coil_pulse c_bumper_up_r
coil_pulse c_bumper_lower
coil_pulse c_slingshot_left
coil_pulse c_slingshot_right
coil_pulse c_trap_closed
coil_pulse c_trap_door_open
coil_pulse c_steps_gate
coil_pulse c_dummy_eject
coil_pulse c_ramp_diverter
coil_pulse c_tunnel_kickbig
coil_pulse c_eyes_right
coil_pulse c_eyes_left
coil_pulse c_eyelids_open
coil_pulse c_eyelids_closed
coil_pulse c_mouth_motor
coil_pulse c_mouth_up_down
coil_pulse c_kickbig
coil_pulse c_multiball_release
```

Confirm the correct physical coil fires each time (sound/motion), not a neighboring one.

Note: flippers, trough, and auto launcher coils already work (confirmed), so no need to
re-test those unless something changes.

Bonus: a few coils are already wired to fire automatically off their switch via
`coil_player` (bumpers, slingshots, kickbig, dummy eject, tunnel kickbig) — for those
you can skip the service CLI and just trip the switch by hand while `mpf` runs normally.

## Testing lights

The config already runs a one-at-a-time chase show automatically on boot
(`l_p1` → `l_p32`, then `l_b1` → `l_b20`, 0.4s each, looping forever). Just run `mpf`
and watch — note the **last light number that actually lights up** before it goes dark
or stops making sense. That pinpoints whether it's a config address, an expansion
board/port issue, or a physical wiring gap.

To re-test one specific light on demand (e.g. after fixing a connection):

1. Comment out the `show_player: machine_reset_phase_3:` block at the bottom of
   `config/config.yaml` (it will otherwise fight with manual commands).
2. Restart `mpf`, then in the service CLI:
   ```
   list_lights
   light_color l_p5 red
   light_off l_p5
   ```
3. Uncomment the `show_player` block when done so the chase resumes on the next boot.

## Recording results

For each failed/mismatched item, note:
- The MPF name (e.g. `s_gangway_up_l`, `c_bumper_lower`, `l_p14`)
- What you expected vs. what actually happened
- The board/connector it's on (see comments in `config/config.yaml`)

Bring that list back for the next round of fixes.
