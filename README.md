# Therapist Matcher/Find My Therapy? todo

**Automation of Patient-Therapist Matching.** This tool digitalizes and simplifies the process for patients seeking a fitting and available therapist after receiving a prescription. By matching patients against professional criteria, capacity, and individual patient needs, it ensures a more efficient search for the patient and provides therapists with a clear digital interface to efficiently accept or decline new patients.

# Abstract

With this project, a Machine Learning based therapist-matching process has the goal to handle a patient request with the outcome to get a proposal to a initial therapist meeting back.



# People Involved 

## Project Team / Authors
| Name          | Email                  |
|---------------|------------------------|
| Daniel Fuhst| jdaniel.fuhst@students.fhnw.ch |
| Jana Stojanovic | jana.stojanovic@students.fhnw.ch |
| Christine Remy| christine.remy@students.fhnw.ch |

## Supervisors
| Name          | Email                  |
|---------------|------------------------|
| Andreas Martin | andreas.martin@fhnw.ch |
| Charuta Pande  | charuta.pande@fhnw.ch |
| Devid Montecchiari | devid.montecchiari@fhnw.ch |



# Description of the Project

This project aims to implement a digitized patient-psychotherapist matching process for people looking for an initial meeting.

## Motivation



![image](https://github.com/user-attachments/assets/d546845f-91d6-41a4-9faa-466a607c3a4a)


## Project Goal

The project-goal is to **accelerate patient-psychotherapist satisfaction** by introducing:
---


## Current (AS-IS) Process

![As-Is-Process](https://github.com/user-attachments/assets/5a498502-e5b0-42ee-900f-ded90232ac1a)


## Challenges and Requirements to be adressed with the To-Be Process
| Challenge          | Requirement                  |
|---------------|------------------------|
| Feedback is collected **informally** via email, phone, or paper | Standardizing how feedback is analyzed and handled internally |
| Processing is **inconsistent and undocumented** | Documentation of feedback and Compliance with ISO-Standards |
| There is **no central system or responsible role**  | Clear process ownership and traceability |
| There is **no structured categorization** or trend reporting| Standardized categorization |
| **No standardized escalation** exists for unresolved or critical feedback | Ensuring every feedback is recognized, proceed and answered |
| **Updates or Closure is not guaranteed** for the Feedback Givers  | Automated follow-ups and stakeholder communication |

--- 

# To-Be Process
The **To-Be process** operationalises SVK’s feedback management as an executable BPMN 2.0 model. The whole workflow is orchestrated end-to-end by a **Camunda 7** BPMN engine. Its logic can be summarised in five stages:

1. [**Intake**](#intake) – A person submits a form via Camunda-Form with mandatory fields to search for a potentially psychotherapist
2. [**Classification**](#classification) – A classification Machine Learning Model "Logistic Regression" takes the input values from the form as "features" to be trained 
3. 


![To-Be Process Model](Readme%20-%20Appendix/Pictures/To-Be%20Process%20Model.png)

## Intake
A new case begins when a stakeholder submits feedback via a **Jotform** which is going to be embedded on the SVK website. The submission payload is forwarded through a **Make** scenario  ([see further details](Readme%20-%20Appendix/Make%20Scenarios.md)) which instantiates a Camunda process instance; the Jotform *submission ID* (created by Jotform on submission) serves as the **business key**.

![Initial data flow](Readme%20-%20Appendix/Pictures/DataFlow_initialSubmission.png)

Immediately after instantiation, the feedback is persisted in SVK’s central data store - an Excel workbook on SVK's local server - and assigned the status **`open`** (see [Process Variables and Database](Readme%20-%20Appendix/Process%20Variables%20and%20Database.md) for further details to statuses). A confirmation e-mail is dispatched to the submitter.

![Database save + confirmation](Readme%20-%20Appendix/Pictures/Dataflow_initialSubmissionSaveConfirm.png)

## Classification
The case is then routed to the newly created role **Feedback Master** (see [Role Definitions.md](Readme%20-%20Appendix/Role%20Definitions.md)). The Feedback Master works exclusively in the **Camunda Tasklist**, where a Camunda form displays all submission details.

![Classification task](Readme%20-%20Appendix/Pictures/Dataflow_classification.png)

During classification the Feedback Master records three attributes:  

* `feedbackType` – semantic category (positive, negative, suggestion)  
* `urgency` – low, medium, or high  
* `impactScope` – specific, small, large

Please see ([Classification Guardrails.md](Readme%20-%20Appendix/Classification%20Guardrails.md)) for additional information.

If essential information is missing, the Feedback Master sets the Boolean `needsClarification`.  
She or he also indicates - via `immediateAction` - whether the feedback can be resolved immediately; otherwise the responsible department is selected from a drop-down menu which contains SVKs department names.

After these entries, the record is re-written to the database. An **inclusive gateway** checks `urgency`; if the value is *high*, an escalation e-mail is sent to the CEO.  

An **exclusive gateway** then directs the token either to a *Clarification* sub-flow or to scenario determination, depending on `needsClarification`.

![Save → CEO alert → decision](Readme%20-%20Appendix/Pictures/DataFlow_saveCEOqueryDecision.png)

## Clarification
If clarification is required, Camunda generates a new task for the Feedback Master to phrase a query to the submitter.  
The query text is added to the database and the status changes to **`clarification`**.  
A pre-filled Jotform - including the original submission and the query - is generated, and the submitter receives an e-mail with the link.

![Querying the submitter](Readme%20-%20Appendix/Pictures/DataFlow_querying.png)

Camunda then waits at a *Receive Task* for the supplementary submission. A reminder is e-mailed after **7 days**; if no response arrives within **14 days**, the case is marked **`withdrawn`** and the process instance is terminated.

![Receive supplementary data](Readme%20-%20Appendix/Pictures/DataFlow_ReceiveQueryAnswer.png)

When the submitter answers, another Make scenario ([see further details](Readme%20-%20Appendix/Make%20Scenarios.md)) correlates the message to Camunda.

![Supplementary form submission](Readme%20-%20Appendix/Pictures/DataFlow_supplementarySubmission.png)

The query and the reply are appended - timestamped - to the process variable `feedbackText`. Control returns to the **Classify Feedback** user task; the sub-flow may loop until all clarifications are complete. The final classification is submitted when  `needsClarification` is cleared, allowing the main flow to continue to define the appropriate scenario for the feedback.

## Scenario Handling & Closure
A **Business Rule Task** evaluates the classified variables and outputs one of four predefined handling scenarios (see [Feedback Scenarios.md](Readme%20-%20Appendix/Feedback%20Scenarios.md) for further details). An exclusive gateway routes accordingly:

*Scenario 1 and 4 – Non-critical items*  
Status is set to **`review-board`**; the submitter receives an acknowledgement e-mail (gratitude in Scenario 4, processing notice in Scenario 1). The item is placed on the agenda of the bi-weekly **Feedback Review Board**.

![Scenario 1 & 4 path](Readme%20-%20Appendix/Pictures/Dataflow_scenario1Scenario4.png)

*Scenario 2 – Department measure required*  
A **Department Measure Documentation** Jotform is pre-populated with `feedbackText` and the submitters contact data (provided with the initial submission form). An e-mail with the form link is sent to the department selected earlier.  
If no response is received within **3 days**, Camunda sends cyclic reminders (3-day interval) until submission arrives. If the department does not respond, feedback review board will get aware of the case.

![Receive department measure](Readme%20-%20Appendix/Pictures/DataFlow_ReceiveDepartmentMeasureForm.png)

As soon as the departments responsible person submits the form, the form values will again be posted to the camunda workflow engine with the measure documentation by another Make scenario ([see further details](Readme%20-%20Appendix/Make%20Scenarios.md)).

![Scenario 2 & 3 path](Readme%20-%20Appendix/Pictures/Dataflow_scenario2Scenario3.png)

*Scenario 3 – Immediate action by Feedback Master*  
A follow-up user task prompts the Feedback Master to document the actions taken directly.

For Scenarios 2 and 3 the documented measures are persisted to the database, and the submitter is informed that the feedback has been resolved.

## Feedback Termination and Lifecycle Management
Throughout the lifecycle the Feedback Master and Review Board members can monitor the case in the dedicated **Feedback Manager Web-App**.  

The landing page provides a dashboard and lists all feedback items, grouped by their status.

![Feedback-Manager dashboard](Readme%20-%20Appendix/Pictures/webapp.png)

Selecting a row opens a detailed view of the chosen feedback.

![Detailed feedback view](Readme%20-%20Appendix/Pictures/webappDetail.png)

In this view the Feedback Master can set a case to **`terminate`** (e.g., when the submitter withdraws the issue) by clicking the red button.
The web-app then correlates a terminate message to Camunda; the workflow instance ends and the database status becomes **`cancelled`**.

![Event Sub Process: Termination](Readme%20-%20Appendix/Pictures/DataFlow_Termination.png)

The detailed view also allows to record measures taken and grant final approval, which updates the status to **`complete`**.


## Further Documentation  

For implementation details that exceed the scope of this overview, consult the companion documents listed below. Each file resides in *25DIGIBP1/Readme – Appendix/* and expands a specific aspect of the solution.
Use the links below to open each file directly:

| File Name | Scope |
|------------|-------|
| [**Classification Guardrails.md**](Readme%20-%20Appendix/Classification%20Guardrails.md) | Decision criteria for `feedbackType`, `urgency`, and `impactScope`. |
| [**Feedback Scenarios.md**](Readme%20-%20Appendix/Feedback%20Scenarios.md) | Rationale and routing logic of the DMN *Define Scenario* table. |
| [**Jotform.md**](Readme%20-%20Appendix/Jotform.md) | Public URLs, and cloning instructions. |
| [**Make Scenarios.md**](Readme%20-%20Appendix/Make%20Scenarios.md) | Description of the Make blueprints and cloning instructions. |
| [**Process Variables and Database.md**](Readme%20-%20Appendix/Process%20Variables%20and%20Database.md) | Complete list of process variables, database schema, and status semantics. |
| [**Python (Service Workers and Web App).md**](Readme%20-%20Appendix/Python%20(Service%20Workers%20and%20Web%20App).md) | Mapping of each external-task topic to its Python script with a functional summary and description of the web app. |
| [**Role Definitions.md**](Readme%20-%20Appendix/Role%20Definitions.md) | Formal responsibilities and authority boundaries for Feedback Master and Review Board. |
| [**Docker.md**](Readme%20-%20Appendix/Docker.md) | Guide on how to perform the local deployment. |

These sub-documents provide further insights required to replicate, maintain, or extend this solution.




# Deployment  

The solution can be installed in three logical layers - Camunda, integration artefacts (JotForm + Make), and Python workers.  
Wherever further detail is required, links point to the dedicated README Appendices in *Readme – Appendix/*.

---

## 1 Camunda Engine  

1. **Provision Camunda 7** on an on-premise server and confirm that `/engine-rest` is reachable.  
2. **Deploy Camunda artefacts** (`TOBE_Technical_Model.bpmn`, `DMN.dmn`, and the three embedded Camunda forms  
   (`classificationForm.form`, `queryForm.form`, `feedbackMasterMeasureDocumentationForm.form`). .  
3. **Tenant ID**  
   * All external messages reference the Feedback Master’s tenant (default `25DIGIBP12`).  
   * If Camunda administrators and the Feedback Master use separate tenants, open each user task in the *Feedback Master* lane and set the **User Assignment → Tenant ID** property accordingly.  
4. **Classification form dropdown** — add every department name in the `forwardToDepartment`.

---

## 2 Integration Artefacts  

| Step | Action | Reference |
|------|--------|-----------|
| **Clone JotForms** | Import the three public forms into SVK’s JotForm workspace; do **not** alter field names. | [Jotform.md](Readme%20-%20Appendix/Jotform.md) |
| **Import Make blueprints** | Upload the three JSON files, insert the JotForm API key in each *watchForSubmissions* module, select the cloned form, and configure the Camunda REST URL plus `tenantId` (Feedback Master) in the HTTP request module. | [Make Scenarios.md](Readme%20-%20Appendix/Make%20Scenarios.md) |

---
## 3 Python Workers

Full image and configuration details:  
[Docker.md](Readme%20-%20Appendix/Docker.md) · [Python (Service Workers and Web App).md](Readme%20-%20Appendix/Python%20(Service%20Workers%20and%20Web%20App).md)

---

# Testing (University / Coaches)  

A test environment is hosted in a DeepNote project so reviewers can evaluate the full workflow without installing local infrastructure.  
Camunda artefacts already run on the provided Camunda Enginge on Tenant **`mi25chocolat`**, your only task is to execute the Python workers (and later the web app) from DeepNote.

## 1 Testing-Environment Setup  

| Step | Action |
|------|--------|
| 1 | <a href="https://deepnote.com/workspace/Daniel-4d5b9353-033c-491f-89f6-fc9d81c24a82/project/TherapistMatcher-e5b8e8da-20ef-417a-abb6-fa3bb3ab0c64/notebook/Notebook-1-a01a0767f8094e619010beeaeb6b7c65?secondary-sidebar-autoopen=true&secondary-sidebar=agent">**Open Deepnote project**</a> provided by the team|
| 2 | In the sidebar, select **`Launcher`** and run the first script (import script).<br>Then run the worker script to launch all Python external-task workers. |
| 3 | Keep this kernel running while you execute the functional test below. |

> **Deepnote limitation:** only **one** kernel can run at a time. You will later stop the workers and start the web-app script.

## 2 Functional Test Walk-Through  

1. **Submit feedback** — open the public JotForm <a href="https://form.jotform.com/251255809932058">“Initial Feedback”</a>, fill in your own e-mail, and press *Submit*.  
2. **Classify** — log into Camunda Cockpit (tenant `mi25chocolat`), open the new task, and complete the classification form.  
3. **Clarification loop** (conditional) — check your mailbox, follow the Supplementation link, answer the query, and submit.  

5. **Stop the worker notebook**  
6. **Launch web app** — Run the single cell for the web app, and click the displayed URL below.  



