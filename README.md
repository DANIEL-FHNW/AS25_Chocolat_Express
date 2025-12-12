# Therapist Matcher/Find My Therapy? todo

**Automation of Patient-Therapist Matching.** This tool digitalizes and simplifies the process for patients seeking a fitting and available therapist after receiving a prescription. By matching patients against professional criteria, capacity, and individual patient needs, it ensures a more efficient search for the patient and provides therapists with a clear digital interface to efficiently accept or decline new patients.

# Team Chocolat-Express: Psychotherapist-Matching-Process

# 🧑‍🔧 Team Members

| Name                    | Email                                        |
|-------------------------|----------------------------------------------|
| Jana Stojanovic         | jana.stojanovic@students.fhnw.ch             |
| Christine Remy          | christine.remya@students.fhnw.ch             |
| Daniel Fuhst            | daniel.fuhst@students.fhnw.ch                |


# 💡 Coaches

- Andreas Martin
- Charuta Pande 
- Devid Montecchiari

# 📝 Introduction



# 🧩 Challenges of the Process
The overall challenge is to implement a digitized decision-supporting tool and the definition of possible asked dimensions for persons looking for a therapy.
  
# 🎯 Goal and Vision
The overall goal of the project is to optimize a person-psychtherapist-matching process by implementing a digitized, rule-based
matching process. Both participants, persons looking for the right therapist and the therapist should benefit from the matching process by saving time.
The vision is to offer persons looking for the right therapy a easy to use platform for best matches and confident suggestions by the process.

# 📦 AS-IS Process
* 
 

![As-Is Process]()

**Roles involved in the process**:

**Internal**
-

**External**:
-
  
## 📋 Summarized Process Description

| Process Step | Description                        | Comments                                                                            | Lane                            |
|--------------|------------------------------------|-------------------------------------------------------------------------------------|---------------------------------|
|      1       | Start: Request                     | Start of the overall process: Request for a matching therapist                      |                                 | 
|      2       | Form-Filling


  
# ✨ TO-BE Process


**Key Features**
-
  

#[To-be Process]

![image](https://github.com/DANIEL-FHNW/AS25_Chocolat_Express/blob/main/AS_IS_PROCESS.png)


# Camunda Form

Links:


| Module                    | Purpose                                        | Description |
|-------------------------|----------------------------------------------|------------------|
| Google Sheets           |Watch Rows |Triggers with each new order and starts Camunda Process once an order is entered via Google form
| HTTP Module             | Camunda Integration | https://digibp.engine.martinlab.science/engine-rest/process-definition/key/Process_0ad1ggy/tenant-id/25DIGIBP29/start |

**Example POST endpoint:**

https://digibp.engine.martinlab.science/engine-rest/message

![Post Endpoint](Pictures/Usage_postman.png)

# Recap of the integrated Flow

1. Person wants to find a matching therapist
2. Based on preference, Person can either call a service-number OR fill out a form with given dimensions to look for a psychotherapist
3. Variables from the form are transmitted to a decision table
  -takes variables and transforms them into integer categories
  -per single variable a integer is transmitted to the final decision table
  -decision table consists of implemented patient therapeutic-logic
4. Single variable "therapists_id" with matching Therapists-Name ist generated
5. Matching-Result is transferred via API to the requesting person
6. Person transfers response back (API)
7. Psychotherapist is informed about the possible appointment
8. 

# Decision Table

![Overview Decision Table]()

**Current Limitations**

Currently, Buddy may face challenges with highly specific inquiries, such as incorrect order numbers, leading to response loops. As Buddy is still a test model and not yet live, this is acceptable within the project’s scope. 
To mitigate the risk of misinformation, Buddy is designed with restricted pathways, ensuring that only valid paths are triggered and preventing inaccurate responses. Loosening those restrictions and adding more scenarios will make the chatbot easily scalable to cover additional services (e.g. delivery updates, FAQs and more) in the future.

Future updates will focus on refining phrasing and expanding language capabilities. At the moment the chatbot operates in English only when workflows are triggered.

# Process Improvements
In our analysis of the original order-to-shipment process, we identified several key challenges: fragmented communication, manual data handling, inefficient follow-ups, and a lack of automation. These issues led to delays, errors, and inconsistent customer experiences. To address these pain points, we designed and implemented a digitalized and automated solution. Our improved process reduces manual workload, increases transparency, and speeds up response times, all while maintaining flexibility and human oversight where necessary.

| Challenge               | Solution                                        |
|-------------------------|----------------------------------------------|
| Manual PO checks        |Google Forms + automated trigger |



# 🔮 Future Steps and Opportunities
While the current process delivers major improvements in automation, communication, and efficiency, there are several areas where further enhancements can provide even greater value and scalability.

**Process Enhancements**
- Optimize


**Operational Efficiency and Costs**

- Time-Savings Potential:
- Integration with possible therapist-IT-infrastructure:


# Technologies Used
To tackle the current challenges of the AS-IS Process the following technologies have been used and applied:

| Component               | Purpose                                        |
|-------------------------|----------------------------------------------|
| Camunda 7               | Modeling the business process                |
| BPMN 2.0                | Used Modeling-Language                       |
| Deepnote                | API-Gateways to send Mails                   |
| Deepnote                | ML-Model Training, Deployment                |
