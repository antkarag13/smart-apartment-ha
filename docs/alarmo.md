# Alarmo setup

Alarmo stores its sensor settings in its own panel, not in YAML, so this is
the table to reproduce there (**Alarmo → Sensors**).

| Sensor | Away | Night | Entry delay | Notes |
|---|:---:|:---:|:---:|---|
| Entrance door contact | ✓ | ✓ | ✓ 30 s | the only entry/exit route; add one if it isn't fitted yet |
| Window contacts (label `windows`) | ✓ | ✓ | — | instant |
| Balcony door contacts | ✓ | ✓ | — | instant |
| FP300 living room, kitchen | ✓ | — | ✓ 30 s | on the way to the keypad |
| FP300 bedrooms | ✓ | — | — | people sleep here in Night mode |
| FP300 bathroom, WC | ✓ | — | — | |
| SNZB-06P storage | ✓ | — | — | |

**Modes**

| Mode | Exit time | Entry time | Trigger time |
|---|---|---|---|
| Away | 60 s | 30 s | 180 s |
| Night | 0 s | 30 s (entrance door only) | 180 s |

Leave **"Allow open"** off on the window and balcony contacts, so Alarmo refuses
to arm with one open; `security_open_windows_on_leave` tells you which one.

Put the disarm code in `secrets.yaml` as `alarmo_code`. It is used only for
the automatic disarm when Night mode ends, i.e. when someone is already
inside and has been seen in the kitchen or living room.
