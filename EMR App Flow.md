This is a well-structured core workflow. Let me flush it out into a proper functional specification, adding the detail, edge cases, and data modeling that a developer would need to actually build this.

---

# **EMR App — Functional Specification (Crude V1)**

## **System Overview**

*Current build note: the live prototype is a single-page, front-end-only React app with in-memory state — there is no backend, authentication, or persistent storage yet. Anything described below as "system-generated," "saved," or "auto-saved" currently means "held in browser memory for the session" rather than written to a database. Treat this document as the target functional spec; where a gap between it and the current build is the result of a deliberate product decision rather than simply "not built yet" (e.g. Patient ID generation, below), that's called out inline.*

A linear, queue-driven clinical workflow app supporting three sequential care teams: **Triage → Provider (Care) → Pharmacy**. Each team operates in a defined lane with handoff points between queues. Patient state is always owned by exactly one team at a time.

---

## **Detailed Workflow**

---

### **Stage 1 — Triage**

**Step 1: Patient Search**

Triage team lands on a search screen as the default home view.

Search fields:

* Full name (first, last, or both)  
* Phone number  
* Location (clinic site)  
* Patient unique identifier (system ID or external ID if applicable)

Search returns a results list showing: name, DOB, phone, last visit date. If no results, surface a clear "Patient not found — create new patient" CTA.

---

**Step 2a: Existing Patient — Open Chart and Input Vitals**

On selecting an existing patient, the system opens a **new visit record** automatically (does not overwrite prior visits).

Triage inputs current visit vitals:

* Systolic BP  
* Diastolic BP  
* Heart Rate (bpm)  
* Respiratory Rate (breaths/min)  
* Temperature (with unit toggle °F/°C)  
* Weight (with unit toggle lbs/kg)  
* Height (with unit toggle in/cm)  
* SpO2 (%)  
* Allergies & Intolerances — pre-populated from prior visits, editable and confirmable ("Confirmed unchanged" checkbox to reduce re-entry burden)  
* If a photo exists from a prior visit, it is displayed with two options: **"Keep Existing Photo"** or **"Take New Photo"**  
* If no prior photo exists, photo capture is required before check-in can proceed  
* Photo is taken via device camera (mobile) or uploaded from file (desktop)  
* Basic guidance displayed on screen: *"Center patient's face, neutral expression, adequate lighting"*  
* Triage can retake before confirming

All vitals fields should have basic range validation with soft warnings (not hard blocks) — e.g. SpO2 below 90% triggers a yellow flag, not a form error.

---

**Step 2b: New Patient — Create Record \+ Vitals**

If patient not found, triage opens a new patient creation form.

Fields:

* First name, last name (required)  
* Extension name — an optional second last name used as an additional identifier  
* Date of birth, gender (required; dropdown: Male / Female / Non-binary / Prefer not to say / Other)  
* Phone number — optional; not required to register or check in  
* Email (optional, used for automated report delivery post-visit)  
* Mother's name (optional)  
* Location (clinic location or patient address depending on context) (required)  
* Photo taken (Photo capture is **mandatory** for new patient creation — no check-in can be completed without it. This becomes the patient's permanent photo of record, updateable at future visits.) Before capturing the photo, the volunteer reads a bilingual (English/Spanish) consent disclosure aloud to the patient, explaining the photo is used only to confirm identity at a future visit.

Then immediately flows into vitals entry as in Step 2a.

**Duplicate detection**: Before saving, system checks for existing records with same name \+ DOB or same phone number and surfaces a warning — *"A patient with this name and date of birth already exists. Did you mean \[name\]?"* — to reduce duplicate record creation.

**Patient ID generation**: On record creation, the system generates a patient ID in the format `PT-XXXX-XXXX` (e.g. `PT-7K4M-92QX`) using a cryptographically secure random generator (`crypto.getRandomValues`) over a 32-character alphabet that excludes visually ambiguous characters (`0`, `O`, `1`, `I`). The generator checks the new ID against all patient IDs currently in memory and regenerates on collision. *This is an in-memory uniqueness check only — the current build has no database. Once persistent storage exists, patient ID must carry a real database-level UNIQUE constraint; this generator is designed to make that constraint trivially satisfiable (collisions should be effectively unreachable at any realistic patient volume), not to replace it.*

---

**Step 2c: Triage Classification, History & Maternity (All Patients)**

Applies to every check-in, new or returning, immediately after vitals.

* **BMI** — calculated automatically once weight and height are both entered; recalculates live if either value or its unit changes. Displayed with a category: Underweight / Normal / Overweight / Obese.  
* **Triage Acuity Level** — a required, single-select assignment of 1 (Resuscitation) through 5 (Non-Urgent). Each level is color-coded (1 red → 5 green) and carries a hover tooltip describing its clinical condition and examples:  
  * **1 – Resuscitation** (red): Immediate, life-threatening problem. E.g. cardiac arrest, major trauma, severe respiratory distress, active seizures.  
  * **2 – Emergent** (orange): High-risk, confused, or in severe pain/distress. E.g. heart attack, stroke, suspected head injury, severe asthma.  
  * **3 – Urgent** (amber): Stable vitals but needs multiple resources and quick attention. E.g. moderate pain, deep cuts, stable abdominal pain.  
  * **4 – Less Urgent** (lime): Stable, needs one simple resource. E.g. simple sprain, simple UTI, small closed fracture.  
  * **5 – Non-Urgent** (green): Most stable, needs little or no resources. E.g. cold symptoms, sore throat, minor rash.  
  * The assigned level is required to check in and appears as a colored badge on the care queue card — visible without expanding it. It does not currently reorder the queue (still FIFO by check-in time).  
* **Patient History** —  
  * Free-text past medical history (per visit — not carried forward automatically)  
  * Six chronic-condition checkboxes: Hypertension, Diabetes, Arrhythmia, Heart Attack, High Cholesterol, Surgery. These write to the **patient record**, not the visit — once checked, they stay checked on every future visit until unchecked.  
  * Free-text current medications ("What medications are they on")  
  * Free-text family/social history  
* **Maternity** (shown only when the patient's recorded gender is Female) —  
  * "Is patient currently pregnant?" checkbox  
  * Date of Last Period (date field)

---

**Step 3: Check In and Send to Care Queue**

Triage reviews entered information, then clicks **"Check In and Send to Care Queue."**

On confirmation:

* Visit status updates  
* Visit timestamp recorded  
* Patient card appears in the Care Queue visible to all providers  
* Triage user's view returns to the search screen

Care Queue card displays: patient name, age, check-in time, triage user initials, and a colored triage acuity badge — the "priority flag" placeholder from earlier drafts of this spec is now the triage acuity level described in Step 2c.

---

### **Stage 2 — Provider (Care Team)**

**Step 4: Assume Patient from Care Queue**

Provider sees a live Care Queue list ordered by check-in time (FIFO by default).

Each queue card shows: name, age, photo, vitals summary (flagged abnormals highlighted).

Provider clicks **"Assume Patient."** Visit status updates, timestamps the assumption, and locks the patient from being assumed by another provider. Provider's active patients appear in a sidebar or tab.

---

**Step 5: Review Past Patient Notes**

Within the open chart, a **Visit History** tab shows all prior visits in reverse chronological order.

Each past visit expandable to show:

* Vitals from that visit  
* SOAPP note  
* Medications and vaccines administered  
* Prescriptions filled

Read-only. Providers cannot edit past visit records. This is critical for both clinical integrity and audit purposes.

---

**Step 5b: Review Current-Visit Context**

Before or alongside documentation, the chart surfaces what triage already captured so nothing has to be asked twice:

* Mother's name and any chronic-condition flags from the patient record  
* A pregnancy indicator, if applicable  
* The assigned triage acuity level  
* Chief complaint, free-text past medical history, current medications, and family/social history captured at check-in

---

**Step 6: Add SOAPP Note**

Core clinical documentation. Each section is a non-linear free-text field with generous character limits:

* **Subjective** — Patient-reported symptoms, chief complaint, history of present illness  
* **Objective** — Clinician observations, physical exam findings, reference to current vitals  
* **Assessment** — Diagnosis or differential diagnosis  
* **Plan** — Treatment plan, follow-up instructions, referrals  
* **Procedures Completed** — Structured checklist (e.g. wound care, IV placement, ECG) plus free-text for unlisted procedures

Auto-save every 30 seconds. SOAPP note is not submitted until the provider explicitly closes the visit — partial notes are preserved.

Provider can also correct the vitals recorded at triage directly from the chart via an inline edit control, rather than sending the patient back to triage. The correction replaces the current visit's vitals and is reflected everywhere this visit's vitals are shown, including the pharmacy chart if the visit is sent on.

---

**Step 7: Log Medications and Vaccines Administered**

Separate section within the visit: **Administered in Clinic.**

Provider adds line items:

* Medication or vaccine name (searchable dropdown \+ free text for unlisted)  
* Dose and unit (mg, mL, units)  
* Route (oral, IV, IM, subcutaneous, topical)  
* Date/time administered (defaults to now, editable)

Multiple items can be added. Each line item is saved independently.

Within the same section, provider can also select any **Resources Ordered** from a fixed checklist: Ultrasound, Osteopathic Manipulative Treatment (OMT), Electrocardiogram (ECG/EKG), Pregnancy Test, Stress Test, H&H Blood Test (Hemoglobin and Hematocrit), Urine Test. Multiple selections allowed; carried through to pharmacy/completed record as documentation only — this does not place an order with an external lab or imaging system.

---

**Step 8: Add Pharmacy Prescription Requests**

Separate section: **Pharmacy Requests.**

Provider adds one or more prescription requests:

* Medication name (searchable)  
* Dose and unit  
* Route  
* Instructions / sig (e.g. "take with food," "do not crush")  
* Notes to pharmacist (free text, optional)

Each request is saved as a separate record with status.

---

**Step 9: Close Chart and Send to Pharmacy Queue**

Provider reviews all entries, then clicks **"Close and Send to Pharmacy."**

System checks: at least one pharmacy request must exist to send to Pharmacy Queue. If no prescriptions, provider can alternatively **"Close Visit"** directly (skips pharmacy stage — visit goes to completed).

On send to pharmacy:

* Visit status updates  
* Patient card appears in Pharmacy Queue  
* Provider's chart closes, returns to Care Queue view

---

### **Stage 3 — Pharmacy Team**

**Step 10: Assume Patient from Pharmacy Queue**

Pharmacy Queue mirrors Care Queue UX — FIFO list, each card showing patient name, wait time, number of prescription requests, and the assuming provider's name.

Pharmacist clicks **"Assume Patient."** Visit locks to that pharmacist.

---

**Step 11: Review Medication Requests and Log Fulfillment**

Pharmacist sees all pending prescription requests from the provider.

For each request, pharmacist:

* Reviews medication name, dose, route, instructions, with allergies, chronic-condition flags, and a pregnancy indicator (if applicable) all shown prominently before any dispensing action, alongside the full SOAPP note (Subjective, Objective, Assessment, Plan, Procedures) for complete clinical context  
* Marks status: **Filled / Partial / Unable to Fill**  
* If filled: logs quantity dispensed  
* If partial or unable: adds a reason note (out of stock, contraindication, formulary issue)

Each request is handled independently — a visit can have some filled and some declined.

---

**Step 12: Log Frequency and Set Reminders**

For each filled medication:

* **Frequency**: dropdown (once daily / twice daily / three times daily / every N hours / as needed / other)  
* **Reminder trigger**: toggle Yes/No  
* If Yes: reminder schedule is recorded (time of day, days of week, duration)

In V1, "reminder trigger" can simply flag the record — actual push notification or SMS delivery can be a V2 feature. But capture the data now so you're not retrofitting.

---

**Step 13: Close Patient Chart**

Pharmacist confirms all requests have been addressed, then clicks **"Close Patient Chart."**

Visit status updates. Timestamp recorded. Chart is now read-only across all roles.

On close, system triggers:

* **Email report to patient** (if email on file) — summarizing: visit date, provider name, diagnoses (from SOAPP assessment field), medications dispensed, frequency/reminder schedule, follow-up instructions (from SOAPP plan field)  
* Record moves to Completed Patients Dashboard

---

### **Stage 4 — Completed Patients Dashboard**

**Step 14: Aggregated View**

A chronological (most recent first) list of all completed visits, accessible to pharmacy team and admin roles.

Each row shows:

* Patient name, DOB  
* Visit date  
* Provider name, Pharmacist name  
* Number of prescriptions requested vs. filled  
* Visit duration (check-in to chart close)

Filterable by: date range, provider, pharmacist, location.

Clickable to open read-only visit summary.

**Metrics panel (top of dashboard):**

* Total completed visits today / this week / this month  
* Average wait time per stage (triage → care, care → pharmacy)  
* Prescription fill rate (filled / total requested)

---

### **Live Operations Dashboard (Command Center)**

A read-only, three-column board — Triage, Care, Pharmacy — showing every patient currently waiting at each stage (name, age, gender, wait time, chief complaint, allergy and triage-acuity indicators), plus a metrics strip (average wait per stage, prescription fill rate, completed-visit count, active-patient count). Unlike the Completed Patients Dashboard above, this is a live view of work-in-progress, not a historical record.

Available to every clinical role — Triage, Provider, and Pharmacy — not only Program Manager. It is strictly read-only: no role can assume, discharge, or edit a patient from this view.

---

## **Edge Cases to Design For**

* **Patient walks out mid-visit** — each stage needs a "Discharge / Remove from Queue" action with a reason code (left without being seen, refused care, etc.)  
* **Provider needs to correct vitals** — handled in place via the inline vitals-edit control on the provider's chart (see Step 6); the patient is no longer routed back to triage for this. A formal "Return to Triage" action remains a candidate for cases a vitals correction can't resolve (e.g. the patient needs to be physically re-examined), but is not present in the current build.  
* **Pharmacy has a question for the provider** — a simple internal note/flag on the prescription request that alerts the provider without moving the visit backward in the queue  
* **Duplicate visits** — if a patient is checked in twice in the same day, surface a warning but allow override with a reason  
* **No pharmacy needed** — as noted above, provider can close without routing to pharmacy; this path needs to be clean and intentional, not accidental  
* **Duplicate roles** \- Volunteer can be a Provider one day, and a Pharmacist the next day. Interoperability of certain roles.  
* **Volunteer navigates away mid-entry** — any screen with unsaved input (new patient form, vitals, SOAPP note, pharmacy fulfillment) prompts a confirm-before-discard dialog rather than silently losing work; role switches are guarded the same way.

---

## **What to Defer to V2**

To keep V1 crude and buildable:

* Actual SMS/push reminder delivery (capture the data, send manually for now)  
* Insurance and billing  
* Lab orders and results  
* Appointment scheduling / pre-registration  
* Role-based permission granularity beyond the four roles above  
* Multi-clinic / multi-location support (build location field now, don't build the routing logic yet)  
* Audit log UI (log the events in the database from day one, build the UI later)  
* Actual order integration for Resources Ordered — it is documentation only in V1 and does not place a real order with a lab or imaging system

