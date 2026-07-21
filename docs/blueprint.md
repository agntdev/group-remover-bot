# Group Member Removal Bot — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets group admins remove members via a simple interface. Admins select a user from a list and confirm removal; the bot handles the API call.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group admins
- Moderators of community groups

## Success criteria

- Admin can remove a member in under 10 seconds
- Bot shows clear error messages if removal fails
- No unauthorized users can trigger removals

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/remove** (command, actor: admin, command: /remove) — Open the member removal interface
- **Remove user** (button, actor: admin, callback: remove:select) — Select a user to remove
  - inputs: user_id
  - outputs: confirmation prompt
- **Yes, remove** (button, actor: admin, callback: remove:confirm) — Confirm removal
  - inputs: user_id
  - outputs: success/failure message
- **Cancel** (button, actor: admin, callback: remove:cancel) — Cancel the removal
  - outputs: back to member list or menu

## Flows

### Remove member
_Trigger:_ /remove

1. Show member list
2. Wait for user selection
3. Show confirmation
4. Remove user
5. Report result

_Data touched:_ group_members, user_id

### Cancel removal
_Trigger:_ Cancel button

1. Clear selection
2. Return to menu

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **group_members** _(retention: none)_ — Current members of the group
  - fields: user_id, username, first_name, last_name
- **selection** _(retention: session)_ — Selected user for removal
  - fields: user_id

## Integrations

- **Telegram** (required) — Bot API messaging
- **Telegram Bot API** (required) — removeUserFromChat API call
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Remove a member from the group
- Cancel a pending removal
- View the member list

## Permissions & privacy

- Bot must be admin in the group
- Only admins can trigger removals
- No logging of removals
- No user data stored beyond the current session

## Edge cases

- User already left the group
- Bot lacks permission to remove users
- Admin tries to remove themselves
- Member list is empty
- Selection timeout (user doesn't respond)
- User has no username

## Required tests

- Admin can remove a member successfully
- Bot shows error if admin lacks permission
- Bot shows error if user already left
- Bot shows error if admin tries to remove themselves
- Bot shows error if bot lacks kick permission
- Bot shows error if member list is empty
- Bot shows error if selection times out
- Bot shows error if user has no username
- Bot shows error if non-admin tries to use /remove

## Assumptions

- Bot is added as admin to the group
- Admin has kick permission
- Bot can fetch member list
- Member list shows up to 50 members
- Confirmation is required before removal
- No logging of removals
- No user data stored beyond the current session
