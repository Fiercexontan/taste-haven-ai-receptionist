# Demo Video Script & LinkedIn Caption

A ready-to-use script for a 60–90 second demo video and a LinkedIn caption to publish it.

---

## Video Script (target: 60–90 seconds)

### Recording setup
- Screen-record your laptop with the **Vapi test call window** and the **n8n canvas** visible.
- Keep your phone / a second window ready to show the **email confirmation** and **Google Calendar** arriving in real time.
- Do one full dry run before your final take.

### Shot 1 — Hook (0:00–0:07)
> "I built an AI receptionist that answers restaurant calls, books real reservations, and checks availability — with zero humans involved. Here's how it works."

### Shot 2 — The call (0:07–0:30)
On screen: the Vapi call interface. Say naturally:
> "Hi, I'd like to book a table for 4 people this Friday at 7pm. My name is Ada, my number is 0803…, and my email is ada at gmail dot com."

Let the AI respond fully — including reading the email back to confirm. This is the core proof; don't cut it short.

### Shot 3 — Behind the scenes (0:30–0:50)
Switch to the n8n canvas as nodes light up green:
> "Behind the call: it parses natural speech, checks Google Calendar for real-time availability, books the slot, and sends a confirmation — all in seconds."

Zoom briefly on: `Get many events` → `Decide Availability` → `If` → `Create an event`.

### Shot 4 — Proof it's real (0:50–1:05)
Quick cut between Google Calendar (new event) and the email inbox (confirmation):
> "No fake demo — that's a real calendar event and a real confirmation email, generated the moment the call ended."

### Shot 5 — The "unavailable" case (1:05–1:20)
Call again, ask for the same slot:
> "Hi, can I book a table for Friday at 7pm?"

Let the AI say it's taken and offer an alternative.
> "It also knows when NOT to book — checking real availability before confirming anything."

### Shot 6 — Close / CTA (1:20–end)
> "Built with Vapi, OpenAI, n8n, and Google Workspace (Calendar, Gmail, Sheets). If you're a business losing bookings to missed calls, this is the kind of system I build. Let's talk."

---

## v1.1 add-on demo (call logging + self-cleaning calendar)

Use this as a short follow-up clip or an extension of the main video.

### Call logging
1. Open the **reservations Google Sheet** full-screen, cleared to just the header row (`Timestamp, Name, Phone, Email, Party Size, Date, Time, Outcome`).
2. Place a booking call for a free slot → after the call, refresh the sheet → a new **"Booked"** row appears with caller details + timestamp.
3. Place a second call for the *same* slot → the AI declines → refresh → a new **"Declined - Slot Unavailable"** row appears.
> "Every call is now logged automatically — booked or declined — so the restaurant gets a live record of demand and lost bookings, with zero manual entry."

### Self-cleaning calendar
- Show the **Table Cleanup** workflow canvas and its green execution history.
> "And a scheduled job runs every hour to clear out reservations that have already passed — keeping the calendar clean automatically. It only touches past reservations, never future bookings or personal events."

### v1.1 LinkedIn caption

> 🎙️ Update: my AI voice receptionist for restaurants just got smarter.
>
> A few weeks ago I built an AI agent that answers the phone, understands a caller's reservation request, checks the calendar in real time, and books the table — hands-free.
>
> This week I shipped two upgrades that make it a real operational tool, not just a demo:
>
> 📊 **Automatic call logging** — every call is now recorded to a live spreadsheet: caller name, phone, party size, requested time, and outcome (booked or declined). Instant analytics and lead capture — no clipboard, no manual entry.
>
> 🧹 **Self-cleaning calendar** — a scheduled job runs every hour and automatically clears out reservations that have already passed, keeping the booking calendar clean with zero manual work.
>
> The result: a receptionist that books tables, keeps perfect records of every call, and tidies up after itself. 24/7.
>
> Built with n8n, Vapi, and Google Workspace.
>
> Demo below 👇
>
> #AI #Automation #n8n #VoiceAI #NoCode #RestaurantTech #Vapi

---

## LinkedIn Caption

> I built an AI Voice Receptionist for restaurants — and it doesn't just answer calls, it actually runs the front desk. 🍽️
>
> The brief I set myself: build something that could realistically replace the repetitive parts of a host's job — answering calls, checking availability, booking tables, confirming details — without a human touching it.
>
> **What "Taste Haven's" AI receptionist does:**
> ✅ Answers calls with a natural voice and understands real customer requests
> ✅ Checks Google Calendar in real time before confirming anything
> ✅ Books the reservation instantly if the slot is free
> ✅ Politely declines and offers alternatives if it's already taken
> ✅ Sends an automatic email confirmation the moment the call ends
> ✅ Handles FAQs (hours, menu, location)
>
> **Stack:** Vapi (voice AI) + OpenAI (reasoning) + n8n (automation/orchestration) + Google Calendar API (real availability) + Gmail (confirmations) + Google Sheets (call logging)
>
> The interesting part wasn't getting the AI to *talk* — that's the easy part now. It was getting the automation layer to reliably check real-world availability, avoid double-bookings, and only tell the caller "confirmed" when it's actually true. That reliability piece is where the real engineering is.
>
> This is a portfolio build, but the architecture is the same one I'd bring to a real clinic, salon, or service business losing bookings to missed calls and manual scheduling.
>
> Video demo below 👇 — happy to walk through the n8n workflow or the Vapi setup for anyone curious.
>
> #AIAutomation #VoiceAI #n8n #Vapi #BusinessAutomation #AIagents #Automation #SystemDesign
