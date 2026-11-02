# Devices, areas and entity IDs

The apartment is being renovated. This is the target setup, taken from the
smart-home sheet (12γ) of the renovation drawings, and the entity IDs the
automations expect. When the devices are paired, rename each entity to the ID
below and every automation works without edits.

## Bill of materials

| Qty | Device | Role |
|---:|---|---|
| 10 | SONOFF ZBMINIR2 (neutral) / ZBMINIL2 (no neutral) | 1-gang Zigbee relay behind a wall switch |
| 2 | SONOFF MINI DUO (MINI-ZB2GS) | 2-gang Zigbee relay, kitchen and entrance |
| 7 | Aqara FP300 | mmWave + PIR presence, illuminance, temperature, humidity |
| 1 | SONOFF SNZB-06P | mmWave presence, storage room (USB 5 V) |
| 2 | Power-metering smart plug, 16 A | washing machine (SP1), chest freezer (SP2) |
| 2 | Zigbee water leak sensor | storage room floor (by the washer), under the kitchen sink |
| 1 | Zigbee temperature sensor with wired probe | inside the chest freezer |
| 1 | SONOFF POWCT | whole-apartment power on the main line (Wi-Fi) |
| 1 | DIN-rail power relay | water heater, own line |
| 1 | Zigbee coordinator | near the panel / router |
| — | Magnetic contacts (existing) | every window and balcony door, used by Alarmo |
| 5 | Ducted indoor units (multi-split) | one per room, climate entities |

Every relay and plug is mains-powered, so each one is also a Zigbee router and
the mesh gets stronger with every room.

## Areas

Create these areas on one floor with the ID `apartment`, and assign every
device to its room. Area and entity IDs stay generic so the config is reusable;
the room names below are what Home Assistant and the dashboard show. Automations use area targets, so a sensor or light added
later is picked up without editing any YAML.

| Area ID | Room (drawing 9γ) |
|---|---|
| `study_bedroom` | Bedroom 1 (Υ/Δ πρώην κουζίνα): single bed + desk |
| `small_bedroom` | Bedroom 2 (Υ/Δ μικρό) |
| `double_bedroom` | Master bedroom (Υ/Δ δίκλινο) |
| `hallway` | Διάδρομος |
| `bathroom` | Main WC (Λουτρό) |
| `wc` | Master WC, en-suite to the master bedroom |
| `storage` | Αποθήκη |
| `living_room` | Ενιαίος χώρος: καθιστικό, κουζίνα, τραπεζαρία |
| `balcony_east` | Ανατολικός εξώστης |
| `balcony_south` | Νότιος εξώστης |

Give every window and balcony-door contact the label **`windows`**.

## Entity map

### Lights (relays)

Expose each relay as a light: **Settings → Devices & services → Helpers →
Create helper → Change device type of a switch → Light**.

| Relay | Entity ID | Load | Switch |
|---|---|---|---|
| R1 | `light.study_bedroom_ceiling` | ceiling | three-way |
| R2 | `light.small_bedroom_ceiling` | ceiling | single |
| R3 | `light.double_bedroom_ceiling` | ceiling | three-way |
| R4 | `light.hallway` | hallway lights | three-way |
| R5 | `light.bathroom` | ceiling + mirror | single |
| R6 | `light.storage` | ceiling | single |
| R7 | `light.wc` | ceiling | single |
| R8 | `light.kitchen_counter_led` | under-cabinet LED | push button on the counter |
| R9 | `light.balcony_east` | balcony light | by the balcony door |
| R10 | `light.balcony_south` | balcony light | by the balcony door |
| D1·1 | `light.kitchen_ceiling` | kitchen ceiling lights | double |
| D1·2 | `light.kitchen_island_pendant` | island pendant | double |
| D2·1 | `light.living_room_ceiling` | living room | three-way with the entrance |
| D2·2 | `light.dining_pendant` | dining pendant | double |

### Sensors

| Sensor | Entity IDs |
|---|---|
| F1 FP300, study bedroom | `binary_sensor.study_bedroom_presence`, `sensor.study_bedroom_illuminance` |
| F2 FP300, small bedroom | `binary_sensor.small_bedroom_presence`, `sensor.small_bedroom_illuminance` |
| F3 FP300, double bedroom | `binary_sensor.double_bedroom_presence`, `sensor.double_bedroom_illuminance` |
| F4 FP300, bathroom | `binary_sensor.bathroom_presence` |
| F5 FP300, WC | `binary_sensor.wc_presence` |
| F6 FP300, dining (by the front door) | `binary_sensor.dining_presence`, `sensor.dining_illuminance` |
| F7 FP300, kitchen + living room | `binary_sensor.kitchen_presence`, `sensor.kitchen_illuminance` |
| F6 + F7 together | `binary_sensor.living_room_presence`: group defined in `lighting.yaml`; it has no device, so assign it to area `living_room` by hand |
| FP300 temperature / humidity | `sensor.<room>_temperature`, `sensor.<room>_humidity` for study_bedroom, small_bedroom, double_bedroom, kitchen (F7), dining (F6) |
| S1 SNZB-06P, storage | `binary_sensor.storage_presence` |
| Balcony door contacts | `binary_sensor.balcony_east_door`, `binary_sensor.balcony_south_door` (device class `door`, label `windows`) |
| Window contacts | `binary_sensor.study_bedroom_window`, `binary_sensor.small_bedroom_window`, `binary_sensor.double_bedroom_window`, `binary_sensor.bathroom_window`, `binary_sensor.kitchen_window`, `binary_sensor.living_room_window` (device class `window`, label `windows`) |
| Leak sensors | `binary_sensor.storage_leak` (area `storage`), `binary_sensor.kitchen_sink_leak` (area `living_room`) |
| Freezer probe | `sensor.freezer_temperature` |

The FP300 temperature drives the room temperature control (each unit is
corrected so the *room* reaches its target) and the humidity drives the mould
check. Bathroom and WC FP300s only report presence to the automations.

### Energy, climate, alarm

| Device | Entity IDs |
|---|---|
| POWCT | `sensor.mains_power` (W), `sensor.mains_energy` (kWh) |
| Water heater relay | `switch.water_heater` |
| SP1 washing machine | `switch.washing_machine`, `sensor.washing_machine_power` |
| SP2 chest freezer | `switch.freezer`, `sensor.freezer_power` |
| A/C units E1–E5 | `climate.study_bedroom`, `climate.small_bedroom`, `climate.double_bedroom`, `climate.living_room` (E4), `climate.kitchen_dining` (E5) |
| Alarmo | `alarm_control_panel.home` |
| Weather | `weather.forecast_home` (default Met.no integration) |
| Notifications | `notify.family`: a notify group helper with everyone's phones, for plain messages · `notify.family_phones`: legacy group in `security.yaml` with the `mobile_app_*` actions, for notifications with buttons |

## Installation notes (from the electrical drawings)

- Bring phase **and neutral** to every switch box and use deep boxes (≥ 50 mm).
  Every room gets a new circuit, so all boxes should have neutral and
  ZBMINIR2 everywhere. ZBMINIL2 only where an old cable stays; it needs a
  minimum load, so very small LED lamps may need a bypass capacitor.
- FP300 at about 2.0–2.4 m in a corner, aimed into the room. Never aim it at
  A/C grilles or curtains: their movement reads as presence on mmWave. With
  ducted units this matters: check where each supply grille blows.
- The POWCT is Wi-Fi, not Zigbee.
- Freezer probe: keep the sensor body **outside** the freezer and pass only the
  thin probe cable under the lid seal. At -18 °C a coin cell dies within weeks
  and the Zigbee signal can't get out of the metal box.
- Leak sensors lie flat on the floor at the lowest point, where water would
  collect first.
