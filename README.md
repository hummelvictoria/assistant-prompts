# Morgen-Digest — Build Guide

Turns the "Morgen-Digest" flow from the concept diagram into an actual Zapier
build. Two zaps: the 7:30 push, and the reply-handler that reads Julian's
answers back in.

## Before you start (blocking prerequisites)

1. **WhatsApp-Setup must be done first** — direct via Meta Business Manager,
   no BSP, using Zapier's native **WhatsApp Business** app (search for it
   when adding a step). It's marked "Premium" in Zapier's app directory,
   which usually means a paid plan is required to connect it — try
   connecting an account and see if Zapier prompts an upgrade before
   assuming you're blocked.
   - Create/use a Meta App (developers.facebook.com) and add the "WhatsApp"
     product to it.
   - In Meta Business Manager, verify the business and add a WhatsApp phone
     number under WhatsApp Manager.
   - When you add a "WhatsApp Business" step in Zapier and click "Connect
     a new account", Zapier walks you through authorizing against your Meta
     Business Account directly (OAuth) — no manual system-user token or
     phone-number-ID copying needed, Zapier handles that.
   - Submit the `morgen_digest` utility template (WhatsApp Manager >
     Message Templates) — body is just `{{1}}`, see the note in the Zap 1
     step 7 row below.
   - No separate webhook relay needed: the native "New Message Received"
     trigger handles Meta's webhook verification handshake itself.

   Nothing in step 7 (Zap 1) or Zap 2 works until this is done. If you want
   to test steps 1–6 of Zap 1 sooner, temporarily swap the WhatsApp send
   step for an email (native Zapier Gmail "Send Email" action) so you can
   see Claude's output end-to-end.
2. **Calendar**: Julian's appointments are kept in the Google Calendar of
   the same consolidated Gmail account (not Outlook) — so calendar data
   comes from the same Apps Script as the emails, no separate setup needed.
   For the reply-handling zap's "termin" path, use Zapier's native
   **Google Calendar: Create Event** action.
   - An Azure app registration (one-time, needs a Microsoft 365 admin) is
     still needed, but only for OneDrive (see point 5 below) — Outlook
     itself is no longer used anywhere in this build.
3. **Trello**: get an API key + token at trello.com/app-key. **The token
   must be authorized under Julian's own Trello account**, not whoever is
   building this Zap — the Trello API's "me" endpoints (assigned cards,
   notifications/mentions) resolve to whoever owns the token, so if you
   authorize it yourself instead of Julian, the digest would show your
   cards and mentions, not his. No board/list IDs needed for this step.
4. **Anthropic API key** from console.anthropic.com — separate from any
   Claude.ai login, this is the developer API key Zapier's Code steps call.
5. **OneDrive folder** with the two prompt files from this package uploaded:
   `prompt-morgen-digest.md` and `prompt-notiz-klassifizieren.md`. Zapier's
   native OneDrive app has no "read file content" action, so both Zaps
   read these via `code-onedrive-prompt.js` (Microsoft Graph API). This is
   the one thing that still needs the Azure app registration:
   - portal.azure.com (or entra.microsoft.com) > Microsoft Entra ID > App
     registrations > New registration.
   - Certificates & secrets > New client secret > copy the value
     immediately (only shown once).
   - Copy the Application (client) ID and Directory (tenant) ID from the
     Overview page.
   - API permissions > Add a permission > Microsoft Graph > Application
     permissions > add `Files.Read.All` > **Grant admin consent**.
   - See `code-onedrive-prompt.js` for exactly where these four values go.
6. Every secret above goes into **Storage by Zapier** (Zapier's built-in
   key-value store), not hardcoded into any Code step. Code steps read them
   via `StoreClient` or, simpler for a first build, as manually-entered
   Input Data fields on each step (still not visible in the code itself).

## How to actually build a step in Zapier's UI

Zapier's model: one Zap = one trigger + a chain of numbered action steps,
built top to bottom, tested one at a time. Every "Code by Zapier" row in the
tables below maps to the same recipe:

1. Click **+** to add a step, search for the app named in the table
   ("Code by Zapier", "Webhooks by Zapier", "Trello", "Microsoft Outlook",
   "WhatsApp Business", "Filter by Zapier", "Paths by Zapier").
2. For **Code by Zapier**: choose action event **"Run Javascript"**. You'll
   get two boxes — **Input Data** (key/value pairs) and **Code**.
   - In *Code*, paste the full contents of the matching `.js` file from this
     folder.
   - In *Input Data*, add one row per `inputData.xxx` used at the top of
     that file. For a static secret (API key, Trello token, phone number
     ID), just type the value straight into the field — Zapier stores it
     encrypted per step, so you don't need Storage by Zapier for a first
     build. For a dynamic value (e.g. `digest_text`, `system_prompt`), click
     the field and pick the output of an earlier step from the dropdown
     instead of typing.
3. For **Webhooks by Zapier**: choose **GET** or **POST** depending on the
   table, paste the URL (your Apps Script `/exec` link), add headers/query
   params if needed.
4. Click **Test step** — Zapier runs it live and shows you the real output.
   This is what lets the *next* step's Input Data dropdown offer that data
   as a mappable field, so always test steps in order, top to bottom.
5. Repeat for every row in the table below, then hit **Publish** and flip
   the Zap from Draft to **On**.

Practical order for Zap 1: build steps 1–6, testing each one, before
touching step 7 (WhatsApp send) — that way you can see Claude's actual
`digest_text` output in step 6's test panel before wiring up delivery. If
WhatsApp isn't ready yet, swap step 7 for a plain Gmail "Send Email" action
temporarily so the whole chain is testable end-to-end.

## Zap 1: Morgen-Digest (7:30, werktags)

| # | Step | App / Type | What it does |
|---|------|-----------|---------------|
| 1 | Trigger | Schedule by Zapier | Every Day, 07:30 |
| 2 | Filter | Filter by Zapier | Only continue if trigger day is Mon–Fri (use a Formatter "Format Date" step first to extract weekday name, then filter "does not contain" Sat/Sun) |
| 3 | Action | Webhooks by Zapier (GET) | Call your Apps Script `/exec` URL — see `apps-script.gs`. Returns unread/stale emails + today's Google Calendar events as JSON. |
| 4 | Action | Code by Zapier | `code-trello-cards.js` — fetches Julian's assigned cards (with due dates) + his unread mentions/comments, via Trello's `/members/me/...` endpoints |
| 5 | Action | Code by Zapier | `code-onedrive-prompt.js` — reads `prompt-morgen-digest.md` fresh via Microsoft Graph (`file_path` input = path to that file) |
| 6 | Action | Code by Zapier | `code-claude-digest.js` — calls Claude Sonnet with steps 3+4 data and the step-5 prompt, returns `digest_text` |
| 7 | Action | WhatsApp Business: Send Template Message | Template = `morgen_digest`, its one `{{1}}` variable mapped to `digest_text` from step 6, To = Julian's number |

Steps 3, 4, and 5 don't depend on each other — order between them
doesn't matter, they all just need to run before step 6.

## Zap 2: WhatsApp-Antwort → Trello/Kalender (Rückkanal)

Same pipeline also covers the "Notiz unterwegs" flow, since an unprompted
WhatsApp message and a reply to the digest are handled identically once
they hit the webhook.

| # | Step | App / Type | What it does |
|---|------|-----------|---------------|
| 1 | Trigger | WhatsApp Business: New Message Received | Native trigger, fires instantly when Julian messages. Check its test output for the exact field names (message text, sender, message type, media ID) — map those into later steps. |
| 2 | Action | WhatsApp Business: Get Attachment (only if voice note) | Input = Media ID from step 1. Follow with a transcription step (Whisper by Zapier / OpenAI) to turn it into text. |
| 3 | Action | Code by Zapier | `code-onedrive-prompt.js` — reads `prompt-notiz-klassifizieren.md` via Microsoft Graph (`file_path` input = path to that file) |
| 4 | Action | Code by Zapier | `code-classify-reply.js` — Claude Haiku returns `{type, title, description, due_iso, answer}` |
| 5 | Branch | Paths by Zapier | Branch on `type` from step 4 |
| 5a | Path: aufgabe | Trello: Create Card | List = wherever new tasks should land (pick a board/list when configuring this step — independent from the "assigned to me" reading logic in Zap 1), name = `title`, desc = `description`, due = `due_iso` |
| 5b | Path: termin | Google Calendar: Create Event | title = `title`, start/end from `due_iso` |
| 5c | Path: idee | Trello: Create Card | List = "Ideen" |
| 5d | Path: frage | — | skip straight to confirmation, using `answer` as the text |
| 6 | Action (end of each path) | WhatsApp Business: Send Freeform Message | To = Julian's number (from step 1), Text = confirmation built from the path's fields (see below) |

Confirmation text per path (type it directly into the "Send Freeform
Message" field, mixing static text with mapped fields from step 4):
- aufgabe: `✓ Als Aufgabe angelegt: "{title}"` (+ ` (fällig {due_iso})` if due_iso is set)
- termin: `✓ Termin im Kalender: "{title}" am {due_iso}`
- idee: `✓ Idee gespeichert: "{title}"`
- frage: just map `answer` directly as the message text

## Files in this folder

- `apps-script.gs` — deploy to script.google.com under the consolidated Gmail account (Gmail + Google Calendar data endpoint)
- `code-trello-cards.js` — Zap 1, step 4
- `code-onedrive-prompt.js` — Zap 1 step 5 (`file_path` = prompt-morgen-digest.md) and Zap 2 step 3 (`file_path` = prompt-notiz-klassifizieren.md) — same code, different Input Data
- `prompt-morgen-digest.md` — upload to OneDrive
- `code-claude-digest.js` — Zap 1, step 6
- `prompt-notiz-klassifizieren.md` — upload to OneDrive
- `code-classify-reply.js` — Zap 2, step 4

WhatsApp sending/receiving (Zap 1 step 7, Zap 2 steps 1/2/6) now uses
Zapier's native **WhatsApp Business** app directly — no custom code or
relay script needed for those steps anymore.

## Standing rule: German output

Every Claude-facing prompt in this system must explicitly force German
output, not just be written in German — source data (emails, transcripts,
voice notes) can be in other languages, and without an explicit instruction
Claude may mirror the source language instead of answering in German. Both
prompt docs here (`prompt-morgen-digest.md`, `prompt-notiz-klassifizieren.md`)
now state this. Carry the same explicit rule into the prompts for
Anruf-Pipeline, Anmeldungen, and Tagesabschluss when those get built.

## Known gaps / next decisions

- **Filter step for weekdays**: exact Zapier UI for date-formatting varies
  by account version — if "Format Date" doesn't expose weekday name
  directly, use a Code step instead (`new Date().toLocaleDateString('de-DE',
  {weekday:'long'})`).
- **This only covers Morgen-Digest.** Anruf-Pipeline, Anmeldungen, and
  Tagesabschluss are separate builds — say the word when you want the next
  one and I'll do the same treatment.
