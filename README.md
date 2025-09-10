All-American English — AI SDR (Resend + Inbox Replies)

AI SDR for an online English school. Drafts cold emails, converts them to branded HTML, and sends via Resend. Replies are routed to your chosen inbox with reply_to. An optional IMAP poller can generate short, context-aware answers with an LLM. Full automatic reply is work in progress while evaluating providers that support simple two-way email (send + receive) with inbound webhooks.

Features

✉️ Outbound (fast path): single-draft turbo flow → draft → subject → HTML → send (Resend)

🎨 Branding: clean, mobile-friendly HTML template (header, CTA, offer list, footer + socials)

🔁 Threading: In-Reply-To / References to keep the same email thread

📥 Replies: routed to any mailbox via reply_to

🤖 Auto-reply (optional): IMAP poller + reply agent for short, context-aware responses (WIP)

Requirements

Python 3.10+

Install:

pip install -U agents resend python-dotenv nest_asyncio
# IMAP auto-reply uses stdlib: imaplib, email  (no extra install)
# If you later use Gmail API/push instead of IMAP, install Google client libs.

.env (required)

Create a .env file in the repo root:

# Resend (outbound)
RESEND_API_KEY=your_resend_api_key
RESEND_FROM=onboarding@resend.dev         # or a verified sender at your domain
RESEND_TO=you+tests@example.com           # default recipient for quick tests
RESEND_SUBJECT=Want to speak native-level American English?

# Where human replies should go (your real inbox)
REPLY_TO=english.all.american@gmail.com

# Optional display prices (used in template text)
PRICE_PLN=120
PRICE_USD=25

# Optional IMAP auto-reply (Gmail)
IMAP_SERVER=imap.gmail.com
IMAP_USER=english.all.american@gmail.com
IMAP_PASS=your_16_char_gmail_app_password


Notes

Use onboarding@resend.dev while testing. For production, verify a sender/domain in Resend and set RESEND_FROM accordingly.

For Gmail IMAP, enable 2FA and create an App Password.

Quick start
1) Notebook (recommended)

Open all_american_english_sdr.ipynb and run Part 1 → Part 2 → Part 3.

Send a fast test email with the turbo path:

await turbo_send(
    "Write a cold email for a potential student who wants online English lessons; "
    "greet with 'Greetings' and sign from Mike Taylor III — Founder & English Teacher."
)

2) Script mode (optional)
import asyncio
asyncio.run(turbo_send("...your prompt..."))

Project structure (what each “Part” does)

Part 1 — Imports & Sender Tools

Loads .env

Fast function tools: subject_writer, html_converter

Senders:

send_html_email(subject, html_body) → sends to RESEND_TO

send_html_email_dynamic(to_email, subject, html_body, in_reply_to, references) → threaded replies

Plain-text fallback for deliverability; sets reply_to to route human replies

Part 2 — Tools & Email Manager

Three sales agents for drafting:

benefit-led, social-proof, concise (plain text only)

Email Manager agent:

calls subject_writer → html_converter → send_html_email

outputs a one-line confirmation and stops

Reply Writer agent:

short, context-aware replies; uses lesson/teacher facts; never echoes the user

Part 3 — Turbo pipeline (fast send)

turbo_send(prompt):

drafts once with the concise agent

hands off to Email Manager (subject + HTML + send)

Part 4 — Inbound replies (optional)

handle_inbound_email(from_email, subject, text, message_id, references):

uses Reply Writer → html_converter → send_html_email_dynamic

keeps the thread via headers

Part 5 — IMAP poller (optional)

Polls your inbox and calls handle_inbound_email(...) automatically

How replies work today

Outbound emails include:

"reply_to": ${REPLY_TO}


When the recipient replies, it lands in that inbox.

(Optional) An IMAP poller fetches unread messages and calls the Reply Writer agent to draft a short response, converts to HTML, and sends via Resend with threading headers.

Automatic reply is WIP. I’m developing a dedicated agent for reliable back-and-forth and evaluating a provider that supports send + receive with simple inbound webhooks (to avoid IMAP complexity).

Troubleshooting

Nothing sends
Check RESEND_API_KEY. Use RESEND_FROM=onboarding@resend.dev or verify your domain in Resend.

Takes too long
Use Turbo (Part 3). It does one LLM call, then local function tools.

HTML shows <br> as text
Part 1 normalizes HTML-ish breaks before escaping (already included).

No auto-reply
Ensure your listener calls handle_inbound_email(...) (enable Part 5 and set IMAP creds).
Remember: full automated replies are in progress.

Agents loop
Keep max_turns small and follow the STOP rules in agent instructions.

Security

Keep .env out of git (ensure it’s in .gitignore).

Use a Gmail App Password (never your normal password).

Rotate API keys regularly.

Roadmap

 Turbo send (single LLM draft → subject → HTML → send)

 Threading + reply_to

 IMAP poller (optional) for inbound reads

 Provider with native inbound webhooks (send + receive)

 Dedicated auto-reply agent for continuous, safe back-and-forth

License

MIT (or your preferred license)
