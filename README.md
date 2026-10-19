# Fake Guard

Scan a product, understand the evidence, protect your community. Fake Guard is a Node.js + Express app with an installable web frontend in `frontend/public/`; its current backend database is MongoDB. A Supabase/PostgreSQL cutover is not complete yet. See [Supabase migration status](./supabase/README.md) and the secret-free [Supabase config example](./supabase/demo.env.example).

## Run it (5 minutes)

1. Copy `.env.example` to `.env`, then set `MONGO_URI` (your existing MongoDB) and a long random `JWT_SECRET`.
2. In the project folder:
   ```
   npm install
   npm run seed      # loads the demo product, the flagged batch and a demo user
   npm run dev
   ```
3. Open http://localhost:3000

See [DEMO.md](./DEMO.md) for a guided tour of the app and a two-account Messenger/call walkthrough.

With `AUTO_VERIFY=true` new accounts are treated as verified, so you can test everything without email or SMS.
Reset links and phone codes are printed in the server console while SMTP / Twilio are not configured.

## Admin panel and user panel

### Admin panel

Open `http://localhost:3000/manual` (or the port configured by `PORT`) and enter the `ADMIN_KEY` value from the server's `.env` file. The key is kept only in that browser tab's session storage. Set a long, unique random value for `ADMIN_KEY` and restart the server after changing it. Never publish the key, put it in frontend code, or share it with regular users. If `ADMIN_KEY` is empty, admin API actions are disabled.

The admin console supports sourced AI knowledge notes, product/batch records, alerts, and AI/Python-agent connection checks. The alert form requires explicit confirmation and a source reference before it can notify users. This is a data/AI administration console, not a user-account management dashboard; it does not provide tools to list, edit, suspend, or delete user accounts.

### User panel

Open the app at `http://localhost:3000`, register or sign in, then use the bottom navigation and Profile/Settings. Users can manage their own profile and preferences, use the community feed and notifications, and view their activity. Users do not need or receive the admin key.

For development-only test data, run `npm run seed` and sign in with `maya.rahman@example.com` / `Demo-Pass-2026!`. It also creates the verified `Fake Guard Test` Messenger peer with fictional phone `+1 (202) 555-0142`; sign into that peer in a second browser with `messenger.test@example.invalid` / `Fake-Guard-Test-2026!` to test both sides of a call. The seed command refreshes demo data and deletes/recreates both demo accounts; do not use it against production data. In production, set `AUTO_VERIFY=false`, configure real verification providers, use HTTPS, and store secrets in the hosting provider's secret manager rather than committing `.env`.

## Demo data (after `npm run seed`)

| Try this on the Scan screen > "Enter a code manually" | Result |
|---|---|
| code `8901234567890`, batch `A24-0718` | Product record matched |
| code `8901234567890`, batch `A24-0991` | Already flagged as counterfeit (alert DEMO-014) |
| code `8901234567890`, no batch | Asks for the batch |
| code `8901234567890`, batch `ZZZ` | Batch not on record |
| any other code | No record found |

Demo login: `maya.rahman@example.com` / `Demo-Pass-2026!`

To test the camera, make QR codes from these texts with any QR generator:
`8901234567890|A24-0718` and `8901234567890|A24-0991`.
The Scan screen can also generate text QR codes and EAN-13 barcodes (12 digits to calculate the check digit, or 13 digits to validate it). Keep the camera running in QR or Barcode mode: recognized codes open their decoded information or product-record result automatically. Links in QR codes are shown but never opened automatically.
GS1 links such as `https://id.gs1.org/01/08901234567890/10/A24-0718` also work. The camera needs https (or localhost). The Scan screen offers **Product**, QR code, Barcode, and **Macro** modes. Product and Macro open the AI-assisted inspection page; Macro adds tap-to-focus digital zoom up to 300× and sends the visible crop through the same product-analysis agents. Digital zoom is not 300× optical magnification and can reduce detail significantly. Product inspection offers the initial 100 product-category choices as user-selected hints and an optional focus note; Gemini may use the hint but it is not a trained 100-class classifier. A local center-area median sRGB estimate is available for photos or camera frames, but it is uncalibrated: no segmentation, color-reference catalog, colorimeter, CIELAB, or ΔE00 comparison is implemented, so it is not precise color matching. Inspection reads visible text with Gemini and asks Gemini for visual guidance when a sourced local record cannot answer. The configured `GEMINI_MODEL` is tried first; if that model is unavailable or rate-limited, the client tries models supported by the API key and then stable Flash fallbacks. The Python vision agent uses generic pretrained COCO classes for common objects (including bottles, phones, backpacks and handbags) when custom weights are absent; COCO has no shoe or brand classes and cannot determine authenticity. Custom object classes require collecting/labeling images and training a YOLO model with `backend/agent/train.py`. There is no million-variant catalog or product retrieval model configured, and no CNN training or evaluation has been run. **Create QR from a photo** makes a downloadable QR containing a small compressed copy of the selected image; **Open camera to create QR** captures a frame and uses the same local QR flow. The original image stays on-device and is not uploaded.

## Folder map

```
server.js                     compatible app entry point
frontend/public/              web app (HTML, CSS, JavaScript, PWA assets)
backend/server.js             Express API and static-file server
backend/routes/               authentication, scans, alerts, AI and profile APIs
backend/models/               MongoDB collections
backend/middleware/           authentication middleware
backend/utils/                email/SMS, AI, Apple and shared helpers
backend/seed.js               demo data
backend/seed-manual.js        optional example AI knowledge data
backend/agent/                optional Python vision and barcode service
test/                         automated Node.js tests
uploads/                      uploaded report evidence
.well-known/                  Apple's domain verification file
.env, package.json            root-level configuration and npm commands
```

The frontend is served by the backend from `frontend/public/`; keep browser assets and API routes in their respective folders. Run `npm test` for the automated tests and `npm run seed:manual` to load the optional sample notes.

## Community feed and reels

**Current navigation and Messenger layout:** the five equal-width bottom destinations are Scan, Home, Reels, Messenger and Profile. Notifications remain available from the top bar and are not a bottom tab. Messenger is branded with Fake Guard's own name/logo and presents Chats, Updates, Communities, Calls and Settings; Chats includes All/Unread/Groups filters, Communities lists existing group conversations, and Updates links to Home Stories, which expire 24 hours after creation. WhatsApp branding is not used. Features not implemented in this app—such as chat document attachments, typing/presence, standalone 24-hour messaging status and end-to-end encryption—are not presented as active.

**Feed and Stories update:** Home reuses the existing authenticated Node/Express and MongoDB feed with cursor-based loading and persisted post interactions. Stories now expire 24 hours after creation; older references to 48 hours in the overview below are superseded. The web application has not been migrated to React Native or Supabase.

The bottom navigation includes **Scan, Home, Reels, Messenger and Profile** in equal-width slots; Notifications remain in the top bar. Signed-in Home headers provide profile search, an unread notification badge beside Messenger, and links to their respective screens. Messenger is a separate route and is not embedded in Home or Reels. The Scan page combines QR and barcode recognition in one camera mode; generated text QR codes and validated EAN-13 barcodes are available. Macro/Product camera mode includes live object detections, camera switching, device/digital zoom, movable crop point, autofocus, manual focus and torch when supported by the camera, plus gallery import and scan history. Signed-in users can publish/edit/delete posts, attach up to five photos or one video, tag public profiles, add a feeling or place label, react, comment/reply and share links. Clicking Home's post prompt opens the existing post editor; it supports direct rear/front camera capture, tap-to-focus requests, zoom, supported torch/exposure controls, up to five angle-labeled photos, gallery selection and advisory low-light/glare checks. Camera permissions and hardware controls are browser/device-dependent; captured images stay in the post composer until submission, and offline publishing/queued uploads and automatic stable-product capture are not implemented. Use Product camera for live detections and record-backed inspection; visual AI guidance and image-quality hints do not establish authenticity. Comments support emoji reactions and optional microphone-recorded audio notes where browser recording is available. Post visibility is **Public** or **Only Me**; private posts and their media are restricted to the author. Choose relevant/recent/favorites feed order in Settings. In-app notifications cover reactions, comments, shares, follows, and explicit admin-published counterfeit alerts; admins must confirm a sourced counterfeit finding before broadcasting it to accounts with notifications enabled. Sample/example findings, unconfirmed statuses, and individual scan/AI estimates are never broadcast. Stories expire after 24 hours; owners can delete their stories from the story viewer, and uploaded story media is removed on deletion or expiry. Public-profile stories support emoji reactions and private replies, visible to the author only. Reels snap vertically, loop, allow tap-to-toggle sound, show a public rotating sound disc using the creator's audio label, and support double-tap likes, follow, bookmarks, share, download, camera recording, and browser-based trimming, speed, preset color filters, adjustable filter strength and audio export where supported. Portrait F1–F12, Landscape B1–B12, Food G1–G7, and Vibe V1–V10 color presets render through the browser canvas export; these are color grades, not face-aware beauty or AR effects. Reel discovery uses a MongoDB text index over captions and user-provided audio information (hashtags are part of captions), then ranks a bounded candidate set using text relevance, recency-weighted likes/comments and a small boost for creators the signed-in user follows. Spoken-word transcription, OCR indexing, individual watch-time/replay tracking, AI moderation, copyright detection, server-side transcoding and seeded/tiered audience distribution are not configured. Reel recording and filtering happen locally in supported browsers; uploaded video is stored as supplied. Live streaming is not enabled: a public HTTPS service, WebRTC signaling, and STUN/TURN infrastructure must be deployed before the Live entry can work. Face tracking/beautification, segmentation, and AR effects are not included. Friends-only/custom audiences and page/group feeds are not configured. Uploaded media and public profile photos are stored under `uploads/`; use persistent storage when deploying.

### Reels comments

Reel comment threads are integrated with existing `Post` records of kind `reel` and the shared MongoDB `Comment` model; they are not a separate demo or a React Native/Supabase implementation. The sheet supports top/newest cursor-paginated top-level comments, on-demand paginated replies (up to three levels), optimistic comment counts and likes with rollback, editing/deletion, creator pinning, emoji insertion, mentions/hashtags formatting, reports, user blocks, creator-selected everyone/followers/off permissions, and creator word filters. Video playback follows the **Pause while comments are open** Reels preference. Comment/reply/mention notifications use the existing global notification preference and deduplicated notification event keys. Soft-deleted comments remain as thread placeholders. Comment reports are stored for review; a moderator review console, automated abuse classifier, Supabase Realtime, and per-notification-type preferences are not implemented.

The Express routes are mounted at `/api`:

| Method | Route | Access |
|---|---|---|
| GET, POST | `/api/videos/:videoId/comments` | Public read when the Reel is public; sign-in required to write |
| GET, POST | `/api/comments/:commentId/replies` | Public read when visible; sign-in required to write |
| PATCH, DELETE | `/api/comments/:commentId` | Author edits; author or Reel owner removes |
| POST, DELETE | `/api/comments/:commentId/like` | Sign-in required; duplicate likes are prevented |
| POST, DELETE | `/api/comments/:commentId/pin` | Reel owner only |
| POST | `/api/comments/:commentId/report` | Sign-in required |
| POST, DELETE | `/api/users/:userId/block` | Sign-in required |
| GET, PATCH | `/api/videos/:videoId/comments-settings` | Reel owner only |

Run from the project folder in PowerShell on Windows:

```powershell
Set-Location 'C:\path\to\fake-guard'
npm install
npm test
npm run dev
```

Set `MONGO_URI` and `JWT_SECRET` in `.env` as described above before starting the app. No SQL migration was added: the running app uses MongoDB/Mongoose, and an unconnected PostgreSQL migration would not enforce these routes' permissions.

Home shows a post composer, stories and community posts, while Reels stays on its own screen and has separate navigation entries. Desktop Home uses a three-column layout with supported navigation, profile discovery and product-safety links; narrower screens remain a single-column feed. Stories expire 24 hours after creation unless the owner deletes them earlier; legacy stories are also inaccessible after 24 hours and their media is removed by the server's periodic cleanup. Search results show public profile counts, quick follow/unfollow, and a Friends badge when both accounts follow each other. Private-account mode removes the account's public posts and profile from discovery; it does not yet implement manual follow approvals. Users can set their following list to public/private, choose system/light/dark appearance, and manage their photo by clicking it on the profile. Profile totals show followers, following, and reactions received on posts. New registrations receive a persisted unread notification welcoming them from Your Brother Zabih Ullah.

Messenger has separate **Chats** and **Calls** tabs; call history is separate from chat threads. Chats support one-to-one and group text conversations, admin-managed group membership, cursor-based older-message loading, editable text and voice notes, replies, emoji reactions, unread counts, and sent/delivered/read indicators. Voice notes are private to conversation members and limited to 10 MB. One-to-one voice/video calls use WebRTC signaling, browser device permissions, call history, and missed-call handling. Incoming calls show an in-app alert from any app screen while Fake Guard is open; optional desktop notifications are controlled in Messenger settings. STUN is configured by default; TURN must be deployed for networks requiring relay. Group calls, arbitrary file attachments, presence, typing indicators, blocking/reporting, and end-to-end encryption are not implemented. See [DEMO.md](./DEMO.md) to test the app and two-party calling with separate development demo accounts. Optional SMS two-step verification can be enabled in Settings with an internationally formatted phone number; it applies to password, phone-code, and social sign-ins. Configure Twilio for production; development without SMS configured displays a test code in the app. SMS delivery and carrier availability vary by country. Reel controls show Like, Comments, Share, and More; bookmark, download, sound, and owner edit/delete actions are in More. Creator profile, follow/unfollow, and direct-message shortcuts are available beside the reel caption. Settings provides an English-language country/region selector with ISO country and territory names. The Live entry is disabled until public HTTPS WebRTC signaling and STUN/TURN infrastructure are configured. Settings includes an on-device face-framing preview with continuously animated green guide markers. The animation is visual only; face-based identity recognition, social-media lookup, selfie storage, and sending face video to AI are not provided.

Profile settings let users upload/remove their photo and explicitly check browser location permission. The app stores only the location preference, not coordinates. The profile dashboard graphs recorded posts/scans/comments and offers an optional AI summary based on actual counts. AI avatar creation uses Gemini's image-generation Interactions API; set `GEMINI_API_KEY` and, if needed, `GEMINI_IMAGE_MODEL` (default `gemini-nano-banana-2.1`). If the configured key/model lacks image access, the UI shows the provider error; it never substitutes a fake image.

Optional identity verification uses Persona's hosted flow. Configure `PERSONA_API_KEY`, `PERSONA_TEMPLATE_ID`, and `PERSONA_WEBHOOK_SECRET` in the server `.env`; the Persona template determines which document, selfie, image-quality, and liveness checks are requested. Register `https://YOURDOMAIN/api/kyc/persona/webhook` in the Persona dashboard and enable the `inquiry.approved` and `inquiry.declined` events. The webhook signature must be fresh (within five minutes), and the app verifies its inquiry and user reference before changing status. The blue identity badge is awarded only after a valid signed Persona `inquiry.approved` webhook for that user's inquiry. The Trust profile separates Persona's overall inquiry decision from social OAuth connections and records consent and minimal verification audit events. The integration does not receive distinct face-match or liveness results, so it explicitly reports them as unavailable; Fake Guard does not perform face recognition. The app stores the provider inquiry ID, consent history and resulting status, not selfie or document images. Withdrawing consent stops Fake Guard from accepting a pending result, but provider-side retention/deletion requests must be made to Persona. Use a public HTTPS endpoint for webhook testing; localhost cannot receive Persona webhooks directly. Apple Face ID does not share facial biometrics or confirm a person's legal identity to this web app. Contact verification is not identity verification. Phone login supports either the account password or a one-time SMS code. Configure Twilio for production SMS; local development can display a test code.

## Social login

Social login requires real provider credentials; empty or placeholder values cannot sign a user in. The app buttons show a clear configuration error until their server-side settings are present. To enable each provider:

- **Google**: Google Cloud Console > Credentials > OAuth client ID (Web). Add the exact app origin (for local development: `http://localhost:3000`) as an authorised JavaScript origin. Put that Web client ID in `GOOGLE_CLIENT_ID`.
- **Facebook**: Meta for Developers > create app > Facebook Login. Add the exact app domain and enable the Facebook Login product. Set `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET`, and a currently supported Graph API version in `FACEBOOK_API_VERSION`.
- **Apple**: needs a paid Apple Developer account. Create an App ID and a Services ID with Sign in with Apple, add your https domain and the return URL `https://YOURDOMAIN/api/auth/apple/callback`, put Apple's verification file in `.well-known/apple-developer-domain-association.txt`, and create a Sign in with Apple key. Set `APPLE_SERVICES_ID`; also set `APPLE_TEAM_ID`, `APPLE_KEY_ID`, and `APPLE_PRIVATE_KEY` so account deletion can revoke access.

Apple, Google and Facebook sign-in all need a real https domain. They will not work on a cloudflared quick link because that address changes every run. The current `.env` is missing the Google client ID and Facebook app ID/secret; Apple web sign-in also needs a valid Services ID and verified domain. Add the credentials from each provider's developer console to `.env` before expecting those buttons to authenticate.
Provider credentials are issued by Google, Meta and Apple and cannot be generated by this app. If a provider's required value is missing, its button reports that configuration is needed; the app never pretends the sign-in succeeded. Settings can connect a provider to an already signed-in Fake Guard account using a separately authorized OAuth flow. A connected OAuth account demonstrates account authorization at connection time, not the holder's real-world identity.

## AI agents and model

The product inspection uses three local checks: a YOLO vision agent, a QR/barcode reader, and a sourced product/alert record lookup. If those do not produce an exact record-backed answer, the configured AI model can assist with image analysis. The scanner can optionally sample the live camera about every 1.5 seconds (network/model response time may be longer), read visible object text, show decoded QR content without opening it automatically, and look up recognized product barcodes. A photo QR stores a compressed thumbnail directly in the QR and can be read back in Fake Guard without a server upload. Macro zoom uses device lens zoom if supported and a digital crop otherwise. AI analysis is not manufacturer verification. The optional specialized packaging detector requires custom trained weights. If the Python service is unavailable, the browser scanner and configured AI model remain available.

To enable the local Python vision and QR/barcode agents, follow the Windows setup in [SETUP-AI.md](./SETUP-AI.md). Configure `PY_AGENT_URL=http://127.0.0.1:8001`; start the Node app separately with `npm run dev`.

## Deploy

This repository includes a Render Blueprint (`render.yaml`) for a free demo deployment. It builds with `npm install`, starts with `npm start`, and checks `/api/health`. The health endpoint returns unavailable until MongoDB is connected.

1. Push the project to a GitHub repository. Do not add `.env`, `node_modules`, or `uploads`.
2. In Render, choose **New > Blueprint**, connect the repository, and deploy `render.yaml`.
3. During setup, provide the production `MONGO_URI` and a temporary `BASE_URL` such as `https://fake-guard.onrender.com`. After Render assigns the actual service URL, update `BASE_URL` in the service's environment settings to that exact HTTPS URL and redeploy. If the chosen Render hostname differs, use the assigned one.
4. Render generates `JWT_SECRET` and `ADMIN_KEY`. Keep both private. Copy the generated `ADMIN_KEY` from the Render environment settings to sign in to `/manual`; never put it in frontend code or commit it.
5. The free Render plan uses an ephemeral filesystem: uploaded community media can disappear when the service restarts or redeploys, and the service may sleep when idle. This is suitable for a demo, not durable user uploads. Before production use, move media to object storage or select hosting with persistent storage. Configure SMTP for email verification and password reset; configure Twilio if phone OTP is needed. Add any optional AI, social sign-in, or Persona secrets directly to Render's secret environment settings.

### Railway

Set the Railway service's **Root Directory** to the project root (the folder containing `package.json`), with build command `npm install` and start command `npm start`. Ensure the deployed branch/commit includes `server.js`, the entire `backend/` directory, `frontend/`, `package.json`, and `package-lock.json`; the root entrypoint loads `backend/server.js`. Set `MONGO_URI` to a reachable MongoDB deployment and `JWT_SECRET` to a long random secret in Railway's Variables. Set the healthcheck path to `/api/health`. This endpoint returns HTTP 503 until the MongoDB connection is ready, so missing/invalid credentials or an unreachable database will fail the healthcheck even if the Node process starts. Do not upload or commit `.env`, `node_modules/`, or `uploads/`.

## Important notes

- Product records and alerts are **examples**. There is no live manufacturer verification and no government connection. Reports are private drafts and nothing is ever sent to an authority.
- "Automatically prepare reports" is opt-in and off by default. It only creates a private draft when a flagged batch is scanned.
- The Terms, Privacy and Guidelines text in the app are short summaries. Replace them with real legal text before a public launch.
- The Apple button uses a simple inline logo. For App Store review, follow Apple's Sign in with Apple button guidelines.
- Phone push notifications are not configured; the Notifications screen is an in-app inbox. For a phone-share/Jenkins URL, deploy the app to a public HTTPS domain and set `BASE_URL`; Jenkins also needs a configured server and job URL. A localhost URL is not reachable by other phones.
