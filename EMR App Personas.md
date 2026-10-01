# **Mission EMR — User Personas**

---

## **1\. Triage Volunteer**

**Role:** Entry point of every encounter — builds the picture the provider walks into 

**Workflow stage:** Clinical

### **Clinical background**

* **Who they are:** Non-clinical volunteer to RN — background varies widely and cannot be assumed; may have no prior clinical exposure at all  
* **Role consistency:** May rotate into this role from another role across trips or even across days within the same trip  
* **Key implication:** The system cannot rely on clinical judgment at this stage — clarity and guided input matter more than flexibility

### **Tech profile**

* **Comfort with software:** Low to medium — cannot assume digital fluency; role must be learnable in minutes with no training  
* **Device:** Phone or tablet most likely — no guaranteed device standardization across volunteers  
* **Prior EMR exposure:** Usually none — this is often their first encounter with any clinical software

### **What they own in the workflow**

* **Owns:** Patient identification, vitals, chief complaint, preliminary history, and handoff to the provider  
* **Hands off to:** Provider — everything captured at triage becomes the foundation of the clinical encounter  
* **Receives from:** No upstream role — this is the first touch; the patient arrives with nothing in the system

### **What they need from the system**

* **Must see:** Whether the patient has been seen before, their prior history at a glance — including chronic conditions already on file, which stay checked from visit to visit rather than being re-asked — and any flags that need confirmation before handoff  
* **Must capture:** Identity, vitals (BMI is calculated automatically once weight and height are entered), known allergies, chief complaint, a triage acuity level, maternity status and date of last period for female patients, chronic-condition history, current medications, and family/social history — enough for the provider to begin without asking the patient to repeat themselves  
* **Should not see:** Clinical documentation, prescriptions, and pharmacy activity — information outside their role increases cognitive load without benefit

### **Operating conditions**

* **Pace & volume:** High volume and fast-moving — the system must keep pace with the team, never slow it down  
* **Physical environment:** Field clinic setting — standing, moving, variable lighting and noise; not a desk environment  
* **Connectivity:** Requires internet access to register patients and sync records across the care team

### **Failure modes to design against**

* **Allergy not captured:** A contraindicated medication is prescribed and dispensed because the allergy was never in the record  
* **Returning patient treated as new:** Prior diagnoses, medications, and reactions are unknown to the provider because a duplicate record was created  
* **Incomplete handoff:** Chief complaint or history not captured — provider must re-gather from the patient, breaking the intended flow  
* **Triage level misjudged:** The acuity level assigned at check-in is the only prioritization signal in the queue — an inaccurate assessment can leave a higher-acuity patient waiting behind lower-acuity ones, since the queue itself does not reorder by level

### **Signal of success**

* **Provider starts without asking:** The provider walks into the encounter with everything they need — no follow-up questions back to triage  
* **Returning patients recognized:** History surfaces immediately — the volunteer confirms or updates rather than re-entering everything from scratch  
* **Chronic conditions carry forward:** Once documented, they don't need to be re-asked at every visit

---

## **2\. Provider — Physician / Nurse / PA / NP / Medical Student**

**Role:** Owns the clinical encounter — highest cognitive load role in the workflow 

**Workflow stage:** Clinical

### **Clinical background**

* **Who they are:** Licensed clinicians — MD, DO, RN, PA-C, NP — and medical students working under direct supervision of a licensed clinician; clinical skill is high across the board  
* **Supervision context:** Medical students may be documenting or ordering under a supervising clinician's authority — the record must reflect who bears clinical responsibility  
* **Key implication:** Tech comfort varies widely across this group; the system must not assume familiarity with software even when clinical confidence is high

### **Tech profile**

* **Comfort with software:** Variable — high clinical skill does not correlate with tech comfort; the range within this persona is the widest of any role  
* **Device:** Laptop or tablet most common for documentation; may shift to phone when moving between patients  
* **Prior EMR exposure:** Mixed — some bring ingrained habits from other systems; others arrive with none; neither should be assumed

### **What they own in the workflow**

* **Owns:** The clinical encounter — assessment, documentation, treatment decisions, and what gets sent to pharmacy  
* **Hands off to:** Pharmacy if prescriptions are written; may close the visit directly if no pharmacy step is needed  
* **Receives from:** Triage — vitals, chief complaint, preliminary history, and confirmed allergies must be complete before the encounter begins

### **What they need from the system**

* **Must see:** The patient's full prior history, current vitals and any concerning values (including automatically calculated BMI), confirmed allergies, chronic-condition flags and mother's name from the patient record, a pregnancy indicator where applicable, the triage acuity level, and the chief complaint, past medical history, current medications, and family/social history captured at triage  
* **Must capture:** Clinical assessment and plan, any medications, vaccines, or resources ordered (ultrasound, ECG, labs, etc.) during the encounter, prescriptions for the patient to take home, and corrections to triage-entered vitals when needed  
* **Should not see:** Pharmacy fulfillment details and administrative functions — anything outside the clinical encounter adds noise during a time-pressured visit

### **Operating conditions**

* **Pace & volume:** Time-pressured and frequently interrupted — must context-switch between patients; documentation must never block care delivery  
* **Physical environment:** Moving between exam areas; variable setup; not always seated or stationary when documenting  
* **Connectivity:** Requires internet access to view history, receive patients from triage, and hand off to pharmacy

### **Failure modes to design against**

* **History not reviewed:** A returning patient is treated as new — prior diagnoses, medications, and reactions are unknown, leading to redundant or contradictory care  
* **Allergy not visible at prescribing:** A dangerous order is written because the allergy was not surfaced at the moment of the decision, not elsewhere in the chart  
* **Documentation lost:** No clinical record exists for the encounter — the patient's care is undocumented and unrecoverable

### **Signal of success**

* **No verbal relay needed:** Closes the encounter and hands off to pharmacy without verbally communicating anything that should have been in the record  
* **History informs decisions:** Prior visits and allergies visibly shape the encounter — the record is being used, not ignored

---

## **3\. Pharmacy Volunteer**

**Role:** Most compositionally variable role — last owner before the record is finalized 

**Workflow stage:** Clinical

### **Clinical background**

* **Who they are:** A PharmD or clinical volunteer may lead the station; non-clinical volunteers assist alongside them — both present at the same time  
* **Role consistency:** Composition can change across trips and across clinic days within the same trip — the system cannot assume medication expertise is always present  
* **Key implication:** Design for the least-clinical person at the station; safety-critical information must be visible without requiring clinical judgment to find it

### **Tech profile**

* **Comfort with software:** Variable — non-clinical volunteers may have very low tech comfort; the workflow must be explicit and step-driven  
* **Device:** Phone or tablet; a shared device at the dispensing station is common  
* **Prior EMR exposure:** Rarely; clinical leads may have some, non-clinical assistants almost certainly do not

### **What they own in the workflow**

* **Owns:** Reviewing and fulfilling prescriptions, documenting what was dispensed, and closing the visit — the record ends here  
* **Hands off to:** Completed — closing the visit finalizes the record and triggers the patient's visit summary  
* **Receives from:** Provider — prescription requests and the clinical notes that give context for substitution decisions or addenda

### **What they need from the system**

* **Must see:** All prescriptions, allergies prominently displayed before any dispensing action, patient identity including chronic-condition flags and a pregnancy indicator where applicable — both relevant to dispensing safety — and read access to the full clinical SOAPP note (not only the assessment and plan) for context when a substitution or addendum is needed  
* **Must capture:** What was dispensed and how — including any deviations from what was prescribed and the reason for them  
* **Should not see:** Editing access to clinical notes — they can read for context but the record of what the provider decided is not theirs to change

### **Operating conditions**

* **Pace & volume:** High volume; multiple prescriptions per patient; non-clinical volunteers need clarity at every step — ambiguity leads to dispensing errors  
* **Physical environment:** Standing at a dispensing area; hands may be occupied; shared and often crowded workspace  
* **Connectivity:** Requires internet access to receive patients from the care team and to finalize and sync the completed record

### **Failure modes to design against**

* **Allergy invisible at dispensing:** A contraindicated medication is given — the single highest-stakes failure in the entire system; allergy visibility at this stage is non-negotiable  
* **Substitution undocumented:** What was actually dispensed differs from what was prescribed with no record — future teams have no accurate medication history  
* **Visit closed with gaps:** Not all prescriptions addressed before the record is finalized — incomplete care with no audit trail

### **Signal of success**

* **No verbal relay to provider:** Fulfills or resolves every prescription and closes the visit without calling back to the provider for clarification  
* **Complete dispensing record:** Every prescription has a documented outcome — including any substitutions or addenda — before the visit closes

---

## **4\. Program Manager**

**Role:** Manages users and trips — not the system itself **Workflow stage:** Administrative

### **Background**

* **Who they are:** A recurring volunteer or non-profit staff member — not a clinician, not a technologist; they know how trips run operationally  
* **Organizational knowledge:** Institutional memory holder — understands volunteer roster patterns, role expectations, and how the org structures trips  
* **Key implication:** Needs enough control to fully prepare a trip without needing to escalate to the master admin for routine setup

### **Tech profile**

* **Comfort with software:** Medium — comfortable with web-based tools; not a developer; should not need technical knowledge to do their job  
* **Device:** Laptop primarily; pre-trip setup happens in a normal working environment, not the field  
* **Prior EMR exposure:** Unlikely; familiar with the org's own processes but not clinical software conventions

### **What they own in the workflow**

* **Owns:** Creating and configuring trips, managing volunteer accounts, assigning roles, and maintaining the roster before and during a trip  
* **Hands off to:** The clinical team — the system must be fully ready before the first patient arrives  
* **Receives from:** Master admin for system availability; volunteers who need access corrections or role adjustments

### **What they need from the system**

* **Must see:** Trip list, volunteer roster, role assignments, and the status of user accounts — enough to confirm everything is ready before a trip begins  
* **Must capture:** Trip details, volunteer accounts, and role assignments — the configuration that makes the clinical workflow possible  
* **Should not see:** Patient records, clinical documentation, and system-level configuration — those belong to the clinical team and the master admin respectively

### **Operating conditions**

* **Pace & volume:** Pre-trip setup is the critical window; during-trip access is exception-handling only — not a high-frequency user of the system  
* **Physical environment:** Office or remote; stable conditions — this role operates before the field environment begins  
* **Connectivity:** Requires reliable internet; admin configuration is not a field task

### **Failure modes to design against**

* **Wrong role assigned:** A volunteer cannot access their queue on trip day — discovered at the worst possible moment with no time to fix it  
* **Account not created:** A team member is locked out when patients arrive — one missing account can stall an entire stage of care  
* **Trip not configured:** The clinical team arrives to a system that isn't ready — everything breaks from the first patient

### **Signal of success**

* **Zero trip-day access issues:** Every volunteer reaches their correct role in the system before the first patient is seen  
* **No escalation to master admin:** All trip and user configuration was handled within their own access — no system-level intervention needed

---

## **5\. Master Admin**

**Role:** Full system rights — operates outside the clinical workflow **Workflow stage:** System

### **Background**

* **Who they are:** A technically capable staff member or contractor at the non-profit — may be a developer, IT lead, or highly technical ops person; small set of people  
* **Technical knowledge:** Comfortable with system configuration, permissions architecture, and data management — this is their domain  
* **Key implication:** Their mistakes are high-stakes and potentially irreversible — the system must make destructive actions visible and hard to do accidentally

### **Tech profile**

* **Comfort with software:** High — expects administrative interfaces to be precise, clearly labeled, and unambiguous about the consequences of actions  
* **Device:** Laptop or desktop; stable working environment with full screen availability  
* **Prior EMR exposure:** Possible but not required — their focus is system integrity, not clinical workflow familiarity

### **What they own in the workflow**

* **Owns:** Full system configuration — roles, permissions, data structure, integrations, and overall system health  
* **Hands off to:** Program manager once the system is live and ready to be configured for trips  
* **Receives from:** No upstream dependency — may respond to escalations from program managers when issues exceed their access

### **What they need from the system**

* **Must see:** All system settings, all user accounts across all trips, and a clear audit trail of changes — who changed what and when  
* **Must control:** Role definitions, permission architecture, system-wide settings, and anything that affects how data is structured or accessed  
* **Key principle:** Nothing is restricted from view — but the clinical workflow should be visually and structurally separated to prevent accidental interference with live trip data

### **Operating conditions**

* **Pace & volume:** Deliberate — operates pre- and post-trip; changes are high-stakes, not high-speed; time is available to think before acting  
* **Physical environment:** Office or remote; stable and distraction-minimal — this is not a field role  
* **Connectivity:** Requires reliable internet; system administration is not a field task

### **Failure modes to design against**

* **Role misconfiguration:** Clinical volunteers see wrong data or lose access mid-trip — no clean recovery once patients are in the system  
* **Destructive action without warning:** A configuration change causes data loss or breaks a live trip — irreversible without a clear audit trail and confirmation step  
* **Permission error:** Patient records are exposed to the wrong role — a trust and dignity failure for the patients this system exists to serve

### **Signal of success**

* **No mid-trip escalations:** No one contacts the master admin during a trip for access or configuration issues — the system was right before patients arrived  
* **Full audit trail:** Every configuration change is attributable — nothing was changed silently or without a record of who did it

