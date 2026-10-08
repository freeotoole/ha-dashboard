# Dynamic Person Card

<img alt="my dynamic person cards in home assistant" src="../../docs/screenshots/dynamic-person-card.png" width="460px">

## Dependencies

Requires the following HACs addons

- bubble-card
- mushroom-card
- stack-in-card
- card-mod

## Files

The reusable component is `dynamic-person-card`; each instance references it with
`template: dynamic-person-card`.

- `dynamic-person-card/dynamic-person-card-templates.yaml`: merge `decluttering_templates` into the
  dashboard's raw configuration at the root, alongside `views`. Merge into an
  existing template mapping rather than adding a duplicate root key.
- `dynamic-person-card/dynamic-person-card.yaml`: Bunny's replacement card; paste into the same
  position as the original card.
- `dynamic-person-card/dynamic-person-card-example.yaml`: second-person example with map navigation
  and a light toggle; replace the example entities and avatar paths before use.

For YAML-mode dashboards, copy the files into your HA configuration directory
and merge the template mapping into the dashboard root. UI-managed dashboards
require pasting the mapping into the raw configuration editor. The template file
is dashboard configuration, not an individual card for a view's `cards` list.

## Editing

Make common visual changes in `dynamic-person-card/dynamic-person-card-templates.yaml`: background,
gradient, avatar sizing, spacing, button styling and avatar tap behaviour.
The template retains the original stack with two Bubble rows surrounding the
Mushroom avatar. Each instance supplies independent `top_buttons` and
`bottom_buttons` lists, with the existing Bubble sub-button configuration and
actions. Empty lists do not remove the row or its spacing.

Keep entity, name, avatar paths, work zone, colours and button definitions in
each person instance. `work_zone` must match the person's state exactly, including
case. `state_colours` is a **single-line Jinja dictionary literal**, written as a
folded YAML string with single-quoted keys and values, as in the examples (not a
YAML mapping). Add any exact zone/state keys there; unmatched states use
`away_colour`. Only `home` and `work_zone` select special avatars; other states
use `away_avatar`.

The original card has no name label. `display_name` is supported but hidden by
default to preserve that appearance; add `- show_name: true` to an instance to
display it above the avatar. Keep `grid_options` on each instance so the dashboard
sees its six-column width on the outer Decluttering card.
