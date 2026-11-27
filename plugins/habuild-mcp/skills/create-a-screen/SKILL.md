---
name: create-a-screen
description: Use when someone wants a new screen in a Habuild app, or a new version of an existing screen for a group of members (a variation). Claude builds the screen from the building blocks that exist and sends it as a proposal for a reviewer to approve in the admin panel. Nothing goes live from Claude.
---

# Create a screen

## What this does

You describe a new screen: what it is for, what it shows and who should see it. Claude builds it from Habuild's
existing building blocks and member data, shows you the layout in plain words, and sends it as a **proposal**.

A proposal is a request, not a change. **Nothing goes live until a reviewer approves the proposal in the admin panel.**
Even after approval the new screen is created as a draft, and it reaches members only when it is published through the
normal flow. Claude cannot approve, publish or skip review.

## Tools

The tools come from the Habuild connector. Their names end in the names below; the app you use may add a prefix in front
(for example `mcp__habuild__ux__list_components`). Match on the ending. If a tool you need is missing, tell the user which
one and stop: do not work around it.

| Tool | Used for |
|---|---|
| `ux__get_authoring_guide` | The rules for writing a screen. Always read first. |
| `ux__list_surfaces`, `ux__get_surface_tree` | Check whether a screen with that key exists; look at similar screens |
| `ux__list_components`, `ux__get_component_schema` | Which building blocks exist and what settings each takes |
| `ux__get_attribute_vocabulary`, `ux__get_data_attribute` | Member data a block can show or be targeted on |
| `ux__list_formatters` | Ways to format a value (dates, numbers) |
| `ux__get_targeting_capabilities` | How to show the screen or a block only to some members |
| `ux__list_actions`, `ux__get_action`, `ux__get_action_capabilities`, `ux__new_object_ids` | What tapping something does; ids for new actions |
| `ux__list_tags` | Tags a screen or block can carry |
| `ux__propose_surface_change` | Send the new screen as a proposal |
| `ux__list_my_proposals`, `ux__get_proposal`, `ux__update_proposal` | Check or change your open proposal |

## Steps Claude follows

1. **Get the brief.** Ask for anything missing: the screen's key (its short name, such as `home` or `challenge-day`), the
   platform (mobile when not said), what it shows, and who should see it.
2. **Read the guide first.** Call `ux__get_authoring_guide` with `section: "overview"`, then `regions`, `block-trees` and
   `bindings`, plus `targeting` or `actions` when the screen needs them. Never write a screen before reading the guide.
3. **Check the key.** Call `ux__list_surfaces` with the key. If a screen with that key already exists on the platform,
   this is a new variation of it, not a new screen: tell the user and agree which members see which version (targeting)
   before going on.
4. **Pick the building blocks.** `ux__list_components` and `ux__get_component_schema` for each block;
   `ux__get_attribute_vocabulary` for member data; the action, formatter and tag tools as needed. Use only what these tools
   return. If something the user wants does not exist, say so plainly instead of guessing.
5. **Build the screen** with no screen id and no block ids, the agreed key and platform, every block written out in full.
   New actions take ids from `ux__new_object_ids`.
6. **Describe the layout in plain words** to the user, top to bottom, and confirm before sending.
7. **Send the proposal** with `ux__propose_surface_change`: `payload` is
   `{ "surfaceId": null, "baseCommitId": null, "tree": <the new screen>, "actions": <new actions, if any> }`, plus a short
   `title` that starts with a verb and says what members see, and a `description` the reviewer can act on.
8. **Report back:** the proposal id and its status, `pending_review`. Say it is waiting for a reviewer in the admin panel.

## Rules

- **Never say the screen is live, done or published.** It is a proposal waiting for review. Use the words "waiting for
  review".
- Only one proposal per service can be open at a time. If the proposal is refused with `open_proposal_exists`, show the
  user their open proposal (`ux__list_my_proposals`) and ask whether to update it, withdraw it or leave it.
- If a tool answers `403`, the user's role cannot do this: tell them and do not retry. `429` means a limit was reached:
  tell them when it resets. `422` names the field to fix.
- Never ask for, accept or paste a token. Sign-in is handled by the connector.
