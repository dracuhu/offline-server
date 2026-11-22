# Monster Mind — Instagram Thread Reply Dashboard

A small Node/Express app that uses Meta's official Instagram Messaging API to **reply inside an existing Instagram conversation** by conversation/thread ID. The dashboard is based on the supplied Monster Mind visual reference.

## Important API behavior

- Instagram Messaging is for eligible Instagram Professional accounts and requires the appropriate Meta app setup, permissions, token, and (for this Facebook Login integration) a linked Facebook Page.
- A conversation/thread ID is **not** itself the recipient ID for the Send API. This app first reads the conversation, finds the other participant's Instagram-scoped ID, then sends to that participant using the Instagram Send API. You can optionally supply a message ID as `reply_to.mid` to reply to a particular message.
- This is deliberately an inbox reply tool, not a follower scraper or cold-DM/bulk sender. It does not message people who have not initiated an eligible conversation. Meta's messaging windows, policy, app review, and conversation-routing ownership rules still apply. If Meta rejects a send because the window expired or another app owns the thread, the API error is shown in the dashboard.

## Run locally

1. Install Node.js 20+.
2. Copy `.env.example` to `.env` and fill in the values.
3. Run:

   ```bash
   npm install
   npm start
   ```
4. Open `http://localhost:10000`.

If `DASHBOARD_PASSWORD` is set, the browser asks for HTTP Basic Auth. Use any username and that password.

## Meta configuration

Create/configure a Meta developer app with Instagram Messaging enabled. For this implementation, configure the Facebook Login / Page-linked Instagram professional account flow and obtain a Page access token for a person who has the Page's messaging task. Ensure the app has the permissions required by Meta for the endpoints you use, typically `instagram_manage_messages`, `instagram_basic`, and `pages_manage_metadata` (plus any permissions Meta currently requires for your app/review state). Add the app's webhook subscriptions and complete App Review / business verification where required.

Set these server-side environment variables:

| Variable | Meaning |
|---|---|
| `META_PAGE_ACCESS_TOKEN` | Page access token. Keep secret. |
| `INSTAGRAM_ACCOUNT_ID` | Instagram Professional Account ID (not username, not conversation ID). |
| `GRAPH_API_VERSION` | Defaults to `v27.0`; use a version supported by your Meta app. |
| `DASHBOARD_PASSWORD` | Strong password protecting the dashboard and API. |
| `PORT` | Defaults to `10000`; Render supplies this automatically. |

Never commit `.env` or put access tokens in browser JavaScript.

## Deploy to GitHub + Render

1. Create a new GitHub repository and upload/push this folder's contents.
2. In Render, choose **New → Web Service**, connect the repository, and use:
   - Build command: `npm install`
   - Start command: `npm start`
   - Health check path: `/health`
3. Add the environment variables above in Render → Environment. Do not add the token to GitHub.
4. Deploy. The app listens on `0.0.0.0` and Render's `PORT`.

A `render.yaml` Blueprint is included as an alternative; secret values are marked `sync: false` and must be entered in the Render dashboard.

## API routes

- `GET /health` — health check.
- `GET /api/conversations/:threadId` — reads a conversation and resolves the recipient participant.
- `POST /api/send-thread` — JSON body: `{ "threadId": "...", "message": "...", "replyToId": "optional message id" }`.

## Official references

- [Instagram Messaging API changelog](https://developers.facebook.com/documentation/business-messaging/instagram-messaging/changelog/)
- [Conversation object](https://developers.facebook.com/docs/graph-api/reference/conversation/)
- [Conversation messages edge](https://developers.facebook.com/docs/graph-api/reference/conversation/messages/)
- [Render Node/Express deployment](https://render.com/docs/deploy-node-express-app)
