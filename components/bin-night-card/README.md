# Bin Night Card

A three-tile (or any number) card showing how many nights until each bin needs to go out. Tapping a
tile toggles an `input_boolean` to mark the bin as taken out.

<img alt="my bin. night cards in home assistant" src="../../docs/screenshots/bin-night-card.png" width="460px">

## Dependencies

Requires the following HACS add-ons

- [button-card](https://github.com/custom-cards/button-card)
- [decluttering-card](https://github.com/custom-cards/decluttering-card)

Requires the following Home Assistant integration and helpers

- [Waste Collection Schedule](https://github.com/mampfes/hacs_waste_collection_schedule): one sensor per
  bin type (for example `sensor.waste_collection_schedule_green_bin`). The sensor state must contain
  the next collection date as `DD.MM.YYYY` (for example `on Tue, 13.10.2026`) or `YYYY-MM-DD`.
- One `input_boolean` helper per bin (for example `input_boolean.green_bin_out`), created under
  Settings > Devices & services > Helpers > Toggle.

## Files

- `bin-night-card.template.yaml`: merge `decluttering_templates` into the dashboard's raw configuration
  at the root, alongside `views`. Merge into an existing template mapping rather than adding a
  duplicate root key.
- `bin-night-card.yaml`: example grid of three bins; paste into a view's `cards` list and replace the
  entities, titles and colours.

## Behaviour

- Bin night is the night before the collection date. The label shows `Tonight`, `Tomorrow`,
  `In N nights`, or `Collection today`.
- Tap toggles `completion_entity` (`on` = taken out). Hold opens its more-info dialog.
- Completion is stored in the helper, so it persists across refreshes and devices.
- 📋 **Helpers are currently not reset automatically** - An automation to turn them off after collection is coming.

## Variables

| Variable               | Required | Default                                        | Description                                                                       |
| ---------------------- | -------- | ---------------------------------------------- | --------------------------------------------------------------------------------- |
| `completion_entity`    | Yes      |                                                | `input_boolean` helper tracking whether the bin is taken out.                     |
| `collection_entity`    | Yes      |                                                | Waste Collection Schedule sensor with the next collection date.                   |
| `title`                | No       | `Bin`                                          | Tile name. Only displayed when `show_title` is `true`.                            |
| `show_title`           | No       | `false`                                        | Show the title above the label.                                                   |
| `icon`                 | No       | collection sensor's icon, else `mdi:trash-can` | Icon override.                                                                    |
| `color`                | No       | `#777777`                                      | Tile background colour (the bin colour).                                          |
| `text_color`           | No       | `var(--primary-text-color)`                    | Text and icon colour when not completed.                                          |
| `completion_style`     | No       | `background`                                   | `background`, `icon` or `both`. Controls what changes when completed (see below). |
| `completed_color`      | No       | HA card background                             | Background colour when completed (`background` and `both`).                       |
| `completed_icon_color` | No       | `color`                                        | Icon colour when completed. Falls back to `color` if empty.                       |
| `completed_label`      | No       | `Taken out`                                    | Label shown when completed.                                                       |

### Completion styles

- `background`: background changes to `completed_color` and text changes to `color`.
- `icon`: background stays as `color`; the icon changes to `completed_icon_color`.
- `both`: background, text and icon all change.

## Example

```yaml
- type: custom:decluttering-card
  template: bin-night
  variables:
    - completion_entity: input_boolean.green_bin_out
    - collection_entity: sensor.waste_collection_schedule_green_bin
    - title: Green Waste
    - color: "#13BD3E"
    - text_color: "#111111"
```
