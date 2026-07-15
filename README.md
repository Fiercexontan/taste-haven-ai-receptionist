# taste-haven-ai-receptionist
# 🍽️ Taste Haven — AI Voice Receptionist

An AI phone receptionist for a restaurant that **answers calls, understands natural speech, checks real-time table availability, books reservations, and sends branded email confirmations — with no human involved.**

Built as a portfolio project to demonstrate practical AI-agent + automation engineering: the hard part isn't making an AI *talk*, it's making the automation layer reliably check the real world, avoid double-bookings, and only tell a caller "confirmed" when it's actually true.

---

## ✨ What it does

- 📞 **Answers calls** with a natural voice and understands real customer requests ("a table for 4 this Friday at 7pm")
- 🗓️ **Checks Google Calendar in real time** before confirming anything
- ✅ **Books the reservation instantly** if the slot is free
- 🚫 **Politely declines and offers alternatives** if the slot is already taken
- 📧 **Sends an automatic, branded email confirmation** the moment the call ends (to the customer, with a copy to the restaurant)
- 💬 **Handles FAQs** — hours, menu, and location — directly in the voice assistant
- 📊 **Logs every call** — books *and* declines — to a Google Sheet for analytics and lead capture *(v1.1)*
- 🧹 **Self-cleaning calendar** — an hourly job automatically clears out reservations whose time has passed *(v1.1)*

---

## 🆕 What's new in v1.1

Two upgrades that turn the demo into a real operational system:

1. **Automatic call logging (Google Sheets).** Every reservation attempt is appended to a spreadsheet — whether the table is **Booked** or **Declined**. Captured fields: `Timestamp, Name, Phone, Email, Party Size, Date, Time, Outcome`. This gives the restaurant a live analytics log and lead-capture record with zero manual entry. Both log steps are **non-blocking**, so a logging hiccup never affects the caller's response.
2. **Automatic table cleanup (scheduled workflow).** A separate workflow (`workflow/taste-haven-table-cleanup.workflow.json`) runs **every hour**, scans the calendar, and deletes reservations whose end time has already passed — keeping the booking calendar clean and current. It **only** removes events titled `Reservation -` whose end time is in the past, so future bookings and the owner's personal calendar events are never touched.

---

## 🏗️ Architecture

```
                    ┌─────────────┐
   Customer  ──────▶│    Vapi     │  Voice AI: speech-to-text, reasoning (LLM),
   (phone call)     │  Assistant  │  text-to-speech, and tool calling
                    └──────┬──────┘
                           │  HTTP POST (tool call: book_table)
                           ▼
                    ┌─────────────┐
                    │   n8n       │  Orchestration / automation layer
                    │  Webhook    │
                    └──────┬──────┘
                           ▼
        ┌──────────────────────────────────────────┐
        │  1. Is Tool Call?  (filter non-booking events)
        │  2. Parse natural date/time/phone/email   │
        │  3. Get calendar events for that slot     │
        │  4. Decide availability (conflict check)  │
        │  5. IF available ──▶ Create event         │
        │                 ──▶ Respond to Vapi       │
        │                 ──▶ Send email (+ BCC)    │
        │                 ──▶ Log "Booked" to Sheet │
        │     ELSE        ──▶ "already booked" reply│
        │                 ──▶ Log "Declined" to Sheet
        └──────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────┐
  │  Separate scheduled workflow (v1.1):             │
  │  Every hour ──▶ Get past events ──▶ filter       │
  │  "Reservation -" events whose end time passed    │
  │  ──▶ delete them (calendar housekeeping)          │
  └─────────────────────────────────────────────────┘
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
   ┌──────────────────┐        ┌──────────────────┐
   │ Google Calendar  │        │  Gmail (email     │
   │ (real bookings)  │        │  confirmation)    │
   └──────────────────┘        └──────────────────┘
```

### Tech stack

| Layer | Tool |
|-------|------|
| Voice AI (STT, LLM reasoning, TTS, tool calling) | [Vapi](https://vapi.ai) |
| Automation / orchestration | [n8n](https://n8n.io) |
| Real-time availability & bookings | Google Calendar API |
| Confirmations | Gmail (email) |
| Call logging & analytics | Google Sheets *(v1.1)* |

---

## 🔄 How the workflow works (node by node)

1. **Webhook** — receives the POST from Vapi when the AI calls the `book_table` tool.
2. **Is Tool Call** — Vapi sends many event types (status updates, end-of-call reports, etc.). This IF node only lets real `tool-calls` through; everything else gets a harmless `200 ignored` response. *(This guard prevents the workflow from crashing on non-booking events.)*
3. **Code in JavaScript** — parses messy, human speech into structured data:
   - Natural dates → ISO dates ("this Friday", "tomorrow", "December 15" → `2026-12-15`)
   - Natural times → 24h (`7pm`, `7:30 pm` → `19:00` / `19:30`)
   - Nigerian phone normalization (`080...` → `+23480...`)
   - Email validation, with a fallback to the restaurant inbox if none was captured
4. **Get many events** — queries Google Calendar for events overlapping the requested slot.
5. **Decide Availability** — flags a conflict if any event already occupies the slot.
6. **If** — routes to either the booking path or the "unavailable" path.
7. **Create an event** — writes the reservation to Google Calendar.
8. **Respond to Webhook** — returns a spoken confirmation sentence to Vapi (the AI reads it out on the call). Uses the Vapi tool-call response format (`results[].toolCallId` + `result`).
9. **Send Confirmation Email** — sends a branded HTML confirmation to the customer, BCC to the restaurant. Non-blocking (`continueRegularOutput`) so a mail hiccup never breaks the call response.
10. **Log Booked Call** *(v1.1)* — appends a `Booked` row to the Google Sheet. Non-blocking.
11. **Respond to Webhook1** — the "that slot is already booked, want another time?" reply for the unavailable path.
12. **Log Declined Call** *(v1.1)* — appends a `Declined - Slot Unavailable` row to the Google Sheet. Non-blocking.

### Table Cleanup workflow *(v1.1, separate file)*

1. **Every Hour** — Schedule Trigger, runs hourly.
2. **Get Past Events** — fetches calendar events from the last 30 days up to now.
3. **Filter Ended Reservations** — keeps only events titled `Reservation -` whose end time has already passed (ignores personal events and future bookings).
4. **Delete Ended Reservation** — deletes each matched event. Non-blocking.

---

## 🧠 Engineering notes & lessons learned

These are the real problems solved during the build — the interesting part of the project:

- **Voice platforms send more than tool calls.** The webhook receives status updates, end-of-call reports, and more. The original workflow crashed reading `toolCalls[0]` on events that had none. Fix: an **"Is Tool Call" guard** that routes non-tool events to a safe acknowledgement.
- **Order of operations matters for voice UX.** The calendar write and the caller's spoken confirmation must happen *before* any slower side effect (email, logging). If a notification step runs first and fails, the caller is wrongly told the booking failed. Fix: **book → respond to caller → then notify/log**, with those steps set to non-blocking.
- **Real-world availability is the hard part.** Anyone can make an AI say "booked." Making it query a live calendar, detect a conflict, and *refuse* to double-book is where the engineering value is.
- **Email captured by voice is error-prone.** Solution combines a schema `email` field + a Vapi prompt rule to **read the address back and confirm**, plus a **restaurant-inbox fallback** so a booking record is never lost even if the email is missing/misheard.
- **Deliverability:** a brand-new Gmail sender can land in spam at first. For production, move to a **custom domain with SPF/DKIM**.
- **Google Sheets append is header-sensitive (v1.1).** The Sheets node matches incoming fields against the sheet's existing header row. If a node upstream (e.g. Gmail) passes its own output through and the log step auto-maps, it can write the wrong headers; and if the header row later drifts, the node errors with *"Column names were updated after the node's setup."* Fix: use **explicit field mapping** (not auto-map) and keep the header row exactly `Timestamp, Name, Phone, Email, Party Size, Date, Time, Outcome`.
- **Deletion safety (v1.1).** The cleanup job never deletes blindly — it filters on the `Reservation -` title prefix **and** a past end time, so personal calendar events and future bookings are always left alone.

---

## 🚀 Setup

See **[docs/SETUP.md](docs/SETUP.md)** for full step-by-step instructions.

Quick version:
1. Import `workflow/taste-haven-booking.workflow.json` **and** `workflow/taste-haven-table-cleanup.workflow.json` into your n8n instance.
2. Connect your **Google Calendar**, **Gmail**, and **Google Sheets** credentials.
3. Replace the placeholders (`YOUR_CALENDAR@example.com`, `YOUR_RESTAURANT_EMAIL@example.com`, `REPLACE_WITH_YOUR_SPREADSHEET_ID`, credential IDs).
4. Create a Google Sheet with a header row of exactly: `Timestamp | Name | Phone | Email | Party Size | Date | Time | Outcome`, and point both log nodes at it.
5. Publish the booking workflow and copy the production webhook URL. Publish the cleanup workflow so it runs hourly.
6. In Vapi, create an assistant, add the `book_table` tool pointing at your webhook, and paste the system prompt from `vapi/system-prompt.md`.
7. Place a test call.

---

## 📁 Repository structure

```
taste-haven-ai-receptionist/
├── README.md                          # You are here
├── LICENSE
├── .gitignore
├── workflow/
│   ├── taste-haven-booking.workflow.json        # Main booking workflow
│   └── taste-haven-table-cleanup.workflow.json  # Hourly calendar cleanup (v1.1)
├── vapi/
│   ├── system-prompt.md               # Full Vapi assistant system prompt (FAQ, menu, rules)
│   └── book_table.tool.json           # Vapi tool schema for the booking function
└── docs/
    ├── SETUP.md                       # Step-by-step setup guide
    └── DEMO.md                        # Demo video script + LinkedIn caption
```

---

## ⚠️ Notes

- This is a **portfolio / demonstration project**. "Taste Haven" is a fictional restaurant.
- The default, zero-cost confirmation channel is **email (Gmail)**.
- The exported booking workflow may still contain a **disabled Twilio/WhatsApp node** from an earlier iteration. It is turned off and not part of the active flow — you can safely ignore or delete it.

---

## 📬 Contact

Built by **LincolnAdura(Fiercexontan)** — I build AI voice agents and automation systems for businesses losing bookings to missed calls and manual scheduling.

If you'd like a system like this for a clinic, salon, or restaurant, let's talk.
