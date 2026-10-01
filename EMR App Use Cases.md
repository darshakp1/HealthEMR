**Mission EMR — Use Case Specification V1**

---

### **UC-1 — Search & identify patient**

Actor: Triage volunteer Trigger: A patient presents at the clinic for care. Preconditions: Triage volunteer is logged in and assigned to an active trip. At least one prior patient record may exist in the system.

Main flow:

1. Triage volunteer lands on the patient search screen (default home view).  
2. Volunteer enters one or more search parameters: full name, phone number, or patient ID.  
3. System returns a results list showing name, DOB, phone, and last visit date.  
4. Volunteer identifies the correct patient and selects their record.  
5. System opens the patient chart and proceeds to UC-3.

Alternate flows: No results found → system surfaces "Patient not found" prompt; volunteer proceeds to UC-2. Multiple possible matches → system displays all candidates; volunteer selects the correct one or confirms none match and proceeds to UC-2.

Exception flows: Duplicate record detected (same name \+ DOB or same phone number) → system flags the conflict with a warning. Volunteer selects the correct record or overrides with a documented reason.

Postconditions: An existing patient record is identified and ready for a new visit, or the volunteer is routed to UC-2.

Business rules: Search must match on at least one field; blank search is not permitted. Duplicate detection runs on name \+ DOB and phone number independently — either match triggers the warning. Search results are scoped to the current organization across all trips.

Data requirements: Search inputs — first name, last name, phone number, patient ID (any one sufficient). Results display — name, DOB, phone, last visit date.

---

### **UC-2 — Register new patient**

Actor: Triage volunteer Trigger: UC-1 returns no match, or volunteer confirms no existing record applies. Preconditions: Triage volunteer is logged in. No matching patient record exists.

Main flow:

1. Volunteer opens the new patient registration form.  
2. Volunteer enters demographic fields: first name, last name, extension name (an optional second last name used as an additional identifier), date of birth, gender, phone number (optional), email, clinic site, and mother's name.  
3. Volunteer reads the photo consent disclosure aloud to the patient, in English or Spanish, explaining that the photograph is used solely to confirm identity at a future visit. Volunteer then captures a patient photo via device camera (mobile) or file upload (desktop). Basic guidance is displayed on screen.  
4. Volunteer reviews entered information and submits.  
5. System runs duplicate detection on name \+ DOB and phone number before saving.  
6. System creates the patient record and proceeds to UC-3.

Alternate flows: Volunteer retakes photo before submitting → system replaces the preview; submission is not blocked. Email not available → field is optional; system records absence; patient will not receive automated visit summary.

Exception flows: Duplicate detected at submission → system surfaces warning with the matching record; volunteer either navigates to the existing record or confirms this is a distinct patient and overrides with a reason code. Photo capture fails → system blocks submission; check-in cannot proceed without a photo; volunteer must resolve before continuing.

Postconditions: A new patient record exists with demographic data and a photo of record. The visit is initialized and ready for vitals entry.

Business rules: Photo is mandatory for new patient creation — no exception path exists in V1. The photo consent disclosure must be presented in both English and Spanish and read aloud before capture; it is a verbal consent only in V1, not a recorded or signed artifact. Phone number is optional and is not required to complete registration or check-in. Email is optional but must be valid format if entered. Gender field must offer: Male, Female, Non-binary, Prefer not to say, Other. Location is captured as clinic site only — no patient home address collected in V1. Guardian/proxy field is not supported in V1. Patient ID is system-generated on record creation, in the format PT-XXXX-XXXX using a cryptographically secure random generator (crypto.getRandomValues) over a 32-character alphabet that excludes visually ambiguous characters (0, O, 1, I); the generator checks for collisions against IDs already in use and regenerates until it finds one that isn't. This uniqueness check runs in application memory only — once the system has persistent storage, patient ID must carry a real database-level UNIQUE constraint; this generator is designed to make that constraint trivially satisfiable, not to replace it.

Data requirements: Required — first name, last name, DOB, gender, clinic site, photo. Optional — extension name, phone, email, mother's name.

---

## **UC-3 — Record vitals & check in**

Actor: Triage volunteer Trigger: Patient record is opened from UC-1 or UC-2. Preconditions: Patient record exists. A new visit has been initialized. Triage volunteer is logged in and assigned to an active trip.

Main flow:

1. System opens the vitals entry form. For returning patients, confirmed allergies and previously documented chronic conditions from the patient record are pre-populated.  
2. Volunteer records all vitals: systolic BP, diastolic BP, heart rate, respiratory rate, temperature (°F/°C), weight (lbs/kg), height (in/cm), SpO2 (%). Once both weight and height are present, the system automatically calculates and displays BMI with a category (Underweight / Normal / Overweight / Obese), recalculating immediately if either value or its unit changes.  
3. Volunteer assigns a triage acuity level — a required, single-select choice of 1 (Resuscitation) through 5 (Non-Urgent). Each level is color-coded (1 red through 5 green) and carries a hover tooltip describing its clinical condition and examples. Check-in cannot proceed without a level selected.  
4. Volunteer captures chief complaint (free text, required).  
5. For returning patients: volunteer reviews pre-populated allergy list and either confirms unchanged or edits as needed, directly below the allergy field.  
6. Volunteer completes the Patient History section: free-text past medical history (captured per visit), six chronic-condition checkboxes (Hypertension, Diabetes, Arrhythmia, Heart Attack, High Cholesterol, Surgery) that write to the patient record rather than the visit, free-text current medications, and free-text family/social history.  
7. For female patients only: volunteer completes the Maternity section — whether the patient is currently pregnant, and date of last menstrual period.  
8. For returning patients with an existing photo: system displays the prior photo with options to keep or retake.  
9. Volunteer reviews all entered data and clicks "Check in and send to care queue."  
10. System timestamps the check-in, updates visit status, and places a patient card in the care queue visible to all providers.  
11. Triage volunteer's view returns to the search screen.

Alternate flows: New patient (no prior allergies) → allergy field is blank; volunteer enters known allergies or documents NKDA. Returning patient selects photo retake → new photo becomes the photo of record going forward. One or more vitals unavailable → volunteer may leave individual fields blank; system flags missing vitals on the care queue card but does not block check-in.

Exception flows: Abnormal vital value entered → system displays a soft warning (yellow flag); specific thresholds to be defined with clinical lead prior to implementation; warning is informational only and does not block check-in. Duplicate check-in same day → system surfaces a warning; volunteer may proceed with override and documented reason, or cancel. Patient leaves before check-in is complete → volunteer uses UC-13 with reason code "left before check-in complete."

Postconditions: Visit is active. Patient card appears in the care queue with vitals summary, flagged abnormals highlighted, check-in timestamp, and triage volunteer initials. Triage's ownership ends; ownership transfers to the provider queue.

Business rules: A new visit record is created for each check-in, even for returning patients. Prior visit records are never modified by triage — confirmed allergy update writes to the current visit only but propagates as the new default for future visits. The "Confirmed unchanged" allergy checkbox must be explicitly selected — it cannot be the default state. Chief complaint is required; check-in cannot be completed without it. Triage acuity level is required; check-in is blocked without a selection. The triage acuity level is captured on the care queue card as a colored badge, visible without expanding the card — this replaces the earlier placeholder priority flag referenced in prior drafts of this spec. It does not reorder the queue; ordering remains FIFO by check-in time (see UC-4). Chronic-condition checkboxes write to the patient record and persist across all future visits — they are not reset or re-asked at subsequent check-ins unless a volunteer changes them. Free-text past medical history, current medications, and family/social history are captured per visit and do not carry forward automatically. Maternity fields (pregnancy status, date of last period) are shown only when the patient's recorded gender is Female.

Data requirements: Vitals — systolic BP, diastolic BP, HR, RR, temperature \+ unit, weight \+ unit, height \+ unit, SpO2, BMI (calculated, stored with the visit). Triage acuity level — integer 1–5, required. Chief complaint — free text, required. Allergies — free text list with confirmation status. Patient history — free-text past medical history (per visit), six chronic-condition booleans (patient-level, persistent), free-text current medications, free-text family/social history. Maternity — pregnant (boolean), date of last period (date), female patients only. Photo — image file with timestamp.

Open questions: None.

---

## **UC-4 — Assume patient from care queue**

Actor: Provider Trigger: Provider is ready to see the next patient and selects a card from the care queue. Preconditions: Provider is logged in and assigned to an active trip. At least one patient card exists in the care queue with status "waiting." Provider does not currently have an active patient locked to them. Patient has been checked in by triage (UC-3 complete).

Main flow:

1. Provider views the care queue — a live FIFO list, each card showing patient name, age, photo, a colored triage acuity badge, vitals summary, flagged abnormal vitals, chief complaint, check-in time, and triage volunteer initials.  
2. Provider selects a patient card and clicks "Assume patient."  
3. System timestamps the assumption, updates visit status to "in care," and locks the record to this provider.  
4. Patient moves from the care queue to the provider's active patient view.  
5. Provider proceeds to UC-5 or directly to UC-6 for new patients.

Alternate flows: Care queue is empty → provider sees an empty state; no action available until triage checks in a new patient.

Exception flows: Provider attempts to assume a patient while already holding an active patient → system prevents the action; provider must close or hand off their current patient first. Provider attempts to assume a patient locked by another provider → system prevents the action and displays which provider holds the lock.

Postconditions: Visit status is "in care." Record is locked to the assuming provider. Patient card is removed from the shared care queue. Triage can no longer modify the visit record.

Business rules: A provider may hold only one active patient at a time. Only one provider can hold the lock on a patient record at a time. Lock is released only by: provider closing the visit, provider sending to pharmacy, or provider returning the patient to triage (UC-14). FIFO ordering is the default; the triage acuity level is visible on each card as a colored badge but does not reorder the queue in V1 — reordering by acuity remains a future enhancement.

Data requirements: Care queue card — name, age, photo, triage acuity level, check-in timestamp, triage volunteer initials, vitals summary with flagged abnormals, chief complaint. Assumption timestamp recorded on the visit record.

Open questions: None.

---

### **UC-5 — Review patient history**

Actor: Provider Trigger: Provider opens a patient chart after assuming from the care queue. Preconditions: Provider has assumed the patient (UC-4 complete). Patient has at least one prior visit record in the system.

Main flow:

1. Provider opens the Visit History tab within the patient chart.  
2. System displays all prior visits in reverse chronological order, each showing visit date, trip location, and treating provider name.  
3. Provider expands one or more prior visits to review: vitals, SOAPP note, medications and vaccines administered, and prescriptions filled.  
4. Provider uses the history to inform the current encounter and proceeds to UC-6.

Alternate flows: New patient with no prior visits → Visit History tab displays an empty state; provider proceeds directly to UC-6. Provider reviews multiple prior visits → each is independently expandable with no limit.

Exception flows: None specific to this use case. Missing data from a prior trip surfaces as a visible gap in the record, not a system error.

Postconditions: Provider has reviewed available history. No data is written or modified — this use case is read-only throughout.

Business rules: All prior visit records are permanently read-only for all roles; corrections are handled via UC-12. History is scoped to the patient across all trips and locations, not filtered to the current trip only. Allergies confirmed at prior visits are surfaced within the Visit History tab — not as a persistent banner. Clinical responsibility for decisions rests with the assuming provider of record.

Data requirements: Per prior visit — visit date, location, provider name, vitals, full SOAPP note, administered medications and vaccines (name, dose, route, date/time), prescriptions filled (name, dose, route, quantity dispensed, fill status).

---

## **UC-6 — Document SOAPP note**

Actor: Provider Trigger: Provider begins clinical documentation for the current encounter. Preconditions: Provider has assumed the patient (UC-4 complete). Visit is in "in care" status.

Main flow:

1. Provider opens the current visit. The chart displays the patient's mother's name, any chronic-condition flags and a pregnancy indicator carried on the patient record, current vitals (with BMI) and any flagged abnormal values, confirmed allergies, the assigned triage acuity level, and the chief complaint, past medical history, current medications, and family/social history captured at triage. Provider opens the SOAPP note section to begin documentation.  
2. Provider enters free text across five sections: Subjective, Objective, Assessment, Plan, and Procedures Completed. Sections are non-linear — provider may fill in any order.  
3. System auto-saves every 30 seconds. A last-saved timestamp is displayed.  
4. Provider continues documenting until the encounter is complete, then proceeds to UC-7 or UC-8.  
5. If a triage-entered vital needs correction, provider edits it directly from the chart via an inline edit control, rather than sending the patient back to triage (UC-14). The correction replaces the current visit's vitals and is reflected everywhere this visit's vitals are shown, including the pharmacy chart if the visit is sent on.  
6. SOAPP note is finalized when the provider closes the visit via UC-8. Until then it remains editable.

Alternate flows: Provider navigates away mid-documentation → auto-save preserves all content; provider can return and resume without data loss. Procedures Completed — provider selects from a structured checklist and/or enters free text for unlisted procedures; both inputs are valid simultaneously. No procedures performed → section left blank; does not block visit closure.

Exception flows: Auto-save fails (connectivity loss) → system displays a visible warning; local draft preserved on-device where possible. Provider attempts to access another provider's in-progress note → note is private until visit closure; other providers see the patient as locked and cannot view note content.

Postconditions: SOAPP note is saved and associated with the current visit. Note remains editable until visit is explicitly closed.

Business rules: SOAPP note is private to the assuming provider until visit closure. Co-signature is not required in V1; clinical responsibility is attributed to the assuming provider of record. A provider's vitals correction overwrites the current visit's vitals — the original triage entry is not separately retained as a distinct historical value in V1 — and is timestamped as part of the provider's save activity. All five sections have generous character limits with no hard maximum in V1. Auto-save interval is 30 seconds; manual save is also available. The Procedures Completed checklist is configurable by the master admin. A finalized SOAPP note is permanently read-only; corrections require UC-12.

Data requirements: Per section — Subjective (free text), Objective (free text), Assessment (free text), Plan (free text), Procedures Completed (checklist selection \+ free text). Auto-save timestamp displayed to provider.

Open questions: None.

---

### **UC-7 — Log administered medications & vaccines**

Actor: Provider Trigger: Provider administers a medication or vaccine during the encounter and needs to document it. Preconditions: Provider has assumed the patient (UC-4 complete). Visit is in "in care" status.

Main flow:

1. Provider opens the "Administered in clinic" section of the current visit.  
2. Provider selects medication or vaccine name from a searchable dropdown, or enters free text for an unlisted item.  
3. Provider enters dose and unit (mg, mL, units), route (oral, IV, IM, subcutaneous, topical), and date/time administered (defaults to current time, editable).  
4. Each line item is saved independently on entry.  
5. Within the same section, provider selects any resources ordered during the encounter from a fixed checklist: Ultrasound, Osteopathic Manipulative Treatment (OMT), Electrocardiogram (ECG or EKG), Pregnancy Test, Stress Test, H&H Blood Test (Hemoglobin and Hematocrit), and Urine Test. Multiple selections are allowed and saved as part of the visit record.  
6. Provider adds additional line items as needed, then proceeds to UC-8.

Alternate flows: Multiple items administered → provider adds each as a separate line item with no limit. Item not found in dropdown → provider enters free text; saved identically to a dropdown-selected item. Date/time adjusted → provider may backdate within the current visit day.

Exception flows: Administered medication matches a known patient allergy → system displays a prominent allergy conflict warning; non-blocking; provider may proceed with documented override; warning and override recorded on the visit; pharmacy is not notified. Provider leaves section without adding any items → section remains blank; valid, as not all encounters involve in-clinic administration.

Postconditions: All administered medications and vaccines are recorded with dose, route, and timestamp. Each entry is independently saved and immediately visible in the visit record.

Business rules: Administered medications are distinct from pharmacy prescription requests — this section documents what was given in the clinic, not what the patient is taking home. Resources Ordered is a fixed, non-extensible checklist in V1; a selection is documentation only and does not place an actual order with a lab or imaging system. Allergy conflict check runs against allergies confirmed at triage for the current visit and all prior visits. Allergy conflict warning is surfaced to the provider only — pharmacy is not notified. Free text entries are saved as-is with no normalization in V1. Each line item is immutable once the visit is closed; corrections require UC-12.

Data requirements: Per line item — medication/vaccine name (dropdown or free text), dose, unit, route, date/time administered, administering provider ID (system-recorded). Resources ordered — multi-select from the fixed checklist above, carried through to the pharmacy/completed record.

---

## **UC-8 — Submit pharmacy requests & close provider encounter**

Actor: Provider Trigger: Provider has completed the clinical encounter and is ready to hand off or close the visit. Preconditions: Provider has assumed the patient (UC-4 complete). SOAPP note has been documented (UC-6). Visit is in "in care" status.

Main flow:

1. Provider opens the "Pharmacy requests" section and adds one or more prescription requests, each with: medication name (searchable), dose and unit, route, sig/instructions, and optional notes to pharmacist.  
2. Provider reviews all visit entries (SOAPP, administered items, Rx requests) and clicks "Close and send to pharmacy."  
3. System validates that at least one pharmacy request exists.  
4. System finalizes the SOAPP note (now read-only), updates visit status to "awaiting pharmacy," timestamps the handoff, and places the patient card in the pharmacy queue.  
5. Provider's chart closes and returns to the care queue view.

Alternate flows: No pharmacy needed → provider clicks "Close visit directly"; system skips pharmacy queue; visit status moves to "completed"; email summary triggered immediately; provider returns to care queue. Multiple Rx requests → provider adds each as a separate line item, tracked independently through fulfillment. Medication not in dropdown → provider enters free text; saved identically to a dropdown selection.

Exception flows: Provider clicks "Close and send to pharmacy" with no Rx requests → system blocks the action and prompts provider to either add a request or use "Close visit directly." Connectivity lost before handoff completes → system retains visit in "in care" status and alerts provider; handoff must be re-attempted on reconnect; auto-saved SOAPP content is preserved.

Postconditions — send to pharmacy path: SOAPP note is finalized and read-only. Visit status is "awaiting pharmacy." Patient card appears in pharmacy queue. Provider lock is released. Provider retains read-only access to the visit record including pharmacy fulfillment status as it updates.

Postconditions — close directly path: Visit status is "completed." SOAPP note is finalized and read-only. Automated visit summary email sent to patient if email is on file. Record moves to completed visits dashboard.

Business rules: A provider cannot send to pharmacy without at least one Rx request. The two close paths must be visually distinct and require deliberate selection — accidental pharmacy skip must not be possible. Once sent to pharmacy, provider cannot recall or re-open the visit except via UC-12 or Program Manager/Master Admin reopen. Notes to pharmacist are visible to pharmacy only — excluded from the patient-facing email. Each Rx request is saved as an independent record with its own fulfillment status. Email delivery is triggered on visit closure by either provider (direct close) or pharmacy (UC-11); content and format to be finalized with stakeholders.

Data requirements: Per Rx request — medication name (dropdown or free text), dose, unit, route, sig/instructions (free text), notes to pharmacist (free text, optional). Handoff timestamp recorded on visit record.

Open questions: None.

---

## **UC-9 — Assume patient from pharmacy queue**

Actor: Pharmacy volunteer Trigger: Pharmacy volunteer is ready to fulfill the next patient's prescriptions. Preconditions: Pharmacy volunteer is logged in and assigned to an active trip. At least one patient card exists in the pharmacy queue with status "awaiting pharmacy." Pharmacy volunteer does not currently hold an active patient lock. Provider has closed the encounter and sent to pharmacy (UC-8 complete).

Main flow:

1. Pharmacy volunteer views the pharmacy queue — a live FIFO list, each card showing patient name, wait time, number of Rx requests, and provider name.  
2. Volunteer selects a patient card and clicks "Assume patient."  
3. System timestamps the assumption, updates visit status to "in pharmacy," and locks the record to that pharmacy volunteer.  
4. Patient card moves from the shared pharmacy queue to the volunteer's active view.  
5. Volunteer proceeds to UC-10.

Alternate flows: Pharmacy queue is empty → volunteer sees an empty state; no action available until a provider sends a patient to pharmacy.

Exception flows: Volunteer attempts to assume a patient already locked by another pharmacy volunteer → system prevents the action and displays which volunteer holds the lock.

Postconditions: Visit status is "in pharmacy." Record is locked to the assuming pharmacy volunteer. Patient card is removed from the shared pharmacy queue.

Business rules: A pharmacy volunteer may only hold one active patient at a time, consistent with the provider constraint in UC-4. Lock is released only by: pharmacy volunteer closing the visit (UC-11) or a master admin override. Provider retains read-only access to the record throughout the pharmacy stage.

Data requirements: Pharmacy queue card — patient name, wait time since provider handoff, number of Rx requests, provider name. Assumption timestamp recorded on visit record.

Open questions: None.

---

## **UC-10 — Fulfill prescriptions**

Actor: Pharmacy volunteer Trigger: Pharmacy volunteer has assumed the patient from the pharmacy queue (UC-9 complete). Preconditions: Visit is in "in pharmacy" status. At least one Rx request exists on the visit record. Patient allergies are available from the visit record.

Main flow:

1. System displays all pending Rx requests from the provider, with patient allergies, chronic-condition flags, and a pregnancy indicator (if applicable) shown prominently at the top of the screen before any dispensing action is available.  
2. Volunteer reviews each Rx request: medication name, dose, route, sig/instructions, and provider notes.  
3. For each request, volunteer selects a fulfillment status: Filled, Partial, or Unable to fill.  
4. If filled: volunteer logs quantity dispensed.  
5. If partial or unable to fill: volunteer enters a reason note (out of stock, contraindication, formulary issue, or free text).  
6. For each filled medication, volunteer sets frequency (once daily, twice daily, three times daily, every N hours, as needed, other) and toggles reminder flag on or off.  
7. Each request is handled and saved independently. Volunteer proceeds to UC-11 when all requests are addressed.

Alternate flows: Multiple Rx requests → each worked through independently in any order; visit cannot be closed until all have a documented status. Reminder toggle set to yes → volunteer records time of day, days of week, and duration; data captured for V2 delivery; no push notification sent in V1. Volunteer needs clinical context → volunteer reads provider SOAPP note in read-only mode within the same view.

Exception flows: Filled medication matches a known patient allergy → system displays a prominent non-dismissible allergy conflict warning before "filled" status can be confirmed; volunteer must explicitly acknowledge and document an override reason; warning and override permanently recorded on the visit. Pharmacy volunteer has a question for the provider → volunteer adds an internal flag and note on the specific Rx request; system sends an active in-app notification to the provider that persists until acknowledged; visit does not move backward in the queue; pharmacy retains ownership; provider responds via the same flag thread. Volunteer attempts to close with unaddressed requests → system blocks closure and highlights unresolved items.

Postconditions: Every Rx request has a documented fulfillment status. Quantity dispensed, frequency, and reminder flag are recorded for all filled items. Deviations from the prescription are documented with reason notes.

Business rules: Allergy list must be displayed prominently before any dispensing action is available — it cannot be collapsed or dismissed during the fulfillment workflow. A filled status requires quantity dispensed to be entered. Partial and unable to fill statuses require a reason note. Reminder data is captured in V1 but delivery mechanism is deferred to V2. Pharmacy volunteer may read the SOAPP note for clinical context but cannot edit any provider-authored content. Allergy conflict override is permanently visible in the audit trail. Provider receives an active in-app notification when a pharmacy flag is raised; notification persists until acknowledged.

Data requirements: Per Rx request — fulfillment status (filled/partial/unable), quantity dispensed, deviation reason if applicable, frequency, reminder flag, reminder schedule if applicable, internal flag and note if applicable, allergy conflict override reason if triggered.

Open questions: None.

---

## **UC-11 — Close visit & trigger record handoff**

Actor: Pharmacy volunteer (primary closer). Program Manager or Master Admin (reopen authority). Trigger: All Rx requests have been fulfilled or resolved (UC-10 complete). Preconditions: Visit is in "in pharmacy" status. Every Rx request has a documented fulfillment status. Pharmacy volunteer holds the active lock on the record.

Main flow:

1. Pharmacy volunteer reviews the complete visit summary: patient demographics, vitals, SOAPP note (read-only), administered medications, and all Rx fulfillment records.  
2. Volunteer confirms all requests have been addressed and clicks "Close patient chart."  
3. System validates that no Rx requests remain unaddressed.  
4. System updates visit status to "completed," records closure timestamp, and makes the chart read-only across all roles.  
5. System triggers automated visit summary email to patient if email is on file.  
6. Record moves to the completed visits dashboard, visible to all roles with read access.

Alternate flows: Patient has no email on file → email step is skipped; record closed and moved to dashboard. Program Manager or Master Admin reopens a closed visit → visit returns to "in care" status and re-enters the care queue; reopening actor must supply a mandatory reason note; reopen event recorded in audit trail with timestamp, actor, and reason; provider may update vitals (new entry, not overwrite) or amend the SOAPP note directly; provider then closes via UC-8 paths (send to pharmacy or close directly); if sent to pharmacy, pharmacy volunteer sees the full prior dispensing record alongside new Rx requests, clearly labeled as "prior dispensing — visit reopened."

Exception flows: Unaddressed Rx requests detected at closure attempt → system blocks closure and highlights unresolved items. Connectivity lost during closure → system retains visit in "in pharmacy" status; closure must be re-attempted on reconnect; all fulfillment data preserved.

Postconditions: Visit status is "completed." Chart is read-only across all roles unless reopened by Program Manager or Master Admin. Visit summary email sent to patient if email is on file. Record appears in completed visits dashboard. All roles have read-only access to the full visit record.

Business rules: Chart closure is reversible by Program Manager and Master Admin only. All other roles use UC-12 for corrections. On reopen, the visit returns to "in care" status; provider updates vitals or SOAPP directly (no addendum required for a reopened visit); standard UC-8 close paths apply on re-closure. When a reopened visit is sent to pharmacy, pharmacy sees the full prior dispensing record alongside new Rx requests, clearly labeled. Automated email content and format to be finalized with stakeholders prior to implementation. Notes to pharmacist are excluded from the patient-facing email. Closure timestamp is system-generated and cannot be edited. Read access to completed records is granted to all roles.

Data requirements: Visit summary email — visit date, provider name, assessment/diagnosis, dispensed medications (name, dose, frequency, reminder schedule), plan/follow-up instructions; full spec pending. Closure timestamp recorded on visit record. Reopen audit entry — timestamp, actor user ID, actor role, reason note.

Open questions: None.

---

## **UC-12 — Add addendum**

Actors: Triage volunteer, Provider, Pharmacy volunteer, Program Manager, Master Admin Trigger: A role identifies an error, omission, or additional information that needs to be added to a visit record. Preconditions: A visit record exists. The actor is logged in and was involved in the visit, or holds Program Manager/Master Admin access. The section being annotated is within the actor's role scope.

Main flow:

1. Actor navigates to the relevant visit record via the completed visits dashboard or active chart.  
2. Actor opens the addendum section associated with the relevant part of the record (triage data, SOAPP note, or dispensing record).  
3. Actor enters addendum text (free text, required) and submits.  
4. System timestamps the addendum, records the authoring role and user ID, and appends it beneath the original entry.  
5. Original content is never modified or overwritten — the addendum is always a separate, attributed append.

Alternate flows: Actor adds multiple addenda to the same visit → each saved as an independent entry with its own timestamp and attribution; no limit on number. Addendum added to an active (not yet closed) visit → permitted; appended immediately and visible to the current record owner.

Exception flows: Actor attempts to add an addendum outside their role scope → system prevents the action. Connectivity lost during submission → system preserves the draft locally and retries on reconnect; actor is alerted the addendum has not yet been saved.

Postconditions: Addendum is permanently appended to the visit record with authorship and timestamp. Original record content is unchanged. Addendum is visible to all roles with read access.

Business rules: Addenda are append-only — no role can delete or edit a submitted addendum, including master admin. Each addendum is attributed to the specific user and role that authored it. Role scope for addenda mirrors role scope for original documentation: triage annotates triage data, providers annotate SOAPP and administered medications, pharmacy annotates dispensing records. Program Manager and Master Admin may add addenda to any section of any visit record. Addenda are included in the permanent audit trail. Addenda do not trigger a new patient-facing email in V1. A corrective addendum by a provider after pharmacy has closed the visit does not trigger a pharmacy notification.

Data requirements: Per addendum — free text content, authoring user ID, authoring role, timestamp, section reference.

Open questions: None.

---

## **UC-13 — Discharge patient mid-visit**

Actors: Triage volunteer, Provider, or Pharmacy volunteer — whichever role currently owns the patient. Trigger: A patient leaves, refuses care, or must be removed from the active queue before their visit is complete. Preconditions: A visit is active and owned by the acting role. Patient has not yet reached a completed status.

Main flow:

1. Acting role locates the active patient in their queue or active patient view.  
2. Actor selects "Discharge / remove from queue."  
3. System prompts for a reason code (required): left without being seen, left against medical advice, refused care, administrative removal, or other (free text).  
4. Actor confirms the discharge.  
5. System timestamps the discharge, records the reason code and acting user, updates visit status to "discharged — incomplete," and removes the patient from all active queues.  
6. System sends an active in-app notification to the Program Manager with: patient identifier, discharging role, reason code, and timestamp.  
7. Record moves to the completed visits dashboard with status clearly marked as "discharged — incomplete."

Alternate flows: Discharge initiated by triage before provider assumption → visit has minimal data; record closed with whatever was captured. Discharge initiated by provider mid-encounter → SOAPP note in progress is auto-saved at point of discharge; partial note preserved with a "visit discharged" flag. Discharge initiated by pharmacy → all Rx requests remain in their current fulfillment state; partial fulfillment documented as-is with the discharge reason.

Exception flows: Actor attempts to discharge a patient locked by a different role → system prevents the action and displays which role holds the lock; lock-holding role must initiate the discharge or release the lock first.

Postconditions: Visit status is "discharged — incomplete." Record is closed and read-only (addenda still permitted via UC-12). Patient is removed from all active queues. Program Manager has been notified. No patient-facing email is sent for discharged visits in V1.

Business rules: Reason code is mandatory — discharge cannot be confirmed without a selection. "Other" reason code requires free text explanation. A discharged visit record preserves all data captured up to the point of discharge. No patient-facing email is triggered on discharge in V1 — deferred to V2 with clinical input. A discharged visit can be reopened by Program Manager or Master Admin if the patient returns and continuity is needed, consistent with UC-11 reversal rules. Program Manager receives an active in-app notification for every mid-visit discharge.

Data requirements: Discharge reason code (structured \+ optional free text), discharging user ID, discharging role, discharge timestamp, visit status updated to "discharged — incomplete." Notification payload to Program Manager — patient name, discharging role, reason code, timestamp.

Open questions: None.

---

## **UC-14 — Return patient to triage**

*Implementation status: the explicit "Return to Triage" action has been removed from the current provider interface. In practice, the most common trigger for this use case — incorrect or incomplete vitals — is now resolved in place via the vitals-correction capability described in UC-6, without handing the patient back to triage. This use case is retained as the documented workflow for situations an in-place vitals correction can't resolve (e.g. the patient needs to be physically re-examined by triage, or more than vitals need to be re-collected); re-introducing it as an explicit action is a candidate for a future version if that need shows up in practice.*

Actor: Provider Trigger: Provider determines that a patient needs to return to triage — most commonly due to incorrect or incomplete vitals. Preconditions: Provider has assumed the patient (UC-4 complete). Visit is in "in care" status. Provider holds the active lock on the record.

Main flow:

1. Provider opens the active patient chart and selects "Return to triage."  
2. System prompts for a reason note (required, free text).  
3. Provider confirms the action.  
4. System timestamps the return, records the reason note and provider ID, releases the provider lock, and updates visit status to "returned to triage."  
5. Patient card re-enters the triage queue.  
6. System sends an active in-app notification to the original triage volunteer, identifying the patient and displaying the provider's reason note.  
7. Provider's active patient view clears; provider may now assume a new patient from the care queue.

Alternate flows: Original triage volunteer is no longer active in the system → notification is sent to all triage volunteers currently active on the trip; any available triage volunteer may handle the re-entry.

Exception flows: Provider attempts to return a patient who has already been sent to pharmacy → action is not available once visit status is "awaiting pharmacy"; provider must use UC-12 or contact pharmacy directly.

Postconditions: Visit status is "returned to triage." Provider lock is released. Patient card is visible in the triage queue. Original triage volunteer or active triage team has been notified. Any SOAPP content entered by the provider is auto-saved, preserved on the record, and visible to the returning triage volunteer in read-only mode.

Business rules: Reason note is mandatory. Triage cannot edit the provider's partial SOAPP note — triage updates vitals only and re-checks the patient in. Re-entry vitals form is pre-populated with the original vitals; volunteer corrects specific fields; both original and corrected vitals entries are preserved on the visit timeline. Re-check-in after a return creates a new vitals entry on the same visit record rather than a new visit. A provider may not return a patient to triage more than once per visit without master admin review. The return event is recorded in the visit audit trail with timestamp, provider ID, and reason note.

Data requirements: Return reason note (free text), returning provider ID, return timestamp, visit status update. Notification payload to original triage volunteer — patient name, provider name, reason note. Pre-populated vitals on re-entry form — all fields from the original triage entry.

Open questions: None.

---

## **UC-15 — Volunteer role assignment**

Actor: Program Manager Trigger: A new trip is being configured, or a volunteer's role needs to be changed before or during a trip. Preconditions: Program Manager is logged in with trip configuration access. The trip record exists in the system. The volunteer account exists or needs to be created.

Main flow:

1. Program Manager navigates to the trip configuration view and opens the volunteer roster.  
2. Program Manager locates an existing volunteer account or creates a new one (name, email, temporary credentials).  
3. Program Manager assigns the volunteer to the trip and selects their role: Triage, Provider, Pharmacy, or Program Manager.  
4. System saves the assignment and makes the volunteer's role active for the trip.  
5. Volunteer receives an account notification email with login credentials and assigned role.

Alternate flows: Volunteer exists from a prior trip → Program Manager locates the existing account, assigns to the new trip, and sets role; no duplicate account created. Volunteer role changes during a trip → Program Manager updates the role assignment; change takes effect immediately on save; volunteer receives an active in-app notification and must log out and back in for the new role to apply. Volunteer serves multiple roles on different days → Program Manager updates the role assignment per day; system retains a history of role assignments per volunteer per trip.

Exception flows: Program Manager attempts to create a duplicate volunteer account (same email) → system detects the conflict and surfaces the existing account; Program Manager assigns the existing account rather than creating a new one. Role change attempted while volunteer has an active patient lock → system warns that the volunteer currently holds an active patient record; role change is blocked until the lock is released or a master admin overrides.

Postconditions: Volunteer account is active for the trip with the correct role assigned. Volunteer can log in and access their role-specific queue. Role assignment is recorded in the trip audit trail.

Business rules: Each volunteer has exactly one active role per trip day — multi-role access within a single day is not permitted in V1. Program Manager cannot create or modify master admin accounts — reserved for master admin only. Role assignments are trip-scoped and do not carry over to the next trip automatically. Volunteers receive an active in-app notification when their role is changed mid-trip, in addition to the logout/login requirement. All role assignment changes are logged with timestamp and the Program Manager's user ID.

Data requirements: Per volunteer — name, email, temporary credentials, assigned trip, assigned role, assignment timestamp, assigning Program Manager ID. Role change history retained per volunteer per trip.

Open questions: None.

---

## **UC-16 — View live operations dashboard (Command Center)**

Actor: Triage volunteer, Provider, Pharmacy volunteer, or Program Manager. Trigger: User wants situational awareness of the clinic's overall patient flow without owning a specific record. Preconditions: User is logged in and assigned to an active trip.

Main flow:

1. User selects "Command Center" from their role's navigation — available to every clinical role, not only the Program Manager.  
2. System displays a live, read-only, three-column board — Triage, Care, Pharmacy — each showing the patients currently waiting in that stage with name, age, gender, wait time, chief complaint, and (for Triage and Care) allergy and triage-acuity indicators.  
3. A metrics strip shows average wait time per stage, prescription fill rate, total completed visits, and current active-patient count.  
4. User returns to their own queue or active patient at any time; the Command Center does not lock or claim any patient record.

Alternate flows: A stage has no waiting patients → that column displays an empty state ("Queue empty") rather than being hidden.

Exception flows: None — this view is read-only and has no write actions to fail.

Postconditions: No data is written or modified. Any team member can check overall clinic status without interrupting whoever currently owns a given queue.

Business rules: Command Center is read-only for all roles — no role can assume, discharge, or edit a patient from this view; those actions remain scoped to UC-4, UC-9, and UC-13 respectively. Metrics are computed live from current queue and completed-visit state and are not persisted as a separate report in V1.

Data requirements: Per queued patient — name, age, gender, wait time, chief complaint, allergy flag, triage acuity level (Triage/Care columns only). Metrics — average wait per stage, prescription fill rate, completed-visit count, active-patient count.

Open questions: None.

