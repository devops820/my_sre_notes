# my_sre_notes
SRE Key Concepts

## Introduction

### Why measure service performance ?
- software is delivered continously and user expectation is higher than ever.
- customers expect high quality features whenever they want them.
- to support this reality, developers and operations teams, both, must work together to ensure their services are performing acceptably. How can they do this ? Enter SLO's.
- SLO's
  - sets service performance expectations
  - ensure teams take data driven decisions
  - balance reliability efforts with innovation
 
### Pre-req's
- Basic understanding of Service Oriented Architecture
- Basic understanding of Client-Server Architecture

## Service Level Indicators

### Defining Reliability
- Basic building block for defining reliability of a service (SLI)
- Its a level of service you provide interms of ratio of two numbers.
- It is recommended to think SLI ~ `'Good' Events/Total Events`
- When picking SLI's for a service
  - What makes users happy ?
  - What behaviours really matter to them ?
- To support answering the above questions
  - Define users
  - Identify critical activities
  - Draw an high level arch diagram of your systems components.
- **SLI Specification** - The assessment of a service outcomes that you think matters to the users, idependent of how it is measured. (`Measurement Agnoistic`)
- Example - Latency of a request in a web application. 1 SLI specification could be ratio of page requests that load in less than 100 ms to all requests.
- Within the SLI specification, you do not mention the specifics of indicator measurement. It is abstract. Measuring the assessment is reffered to as SLI Implementation.

### Implementing measurements
- **SLI Implementation** - An SLI specification and how to measure it.
- SLI's should be specific and measurable
- Implementation sources
  - Application Server Logs
  - Load balancer logs or monitoring
  - Black-box monitoring
  - Client-side instrumentation
- Example - If you want to measure the load time of a page on a website, you could use the any of the aforementioned sources to calculate the latency, each with its own pros and crons.

<img width="901" alt="sli_implementation_sources" src="https://github.com/devops820/my_sre_notes/assets/34048837/4984326f-0a95-4305-99a8-9216b373f340">

- If we chose server logs for your source of measurement.

<img width="586" alt="sli_implementation_source_server" src="https://github.com/devops820/my_sre_notes/assets/34048837/4efacc56-ac17-405f-821e-17e1012c9859">

- Pros
  - Highly Accurate Data
- Cons
  - Miss request that fail to reach the server
 
<img width="586" alt="sli_implementation_source_client" src="https://github.com/devops820/my_sre_notes/assets/34048837/95170230-061b-4ffc-b030-4be5116bbf20">

- If we chose to instrument the javascript loaded by each page, and report that back to a metrics aggregation platform.
- Pros
  - Accurately capture UX
- Cons
  - Need to modify the client code
  - Need infrastructure support

- When **chosing between various SLI Implemntations**, consider the following 3 things
  - Quality (How accurately the implementation captures the user experience)
  - Coverage (How broadly the experience of all customers is captured)
  - Cost
 
- You do not need to get your SLI completely correctly the first time you try. *Start Measuring Something*
