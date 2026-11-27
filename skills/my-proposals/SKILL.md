---
name: my-proposals
description: Use when someone asks about changes they have sent for review in Habuild, such as "what's the status of my screen change", "was my proposal approved", "why was it rejected", "change my proposal", "undo my last edit to it" or "cancel my proposal". Claude lists, explains, updates, rolls back or withdraws the user's own proposals. Claude cannot approve anything.
---

# My proposals

## What this does

Every change Claude makes for you in Habuild is a **proposal**: a request that a reviewer approves or rejects in the
admin panel. This skill lets you see where your proposals stand, read what a reviewer said, change a proposal that is
still waiting, go back to an earlier version of it, or withdraw it.

**Nothing goes live until a reviewer approves the proposal in the admin panel.** An approved screen change becomes a
draft, and it reaches members only when that draft is published through the normal flow. Claude cannot approve, reject
or publish anything.

## Tools

The tools come from the Habuild connector. Their names end in the names below; the app you use may add a prefix in front
(for example `mcp__habuild__ux__list_my_proposals`). Match on the ending. Each Habuild service has its own set with its
own prefix; `ux__` is screens (UX-Service). If a tool you need is missing, tell the user which one and stop.

| Tool | Used for |
|---|---|
| `ux__list_my_proposals` | Your proposals, newest first. Filter with `status`. |
| `ux__get_proposal` | One proposal in full: what it changes, status, version, who reviewed it and what they said |
| `ux__list_proposal_versions` | Every version of a proposal: who changed it and when |
| `ux__update_proposal` | Change a proposal that is still waiting for review |
| `ux__revert_proposal` | Make an earlier version current again (kept as a new version; nothing is deleted) |
| `ux__withdraw_proposal` | Cancel a proposal that is still waiting for review |
| `ux__get_authoring_guide` | The rules for writing a screen. Read before changing a proposal's screen. |

## What the statuses mean

| Status | In plain words |
|---|---|
| `pending_review` | Waiting for a reviewer. You can still change, roll back or withdraw it. |
| `approving` | A reviewer approved it and it is being applied. Wait. |
| `approved` | Approved. For a screen this means a draft was created; it is **not live** until published through the normal flow. |
| `rejected` | A reviewer turned it down. `ux__get_proposal` shows their reason. |
| `withdrawn` | You cancelled it. |

## Steps Claude follows

1. **List.** Call `ux__list_my_proposals` (with `status` if the user asks about, say, only waiting ones). Show each one's
   title, status and date in plain words, with its id.
2. **Explain one.** Call `ux__get_proposal` with its `proposalId`. Say what it changes for members, its status, and for a
   rejected one the reviewer's reason. To see an older version, pass `version`; to see its history, call
   `ux__list_proposal_versions`.
3. **Change one** (only while `pending_review`). Read `ux__get_authoring_guide` (`overview`, then the section for the
   change) first. Start from the proposal's current payload from `ux__get_proposal`, make the change, explain it in plain
   words, then call `ux__update_proposal` with the whole payload, not just the changed part. A rejected proposal cannot be
   changed: start a new one with the edit-a-screen or create-a-screen skill.
4. **Roll back** (only while `pending_review`). Show the versions, confirm which one, then call `ux__revert_proposal` with
   `proposalId` and `version`.
5. **Withdraw** (only while `pending_review`). Confirm with the user first, then call `ux__withdraw_proposal`.
6. **Report back** the proposal id, its version and its status after any change.

## Rules

- **Never say a change is live.** `approved` is not live. Say "approved, waiting to be published" for an approved screen
  change, and "waiting for review" for `pending_review`.
- Only the person who raised a proposal can change, roll back or withdraw it. A `403` means it is someone else's or the
  user's role cannot do this: tell them and do not retry.
- A `409` means the proposal is no longer waiting for review or changed since it was read: read it again with
  `ux__get_proposal` and tell the user what changed.
- Only one proposal per service can be open at a time.
- Never ask for, accept or paste a token. Sign-in is handled by the connector.
