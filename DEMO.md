# Fake Guard app demo

This is a development walkthrough of the features that are currently implemented. Demo products, posts, findings, and accounts are fictional; they are not real manufacturer or authority data.

## Start the app

1. Configure `.env` with a MongoDB connection and a strong `JWT_SECRET`.
2. From the project root, run:

   ```powershell
   npm run seed
   npm run dev
   ```

3. Open `http://localhost:3000`.

`npm run seed` is development-only. It refreshes the seeded product examples and deletes/recreates the two demo users; do not run it against production or valuable local data.

## Demo accounts

| Account | Sign-in identifier | Password |
|---|---|---|
| Main demo user | `maya.rahman@example.com` | `Demo-Pass-2026!` |
| Messenger peer | `messenger.test@example.invalid` | `Fake-Guard-Test-2026!` |

To test both sides of Messenger, sign in as Maya in one browser profile and the Fake Guard Test account in a separate browser profile or private window. From Maya's account, open Messenger, search for **Fake Guard Test**, and start the conversation. Open the same conversation from the peer account. Send messages in both directions, try replies/reactions/edit/delete, record a voice note with the microphone button, and load older messages after the conversation has more than one page. Use Messenger's separate **Chats** and **Calls** tabs to switch between conversations and call history. To try an actual call, start a voice or video call on one side and accept it on the other. Both clients need microphone permission; video calls also need camera permission. Test on `localhost` or HTTPS.

From the **Chats** tab, choose **New group**, set a name, search and select at least one other account, then create it. The creator is the first administrator. Open **Group info** to add/remove members or leave the group. Members can send text and voice notes; group voice/video calls are not supported.

Voice/video calls use WebRTC signaling and browser media devices; they are not simulated. STUN is configured by default, but networks that block direct peer connections need a deployed TURN service configured with `TURN_URLS` and `TURN_SHARED_SECRET`. The seed data does not fabricate successful calls. Group calls and end-to-end encryption are not implemented.

## Walk through the app

1. **Scan and product camera** — Open **Scan**. Choose **Enter a code manually** and try:

   | Code | Batch | Expected demo result |
   |---|---|---|
   | `8901234567890` | `A24-0718` | Example product record match |
   | `8901234567890` | `A24-0991` | The explicitly labeled demo alert `DEMO-014` |
   | `8901234567890` | `ZZZ` | No matching batch |

   Switch to **Product** or **Macro** for camera-based visual inspection (camera permission required). Product and Macro use configured inspection agents/model integrations and sourced local records; they are not proof of authenticity. QR and barcode modes can scan codes; manual code generation is also available.

2. **Home and stories** — Open **Home** to view the community feed. Use the post composer at the top of the feed to open the post editor, then try choosing an audience, reacting, and commenting. Stories expire after 48 hours; open your own story and choose **Delete this story** to remove it sooner. The feed is community content, not an official safety verdict.

3. **Reels** — Open **Reels** to browse the reel feed or create a reel. The editor supports the implemented recording/upload, trimming, speed, filter, and audio controls. Features that require live streaming or server-side video processing are not part of this demo.

4. **Messenger** — Follow the two-account instructions above. Text and recorded voice notes are stored by the server. Chats include delivery/read indicators, replies, reactions, editing within the allowed edit window, deletion for everyone, and groups with administrator-managed membership. The separate Calls tab contains call history; one-to-one voice/video calls use WebRTC. Arbitrary file attachments, channels, group calls, and E2E encryption are not implemented; do not use this demo for sensitive messages.

5. **Notifications** — Open **Notifications** to review available in-app activity, such as reactions, comments, follows, and missed calls. Browser call notifications require browser permission and the page to remain open.

6. **Profile and settings** — Open **Profile** to review the signed-in account and available profile/privacy preferences.

7. **Admin data console** — If `ADMIN_KEY` is configured, open `http://localhost:3000/manual` and enter it to manage sourced product/AI knowledge data and alerts. This is not a user-account moderation dashboard. Never share or publish the admin key.

## Boundaries of this demo

- The seeded products/findings and accounts are examples only.
- AI results depend on configured providers and available credentials; the app should not claim a product is genuine solely from an AI image assessment.
- Messages and calls are **not end-to-end encrypted**.
- Calls are one-to-one. TURN infrastructure must be deployed for networks that require relay.
- Group messaging, channels, attachments, and some advanced moderation/security features from the broader product plan are not implemented.
