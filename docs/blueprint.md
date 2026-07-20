# ChatGuard — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for group chat moderation with automatic new member confirmation, spam detection, admin commands, and audit logging. Features include button-based verification, configurable thresholds for auto-actions, and reporting for group owners.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group owners
- Community moderators

## Success criteria

- New members must confirm via button before sending messages
- Spam is automatically detected and blocked based on configurable thresholds
- Admins can manage users and review audit logs via commands

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/warn** (command, actor: admin, command: /warn) — Warn a user with optional reason
- **/mute** (command, actor: admin, command: /mute) — Mute a user for specified duration
- **/kick** (command, actor: admin, command: /kick) — Kick a user from the group
- **/ban** (command, actor: admin, command: /ban) — Ban a user from the group
- **/trust** (command, actor: admin, command: /trust) — Mark a user as trusted
- **/untrust** (command, actor: admin, command: /untrust) — Remove trusted status from a user
- **/setgreeting** (command, actor: admin, command: /setgreeting) — Update greeting text and rules
- **/setthresholds** (command, actor: admin, command: /setthresholds) — Configure auto-action thresholds
- **/audit** (command, actor: admin, command: /audit) — Get audit report for specified period
- **/showlogs** (command, actor: admin, command: /showlogs) — View recent action records
- **Я человек** (button, actor: user, callback: confirm:join) — Confirm new member identity

## Flows

### New member join
_Trigger:_ member join event

1. Send greeting with confirmation button
2. Wait for button press or timeout
3. Grant permissions or remove user

_Data touched:_ User, New-join challenge

### Spam detection
_Trigger:_ new message

1. Check for spam triggers
2. Calculate offense severity
3. Apply configured auto-action

_Data touched:_ Offense event, Action record

### Admin command execution
_Trigger:_ /command

1. Validate admin permissions
2. Execute action
3. Record in audit log

_Data touched:_ Action record

### Audit report generation
_Trigger:_ /audit

1. Filter records by period
2. Aggregate statistics
3. Format and return report

_Data touched:_ Audit log

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram participant with status and permissions
  - fields: id, username, display name, status
- **New-join challenge** _(retention: session)_ — Temporary record for new members
  - fields: join_time, message_id, expires_at, confirmed
- **Offense event** _(retention: persistent)_ — Record of detected violations
  - fields: type, user_id, message_id, timestamp, severity
- **Action record** _(retention: persistent)_ — Logged moderation action
  - fields: auto/manual, action_type, target_user, reason, admin_id, timestamp
- **Group settings** _(retention: persistent)_ — Per-chat configuration
  - fields: greeting text, rules text, confirmation timeout, auto-action thresholds, trusted users list, exempt roles
- **Audit log** _(retention: persistent)_ — Chronological list of actions
  - fields: action records

## Integrations

- **Telegram** (required) — Bot API messaging and moderation actions
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Edit greeting text and rules
- Configure auto-action thresholds
- Manage trusted users list
- Generate audit reports
- Set admin notification preferences

## Notifications

- Group-wide messages for automatic actions
- Admin alerts for critical events
- Periodic audit reports on request

## Permissions & privacy

- Only admins can execute moderation commands
- Audit logs visible only to admins
- Trusted users are exempt from auto-moderation
- Private messages to users when appropriate

## Edge cases

- User clicks confirmation button after timeout
- Multiple spam triggers in one message
- Admin tries to act on another admin
- Message contains both blacklisted keyword and link

## Required tests

- Confirm new member flow with timeout handling
- Verify spam detection triggers and thresholds
- Test all admin commands with permission checks
- Validate audit log generation and retention

## Assumptions

- Button-based confirmation is preferred over CAPTCHA
- Default thresholds prevent over-punishment
- Trusted users include admins by default
- Audit logs are retained for 90 days
