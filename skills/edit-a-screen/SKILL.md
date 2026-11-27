---
name: edit-a-screen
description: Use when someone wants to change an existing screen in a Habuild app, such as changing text, adding or removing a card or button, reordering blocks, or changing who sees a block. Claude reads the screen as it is today, makes the change and sends it as a proposal for a reviewer to approve in the admin panel. Nothing goes live from Claude.
---

# Edit a screen

## What this does

You describe a change to a screen in a Habuild app ("make the banner on the home screen say 'Join today's session'",
"remove the streak card for new members"). Claude finds the screen, reads exactly what it shows today, makes the
change and sends it as a **proposal**.

A proposal is a request, not a change. **Nothing goes live until a reviewer approves the proposal in the admin panel.**
Even after approval the change becomes a draft on the screen, and it reaches members only when that draft is published
through the normal flow. Claude cannot approve, publish or skip review.

## Tools

The tools come from the Habuild connector. Their names end in the names below; the app you use may add a prefix in front
(for example `mcp__habuild__ux__list_surfaces`). Match on the ending. If a tool you need is missing, tell the user which
one and stop: do not work around it.

| Tool | Used for |
|---|---|
| `ux__get_authoring_guide` | The rules for writing a screen. Always read first. |
| `ux__list_surfaces` | Find the screen by its key and platform |
| `ux__get_surface_tree` | The whole screen as it is today, every block with its id |
| `ux__list_components`, `ux__get_component_schema` | Which building blocks exist and what settings each takes |
| `ux__get_attribute_vocabulary`, `ux__get_data_attribute` | Member data a block can show or be targeted on |
| `ux__list_formatters` | Ways to format a value (dates, numbers) |
| `ux__get_targeting_capabilities` | How to show a block only to some members |
| `ux__list_actions`, `ux__get_action`, `ux__get_action_capabilities`, `ux__new_object_ids` | What tapping something does |
| `ux__list_tags` | Tags a screen or block can carry |
| `ux__propose_surface_change` | Send the change as a proposal |
| `ux__list_my_proposals`, `ux__get_proposal`, `ux__update_proposal` | Check or change your open proposal |

## Steps Claude follows

1. **Read the guide first.** Call `ux__get_authoring_guide` with `section: "overview"`, then the section that matches the
   change (`regions`, `block-trees`, `bindings`, `required-props`, `targeting`, `sdk-version`, `actions` or `examples`).
   Never write a screen before reading the guide.
2. **Find the screen.** Call `ux__list_surfaces` with the screen's key (and platform when the user names one). If several
   rows come back they are variations of the same screen for different members: ask the user which one.
3. **Read the screen as it is today.** Call `ux__get_surface_tree` with its id. Every edit starts from this, never from
   memory. Keep its `id` and `commitId`.
4. **Check what you will use.** `ux__get_component_schema` for every block you add or whose settings you change;
   `ux__get_attribute_vocabulary` for any member data; the targeting, formatter, action and tag tools as needed. Use only
   what these tools return. If something the user wants does not exist, say so plainly instead of guessing.
5. **If the screen is already published,** read the `sdk-version` section and keep the change working on the app versions
   the screen supports.
6. **Make the change** on the tree from step 3, keeping every other block, id and setting exactly as it was.
7. **Explain the change in plain words** to the user (what members will see, what stays the same) and confirm before
   sending.
8. **Send the proposal** with `ux__propose_surface_change`: `payload` is
   `{ "surfaceId": <id>, "baseCommitId": <commitId>, "tree": <the whole edited screen> }`, plus a short `title` that starts
   with a verb and says what members see, and a `description` the reviewer can act on.
9. **Report back:** the proposal id and its status, `pending_review`. Say it is waiting for a reviewer in the admin panel.

## Rules

- **Never say the change is live, done or published.** It is a proposal waiting for review. Use the words "waiting for
  review".
- Only one proposal per service can be open at a time. If the proposal is refused with `open_proposal_exists`, show the
  user their open proposal (`ux__list_my_proposals`) and ask whether to update it, withdraw it or leave it.
- If a tool answers `403`, the user's role cannot do this: tell them and do not retry. `429` means a limit was reached:
  tell them when it resets. `422` names the field to fix.
- Never ask for, accept or paste a token. Sign-in is handled by the connector.
