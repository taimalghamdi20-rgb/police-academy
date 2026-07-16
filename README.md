# First Town — Access Log Bot

A small Discord bot + HTTP API that logs website login attempts and lets an
admin **Accept** or **Reject** each one with buttons, right from Discord.

Each log message shows:
- Discord Account (tag)
- Discord ID
- Country
- Device
- Time
- Accept / Reject buttons

## 1. Create the Discord bot

1. Go to https://discord.com/developers/applications and open (or create) your application.
2. Go to **Bot** → **Reset Token** → copy the token.
3. Under **Privileged Gateway Intents**, you don't need any of the privileged
   ones for this bot (it only uses Guilds + button interactions).
4. Invite the bot to your server with the **View Channel**, **Send Messages**,
   and **Embed Links** permissions (OAuth2 → URL Generator → scope `bot`).
5. Get the ID of the channel you want logs posted to (enable Developer Mode in
   Discord settings, then right-click the channel → Copy Channel ID).
6. (Optional) Get the ID of a role that's allowed to click Accept/Reject
   (right-click the role in Server Settings → Roles → Copy Role ID). If you
   leave this blank, anyone in the server can click the buttons.

## 2. Configure

```bash
cd bot
cp .env.example .env
```

Fill in `.env`:

```
BOT_TOKEN=your_bot_token_here
LOG_CHANNEL_ID=your_channel_id_here
APPROVER_ROLE_ID=your_role_id_here   # optional
PORT=3000
```

## 3. Install & run

```bash
npm install
npm start
```

You should see:
```
Logged in as YourBot#0000
API listening on port 3000
```

## 4. Deploy it somewhere it stays running

Your HTML page is static and calls this bot over HTTP, so the bot needs to
run on a server that's always on (a laptop that sleeps won't work). Easy free
options: Render, Railway, Fly.io, or a small VPS. Whichever you use:

1. Deploy this `bot/` folder as a Node.js service (`npm install && npm start`).
2. Set the same environment variables (`BOT_TOKEN`, `LOG_CHANNEL_ID`,
   `APPROVER_ROLE_ID`, `PORT`) in that platform's dashboard/secrets — do not
   commit your real `.env` file.
3. Copy the public URL it gives you (e.g. `https://your-app.onrender.com`).

## 5. Point the website at it

In `first-town-test.html`, set:

```js
const API_BASE_URL = "https://your-app.onrender.com"; // no trailing slash
```

## How it works

1. Someone logs in with Discord on the website.
2. The website sends their Discord ID/tag/country/device to `POST /api/verify`.
3. The bot posts an embed with **Accept**/**Reject** buttons to your log channel
   and returns a `requestId`.
4. The website polls `GET /api/status/:requestId` every 3 seconds.
5. When an admin clicks a button, the bot updates the message (shows who
   decided, disables the buttons) and marks the request as approved/rejected.
6. The website sees the new status and either shows the page content or an
   "access denied" message.

## Notes

- Requests are stored in memory — if the bot restarts, any *pending* requests
  are lost (the person would need to log in again). Decided requests remain
  visible in Discord since they're just regular messages.
- CORS is open by default so the static site can call the API from any
  domain; restrict `cors()` in `bot.js` to your site's exact origin once it's
  live if you want to lock that down.
