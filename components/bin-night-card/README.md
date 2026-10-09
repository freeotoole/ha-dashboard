# Bin Night Card

A three-tile (or any number) card using a user-defined Home Assistant calendar to show when each bin
needs to go out and when to bring it in. Tapping a tile toggles an `input_boolean` to mark the bin as
taken out.

<img alt="my bin night cards in home assistant" src="../../docs/screenshots/bin-night-card.png" width="460px">

## Dependencies

Install these frontend cards through HACS:

- [button-card](https://github.com/custom-cards/button-card)
- [decluttering-card](https://github.com/custom-cards/decluttering-card)

In Home Assistant, you also need:

- A calendar you can add your own events to, such as a Local Calendar, with entity ID
  `calendar.bin_night` (or update the sensor YAML to use your calendar's entity ID).
- The template sensor in `bin-night-schedule-sensor.yaml`, creating `sensor.bin_night_schedule`.
- Optionally, one `input_boolean` helper per bin (for example `input_boolean.green_bin_out`), created
  under Settings > Devices & services > Helpers > Toggle, to track bins taken out.

Waste Collection Schedule is no longer required.

## Calendar setup

Create an event for each collection, repeating at your collection frequency if appropriate:

- Set the event title to the bin's card `title`, exactly, including capitalisation and spaces. For
  the example grid, use `General Waste`, `Recycling` and `Green Waste`.
- Set the start to the collection date and time. Bin night is the calendar day before that start.
- Set the end to the deadline for bringing the bin back in. The card shows `Bring in` from the event
  start until its end.

For example, a `Green Waste` event starting at 06:00 on 13 October 2026 and ending at 20:00 that day
shows `Tonight` on 12 October, then `Bring in` during the event on 13 October. Use timed events to
choose these boundaries explicitly.

## Installation and files

1. Create the calendar events and any toggle helpers you want to use.
2. Merge the `template` configuration from `bin-night-schedule-sensor.yaml` into Home Assistant's
   `configuration.yaml`. If you already have a `template` list, append the supplied entry rather than
   adding a duplicate root key. If using a different calendar, replace all three occurrences of
   `calendar.bin_night` in the sensor file. Reload template entities or restart Home Assistant and
   check that `sensor.bin_night_schedule` has a `next_by_type` attribute.
3. Add the contents of `bin-night-card.template.yaml` beneath `decluttering_templates` in the
   dashboard's raw configuration, at the root alongside `views`. The file starts with
   `bin-night-card:`, so it is an entry in that mapping, not a complete dashboard configuration.
4. Paste the grid from `bin-night-card.yaml` or `bin-night-card.example.yaml` into a view's `cards`
   list. Adjust titles, helpers and colours. The `.example.yaml` version also displays titles.
5. If using toggle helpers, import `bin-night-card.automation.yaml` via Settings > Automations >
   Create > Edit in YAML. Set its calendar entity ID to the same calendar used by the sensor, and
   adjust the `helpers` mapping so each exact event title points to its bin's toggle helper.

The sensor fetches events on Home Assistant startup and every 15 minutes, covering 60 days from
today's midnight. Its state is the number of fetched events; its `next_by_type` attribute stores the
earliest event that has not ended for each event title, with `start` and `end` values. The card reads
this attribute from `sensor.bin_night_schedule`; there is no per-bin `collection_entity` variable.
Calendar edits and ended events can take until the next refresh to appear in the sensor.

If migrating from Waste Collection Schedule, replace the old reset automation with this calendar
automation and remove `collection_entity` from your card variables.

## Behaviour

- Before collection, the label shows `Tonight`, `Tomorrow`, or `In N days`. For bin nights more than
  14 days away, it shows the bin-night date as `DD/MM`. On the collection date before the event
  starts, it shows `Collection today`.
- During the event, the label shows `Bring in`, regardless of the helper's state, and tapping is
  disabled. Once an event ends, the sensor selects the next matching event on its next refresh.
- If the sensor has no matching event in its 60-day window, the label shows `No collection` and
  tapping is disabled.
- Otherwise, tap toggles `completion_entity` (`on` = taken out). When marked as taken out, the label
  shows `completed_label`. Without a valid helper, tapping is disabled.
- Completion is stored in the helper, so it persists across refreshes and devices.
- The reset automation turns off the matching helper when its calendar event ends, ready for the
  next collection. It ignores event titles outside the `helpers` mapping and queues simultaneous
  events so multiple bins can reset together. This uses the calendar end trigger directly and does
  not wait for the schedule sensor's next refresh.
- Hold opens the schedule sensor's more-info dialog.

To test the reset automation, create an event with a mapped title at least 15 minutes in the future
and turn its helper on. Check that the helper turns off when the event ends. Home Assistant polls
calendars every 15 minutes, so events added too close to their trigger time may be missed; see the
[calendar automation documentation](https://www.home-assistant.io/integrations/calendar/#automation).
If Home Assistant is offline when an event ends, clear that helper manually after restarting.

## Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `title` | Yes, to match an event | `Bin` | Exact calendar event title used to find the collection. Displayed when `show_title` is `true`. |
| `completion_entity` | No | Empty | `input_boolean` helper tracking whether the bin is taken out. Omit for a display-only tile. |
| `show_title` | No | `false` | Show the title above the label. |
| `icon` | No | `mdi:trash-can` | Tile icon. |
| `color` | No | HA card background | Tile background colour when not completed. |
| `text_color` | No | `var(--primary-text-color)` | Text colour, and icon colour when not completed. |
| `completion_style` | No | `background` | `background`, `icon` or `both`. Controls what changes when completed (see below). |
| `completed_color` | No | `#777777` | Background colour when completed (`background` and `both`). |
| `completed_icon_color` | No | `var(--secondary-text-color)` | Icon colour when completed (`icon` and `both`). |
| `completed_label` | No | `Taken out` | Label shown when completed, before the collection starts. |

### Completion styles

- `background`: background changes to `completed_color`.
- `icon`: background stays as `color`; the icon changes to `completed_icon_color`.
- `both`: background and icon both change.

Text stays as `text_color` in all completion styles.

## Example

With a calendar event titled `Green Waste`:

```yaml
- type: custom:decluttering-card
  template: bin-night-card
  variables:
    - completion_entity: input_boolean.green_bin_out
    - title: Green Waste
    - show_title: true
    - color: "#13BD3E"
    - text_color: "#111111"
```
