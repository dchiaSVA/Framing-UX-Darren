# CLAUDE.md — Steady Mode CBO Tool

## Project context

A mobile-first web tool for CBO frontline workers and promotoras serving
undocumented Hispanic immigrants in NYC during natural disasters. The worker
uses this tool while sitting with a person to diagnose barriers, deliver
consistent information, and match them to the right resource. Not
public-facing. The trusted human stays in the loop at every step.

One HTML file. No frameworks. Deploys to Netlify as a static site.

---

## System prompt

You are a decision-support assistant for CBO frontline workers and promotoras
working with undocumented Hispanic immigrants in New York City.

You never interact with immigrants directly. You support the worker only.

Your job is three things:
1. Diagnose the actual barrier this specific person is facing — not the
   population-level assumption.
2. Deliver consistent, verified, current information about eligibility
   and data-sharing policies.
3. Match the person's specific barrier profile to the right resource.

RULES:

Never collect, infer, or store immigration status.

Never recommend a resource without a source and verification date.

Never assume the barrier is cultural or linguistic unless the worker
has described it. Default to asking about scheduling and cost first.

Never give answers the same length regardless of complexity. Calibrate.

Never treat superlatives as decoration. "Most effective" requires evidence.

Never give system-level recommendations. Stay at the human level.
The worker is sitting across from a person right now. Answer at that scale.

Always ask one clarifying question at a time. Never a list.

Always show source: [Source · verified date].

Always flag when a resource has unknown availability.

BARRIER TAXONOMY:

- Data fear — afraid the org will share their information
- Scheduling — cannot access services during standard hours
- Cost — cannot afford fees, insurance, or indirect costs
- Documentation gap — lacks required documents for a specific program
- Transportation — cannot reach the service location
- Language — needs a specific language or dialect not available
- Consideration — qualitative observation that does not fit a category

MAPPABLE vs CONSIDERATION:

If input maps to the barrier taxonomy, a known need type (rent, food,
cash, legal, health, childcare), or household context (citizen children,
mixed-status family, single adult, family with children) — it is MAPPABLE.
Generate a visual and a clarifying question.

If input is qualitative and person-specific — it is a CONSIDERATION.
Log as context. No visual. Generate a practical note for the worker instead.

TRANSPARENCY SCRIPT RULES:

Scripts must be program-specific, not generic.
Written to be said out loud. Short sentences. No jargon.
Must name what is and is not collected for this specific program.
Model: "What you tell me stays here. We don't record your name or
immigration status. We don't share anything with DHS or ICE. We only
track what type of help people need — never who they are."
Source: CMS 2022 · 805 UndocuFund model.

RESOURCE MATCHING RULES:

Match to barrier profile, not need type alone.
Rank curated network resources above live lookup results.
Show availability status for every resource.
Show source and verification date for every resource.
Flag when citizen children open additional eligibility pathways.

---

## Build spec

ONE SCREEN. Steady mode only. Mobile-first responsive web.
Max-width 390px, centered on desktop.
Single HTML file — inline CSS in <style>, inline JS in <script>.

THREE TABS: Notes · Session (default active) · Output

---

## Tab 1 — Notes

HARDCODED INCOMING NOTES:

Note 1 (colleague):
  From: Maria R. · 8:45am
  "She came in last month for food assistance. Very cautious
  about sharing info. Prefers Spanish. Has two young kids."

Note 2 (network alert — styled differently from colleague notes):
  Network alert · updated today
  "BronxWorks rent assistance now accepting walk-ins Tue/Thu
  evenings. No docs required."

Worker notes textarea:
  Placeholder pre-filled: "Met briefly at community event last
  week. Said she's been struggling since the storm."

Footer: "Notes stay on this device only · not logged without confirmation"

---

## Tab 2 — Session (default active tab)

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

Each item shows:
- Color dot
- Item text
- Category label below in smaller text
- Visual tag (blue) or context tag (gray)
- Right arrow › on the right edge
- Tappable — opens side panel

FREE INPUT:
Text input + Add button
Keyword map auto-categorizes on add (see keyword map below)
Toast shows "Added to log" on add

QUICK-ADD BUTTONS:
"Has WhatsApp" → "Has WhatsApp, checks regularly"
  household · purple dot · visual
"Spanish" → "Prefers Spanish"
  household · purple dot · visual
"Eviction urgency" → "Worried about eviction timeline"
  barrier · scheduling · red dot · visual

Footer: "No names or status stored · session builds barrier profile only"

---

## Side panel

Slides in from right. Width 86% of phone container.
Overlay behind panel — tapping overlay closes panel.
Header: close ✕ button · item title · category badge
Confirm button → text changes to "Added ✓" → 600ms pause → panel closes.

PANEL CONTENT — BARRIER: DATA FEAR
Visual: bar chart
  Data fear — 74%
  Scheduling — 58%
  Cost — 41%
  Documentation — 29%
Source line: Network sessions this week · CMS 2022
Clarifying question box (warning color):
  Label: CLARIFYING QUESTION
  "Has someone you know had a bad experience sharing
  information with an organization?"
Textarea to note response
Confirm + add to log button
Source below: CMS 2022 · 805 UndocuFund model

PANEL CONTENT — BARRIER: SCHEDULING
Visual: resource availability list
  BronxWorks Tue/Thu eve — Available
  Make the Road evenings — Available
  NYIC Queens evening — Low capacity
  Catholic Charities — Daytime only
Source line: Curated network · verified today
Clarifying question box:
  "Is there a specific evening or weekend time that works
  best — weekdays or weekends?"
Textarea to note response
Confirm button

PANEL CONTENT — NEED: RENT
Visual: availability list
  Mutual aid fund (no docs) — Available
  BronxWorks emergency — Available
  FEMA direct (requires status) — Ineligible
  FEMA via citizen child — Eligible
Source line: NILC · verified Apr 2025 · Morey 2012
Eligibility box (success color):
  Label: ELIGIBILITY — SAY OUT LOUD
  "Emergency rent assistance through local mutual aid does
  not require immigration documentation. You qualify based
  on demonstrated need."
Clarifying question box:
  "Has your landlord given you any written notice, or is
  it just verbal so far?"
Textarea to note response
Confirm button

PANEL CONTENT — HOUSEHOLD: CITIZEN CHILDREN
Visual: eligibility pathway list
  FEMA application (for kids) — Opens pathway
  Medicaid for citizen children — Eligible
  School enrollment support — Available
  Cash assistance (parent) — Status required
Source line: Morey 2012 · NILC · MOIA 2024
Clarifying question box:
  "Do the kids have their birth certificates or social
  security cards accessible?"
Textarea to note response
Confirm button

PANEL CONTENT — CONSIDERATION
No visual.
Gray note box:
  "This is a qualitative observation — logged as session
  context. It doesn't map to a resource or barrier category
  but is worth remembering when deciding which org to
  recommend first."
Practical note box (warning color):
  "Consider leading with the transparency script before
  naming any org. Let her ask which org rather than you
  naming one first."
Textarea: "Which org? What did she say exactly?"
Add to log button

---

## Tab 3 — Output

BARRIER PROFILE (2x2 chip grid):
  Primary: Data fear
  Secondary: Scheduling
  Need: Rent / housing
  Language: Spanish

Match confidence bar: starts at 84%
Updates: each new item added increments by 3–5 points

Suggested next question (warning box) — cycles through these
four states as items are added:
  State 1 (default, 84%):
    "Is there an evening or weekend time that works better for you?"
  State 2 (88%):
    "Has someone you know had a bad experience with an organization?"
  State 3 (91%):
    "Would it help if I explained exactly what we keep and what we don't?"
  State 4 (93%):
    "Is there a time of day that works better — evenings or weekends?"

Divider

TRANSPARENCY SCRIPT:
Label: TRANSPARENCY SCRIPT — SAY OUT LOUD
Sub-label: FOR DATA FEAR + RENT ASSISTANCE
Italicized text:
  "What you tell me stays here. We don't record your name
  or immigration status. We don't share anything with DHS
  or ICE. We only track what type of help people need —
  never who they are."
Source below: CMS 2022 · 805 UndocuFund model

Divider

MATCHED RESOURCES (three items):

Resource 1:
  Name: Make the Road NY — Brooklyn
  Availability badge: Available (green)
  Why: Evening hours · strict no-share policy · rent assistance · Spanish
  Source: Curated network · verified this month · (718) 418-7690

Resource 2:
  Name: BronxWorks — Tue/Thu evenings
  Availability badge: Available (green)
  Why: Walk-in evenings · no docs required · rent + family services
  Source: Network alert today · (718) 508-3000

Resource 3:
  Name: NYIC Queens clinic
  Availability badge: Low capacity (warning)
  Why: Evening screening · no status required · legal + housing
  Source: Curated network · verified Feb 2025 · walk-in

Divider

DELIVER ROW:
  Forward via WhatsApp (green button) →
    Copies to clipboard:
    "Ayuda disponible ahora: renta/vivienda. Sin documentos
    requeridos. Make the Road NY — (718) 418-7690. Evening
    hours available. Comparte."
    Shows toast: "Message copied — paste into WhatsApp"

  Print (secondary button) →
    Calls window.print()

LOG ROW:
  Confirm + log session (info/blue button) →
    Shows toast: "Session logged — no personal data recorded"
    Button text changes to "Logged ✓"

  Discard (secondary button) →
    Shows toast: "Session discarded"

Footer: "No personal data in log · sources shown for each resource"

---

## Keyword map — auto-categorization

fear/scared/share/trust/ICE/DHS
  → barrier · data fear · red dot · visual

schedule/night/day/time/hours/morning/evening/daytime
  → barrier · scheduling · red dot · visual

cost/money/pay/afford/insurance/fee
  → barrier · cost · red dot · visual

rent/housing/eviction/landlord/apartment
  → need · rent/housing · green dot · visual

food/supplies/pantry/eat/hungry
  → need · food/supplies · green dot · visual

legal/lawyer/court/immigration/attorney
  → need · legal aid · green dot · visual

health/doctor/hospital/sick/medical/clinic
  → need · healthcare · green dot · visual

kids/children/born/citizen/family/daughter/son
  → household · citizen children · purple dot · visual

spanish/english/language/dialect/creole/bengali
  → household · language · purple dot · visual

anything else
  → consideration · gray dot · context

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
14px body
12px secondary text
11px tags and badges
10px labels and footers
font-weight: 400 regular · 500 medium only

---

## Do not

- No React, Vue, Tailwind, or any framework
- No external fonts or CDN dependencies
- No backend or API calls
- No actual data persistence — confirm and log is visual only
- No hardcoded hex colors anywhere
- No gradients, drop shadows, or decorative effects
- No color: #333 or similar — invisible in dark mode

---

## File structure

index.html — everything in one file
<style> block at top
HTML body
<script> block at bottom

When done the file deploys by dragging index.html into
the Netlify drop zone with no additional config.    