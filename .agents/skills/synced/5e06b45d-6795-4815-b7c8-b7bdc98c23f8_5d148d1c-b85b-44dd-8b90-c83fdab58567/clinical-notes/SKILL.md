---
name: clinical-notes
description: >
  Generate clinician-facing clinical notes (ED/consult/H&P, daily progress SOAP,
  OB triage/prenatal, psychiatry consult with MSE) that are concise, EHR copy-paste
  ready, and faithful to provided facts. Use when asked to write, rewrite, or template
  a clinical note or note section from a prompt, including specialty add-ons (OB/GYN,
  Psychiatry) and when the user provides partial information that must be carried
  forward without invention. Trigger on any request to write, draft, generate, rewrite,
  or template a clinical note, SOAP note, H&P, progress note, consult note, OB triage
  note, prenatal visit note, or psychiatry note — even if the user only provides
  a rough clinical summary or partial data.
interface:
  display_name: Clinical Notes
  short_description: >
    Generate clinician-facing clinical notes (ED/consult/H&P, daily progress SOAP,
    OB triage/prenatal, psychiatry consult with MSE) that are concise, EHR copy-paste
    ready, and faithful to provided facts. Use when asked to write, rewrite, or
    template a clinical note or note section from a prompt, including specialty
    add-ons (OB/GYN, Psychiatry) and when the user provides partial information
    that must be carried forward without invention.
  icon_small: assets/icon.svg
  icon_large: assets/icon.svg
policy:
  products:
    - chatgpt
    - codex
    - api
    - atlas
  allow_implicit_invocation: true
---

# Clinical Notes Skill

## Operating Principles (Non-Negotiable)

- Output ONLY the clinical note text, directly copy/paste ready for an EHR.
- No greetings, no preamble, no postamble, no meta-commentary, no citations, no evidence sections, no tool artifacts.
- NOT AN EMAIL. Never add email metadata, subject fields, or variant="email" unless the requested artifact is explicitly an email.
- Factual fidelity: every unmarked clinical fact in the note must come from the user's prompt.
- Do NOT add undocumented vitals, exam findings, labs, imaging, meds, diagnoses, histories, dates/times, consults, or pertinent negatives.
- If something is unknown, write `[Add details here]`. Do NOT add frequencies, workup steps, or management not supplied by the user.
- If the user explicitly requests a template, use short bracket placeholders like `[not documented]` or `[details]`. Avoid long blanks or underscores.
- If example wording is requested, clearly label it as `Example:` so it cannot be mistaken for documented fact.
- Prefer problem-focused reasoning and actionable plans using provided data; avoid being so sparse the note is unusable.
- Place information in the correct section (HPI vs PMH vs Meds, etc.) and maintain Subjective vs Objective separation.
- Format hygiene: no emojis, no horizontal rules or linebreak art (e.g. "---") — these paste poorly into EHRs.
- Avoid em dashes and parentheses in clinical notes and RN messages.

---

## Safety and Clinical Consistency Check

Before drafting, assess whether any provided diagnosis, treatment, or plan appears:
- Unsafe or nonstandard
- Internally inconsistent
- Dependent on missing critical context
- Concerning for medication dose, route, frequency, duration, duplicate therapy, or contraindication

If a concern is identified:
- Do NOT silently reproduce the questionable content as correct.
- State the concern explicitly.
- Draft only non-concerning portions when possible.
- If the unsafe/questionable content is central to the document, do NOT draft — return only the concern(s).
- Ask for clarification only when needed for safety.

---

## Default Length Targets

- ED/Consult/H&P: ~250-450 words (~1 page or less)
- Daily progress (SOAP): ~120-250 words
- Specialty add-ons (OB/MSE): include required elements but keep each field 1-2 short lines unless abnormal

---

## Global Formatting Rules

### 1) Header (top of every note)

Include these lines:

```
Writer ID + service + note type (single line)
Example: CT (MSI3) | Internal Medicine | Consult Note

Date/time (spell month; 24-hour time)
Example: Dec 11, 2025 | 16:00

Patient identifiers (facility-appropriate)
Example: Patient: J.D. | MRN: #### | DOB: [not documented] | Location: ED/3W
```

Optional operational fields (include when relevant/available; do not guess):
- Consult requestor/service + question
- Date/time of consult request
- Attending of record
- Code status, isolation
- Lines/tubes/drains/airway (esp. ICU)
- Current diet, VTE prophylaxis, disposition status

### 2) Language and Clarity

- Avoid uncommon or made-up abbreviations; when unsure, write it out.
- Prefer objective data and (when helpful) short, quoted patient statements.
- Use standard, clinician-familiar shorthand (NAD, RRR, etc.) only if it improves readability.

### 3) Template Hygiene

When the user asks for a template:
- Avoid long underscore lines that require deletion.
- Prefer short placeholders: `[ ]`, `[not documented]`, `[details]`.
- Avoid checkboxes unless explicitly asked; never include a checkbox when there is only one option.

### 4) Choose the Right Structure

Determine note type from the user's wording:

| User says... | Use this structure |
|---|---|
| "consult", "initial assessment", "H&P", "admission", "ED provider note" | Consultation / Initial Assessment Note |
| "daily progress", "SOAP", "rounding note" | Daily Progress (SOAP) |
| "psychiatry consult", "MSE" | Psychiatry Consult/Progress Note |
| "OB triage", "labor check", "prenatal visit" | OB/GYN Add-ons |

If the user requests a note type explicitly, follow it exactly.
If unclear, default to Consultation / Initial Assessment (more complete sections) without announcing the choice.

### 5) Closing Line

End with a signature block: `Signed: <Name>, <Role/Level>, <Pager/Contact>`

If the writer is a trainee and a supervisor is provided/expected, append: `, d/w <Supervisor Name (Role/Year)>`

---

## Note Type: Consultation / Initial Assessment

Use these sections in this order:

**0) Consult Context** (include if consult)
- Consult service + requestor + question
- Consult requested date/time
- If missing: Not documented

**1) One-Liner / Reason for Consult** (1 sentence)
- Age + sex/gender + key PMH + why here + key timing
- Example: 54M with HTN/T2DM presenting with 1 hour of exertional chest pressure, improved with rest.

**2) HPI + Relevant ROS**
- Symptom structure: onset, quality, severity, associated symptoms, triggers/relievers
- Include only negatives explicitly provided
- If ROS not provided: `ROS: Not documented.`

**3) PMH / PSH**
- Pertinent conditions; dates/complications for major surgeries when known

**4) Medications**
- Name, dose, route, frequency
- If unknown: `Not documented.`
- Document adherence only if provided; do not speculate

**5) Allergies**
- Allergen + reaction (true allergy vs intolerance if known)

**6) Social History**
- Living situation/supports if relevant
- Tobacco/alcohol/other substances with quantities/timeframe
- Diet/exercise only if relevant and provided

**7) Family History**
- Relevant heritable/condition-related history

**8) Physical Exam**
- Vitals + focused exam
- Include only what is provided; do not add a default "normal" exam
- If not provided: `Vitals: Not documented / Exam: Not documented.`

**9) Labs/Imaging/Studies**
- Summarize key results with dates/times if available
- Do not invent imaging or "normal" results
- If not provided: `Studies: Not documented.`
- If trends/comparisons are provided, include them; do not fabricate prior values

**10) Assessment**
- 1-3 lines: most likely diagnosis + brief why
- Include a status/trajectory phrase when appropriate (e.g., "clinically stable", "improving", "worsening")
- Include key uncertainties

**11) Differential** (only if needed)
- 3-5 items, ordered most to least likely
- Each with 1 supporting/refuting clue

**12) Plan**
- Numbered, actionable steps
- Include disposition (admit vs discharge), monitoring, tests, treatments, consults, follow-up
- Use sub-bullets for readability in long plans
- If multiple independent active issues, organize as a numbered problem list

---

## Note Type: Daily Progress (SOAP)

Keep each section tight (often 2-5 bullets).

- **ID statement** (1 sentence): who + hospital day + diagnosis/status
- **S:** Patient symptoms/function (pain, appetite, sleep, ambulation) + pertinent negatives
- **O:** Vitals + focused exam + key labs/imaging since last note
- **A:** Clinical interpretation and trajectory
- **P:** Numbered plan by problem; include discharge planning if relevant

---

## Note Type: OB/GYN Add-ons

### A) OB Triage/Labor Concern (must-have elements)

If pregnant with triage/labor concern, include:
- GA and basis (LMP vs dating ultrasound) if known
- Fetal status when relevant (movement; FHR/monitoring if available)
- Urgent triage screen (4 questions):
  1. Contractions (start, frequency, intensity 1-10, progression)
  2. Membranes (leak/gush, when, color/odor)
  3. Bleeding (amount, color, clots)
  4. Fetal movement (same/decreased; since when)

### B) Prenatal Routine Visit Rules

- Patient-reported symptoms and counseling go under Subjective
- Vitals, fetal heart rate, fundal height, labs/imaging, blood type/Rh/antibody screen go under Objective
- Assessment: include pregnancy status + stability (e.g., "Singleton IUP at 28w, maternal/fetal status stable today")
- Plan: separate categories, use bullets/sub-bullets

---

## Note Type: Psychiatry Consult/Progress Note

Replace physical exam with MSE; include safety assessment.

**1) Identifying Context**
- Presenting concern, setting (ED/inpatient), and medical clearance status if provided

**2) MSE (IAMSEPTIC)**

| Letter | Element |
|---|---|
| I | Identifying info (apparent age; relevant descriptors) |
| A | Appearance (hygiene, attire) |
| M | Mannerisms/motor |
| S | Speech |
| E | Mood/affect |
| P | Perceptions |
| T | Thought process/content (include SI/HI with intent/plan if present) |
| I | Insight/judgment/reliability |
| C | Cognition |

**3) Assessment** (include safety)
- Working diagnoses + suicide/violence risk summary if relevant

**4) Plan (Biopsychosocial)**
- Bio: meds/somatic treatments
- Psycho: therapy/psychoeducation
- Social: stressors, housing/safety, supports, substance treatment, SW needs

---

## Built-in Quality Checklist (Self-Check Before Finalizing)

- [ ] Header present (writer/service/type + date/time 24h + identifiers)
- [ ] One-liner includes age + sex/gender
- [ ] Clear Subjective vs Objective separation
- [ ] No undocumented facts; unknowns clearly labeled Not documented
- [ ] Assessment includes status/trajectory (stable/improving/worsening) when appropriate
- [ ] Plan is numbered, actionable, easy to scan
- [ ] Specialty requirements included when relevant (OB triage screen; IAMSEPTIC MSE)
- [ ] Within length target; removed anything that does not change decisions
- [ ] No em dashes, no parentheses, no horizontal rules, no emojis

---

## Response Evaluation Criteria

Use a 1-7 scale (7 = excellent) for each criterion:

1. **EHR Usability and Concision**: Copy/paste ready; minimal filler; scannable bullets; no extraneous checkboxes; short placeholders; no emojis; no horizontal rules
2. **Structure and Clinical Organization**: Correct sectioning for note type; clear S vs O separation; specialty add-ons placed correctly
3. **Factual Fidelity and Uncertainty Handling**: Only user-provided facts appear unmarked; missing data labeled "Not documented"; no unsolicited medical advice added
4. **Clinical Reasoning Quality**: Assessment matches provided data; includes trajectory; differential (if present) is prioritized with brief clues
5. **Plan Actionability**: Concrete next steps, monitoring, disposition, follow-up/contingencies; organized by problem/category
