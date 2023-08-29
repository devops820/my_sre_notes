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

### Common Measurements

Think about your system abstractly and segment it into pieces based on common component types. Typically components of a system fall into one of the three categories. 

- Request driven systems (user facing serving systems. eg:- e-commerce applications)
- Big data systems (data processing pipelines)
- Storage systems

a. Request driven systems are the type of systems where user creates some type of an event and expect a response. This could also be a REST ful service, where the user interacts with a browser or an API used by mobile applications.

**Request Driven SLIs** 
1. Availability - Could we respond to the request
2. Latency - How long did it take to respond to the request
3. Throughput - How many requests can be handled over a set period of time

b. Big data systems take input, mutate it, and store the result. Examples :- Report generation services, which take in data from many sources in order to generate reports. As well as video processing systems, that convert video from one format to another. Big data systems lend themselves to the following SLI types.

**Big Data SLIs**
1. Throughput - How much data is being processed
2. End-to-End Latency - How long does it take for data to progress from ingestion to completion.

c. Storage systems accept data and store it for later use.

**Storage SLIs**
1. Latency - How long does it take to read/write data ?
2. Availability - Can we access the data on demand ?
3. Durability - Is the data still there when we need it ?

All system types care about correctness.

<img width="972" alt="arch_online_ecom_platform" src="https://github.com/devops820/my_sre_notes/assets/34048837/f683b12e-a6fe-4270-88ef-ab4d0d2be5f7">

An application running on user's browser interact with a remote API.
The API writes the application state into the application product storage system.
The product state is periodically read by the aggregation pipeline system.
The pipeline system uses both product and user data to generate the mostly highly viewed and purchased items over the last day, week, month by region.
We can segment these architecture components into 3 categories.

API & Website Components fall into Request Driven Category. (because they are interacted with, directly by the users)
Product State, User Data and Region table all represent storage systems because they hold data that is used by the other components.
Aggregation pipeline represent the Big Data system because it takes data and transforms & stores in region table.

It is valid for a request driven system to have Correctness SLI's, which in this case, would measure the proportion of responses that resulted in the correct value. Chose a handful of SLI's that represent the system's most critical usage.

### Measurement and Calculation

The most imporant question for generating SLI's is to ask `what do your users care about ?`
That said `User happiness is difficult to measure`
Think critically to `approximate user needs`
Brain storm with your product team to measure things which are measurable and have direct business impact.
When considering how to calculate service level indicators, you may turn to your company's monitoring systems to find relevant methods.

Example of an e-commerce platform
<img width="886" alt="arch_online_ecom" src="https://github.com/devops820/my_sre_notes/assets/34048837/be76c168-4909-41ac-af97-99eaac59c24e">

The load balancer aggregates product and user data to calculate the most highly viewed and purchased items over the last day, week and year by region.
The statistics (highly viewed and purchased items) are available to the user over API and website. All non-storage systems(Load balancer, API, Website), send metrics to prometheus, an open source whitebox monitoring system.

Let us say you want to define latency related SLI

<img width="943" alt="SLI_Latency" src="https://github.com/devops820/my_sre_notes/assets/34048837/9d469d6e-76d2-4b55-bf27-60e781691077">

The query will give all of the requests in buckets less than or equal to 500 ms that are associated with hosts API divided by all of the requests to API servers. **Be careful when aggregating measurements**.

- Think about metrics as distributions rather than averages. 
- Using percentiles allow you to consider the shape of the distribution. (99th percentile to show the plausible worst case values, 50th percentile shows the median).
- Consider client side instrumentation to most closely approximate the actual user experience.

For example, only measuring the API server response latency from the backend server may miss a host of issues that the user may face client side. (such as Javascript rendering times). A better alternative is to instrument the client to measure how long it takes for the page to become visible.

### Common SLI Standard Definitions
- Aggregation Intervals (for example, "averaged over 30 seconds")
- Aggregation boundaries (for example, "all tasks in a cluster of hosts")
- Measurement frequency (for example, "every 5 seconds")

Additionally include which requests are included in the indicator (for example, all HTTP GET requests from black-box monitoring jobs)
How data is acquired (for example - from our monitoring service.)
How latency is measured (for example - time to last byte)

Having these standards makes it simpler for different people or teams to gain a common understanding of what a specific service level indicator means.

## Service Level Objectives

- SLO builds off of a SLI to provide a target level of reliability for a service to customers.
- SLO = SLI + Threshold + TimeWindow
- SLO's prevent over utilization and under-utilization of a service
- Without an SLO there is no way of saying what kind of downtime is acceptable
- No way to measure if this is a significant service issue
- No path to cutting off additional support tickets from escalating to your on-call engineers
- 100% reliability target is wrong (reduces innovation/deployment)
- SLO's need to be owned by the people who can act on them

**Example of an SLO** -
SLI - API requests < 500 ms
Threshold - 90%
Time Window - 30 days rolling
SLO - 90% of the requests to the API should take fewer than 500 milliseconds over a 30-day period

How will you if something is a resonable SLO ? How to set an initial SLO ?
The short answer is, take a 30 day or 4 week period, track your SLI and see at which percentile's does your SLI's indicate
You set that.

### Choosing time window
- SLO's can be set over multiple time intervals (and can use a rolling time-window or calendar based window) or (short-duration vs long-duration)
- Rolling windows follows more closely the user's experience. For example, if you have a large outage on the last day of the month, the user doesn't forget
  about it on the first day of the next month.
- Also about the rolling windows, choose the window carefully so that it has equal no of weekends, this affects because the traffic patterns may vary during a weekday or a weekend.
- Conversly, Calendar based windows align with business planning, and potentially project planning

### Window Duration
- Shorter windows - allows you to make decisions quickly
- Larger windows - better for strategic decisions (planning multi-month projects based on a week's worth of data is not ideal)
- start with a 4 week rolling window (complement this timewindow with weekly summaries for sprint prioritization and quarterly summaries for long term project planning)

### Stakeholder Aggrement
- In order for a SLO to be useful and effective, you will need to get buy-in from stakeholders - product managers and product developers and (SRE team)
- Product Managers - Is the threshold set high enough for the users? (below the threshold - unacceptably low, such that it is worth the engineering time to fix whatever is causing the missing threshold)
- Product Developers - Do we have capacity to take steps to reduce risk in case of SLO violations? But the accomodation of changes to the service must happen with in the error budgets
- SRE's - Can we meet SLO without excessive or manual work ?

### Documenting SLO's
- Centalized easy to update space
- In the documentation include the following

### Continous Improvements
- Count manually discovered outages and looking at support tickets
- User sentiment on social media
- Periodically sample user happiness
- F2F user interviews

### SLO Coverage
Ask yourselves - Do these periods (Outages and service issues) correlate with SLI trends.
If outages and ticket spikes are not captured or if SLI dips & SLO misses do not correlate with user dissatisfaction, its likely your SLO lacks coverage

3 ways to respond for SLO coverage
- Loosen or Tighten SLO
- change SLI implementation
- create an aspirational SLO

If your SLI's indicate a problem, but your SLO's didn't prompt anyone to notice and respond then tighten your SLO.
Alternatively if you have many false positive events, then loosen your SLO.

SLI coverage
- improve the quality of your metric
  - by moving the measurement closer to the end user
- sample the metric more frequently to capture a higher percentage of user interactions

3 Important Factors

SLO
Toil
Customer Satisfaction

              SLO Met?    High Toil?   Users Happy?
Tighten SLO    Yes          Yes           No

Loosen SLO      No          No            Yes

Offload toil,   No          No            No
fix product 
and
improve
automation


Finally revisit the SLO's at set cadence.



