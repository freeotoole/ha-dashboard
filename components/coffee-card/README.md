# Coffee Card

A Home Assistant Bubble Card for the coffee machine switch, with an estimated
warm-up status and a progress gradient for heating and cooling.

## Dependencies

- [Bubble Card](https://github.com/Clooos/Bubble-Card)

## Files and setup

- `coffee-card.config.yaml`: merge the `input_datetime` helpers and template
  sensors into Home Assistant's configuration. Merge into existing
  `input_datetime` and `template` sections instead of defining duplicate root
  keys.
- `coffee-card.automations.yaml`: add the automation to Home Assistant.
- `coffee-card.yaml`: add the card to the dashboard's card list. Change
  `switch.coffee_machine` if your machine uses a different switch entity.

The configuration defines three helpers:

- `input_datetime.coffee_machine_last_off` records when the machine was switched
  off.
- `input_datetime.coffee_machine_started_at` records when it was switched on.
- `input_datetime.coffee_machine_ready_at` stores the current heating-ready or
  cooling-complete time.

## State and progress

When switched on, the status is `heating` until the estimated ready time, then
`ready`. The warm-up estimate is based on the time since the last switch-off:

| Time off | Estimated warm-up |
| --- | --- |
| Under 15 minutes | Ready immediately |
| 15 to under 30 minutes | 5 minutes |
| 30 to under 60 minutes | 10 minutes |
| 60 to under 90 minutes | 15 minutes |
| 90 to under 120 minutes | 25 minutes |
| 120 minutes or more | 40 minutes |

When switched off, status is `cooling` for 120 minutes, then `cold`. The
`sensor.coffee_machine_progress` reports heating progress while on and elapsed
cooling progress while off, clamped from 0% to 100%.

The card displays `sensor.coffee_machine_status`. Its icon gradient advances
while heating and recedes while cooling; the background and gradient colours
also change with the switch state.
