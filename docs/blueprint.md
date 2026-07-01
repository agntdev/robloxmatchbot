# Roblox Matchmaking Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Automatically matches Roblox players into teams based on registered nicknames and ranks, notifying users instantly when compatible teammates/opponents appear in Telegram.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Roblox players using Telegram
- Casual/competitive gamers seeking quick team formation

## Success criteria

- Users receive instant match notifications with join/skip options
- Persistent user profiles with rank tracking
- Admin controls for pool monitoring and announcements

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding to register Roblox nickname and rank
- **/profile** (command, actor: user, command: /profile) — View/edit profile details
  - inputs: display name, Roblox nickname, rank
  - outputs: profile summary
- **/rank** (command, actor: user, command: /rank) — Quickly update reported rank
  - inputs: rank selection
  - outputs: confirmation message
- **/tags** (command, actor: user, command: /tags) — Manage role preference tags
  - inputs: add/remove tags
  - outputs: updated tags list
- **/leave** (command, actor: user, command: /leave) — Temporarily opt out of matching
  - inputs: confirmation
  - outputs: opt-out confirmation
- **/admin_pool** (command, actor: admin, command: /admin_pool) — View active user pool size
  - inputs: none
  - outputs: pool statistics
- **/admin_announce** (command, actor: admin, command: /admin_announce) — Toggle group announcements
  - inputs: enable/disable
  - outputs: status update

## Flows

### Onboarding
_Trigger:_ /start

1. Prompt for Roblox nickname
2. Collect rank selection
3. Store profile data

_Data touched:_ User Profile

### Match Formation
_Trigger:_ New user registration or pool update

1. Scan pool for compatible users
2. Generate match candidate
3. Send notifications with join/skip buttons

_Data touched:_ Match Candidate, Notification

### Profile Management
_Trigger:_ /profile or /rank

1. Display current data
2. Accept edits
3. Update storage

_Data touched:_ User Profile

### Admin Monitoring
_Trigger:_ /admin_pool

1. Query active users
2. Display pool metrics

_Data touched:_ User Profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User Profile** _(retention: persistent)_ — Telegram user's Roblox identity and preferences
  - fields: telegram_user_id, display_name, roblox_nickname, reported_rank, role_tags, last_active
- **Match Candidate** _(retention: session)_ — Proposed team composition
  - fields: user_ids, team_composition, combined_ranks, timestamp
- **Notification** _(retention: session)_ — Match invitation message
  - fields: recipient_id, team_roster, action_buttons, timeout

## Integrations

- **Telegram** (required) — Direct messaging and admin controls
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Admin chat for pool monitoring
- Toggle group announcements
- Configure admin accounts

## Notifications

- Direct match notifications with action buttons
- Admin pool size updates
- Group announcements (optional)

## Permissions & privacy

- Store user profiles with opt-out
- Send notifications only to registered users
- Admin access restricted to configured accounts

## Edge cases

- No compatible matches found
- User timeout during invitation
- Inactive users in pool

## Required tests

- End-to-end match formation flow
- Profile update persistence
- Admin command responses

## Assumptions

- Rank is primary matching criteria
- Default team size is 5v5
- Single admin account by default
