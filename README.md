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

## 🧾 Camunda Forms & Integration

### Modules Used

| Module        | Purpose             | Description                                               |
| ------------- | ------------------- | --------------------------------------------------------- |
| Google Sheets | Watch Rows          | Triggers Camunda process when a new form entry is created |
| HTTP Module   | Camunda Integration | Starts BPMN process via REST API                          |

**Camunda REST Endpoint**
`/engine-rest/process-definition/key/Process_0ad1ggy/tenant-id/25DIGIBP29/start`

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

## 🧮 Decision Table

The decision table implements the **therapeutic matching logic**, considering factors such as:

* Therapy type
* Modality (online / on-site)
* Availability
* Language
* Specialization constraints

![Overview Decision Table](DECISION TABLE IMAGE)

### Current Limitations

* Static rule definitions
* Manual updates of therapist availability
* Limited scalability with increasing complexity

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
