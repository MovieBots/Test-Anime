# Anime Vault Telegram Bot (starter)

Features:
- Automatic indexing of videos/documents/audio/animations posted in a configured source channel.
- Admin manual upload in private chat.
- Search by typing a title (no `/search` command).
- User menu has exactly four buttons: Anime Search, Anime Movie, Latest Anime Series, Latest Episode.
- Admin can set/change an anime poster with `/setposter Anime Name` by replying to a photo, or by sending a photo captioned `/setposter Anime Name`. Poster is shown when that title is opened.
- Episode/quality buttons and direct Telegram file delivery.
- Duplicate prevention by Telegram `file_unique_id`.
- Optional force-subscribe using normal or request-to-join invite links; pending join requests are not treated as approved membership.
- In groups, search replies mention the user; users can search by writing `@YourBotUsername Naruto`.
- MongoDB Atlas catalog support for persistent storage; PostgreSQL and SQLite fallback support.
- HTTP health endpoint on `PORT` (default 8000) for Koyeb Web Service health checks.

## Important limits
This is a working starter, not a guarantee of zero issues. Title detection is caption-based, not AI-based. For reliable matching, caption each source post with:
```
Anime: Naruto Shippuden
Episode: 12
Quality: 1080p
```
If caption does not contain `Anime:`, the parser tries to infer a title from the first non-metadata line. If the title cannot be inferred, it may become `Untitled Anime`; review captions before uploading. Gemini AI recognition is not included yet.

## Environment variables
- `BOT_TOKEN` required.
- `ADMIN_IDS` required for admin manual uploads; comma-separated numeric Telegram user IDs.
- `SOURCE_CHANNEL_ID` optional source channel ID, usually `-100...`. Add the bot to that channel with permission to read posts.
- `FSUB_CHANNELS` optional comma-separated usernames/IDs.
- `FSUB_LINKS` optional comma-separated join links in matching order.
- `PORT` default `8000`.
- `MONGO_URI` recommended MongoDB Atlas connection string for permanent catalog storage. When set, MongoDB is used first.
- `MONGO_DB_NAME` database name, default `anime_vault`.
- `DATABASE_URL` optional PostgreSQL fallback if `MONGO_URI` is blank.
- `DB_PATH` default `animebot.db` (SQLite fallback only; may be lost when Koyeb replaces the instance).

## Koyeb deployment (Web Service)
1. Put these files in a GitHub repository (upload the ZIP contents, not the ZIP itself).
2. Create a Koyeb service from that repository using the Dockerfile builder.
3. Choose a Web Service if Koyeb requires a port. Set exposed port to `8000`, protocol HTTP, path `/health` if a health-check path can be configured.
4. Add environment variables in Koyeb dashboard. Do not paste tokens into public GitHub files.
5. Deploy, then check logs for `Health server listening on 0.0.0.0:8000` and `Polling started`.
6. Open the bot in Telegram and send `/start`.

## Source channel setup
1. Add the bot to your source channel. For channel posts, make it an admin if needed.
2. Set `SOURCE_CHANNEL_ID` to the numeric channel ID.
3. Post a video with caption like:
   `Anime: Naruto Shippuden\nEpisode: 12\nQuality: 1080p`
4. The bot indexes it silently. Test by searching `Naruto` in the bot private chat.

## Change an anime poster (Admin only)
1. Add your Telegram numeric user ID to `ADMIN_IDS` in Koyeb.
2. Open the bot in private chat as the admin.
3. Send the poster image, then reply to that image with `/setposter Naruto Shippuden`.
   - Alternative: send the image with caption `/setposter Naruto Shippuden`.
4. The bot replies `Poster updated`. Search for that exact anime title and open it; the poster will be displayed with the episode buttons.
5. To change it later, repeat the same steps with a new image and the same title. The new poster replaces the old one.

Poster images are stored as Telegram `file_id` values in MongoDB's `posters` collection when `MONGO_URI` is configured. Existing anime entries are not changed by setting a poster.

## Manual upload
Send a video or document to the bot in a private chat as an ID listed in `ADMIN_IDS`. Use the same caption format above. Then search for the title.

## Force-subscribe / request-to-join channels
Set `FSUB_CHANNELS` and `FSUB_LINKS` with matching comma-separated values. For a request-to-join workflow, create an invite link in Telegram with **Request Admin Approval** enabled, then put that invite link in `FSUB_LINKS`. Do not use a normal public channel URL if you specifically want approval requests. Add the bot as a channel admin so it can check membership and receive join-request updates. The bot tells the requester that the request is pending; access stays blocked until a channel admin approves the request and Telegram reports the user as a member. Test with a non-admin user.

## Group usage and mentions
Add the bot to the group. A user can search by sending `@YourBotUsername Naruto`; the bot removes its own @mention from the query and replies with a clickable mention of the user who searched or pressed a result button. Depending on the group, Telegram bot privacy settings may need to allow the bot to receive the intended messages.

## Permanent database setup with MongoDB Atlas (recommended)
1. Create a MongoDB Atlas account at https://www.mongodb.com/atlas/database and create a free cluster if available for your account/region.
2. In Atlas, create a database user with a username and password. Add the network access rule required by Atlas; for a hosted bot this is often `0.0.0.0/0`, which allows connections from any IP, so use a strong unique database password.
3. Choose Connect → Drivers and copy the connection string beginning `mongodb+srv://`.
4. Replace the username, password, and cluster placeholder with your real details. URL-encode special characters in the password.
5. In Koyeb → Service → Environment variables, add `MONGO_URI` with the full connection string and `MONGO_DB_NAME` as `anime_vault`. Keep `PORT=8000`.
6. Redeploy. The bot creates the collection/indexes automatically when it starts.
7. Check logs, then upload/search a test file. Never post the connection string publicly.

When `MONGO_URI` is set, MongoDB is used first. Existing SQLite/PostgreSQL records are not migrated automatically; newly added files will be stored in MongoDB. Keep the source channel as a backup.

## Security
If a bot token was exposed in logs or screenshots, revoke it with @BotFather and update `BOT_TOKEN`. Never commit `.env` or a live token to GitHub.

## Change the poster shown when users send `/start`

1. Send the new poster photo to the bot in a private chat, as an admin listed in `ADMIN_IDS`.
2. Reply to that photo with `/setstartposter`, then send the command.
   - Alternative: send the photo with caption `/setstartposter`.
3. The bot confirms the welcome poster was updated. Send `/start` to preview it.

This setting changes only the welcome image shown with `/start`; it does not change anime-specific posters.
