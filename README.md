# Smart Apartment: Home Assistant config

Home Assistant configuration for an apartment I'm renovating, written **before
the build** from the renovation drawings. Every light switch gets a Zigbee
relay, every room gets an mmWave presence sensor, and the heat pump, water
heater and appliances are metered and controlled, so on day one the
apartment runs itself.

> Status: design ready, renovation in progress. Entity IDs are the target
> names in [docs/devices.md](docs/devices.md); pair the devices, rename them,
> and the automations work unchanged.

## What it does

| | |
|---|---|
| **Presence lighting in every room** | One reusable blueprint: lights follow the mmWave sensor, switch on only when it's dark, and never fight a person: switch a light off by hand and it stays off until the room empties. |
| **Night-aware** | In Night mode bedroom lights stay off, and the kitchen only lights its counter LED, enough for a glass of water. |
| **House modes** | `Home` / `Night` / `Away` / `Vacation`, worked out from phones and from the day zone going quiet after 22:30. Everything else reacts to the mode, not to raw sensors. |
| **Alarm that arms itself** | Alarmo arms Away when the last person leaves and Night when the house goes to bed; it lists open windows; it only disarms itself from Night mode, when someone is already inside. |
| **Per-room climate** | Five ducted units. An open window pauses that room's unit and restores the exact settings when it closes; a room empty for 45 min switches its unit off (bedrooms excepted at night). |
| **Water heater reconciler** | Schedule + 1 h boost button + house mode + load shedding, combined into one "should it be on?" rule that is re-applied every 10 min, so a missed event can never leave it on all day. |
| **Load shedding** | Whole-apartment power from a POWCT: above the limit, the water heater pauses before the main breaker trips. |
| **Appliances** | "Wash finished" notification without false alarms during soak pauses; freezer watchdog that also refuses to let its plug be switched off. |
| **Maintenance** | A/C filter reminders (60 days), weekly low-battery report, alert when any mains-powered Zigbee device drops off the mesh. |

## How the pieces fit

```mermaid
flowchart TB
  subgraph Sensors
    FP["7× Aqara FP300<br/>presence · lux · temp · humidity"]
    SN["SNZB-06P<br/>storage"]
    WC["Window / balcony<br/>door contacts"]
    PCT["POWCT<br/>mains power"]
    PL["Smart plugs<br/>washer · freezer"]
    PH["Phones<br/>(zone.home)"]
  end

  MODE{{"input_select.house_mode<br/>Home · Night · Away · Vacation"}}

  subgraph Actuators
    REL["14 light channels<br/>ZBMINIR2/L2 · MINI DUO"]
    AC["5 ducted A/C units"]
    WH["Water heater relay"]
    AL["Alarmo"]
  end

  PH --> MODE
  FP --> MODE
  FP -- presence blueprint --> REL
  SN --> REL
  MODE --> REL & AC & WH & AL
  WC -- pause / restore --> AC
  WC --> AL
  FP -- empty room --> AC
  PCT -- load shedding --> WH
  PL --> N["notify.family"]
```

## Repository layout

```
blueprints/automation/antkarag13/
  presence_lighting.yaml    reusable presence-lighting blueprint
packages/                   one self-contained file per feature (helpers + automations)
  house_modes.yaml
  lighting.yaml             8 blueprint instances, hallway, balconies
  security.yaml             Alarmo
  climate.yaml              window pause/restore, empty-room off, away
  energy.yaml               water heater, load shedding, washer, freezer
  maintenance.yaml          filters, batteries, offline devices
docs/
  devices.md                bill of materials, areas, entity map, install notes
  alarmo.md                 which sensor is armed in which mode
configuration.example.yaml  how to load the packages
secrets.example.yaml
```

## Design choices

- **Areas and labels over entity lists.** Climate and balcony automations find
  "the unit in the same room as this window" with `area_id()`, and window
  checks use the `windows` label, so new devices need no YAML edits.
- **Purpose-specific triggers** (`occupancy.detected`, `window.opened`,
  `schedule.block_started`, `timer.finished`, HA 2026.7+) where they fit, with
  plain `to:`-only state triggers where an edge through `unavailable` must
  never be missed.
- **Reconcile, don't toggle.** The water heater automation computes the
  desired state from all inputs and applies it, instead of separate on/off
  automations that can drift.
- **Snapshot and restore.** An open window saves the unit's exact settings
  with `scene.create` and restores them, rather than guessing.
- **Every physical actuator has a failsafe.** Hallway auto-off, freezer plug
  that turns itself back on, offline-device alerts.

## Using it

1. Copy `packages/` and `blueprints/` into your Home Assistant `/config`.
2. Merge `configuration.example.yaml` into `configuration.yaml`.
3. Copy `secrets.example.yaml` to `secrets.yaml` and set the Alarmo code.
4. Create the areas, the `apartment` floor and the `windows` label, and rename
   entities as in [docs/devices.md](docs/devices.md).
5. Set up Alarmo as in [docs/alarmo.md](docs/alarmo.md).

Requires Home Assistant 2026.7 or newer and the [Alarmo](https://github.com/nielsfaber/alarmo) integration (HACS).

## License

[MIT](LICENSE) © 2026 Antonis Karagiannis
