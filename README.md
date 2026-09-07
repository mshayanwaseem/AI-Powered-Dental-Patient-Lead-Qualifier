# AI-Powered Dental Patient Lead Qualifier

An n8n automation that reads incoming patient inquiries from a dental clinic intake form, uses an AI Agent to triage them by urgency, and routes each lead to the right response — automatically.

Built as a practical automation project to solve a real problem clinics face: **missed or delayed callbacks to patients who need urgent care.**

---

## The Problem

Dental clinics get patient inquiries through website forms, but front-desk staff can't monitor submissions in real time. A patient in genuine pain filling out a form at 9 PM might not get a callback until the next morning — by which point they've likely called a competitor. Meanwhile, low-urgency inquiries (routine cleanings, "just researching") don't need the same urgency but still need to be tracked.

This workflow solves that by triaging every submission the moment it comes in, the same way a trained front-desk receptionist would.

---

## How It Works

```
Typeform (Patient Intake Form)
        ↓
AI Agent (Gemini) — classifies lead as Hot / Warm / Cold
        ↓
Switch Node — routes based on classification
        ↓
   ┌────────────┬────────────┬────────────┐
  Hot          Warm         Cold
   ↓             ↓            ↓
Urgent Email   Standard     Low-priority
+ Log to      Email + Log   Email + Log
Sheet         to Sheet      to Sheet
```

### 1. Patient Intake (Typeform)
A patient fills out a short form covering: name, phone, service needed, pain level (1–10), new/returning patient status, insurance, and preferred contact time.

### 2. AI Classification (AI Agent + Gemini)
The AI Agent reads the submission and classifies it using fixed triage rules:

- **Hot** — pain scale ≥ 7, OR service type is "Tooth Pain / Emergency" (regardless of self-reported pain level, since patients often under-rate their own pain)
- **Warm** — real intent to book (checkup, cosmetic, braces, implants) without urgency
- **Cold** — low intent (e.g., "just researching / need a consultation") with no reported pain

The Agent returns structured JSON (`category` + `reason`) via a Structured Output Parser, so the result can be used programmatically downstream — not just as free text.

### 3. Routing (Switch Node)
Based on the `category` field, the submission is routed into one of three branches.

### 4. Automated Response (per branch)
Each branch sends a distinctly styled HTML email reflecting its urgency level:

| | Hot 🚨 | Warm 📋 | Cold 📥 |
|---|---|---|---|
| Purpose | Immediate callback alert | Same-day follow-up | Nurture list |
| Phone shown | Yes, prominent | Yes | No |
| Pain level shown | Yes | Yes | No |

### 5. Logging (Google Sheets)
Every lead — regardless of category — is logged to a shared Google Sheet with timestamp, patient details, AI classification, and the AI's reasoning, giving the clinic a full audit trail of every inquiry.

---

## Tech Stack

- **n8n** (workflow automation)
- **Typeform** (patient intake form)
- **Google Gemini** (AI Agent's language model)
- **Gmail** (automated email alerts)
- **Google Sheets** (lead logging/tracking)

---

## n8n Nodes Used

- Typeform Trigger
- AI Agent
- Structured Output Parser
- Switch
- Gmail (Send Email) × 3 branches
- Google Sheets (Append Row) × 3 branches

---

## Key Design Decisions

- **Safety-net rule for pain classification:** service type can override a low self-reported pain score. A patient describing a "Tooth Pain / Emergency" is always treated as hot, even if they rate their own pain lower than expected — because patients often minimize their own symptoms.
- **No AI memory:** each submission is scored independently. The Agent has no memory of prior conversations, so one patient's data can never influence another's classification.
- **Per-branch email design, not one generic template:** urgency is communicated visually (color, icon, which fields are shown) as well as in the copy — mirroring how a real clinic's triage process prioritizes information differently for an emergency vs. a routine inquiry.

---

## Status

This is a working prototype built as a learning project, tested end-to-end with real form submissions across all three classification branches. It is not a production system — for real clinic deployment, it would need additional error handling (e.g., AI Agent failures, malformed webhook payloads) and a more robust notification channel (SMS in addition to email).

---

## Author

Built by Shayan Waseem as part of a hands-on n8n automation learning track.
