# Therapist Matcher / Find My Therapy

**Automation of Patient–Therapist Matching**
This project digitalizes and simplifies the process for patients seeking a fitting and available psychotherapist after receiving a prescription. By matching patients against professional criteria, availability, and individual needs, it reduces waiting times, increases transparency, and provides therapists with a clear digital interface to efficiently accept or decline new patients.

---

## 👥 Team Chocolat-Express – Psychotherapist Matching Process

### 🧑‍🔧 Team Members

| Name            | Email                                                                       |
| --------------- | --------------------------------------------------------------------------- |
| Jana Stojanovic | [jana.stojanovic@students.fhnw.ch](mailto:jana.stojanovic@students.fhnw.ch) |
| Christine Remy  | [christine.remya@students.fhnw.ch](mailto:christine.remya@students.fhnw.ch) |
| Daniel Fuhst    | [daniel.fuhst@students.fhnw.ch](mailto:daniel.fuhst@students.fhnw.ch)       |

### 💡 Coaches

* Andreas Martin
* Charuta Pande
* Devid Montecchiari

---

## 📝 Introduction

Finding an available and suitable psychotherapist is often a long and frustrating process for patients. Current workflows rely heavily on manual coordination, phone calls, and fragmented information across institutions and practitioners. This project addresses these issues by introducing a **digitized, rule-based matching process** that supports decision-making while keeping human oversight in place.

The system focuses on **efficiency, transparency, and fairness**, ensuring that patients receive suitable therapist suggestions while therapists retain control over their capacity and case acceptance.

---

## 🧩 Challenges of the Current Process

* High administrative burden for patients and providers
* Manual and repetitive data handling
* Limited transparency regarding therapist availability
* Long waiting times and inefficient follow-ups
* No standardized decision logic for matching

The main challenge was to define **relevant matching dimensions** (medical, logistical, and personal preferences) and implement them in a structured, automated decision-support tool.

---

## 🎯 Goal and Vision

**Goal**
To optimize the patient–psychotherapist matching process by implementing a digitized, rule-based workflow that supports faster and more reliable matching.

**Vision**
To provide patients with an easy-to-use platform delivering confident and transparent therapist suggestions, while enabling therapists to manage requests digitally and efficiently.

---

## 📦 AS-IS Process

### Description

The current (AS-IS) process is largely manual and fragmented. Patients typically contact multiple therapists individually, often without knowing availability or specialization fit in advance.

![As-Is Process](IMAGE BPMN OF CURRENT PROCESS)

### Roles Involved

**Internal**

* Administrative staff
* Coordination services

**External**

* Patients seeking therapy
* Licensed psychotherapists

### 📋 Summarized AS-IS Process

| Step | Description               | Comments                                    | Lane    |
| ---- | ------------------------- | ------------------------------------------- | ------- |
| 1    | Start: Request            | Patient initiates search for therapy        | Patient |
| 2    | Form Filling / Phone Call | Information provided manually               | Patient |
| 3    | Manual Matching           | Staff checks therapist fit and availability | Admin   |
| 4    | Feedback Loop             | Calls/emails until a therapist responds     | All     |

---

## ✨ TO-BE Process

The TO-BE process introduces automation and structured decision logic while maintaining transparency and control for all parties.

### Key Features

* Digital intake via form or service hotline
* Rule-based decision table for therapist matching
* Automated communication via APIs
* Clear acceptance/decline workflow for therapists
* Reduced administrative workload

![To-Be Process](https://github.com/DANIEL-FHNW/AS25_Chocolat_Express/blob/main/AS_IS_PROCESS.png)

---

## 🧾 Technologies

### Modules Used

| Module                 | Purpose             | Description                                               |
| -----------------------| ------------------- | --------------------------------------------------------- |
| Camunda Form       | Feature Input        | User Form to provide preferences|
| HTTP Connector             | Deepnote Integration| Sends data to deepnote via REST-API with GET, POST gateways                      |
 |Deepnote                  | API Gateways | Proiding API-Gateways for GET, POST-Routes for client and therpist |

**Camunda REST Endpoint**
`/engine-rest/process-definition/key/Process_0ad1ggy/tenant-id/mi25chocolat/start`

**Deepnote Notebook**
`https://deepnote.com/workspace/DHP25-244a274b-59d3-442f-b7ef-3d5d24503cee/project/chocolatexpress-fbdcce36-fd51-4cfb-8676-e6e544158098/notebook/cc5e3854092d45b89dd08990f5fce491?secondary-sidebar-autoopen=true&secondary-sidebar=agent`

---

## 🔁 Recap of the Integrated Flow

1. A person searches for a suitable therapist
2. The person either calls a service number or fills out a digital form
3. Form variables are sent to a preprocessing decision table

   * Variables are mapped to integer-based categories
   * Each category contributes to the final matching logic
4. A decision table determines a matching `therapist_id`
5. The matching result is returned via API
6. The patient confirms or declines the suggestion
7. The therapist is informed and can accept or reject the request
8. The process ends with confirmation or re-matching

---

## 🧮 Decision Model & Matching Logic

### Input Variables

All patient- and therapist-related variables are normalized and mapped into discrete categories to enable deterministic decision logic.

**Patient Variables**

| Input Variable   | Description                                                   | Type / Encoding                             |
| ---------------- | ------------------------------------------------------------- | ------------------------------------------- |
| therapy_setting  | Requested setting: individual or couple                       | Enum → Int (nbr_of_clients)                 |
| disease_category | Clinical focus: depression, anxiety, coupletherapy, addiction | Enum → Int (category 1–4)                   |
| weekday          | Desired appointment weekday (Mon–Fri)                         | Enum → Constant score (1)                   |
| daytime          | Preferred time of day: morning, afternoon                     | Enum → Int (daytimeDecision)                |
| gender           | Preferred therapist gender: female, male, non-binary          | Enum → Int (genderDecision 1–3)             |
| waiting_time     | Accepted waiting time: 1_week–4_weeks                         | Ordinal → Int (waitingTimeDecision −1 to 2) |

### Output Variable

**Therapist Variables**

| Output Variable | Description                        |
| --------------- | ---------------------------------- |
| Therapy_ID      | Selected therapist identifier/name |

---
**Derived Decision Layers**
The DRD decomposes matching into smaller, focused decisions, each producing intermediate variables consumed by downstream logic.​

| Decision ID / Name                              | Output Variable     | Depends On                                | Purpose                                                          |
| ----------------------------------------------- | ------------------- | ----------------------------------------- | ---------------------------------------------------------------- |
| Decision_TherapySetting / Individual or Couple  | nbr_of_clients      | therapy_setting                           | Encode 1 vs. 2 clients.                                          |
| Decision_DiseaseCategory / Kind of Therapy      | category            | disease_category                          | Map therapy type to category 1–4.                                |
| Decision_Gender / Gender Therapist              | genderDecision      | gender                                    | Encode therapist gender preference.                              |
| Decision_WaitingTime / Waiting List             | waitingTimeDecision | waiting_time                              | Score waiting time (2, 1, 0, −1).                                |
| Decision_Weekday / Day                          | day                 | weekday                                   | Normalize weekday to constant score 1.                           |
| Decision_Daytime / Daytime                      | daytimeDecision     | daytime                                   | Normalize time preference (1).                                   |
| Decision_0i5oonf / Availability                 | availability_score  | day, daytimeDecision, waitingTimeDecision | Combine schedule and waiting-time into availability score (1–3). |
| Decision_TherapistMatches / Which Therapy-Match | Therapy_ID          | All above                                 | Final therapist selection via decision table.                    |

## 🔁 Matching Logic

### 1. Normalization & Encoding

- **Therapy setting**
  - `therapy_setting` → `nbr_of_clients`
    - individual → `1`
    - couple → `2`

- **Disease category**
  - `disease_category` → `category`
    - Encoded as integer values `1–4`

- **Gender**
  - `gender` → `genderDecision`
    - female → `1`
    - male → `2`
    - non-binary → `3`

- **Waiting time tolerance**
  - `waiting_time` → `waitingTimeDecision`
    - very short / immediate → `2`
    - short → `1`
    - neutral → `0`
    - long tolerated → `-1`

---

### 2. Availability Scoring

- **Weekday preference**
  - `Decision_Weekday`
  - Normalizes calendar weekday preferences into a generic score: `day`

- **Daytime preference**
  - `Decision_Daytime`
  - Normalizes time-of-day preferences into `daytimeDecision`

- **Waiting time**
  - `Decision_WaitingTime`
  - Expresses tolerance or flexibility regarding waiting time

- **Combined availability**
  - `Decision_0i5oonf` *(Availability)*
  - Combines:
    - `day`
    - `daytimeDecision`
    - `waitingTimeDecision`
  - Output: `availability_score`
    - `3` → best availability match
    - `2` → good match
    - `1` → lowest configured positive match

---

### 3. Therapist Selection (Main Decision Table)

- **Decision:** `Decision_TherapistMatches`
- **Purpose:** Select the most suitable therapist

- **Inputs**
  - `nbr_of_clients`
  - `category`
  - `genderDecision`
  - `availability_score`

- **Logic**
  - Each rule represents **one therapist**
  - Rules define allowed combinations of:
    - number of clients
    - disease category
    - gender compatibility
    - availability score

- **Output**
  - `Therapy_ID` (e.g. therapist identifier or name)

- **Example therapists**
  - Dr. Jordan
  - Dr. Müller
  - Dr. Ruth
  - Dr. Goth
  - Dr. Biesel
  - Dr. Jung

- **Rule behavior**
  - If an input combination matches a rule’s filters  
    → the corresponding `Therapy_ID` is returned


---

### Final Decision Table (DMN)

**Decision Name:** `Select_Therapist`

**Inputs:**

* therapy_type
* modality


**Output:**

* `Therapy_ID` (therapist_id)

---

### Hit Policy Selection

**Chosen Hit Policy: `COLLECT (MAX)`**

**Rationale:**

* Multiple therapists may satisfy all hard constraints
* Each rule returns `(therapist_id, score)`
* `COLLECT (MAX)` selects the therapist with the **highest matching score**
* Deterministic, explainable, and scalable

**Tie-Break Strategy:**

1. Highest availability
2. Least recent assignment
3. Random fallback (optional)

---

### Example Decision Rule (Simplified)

!!!!!!!
---

### Why Rule-Based (Now)

* Full transparency for stakeholders
* Easy validation by domain experts
* Legally and ethically explainable
* Low data requirement

---

### Migration Path to ML

Once sufficient historical matching data is available:

* **Label (`Therapy_ID`)** becomes training target
* **Features:** all encoded patient variables
* **Models:** Logistic Regression → XGBoost
* **Output:** Probability-ranked Top-3 therapists

The DMN model can then act as a fallback or safety layer.

---

## 🚀 Process Improvements

| Challenge                | Solution                              |
| ------------------------ | ------------------------------------- |
| Manual data collection   | Google Forms with automated triggers  |
| Fragmented communication | Centralized BPMN process              |
| Slow matching decisions  | Decision tables with rule-based logic |
| Lack of transparency     | Structured process & digital tracking |

The improved process significantly reduces delays, errors, and manual workload while improving the overall user experience.

---

## 🔮 Future Steps and Opportunities

### Process Enhancements

* Direct therapist self-service portal for availability updates
* Automated synchronization of vacation and capacity data
* Patient and therapist feedback loops

### Future Outlook

With increasing data availability, the rule-based decision table could be replaced or augmented by **machine learning models**:

* Logistic Regression (transparent, small data friendly)
* XGBoost (higher performance for complex patterns)

The system could generate a **Top-3 matching score** instead of a single result.

---

## ⚙️ Operational Efficiency & Costs

* Reduced administrative effort
* Faster patient placement
* Better utilization of therapist capacity
* Scalable integration with existing IT infrastructures

---

## 🧑‍💻 Technologies Used

| Component    | Purpose                              |
| ------------ | ------------------------------------ |
| Camunda 7    | Business process orchestration       |
| BPMN 2.0     | Process modeling language            |
| Google Forms | Patient data intake                  |
| Deepnote     | API integration & ML experimentation |

---

## 📌 BPMN Process Overview

The BPMN model includes:

* Start Event (Patient Request)
* User Tasks (Form Input)
* Business Rule Tasks (Decision Tables)
* Service Tasks (API Communication)
* User Tasks (Therapist Decision)
* End Events (Match or Re-run)

---

**Project Status:** Prototype / Academic Project
**Context:** Digital Business Processes & Medical Informatics
