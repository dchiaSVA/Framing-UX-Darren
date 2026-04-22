# CLAUDE.md — Steady Mode CBO Tool

## Project context

A mobile-first web tool for CBO frontline workers and promotoras serving
undocumented Hispanic immigrants in NYC during natural disasters. The worker
uses this tool while sitting with a person to diagnose barriers, deliver
consistent information, and match them to the right resource. Not
public-facing. The trusted human stays in the loop at every step.

One HTML file. No frameworks. Deploys to Netlify by dragging index.html
into the drop zone. Uses Gemini 2.0 Flash API for LLM capabilities.

---

## API setup

Model: gemini-2.0-flash
Endpoint: https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent

At the top of the <script> block define:
const GEMINI_API_KEY = "PASTE_KEY_HERE";

The system prompt (defined below under "System prompt") must be passed
as system_instruction on every API call. This is what constrains the
model's behavior. Never call the API without the system prompt attached.

API call function:

const callGemini = async (workerInput, sessionContext) => {
  const prompt = `Session so far: ${sessionContext}\n\nWorker input: "${workerInput}"`;
  const response = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${GEMINI_API_KEY}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        system_instruction: {
          parts: [{ text: SYSTEM_PROMPT }]
        },
        contents: [{
          parts: [{ text: prompt }]
        }],
        generationConfig: {
          temperature: 0.3,
          maxOutputTokens: 400
        }
      })
    }
  );
  const data = await response.json();
  const text = data.candidates[0].content.parts[0].text;
  const clean = text.replace(/```json|```/g, "").trim();
  return JSON.parse(clean);
};

SESSION CONTEXT: build a running plain-text string from items already
logged. Format: comma-separated subtypes.
Example: "data fear, scheduling, rent/housing, citizen children"
Pass this string with every call so the model knows what is already
captured and does not repeat categories.

LOADING STATE: show a small spinner or "Analyzing..." text in the
log area while the API call is in flight. Replace with result on
completion.

ERROR HANDLING: if the API call fails or JSON parse fails, fall back
to the keyword map (defined below). Show small note on the item:
"Auto-categorized — tap to review."

---

## System prompt

Store this as const SYSTEM_PROMPT at the top of the <script> block,
directly below GEMINI_API_KEY. This is passed as system_instruction
on every Gemini API call.

You are a decision-support assistant for CBO frontline workers and
promotoras working with undocumented Hispanic immigrants in New York City.

You never interact with immigrants directly. You support the worker only.

You will receive a worker input describing what a person said or what
the worker observed during a session. You must respond with a JSON
object only — no prose, no markdown, no explanation outside the JSON.

Response format — return exactly this structure:
{
  "category": "barrier" | "need" | "household" | "consideration",
  "subtype": string,
  "dot": "red" | "green" | "purple" | "gray",
  "tag": "visual" | "context",
  "label": string (short display label, max 8 words),
  "clarifying_question": string (one question, written to be said out loud),
  "panel_note": string (practical note for the worker, 1-2 sentences),
  "transparency_script": string or null (only generate if category is
    barrier AND subtype is data fear — otherwise always null),
  "eligibility_note": string or null (only generate if category is
    need — otherwise always null)
}

RULES YOU NEVER BREAK:

Never collect, infer, or store immigration status. If the worker input
mentions status, use it only to determine eligibility category — do
not include it in any output field.

Never recommend a specific resource. The UI handles resource matching.
Your job is categorization and language support only.

Never assume the barrier is cultural or linguistic unless the worker
has explicitly described it. Default to diagnosing scheduling and cost
barriers first.

Never produce a clarifying_question that contains more than one
question. One question only, written to be said out loud by the worker.
Short sentences. No jargon.

Never produce output outside the JSON structure. No preamble.
No explanation. No markdown. Raw JSON only.

Never give the same clarifying_question twice in a session. The session
context string tells you what has already been captured — use it to
avoid repetition.

Always write clarifying_question in plain conversational Spanish or
English depending on context — the worker will say it out loud.

Always write transparency_script in first person from the worker's
perspective. Short sentences. Must name what is and is not collected
for this specific situation. Model: "What you tell me stays here. We
don't record your name or immigration status. We don't share anything
with DHS or ICE. We only track what type of help people need — never
who they are."

Always write eligibility_note as a statement the worker can say out
loud to the person. Start with what the person qualifies for before
naming any limitations.

CATEGORY AND SUBTYPE RULES:

barrier — something preventing the person from accessing help
  subtypes: data fear · scheduling · cost · documentation gap ·
  transportation · language
  dot: red · tag: visual

need — a specific resource or service the person requires
  subtypes: rent/housing · food/supplies · cash assistance ·
  legal aid · healthcare · childcare
  dot: green · tag: visual

household — context about the person's household situation
  subtypes: citizen children · mixed-status family · single adult ·
  family with children · language preference · employment sector
  dot: purple · tag: visual

consideration — qualitative observation that does not fit the above
  dot: gray · tag: context
  clarifying_question: a practical follow-up the worker can note
  panel_note: why this observation matters for this session

EXAMPLE INPUT AND OUTPUT:

Input: "She's scared they'll share her info with ICE"
Output:
{
  "category": "barrier",
  "subtype": "data fear",
  "dot": "red",
  "tag": "visual",
  "label": "Scared information will be shared",
  "clarifying_question": "Has someone you know had a bad experience sharing information with an organization?",
  "panel_note": "Data fear is the most common barrier in this network. Lead with the transparency script before naming any resource.",
  "transparency_script": "What you tell me stays here. We don't record your name or immigration status. We don't share anything with DHS or ICE. We only track what type of help people need — never who they are.",
  "eligibility_note": null
}

Input: "She needs help with rent, landlord raised it after the storm"
Output:
{
  "category": "need",
  "subtype": "rent/housing",
  "dot": "green",
  "tag": "visual",
  "label": "Rent help after storm",
  "clarifying_question": "Has your landlord given you any written notice, or is it just verbal so far?",
  "panel_note": "Emergency rent assistance through mutual aid does not require immigration documentation.",
  "transparency_script": null,
  "eligibility_note": "Emergency rent assistance through local mutual aid does not require immigration documentation. You qualify based on demonstrated need."
}

---

## Build spec

ONE SCREEN. Steady mode only. Mobile-first responsive web.
Max-width 390px, centered on desktop.
Single HTML file — inline CSS in <style>, inline JS in <script>.

THREE TABS: Notes · Session (default active) · Output

---

## Tab 1 — Notes

HARDCODED INCOMING NOTES:

Note 1 (colleague — amber/warning background):
  From: Maria R. · 8:45am
  "She came in last month for food assistance. Very cautious
  about sharing info. Prefers Spanish. Has two young kids."

Note 2 (network alert — styled differently from colleague note):
  Network alert · updated today
  "BronxWorks rent assistance now accepting walk-ins Tue/Thu
  evenings. No docs required."

Worker notes textarea:
  Placeholder pre-filled: "Met briefly at community event last
  week. Said she's been struggling since the storm."

Footer: "Notes stay on this device only · not logged without confirmation"

---

## Tab 2 — Session (default active)

Section label: LOGGED SO FAR — tap any item

HARDCODED LOG ITEMS (load on start, in this order):
1. "Scared to share information"
   barrier · data fear · red dot · visual tag
2. "Works nights, can't do daytime"
   barrier · scheduling · red dot · visual tag
3. "Needs rent help after storm"
   need · rent/housing · green dot · visual tag
4. "Two kids, born in US"
   household · citizen children · purple dot · visual tag
5. "Seemed nervous when org was mentioned"
   consideration · gray dot · context tag

Each item displays:
- Color dot left edge
- Item text (14px)
- Category · subtype label below (10px, muted)
- Visual tag (blue pill) or context tag (gray pill)
- Right arrow › on right edge
- Tappable — opens side panel

NEW ITEMS FROM API:
Worker submits free text input →
  1. Disable input, show "Analyzing..." in log area
  2. Call callGemini(input, sessionContext)
  3. Parse JSON response
  4. Append new log item using response.label, response.dot,
     response.category, response.subtype, response.tag
  5. Store full response object keyed to item id for panel use
  6. Update sessionContext string with new subtype
  7. Update Output tab profile in background
  8. Re-enable input, show toast "Added to log"

QUICK-ADD BUTTONS (also call the API):
"Has WhatsApp" · "Spanish" · "Eviction urgency"
Each triggers callGemini with that phrase as input.

Footer: "No names or status stored · session builds barrier profile only"

---

## Side panel

Slides in from right. Width 86% of phone container.
CSS transition: right -100% to right 0 · duration 0.25s ease.
Semi-transparent overlay behind panel.
Tapping overlay closes panel.
Header: ✕ close · item title · category badge (colored to match dot)

HARDCODED PANEL CONTENT for the five default items:

PANEL 1 — DATA FEAR:
Bar chart (hardcoded percentages):
  Data fear — 74%
  Scheduling — 58%
  Cost — 41%
  Documentation — 29%
Source line: Network sessions this week · CMS 2022
Clarifying question (warning box):
  "Has someone you know had a bad experience sharing
  information with an organization?"
Textarea: "Note her response..."
Confirm + add to log button
Source: CMS 2022 · 805 UndocuFund model

PANEL 2 — SCHEDULING:
Resource availability list:
  BronxWorks Tue/Thu eve — Available (green)
  Make the Road evenings — Available (green)
  NYIC Queens evening — Low capacity (warning)
  Catholic Charities — Daytime only (danger)
Source: Curated network · verified today
Clarifying question:
  "Is there a specific evening or weekend time that works
  best — weekdays or weekends?"
Textarea · Confirm button

PANEL 3 — RENT:
Availability list:
  Mutual aid fund (no docs) — Available (green)
  BronxWorks emergency — Available (green)
  FEMA direct (requires status) — Ineligible (danger)
  FEMA via citizen child — Eligible (green)
Source: NILC · verified Apr 2025 · Morey 2012
Eligibility box (success color):
  "Emergency rent assistance through local mutual aid does
  not require immigration documentation. You qualify based
  on demonstrated need."
Clarifying question:
  "Has your landlord given you any written notice, or is
  it just verbal so far?"
Textarea · Confirm button

PANEL 4 — CITIZEN CHILDREN:
Pathway list:
  FEMA application (for kids) — Opens pathway (green)
  Medicaid for citizen children — Eligible (green)
  School enrollment support — Available (green)
  Cash assistance (parent) — Status required (danger)
Source: Morey 2012 · NILC · MOIA 2024
Clarifying question:
  "Do the kids have their birth certificates or social
  security cards accessible?"
Textarea · Confirm button

PANEL 5 — CONSIDERATION:
Gray note box:
  "Qualitative observation — logged as session context.
  Worth remembering when deciding which org to recommend."
Warning box:
  "Consider leading with the transparency script before
  naming any org. Let her ask which org rather than you
  naming one first."
Textarea: "Which org? What did she say exactly?"
Add to log button

API-GENERATED PANEL CONTENT for new items:
Use fields from stored API response for that item:
- panel_note → gray note box
- clarifying_question → warning box with label CLARIFYING QUESTION
- transparency_script → success box with label SAY OUT LOUD
  (only shows if not null)
- eligibility_note → success box with label ELIGIBILITY
  (only shows if not null)
Textarea to note response · Confirm button

Confirm → text changes to "Added ✓" → 600ms pause → panel closes

---

## Tab 3 — Output

BARRIER PROFILE (2x2 chip grid, updates from session state):
  Primary barrier chip
  Secondary barrier chip
  Need type chip
  Language chip

Default state:
  Primary: Data fear
  Secondary: Scheduling
  Need: Rent / housing
  Language: Spanish

Update rule: as new items are added, update the relevant chip.
First barrier → primary. Second barrier → secondary.
First need → need chip. First language item → language chip.

MATCH CONFIDENCE BAR:
Starts at 84%
Each new item added: increment by random 3–5 points
Cap at 97%
Width animates with CSS transition 0.5s ease

SUGGESTED NEXT QUESTION (warning box):
Cycles through four states as items are added:
  State 1 (default):
    "Is there an evening or weekend time that works better?"
  State 2:
    "Has someone you know had a bad experience with an org?"
  State 3:
    "Would it help if I explained what we keep and what we don't?"
  State 4:
    "Is there a time of day that works better — evenings or weekends?"

Divider

TRANSPARENCY SCRIPT:
Label: TRANSPARENCY SCRIPT — SAY OUT LOUD
Sub-label: FOR DATA FEAR + RENT ASSISTANCE
Secondary background box, italicized text:
  "What you tell me stays here. We don't record your name
  or immigration status. We don't share anything with DHS
  or ICE. We only track what type of help people need —
  never who they are."
Source: CMS 2022 · 805 UndocuFund model

If any API response returned a transparency_script field:
append it below with divider and label:
GENERATED FOR THIS SESSION · [subtype]

Divider

MATCHED RESOURCES (hardcoded — three items):

1. Make the Road NY — Brooklyn
   Available (green badge)
   Evening hours · strict no-share policy · rent · Spanish
   Curated network · verified this month · (718) 418-7690

2. BronxWorks — Tue/Thu evenings
   Available (green badge)
   Walk-in evenings · no docs required · rent + family services
   Network alert today · (718) 508-3000

3. NYIC Queens clinic
   Low capacity (warning badge)
   Evening screening · no status required · legal + housing
   Curated network · verified Feb 2025 · walk-in

Divider

DELIVER ROW:
  Forward via WhatsApp (success/green button):
    Copy to clipboard:
    "Ayuda disponible ahora: renta/vivienda. Sin documentos
    requeridos. Make the Road NY — (718) 418-7690. Evening
    hours available. Comparte."
    Toast: "Message copied — paste into WhatsApp"
    Clipboard API with fallback: if unavailable show selectable
    text box with the message

  Print (secondary button):
    window.print()

LOG ROW:
  Confirm + log session (info/blue button):
    Toast: "Session logged — no personal data recorded"
    Button text → "Logged ✓"

  Discard (secondary button):
    Toast: "Session discarded"

Footer: "No personal data in log · sources shown for each resource"

---

## Keyword map — fallback only when API call fails

fear/scared/share/trust/ICE/DHS
  → barrier · data fear · red · visual

schedule/night/day/time/hours/morning/evening/daytime
  → barrier · scheduling · red · visual

cost/money/pay/afford/insurance/fee
  → barrier · cost · red · visual

rent/housing/eviction/landlord/apartment
  → need · rent/housing · green · visual

food/supplies/pantry/eat/hungry
  → need · food/supplies · green · visual

legal/lawyer/court/immigration/attorney
  → need · legal aid · green · visual

health/doctor/hospital/sick/medical/clinic
  → need · healthcare · green · visual

kids/children/born/citizen/family/daughter/son
  → household · citizen children · purple · visual

spanish/english/language/dialect/creole/bengali
  → household · language · purple · visual

anything else
  → consideration · gray · context

---

## Colors — CSS custom properties only, no hardcoded hex

--color-background-primary
--color-background-secondary
--color-background-info
--color-background-warning
--color-background-success
--color-background-danger
--color-text-primary
--color-text-secondary
--color-text-tertiary
--color-text-info
--color-text-warning
--color-text-success
--color-text-danger
--color-border-tertiary
--color-border-secondary

---

## Typography

font-family: system-ui, sans-serif
14px body · 12px secondary · 11px tags and badges · 10px labels
font-weight: 400 regular · 500 medium only

---

## Do not

- No React, Vue, Tailwind, or any framework
- No external fonts or CDN dependencies
- No hardcoded hex colors anywhere
- No actual data persistence — log and confirm are visual only
- No gradients, drop shadows, or decorative effects
- No backend or serverless functions
- Do not put the actual API key in CLAUDE.md — key goes in
  index.html only, never committed to a repo

---

## File structure

index.html — everything in one file
Structure:
  <style> block — all CSS
  <body> — all HTML
  <script> block — in this order:
    const GEMINI_API_KEY = "AIzaSyCg2-0m_RTyDxcWUEbOgF3uHDFzd6PfwUo";
    const SYSTEM_PROMPT = `...exact text from system prompt above...`;
    const callGemini = async (...) => { ... };
    all other JS

Deploys by dragging index.html into Netlify drop zone.
No config file, no build step, no git required.