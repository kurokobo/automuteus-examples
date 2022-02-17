<!-- omit in toc -->
# Minimal Bot Permissions

Current instruction says to grant administrator permission to the bot, but this is a bit excessive. This is minimal required bot permissions for 6.x or later.

<!-- omit in toc -->
## Table of Contents

- [For the first bot](#for-the-first-bot)
- [For the extra worker bot](#for-the-extra-worker-bot)

## For the first bot

| GENERAL PERMISSIONS           | TEXT PERMISSIONS           | VIOCE PERMISSIONS    |
| ----------------------------- | -------------------------- | -------------------- |
| ⬜ Administrator               | ✅ Send Messages            | ⬜ Connect            |
| ⬜ View Audit Log              | ⬜ Create Public Threads    | ⬜ Speak              |
| ⬜ View Server Insights        | ⬜ Create Private Threads   | ⬜ Video              |
| ⬜ Manage Server               | ⬜ Send Messages in Threads | ✅ Mute Members       |
| ⬜ Manage Roles                | ⬜ Send TTS Messages        | ✅ Deafen Members     |
| ⬜ Manage Channels             | ✅ Manage Messages          | ⬜ Move Members       |
| ⬜ Kick Members                | ⬜ Manage Threads           | ⬜ Use Voice Activity |
| ⬜ Ban Members                 | ✅ Embed Links              | ⬜ Priority Speaker   |
| ⬜ Create Instant Invite       | ⬜ Attach Files             |                      |
| ⬜ Change Nickname             | ✅ Read Message History     |                      |
| ⬜ Manage Nicknames            | ⬜ Mention Everyone         |                      |
| ✅ Manage Emojis and Stickers  | ✅ Use External Emojis      |                      |
| ⬜ Manage Webhooks             | ⬜ Use External Stickers    |                      |
| ✅ Read Messages/View Channels | ✅ Add Reactions            |                      |
| ⬜ Manage Events               | ✅ Use Slash Commands       |                      |
| ⬜ Moderate Members            |                            |                      |

The permission `✅ Use Slash Commands` is just for future enhancements, and is not mandatory at this time.

Permissions Integer: `3234163776`

## For the extra worker bot

| GENERAL PERMISSIONS           | TEXT PERMISSIONS           | VIOCE PERMISSIONS    |
| ----------------------------- | -------------------------- | -------------------- |
| ⬜ Administrator               | ⬜ Send Messages            | ⬜ Connect            |
| ⬜ View Audit Log              | ⬜ Create Public Threads    | ⬜ Speak              |
| ⬜ View Server Insights        | ⬜ Create Private Threads   | ⬜ Video              |
| ⬜ Manage Server               | ⬜ Send Messages in Threads | ✅ Mute Members       |
| ⬜ Manage Roles                | ⬜ Send TTS Messages        | ✅ Deafen Members     |
| ⬜ Manage Channels             | ⬜ Manage Messages          | ⬜ Move Members       |
| ⬜ Kick Members                | ⬜ Manage Threads           | ⬜ Use Voice Activity |
| ⬜ Ban Members                 | ⬜ Embed Links              | ⬜ Priority Speaker   |
| ⬜ Create Instant Invite       | ⬜ Attach Files             |                      |
| ⬜ Change Nickname             | ⬜ Read Message History     |                      |
| ⬜ Manage Nicknames            | ⬜ Mention Everyone         |                      |
| ⬜ Manage Emojis and Stickers  | ⬜ Use External Emojis      |                      |
| ⬜ Manage Webhooks             | ⬜ Use External Stickers    |                      |
| ⬜ Read Messages/View Channels | ⬜ Add Reactions            |                      |
| ⬜ Manage Events               | ⬜ Use Slash Commands       |                      |
| ⬜ Moderate Members            |                            |                      |

Permissions Integer: `12582912`
