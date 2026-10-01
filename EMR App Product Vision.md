**PRODUCT VISION — Mission EMR** *Draft v1*

**Thesis** Existing EMRs are built for connected, permanent clinics with trained staff — this one is built for the field: offline-capable, role-specific, and fast enough to keep pace with a high-volume mission trip without training.

---

**Problem** Medical mission trips run on paper, memory, and improvisation. Care coordination between triage, providers, and pharmacy breaks down. Patient history disappears between trips. There is no continuity — for the patient or the team.

**Goal**

* *For clinicians:* Every person on the care team — triage, provider, pharmacist — knows exactly what they need to know, when they need to know it, without asking anyone else.  
* *For patients:* A record of their care exists, persists, and is accessible — regardless of whether they ever return to a clinic or see the same team again.

---

**Stakes** *These are not edge cases. In a field clinic, any of these can happen on any day.*

* A patient with a known allergy receives a contraindicated medication because the allergy wasn't visible at the point of dispensing.  
* A returning patient is treated as new — prior diagnoses, medications, and reactions are unknown to the provider.  
* The system fails mid-trip and all records for that day are lost, leaving no clinical documentation.

---

**Users & Stakeholders**

*Users*

* Volunteer clinicians (physicians, nurses, PAs, NPs) — high clinical skill, variable tech comfort, time-pressured  
* Triage — often non-clinical volunteers or RNs – may rotate roles across trips  
* Pharmacy volunteers – often run by voluntary clinicians or sometimes pharmDs (may be rotating roles across trips or different clinic days) who may be working side-by-side with non-clinical volunteers 

*Patients (stakeholders, not users)*

* Underserved populations, often without access to their own medical history or ongoing care  
* This system may be the only longitudinal record of their care that exists — it owes them persistence and dignity

**Environment**

* Field clinics in developing regions — unreliable or absent internet, variable power  
* High patient volume, fast pace — the system must move with the team, never block it  
* Mix of personal devices — phones, tablets, laptops; no guaranteed device standardization  
* New volunteers each trip — zero onboarding time, roles must be learnable in minutes

---

**Assumptions** *What must be true for this design to hold. If any of these break, revisit the brief.*

* At least one person per trip can set up and troubleshoot the system before patients arrive  
* Connectivity exists at some point during or after the trip, enabling data sync and record export  
* Care follows a consistent linear sequence: triage → provider → pharmacy (with exceptions, not as the norm)  
* Encounters are episodic — this system documents visits, it does not manage ongoing longitudinal care

---

**Core Use Cases** *Ordered by the natural sequence of a patient encounter.*

1. **Patient intake** — Triage volunteer registers or finds a patient, records vitals and relevant history, and assigns a triage acuity level so the provider has a complete, prioritized picture before the encounter begins.  
2. **History review** — Provider reviews all prior visits for a returning patient so care decisions are informed by history, not made blind.  
3. **Clinical documentation** — Provider documents the encounter and issues medication orders so there is a clear, auditable record of what was decided and why.  
4. **Medication dispensing** — Pharmacist fulfills and logs prescriptions so what was ordered and what was actually given are never ambiguous.  
5. **Record handoff** — At visit close, a complete summary is delivered to the patient and preserved in the system so the record outlives the trip.  
6. **Addendum** – Users must be able to add addendums if a correction needs to be made to a patient encounter or additional information needs to be added to a patient encounter. 

---

**First Principles** *When two good ideas conflict, these decide.*

* **Speed over completeness** — A form that gets filled out is better than a perfect form that gets skipped. When forced to choose, reduce friction. Incomplete data captured is better than complete data abandoned.  
* **One owner at a time** — A patient record is always owned by exactly one role. Clear handoffs are not a UX nicety — they prevent clinical errors caused by ambiguity about who is responsible.  
* **Records persist beyond the trip** — Data is never siloed to a single device or trip. What is recorded today must be accessible to the next team that sees this patient, even years later.  
* **Roles enforce clarity, not hierarchy** — Each role sees what it needs and nothing more. This is not about permission gatekeeping — it's about reducing cognitive load so each person can focus on their job.

---

**Out of Scope** *Deliberate, not accidental. These follow from the principles above.*

Billing & insurance · Appointment scheduling · Lab orders & results · Multi-clinic routing · Ongoing care management · Enterprise HIPAA compliance in the setting of the EMR is being developed by a non-profit organization based in the U.S. and the information will not be shared with any 3rd parties. 

---

**Signals of Success**

* Volunteers use it without asking how — role and workflow are self-evident  
* Returning patients are recognized and their history surfaces without effort  
* After the trip, a complete record exists and is accessible — nothing was lost

**Signals of Failure**

* Volunteers revert to paper mid-trip because the system slows them down  
* Duplicate patient records proliferate because search is too slow or unclear  
* Care team members verbally relay information that should be in the system

