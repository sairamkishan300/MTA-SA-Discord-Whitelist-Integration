# MTA-SA × Discord Whitelist Integration

A complete integration between your MTA:SA server and Discord community.

This system links Discord accounts with MTA serials, manages whitelists automatically, and provides rich logging and staff tools – all directly from Discord.

<img width="735" height="495" alt="image" src="https://github.com/user-attachments/assets/56dfa575-ed14-4c0d-bcb4-4ffdd2a59694" />
-
--
## Features

- **Two-way command bridge**  
  Send commands from Discord to MTA and from MTA back to Discord.

- **Discord ID ↔ MTA Serial linking**  
  Store and manage a secure link between a player’s Discord account and their MTA serial.

- **Automatic whitelist removal on Discord leave**  
  If a whitelisted user leaves the Discord server, they are automatically removed from the whitelist.

- **Whitelist stored in `.txt`**  
  Easy-to-edit whitelist storage in a human-readable `.txt` file.

- **Embed-based whitelist announcements**  
  When a player is whitelisted, an embed announcement is sent to a configured Discord channel.

- **Automatic “Whitelisted” role assignment**  
  On successful whitelist, a dedicated Discord role is assigned.

- **`!whois` lookup command**  
  Staff can quickly search whitelisted entries by Discord ID, MTA serial or name.

- **Discord webhooks support**  
  Webhooks for:
  - Embed announcements  
  - Plain text messages  
  - `.txt` log/whitelist references

- **MTA-SA logs on Discord**  
  Server logs and important events can be relayed to Discord channels.

- **All channels accessible for commands**  
  Commands can be used from any channel (or restricted via permissions, if configured).
<img width="704" height="789" alt="image3" src="https://github.com/user-attachments/assets/c56e1cb7-322e-4c3d-ae6b-e5335ea6ec31" />

---

## Included Discord Commands

### Staff Commands

- `!whitelistadd` / `!wadd <MTA SERIAL> <DISCORD ID> <DISCORD NAME>`  
  Add a player to the whitelist.

- `!whitelistremove` / `!wremove <DISCORD ID>`  
  Remove a player from the whitelist by their Discord ID.

- `!whitelistlist` / `!wlist`  
  Show the current whitelist entries (pulled from the `.txt` file).

- `!whois` / `!find`  
  Look up whitelist info by Discord ID, MTA serial, or name (implementation depends on configuration).

### Self-Whitelist Command

- `!selfwhitelist` / `!sw <MTA SERIAL>`  
  Allows a player to whitelist themselves if they meet your conditions (for example, being in a certain Discord role or channel).
<img width="1048" height="1151" alt="image2" src="https://github.com/user-attachments/assets/4286bd3d-7b78-4ab2-8860-02a3dded0cc3" />

---

## How It Works (Overview)

- **MTA-Side Script (Resource)**  
  - Runs on your MTA:SA server as a resource.  
  - Handles serial checks, whitelisting logic, and in-game events.  
  - Communicates with the `.txt` whitelist file and Discord bridge.

- **Discord-Side Script (Bot / Bridge)**  
  - Runs as a Discord bot or webhook bridge.  
  - Listens to commands (`!wadd`, `!wremove`, etc.).  
  - Sends relevant instructions to the MTA server.  
  - Sends logs, embed announcements, and updates whitelist file if configured on this side.

- **Whitelist Storage (.txt)**  
  - A `.txt` file stores all whitelist entries (e.g. `serial | discord_id | discord_name | timestamp`).  
  - Used for quick editing and backup.

---

## Requirements

- **MTA:SA Server**
  - Ability to add and start a custom resource.
  - File read/write permissions for the whitelist `.txt`.

- **Discord**
  - A Discord server (guild) where the bot will be added.
  - A bot token (from the Discord Developer Portal).
  - Permission for the bot to:
    - Read and send messages  
    - Manage roles (for the whitelisted role)  
    - Manage webhooks (if you use webhook-based logs)

- **Host for the Discord Bot / Bridge**
  - Any machine or VPS that can run the script 24/7.
  - Stable internet connection to reach both Discord and the MTA server.

> The exact runtime (Node.js, Python, etc.) depends on the version of the script you received. Check the provided source files for the language and dependencies.

---

## Installation

### 1. Upload the MTA Resource

1. **Copy the resource folder**  
   Place the provided MTA resource folder into your MTA server’s `resources` directory.

2. **Configure `meta.xml` (if needed)**  
   Ensure the resource’s `meta.xml` is correctly configured and matches the resource folder name.

3. **Whitelist file location**  
   Confirm the path to the `.txt` whitelist file used by the resource.  
   - If a default file is provided, keep it or update the path in the config.

4. **Start the resource**  
   In your MTA console:
   ```
   start <resource_name>
   ```
   Or set it to autostart in `mtaserver.conf`.

---

### 2. Create and Configure the Discord Bot

# mta-discord-bot
Connects MTA:SA servers and Discord channels by sending messages/commands back and forth.  
**Note:** The recent code update has breaking changes to your `config.json`, please replace _server_ with _guild_, thanks.

## Installation

### MTA:SA
The discord resource requires the socket module on your MTA:SA server. You can find the installation guide
and the binaries on the [MTA wiki](https://wiki.multitheftauto.com/wiki/Modules/Sockets).

The resources themselves require no special setup, you only have to edit the `discord/config.xml` to fit your setup.
The resource **discord** is responsible for message transfer between your MTA:SA server and the Discord relay.
On the other hand, **discord-events** is listening for events on the MTA:SA server to send these to the relay
through the **discord** resource.

> mta/discord/config.xml
```xml
<discord>
    <channel>name-of-your-channel</channel>
    <passphrase>equal-to-the-server-passphrase</passphrase>
    <hostname>localhost</hostname>
    <port>8100</port>
</discord>
```

### Relay
The relay server connects to Discord as a bot user; and starts a socket server listening for connections. I never
bothered to keep the code compatible with older releases of node.js, which might be an issue on some distributions.

The relay code needs a little preparation before you can run it. You have to run the command `npm install` in the `src`
directory to download the node.js modules required by the application. Furthermore, you have to copy the file
`src/example.config.json` to `src/config.json` and edit it to fit your setup
(the passphrase and port must be equal to the **discord** config.xml on your MTA:SA server).

You can start the relay with `node app.js`, but you should consider using [pm2](https://github.com/Unitech/pm2) to restart
the relay automatically on crashes.

> src/example.config.json
```json
{
    "port": 8100,
    "passphrase": "key",
    "guild": "guild.id",
    "bots": [{
        "channel": "channel.name",
        "token": "bot.token"
    }]
}
```

## Notes

### Discord Bots

> **Note**
> Make sure your bot account has the `MESSAGE CONTENT INTENT` permission, because otherwise you will only see empty messages coming from Discord (in MTA:SA).
> 
> ![image](https://github.com/botder/mtasa-discord-bot/assets/38703318/f0df9eb7-cc40-44f6-8b22-3a7e38cfbc0d)

This application doesn't magically add your bot(s) to your guild(s). You (better said: the guild owner) have to authorize every bot user. Navigate to your bot application on the following page and note the client id:
> https://discord.com/developers/applications

Then continue to replace the `client_id` field in the URL below and navigate to that page:
> https://discord.com/api/oauth2/authorize?scope=bot&permissions=0&client_id=<your_client_id_here>

Proceed by authorizing the bot for the guild of your choice and you're done.

---

### 3. Configure the Script

You will find configuration files in the package (for example: `config.lua`, `config.json`, `.env`, etc. – names depend on your version).

Configure the following:

- **Discord Section**
  - Bot token  
  - Guild (server) ID  
  - Channel IDs for:
    - Whitelist announcements  
    - Logs  
    - (Optional) Staff commands  
  - Role ID for the **whitelisted** role.  
  - Webhook URLs (if used).

- **MTA Section**
  - RCON / bridge connection (if applicable).  
  - IP / port / auth details for communicating with the MTA server.  
  - Path to the whitelist `.txt` file.

- **Whitelist Settings**
  - Whether self-whitelist is enabled.  
  - Role requirement for `!selfwhitelist` (e.g. only users with “Member” role can self-whitelist).  
  - Logging options (which events go to Discord).

Save your changes and restart the bot / resource after editing configs.

---

### 4. Run the Discord Bot / Bridge

Depending on the script’s language:

- **Node.js example**
  ```
  npm install
  node index.js
  ```

- **Python example**
  ```
  pip install -r requirements.txt
  python bot.py
  ```

> Follow the exact instructions included in the source package for starting the bot process (filename, command, etc.).

Make sure:

- The bot shows as **online** in Discord.  
- No authentication errors appear in the console.

---

## Usage

### Staff Whitelisting

- **Add whitelist**  
  `!whitelistadd <MTA SERIAL> <DISCORD ID> <DISCORD NAME>`  
  - Adds an entry to the `.txt` whitelist file.  
  - Sends an embed announcement (if configured).  
  - Assigns the whitelisted role to the Discord user.

- **Remove whitelist**  
  `!whitelistremove <DISCORD ID>`  
  - Removes entry from the `.txt` list.  
  - Optionally removes the whitelisted role.

- **List whitelist**  
  `!whitelistlist`  
  - Prints the current whitelist entries directly in Discord (or as a file/log).

- **Whois / Find**  
  `!whois <query>` or `!find <query>`  
  - `query` could be a Discord ID, MTA serial, or name (depending on implementation).  
  - Useful to quickly check who is who.

### Self Whitelist

- **Player Self-Whitelist**  
  `!selfwhitelist <MTA SERIAL>` or `!sw <MTA SERIAL>`  
  - The script checks:  
    - If the user meets any role/condition requirements.  
    - If the serial or Discord ID is already whitelisted.  
  - On success:  
    - Adds to whitelist `.txt`.  
    - Assigns whitelisted role.  
    - Sends an embed confirmation.

---

## Automatic Whitelist Removal on Discord Leave

- The script listens for **guild member remove** events.  
- When a user leaves the Discord server:  
  - Their Discord ID is checked against the whitelist `.txt`.  
  - If found, the entry is removed.  
  - A log message and/or embed can be sent to the staff/log channel.

This keeps your MTA whitelist automatically in sync with your Discord community.

---

## Logging & Webhooks

- **MTA Logs**  
  The server can send logs or custom events (kicks, bans, connection attempts, etc.) to Discord channels using:  
  - Plain bot messages  
  - Webhooks with embeds

- **Whitelist Actions**  
  Every whitelist add/remove/self-whitelist can be:  
  - Logged in a staff channel.  
  - Shown publicly in an announcement channel.  
  - Logged to a `.txt` or log channel for audit.

Configure the level of logging in the config file.

---

## Security & Best Practices

- **Limit who can use staff commands**  
  - Restrict `!wadd`, `!wremove`, `!wlist`, `!whois` to staff-only channels or roles (e.g. Admin, Moderator).  
  - Use Discord permission checks in the bot configuration.

- **Protect your bot token**  
  - Never share the token publicly.  
  - Use environment variables or a private config file.

- **Backup the whitelist `.txt`**  
  - Regularly backup the file.  
  - Consider version control or scheduled backups for safety.

---

## Support / Customization

This script is designed to be easily customizable:

- Edit messages, embed colors, and branding in the config or message templates.
- Adjust which events are logged and to which channels.
- Adapt the whitelist structure inside the `.txt` file to fit your needs.

For installation help, customization, or paid support, please contact the script seller.

---

## License & Terms

This script is provided for use on a limited number of servers (as agreed upon at purchase).  
Redistribution, resale, or leaking of the source code is **strictly prohibited** unless explicitly allowed by the author.
