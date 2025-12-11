# Comprehensive Web Application Performance Testing: Blazedemo.com

**Name:** Muhammad Fadzli Bin Mohd Sahid
**Student ID:**  2023234642
**Group:** NBCS2555A
**Course:** ITT440 - Network Programming
**Lecturer:** Sir Shahadan Bin Saad

---

## 1.0 Executive Summary
The primary objective of this individual assignment is to design, execute, and critically analyze a comprehensive performance test plan for a web application. In compliance with the ITT440 coursework requirements, this project focuses on evaluating the stability and scalability of **Blazedemo.com** under various load conditions.

Using **Locust**, a modern Python-based load testing framework, three distinct testing scenarios were executed: Load Testing, Spike Testing, and Soak Testing. The empirical data collected indicates that while the target application maintains stability under consistent nominal loads, it exhibits significant latency degradation during sudden traffic spikes. This report documents the tool selection justification, test environment setup, script logic, data analysis, and technical recommendations for optimization.

---

## 2.0 Introduction
### 2.1 Background
In the domain of Network Programming and Web Administration, application performance is a critical determinant of User Experience (UX) and Service Level Agreement (SLA) compliance. Performance testing is the non-functional testing technique performed to determine the system parameters in terms of responsiveness and stability under various workloads.

### 2.2 Objective
The specific objectives of this project are:
1.  To demonstrate the ability to configure a specialized performance testing tool (Locust).
2.  To formulate a valid hypothesis regarding web server behavior under stress.
3.  To execute Load, Stress, and Endurance tests to identify system bottlenecks.
4.  To interpret key metrics such as Response Time (Latency) and Throughput (RPS).

### 2.3 Target Application
* **URL:** `https://blazedemo.com`
* **Description:** Blazedemo is a sample e-commerce site provided by BlazeMeter for the purpose of learning performance testing. It simulates a travel agency scenario where users can query flights and book tickets. It is a safe, legal, and ethical target for educational testing purposes.

---

## 3.0 Tool Selection and Justification
### 3.1 Selected Tool: Locust
For this assignment, **Locust** (version 2.42.6) was selected as the primary testing instrument. Locust is an open-source load testing tool written in Python.

### 3.2 Justification against Alternatives
While tools like **Apache JMeter** and **LoadRunner** are industry standards, Locust was chosen for the following technical reasons:

* **Configuration as Code:** Unlike JMeter, which uses a complex GUI and XML-based test plans, Locust uses standard Python code. This allows for version control (Git) and makes the test scripts more readable and maintainable.
* **Event-Based Concurrency:** Locust is built on `gevent`, a coroutine-based Python networking library. This allows a single machine to simulate thousands of concurrent users without the high memory overhead associated with thread-based tools like JMeter.
* **Real-Time Monitoring:** Locust provides a built-in web interface (running on localhost:8089) that visualizes requests per second (RPS), failure rates, and response times in real-time, facilitating immediate analysis.

---

## 4.0 Test Environment Setup
To ensure the reproducibility of the test results, the local testing environment was configured as follows.

### 4.1 Hardware Specifications
* **Device:** Laptop VivoBook_Asus Laptop
* **Processor:** Intel(R) Core(TM) i5-1035G1 CPU @ 1.00Ghz
* **RAM:** 12GB RAM
* **Network:** Wired Connection (Ethernet LAN)

### 4.2 Software Configuration
* **Operating System:** Windows 11 (64-bit)
* **Python Runtime:** Python 3.12.8
    * *Note: Python 3.12 was chosen to ensure compatibility with `gevent` library dependencies.*
* **Locust Version:** 2.42.6

### 4.3 Setup Evidence
The following screenshots demonstrate the successful installation and verification of the testing tool via the Windows Command Prompt.

**Figure 1: Locust Installation & Verification**
<img width="624" height="343" alt="Picture1" src="https://github.com/user-attachments/assets/ffc201dd-a367-4f61-858d-3cf23f52a34a" />


---

## 5.0 Test Methodology & Scenarios
The testing strategy involved defining user behavior using a `locustfile.py` script and executing three distinct patterns of load.

### 5.1 The Test Script (`locustfile.py`)
The user behavior was modeled using the `HttpUser` class.
* **Task:** `self.client.get("/")` - Simulates a user visiting the homepage.
* **Wait Time:** `wait_time = between(1, 5)` - This is critical for realism. It introduces a random delay of 1 to 5 seconds between requests, simulating a human user "reading" the page before clicking again.

### 5.2 Scenario 1: Load Test (Baseline)
* **Definition:** Testing the system under expected normal traffic.
* **Configuration:** 50 Users, Spawn Rate 5.
* **Goal:** To establish a performance baseline.

### 5.3 Scenario 2: Spike Test (Stress)
* **Definition:** Testing the system's reaction to a sudden, extreme surge in traffic.
* **Configuration:** 200 Users, Spawn Rate 50 (Very aggressive ramp-up).
* **Goal:** To identify if the server crashes or slows down significantly (Latency Spike).

### 5.4 Scenario 3: Soak Test (Endurance)
* **Definition:** Testing the system's stability over a longer duration.
* **Configuration:** 20 Users, Spawn Rate 1.
* **Goal:** To detect memory leaks or performance degradation over time.

---

## 6.0 Result Analysis & Findings

### 6.1 Analysis of Load Test (50 Users)
**Graph Evidence:**
<img width="624" height="381" alt="Picture2" src="https://github.com/user-attachments/assets/8770fd5c-8e4b-426b-98f8-723330a95bfe" />


**Interpretation:**
The Load Test results indicate a stable system.
* **Response Time:** The Median Response Time remained consistent at approximately **422** ms.
* **Throughput:** The server handled the requested load of approximately **6.1** RPS without generating errors.
* **Conclusion:** The application is well-optimized for low-to-moderate traffic.

### 6.2 Analysis of Spike Test (200 Users)
**Graph Evidence:**
<img width="624" height="367" alt="spike test" src="https://github.com/user-attachments/assets/2780535a-10d5-4d72-846f-878d6d26471d" />


**Interpretation:**
This scenario revealed the most significant findings.
* **Latency Spike:** As shown in the graph (Purple Line), when the user count surged to 200 within 4 seconds, the Response Time spiked dramatically to a peak of **300** ms.
* **Bottleneck Identification:** This indicates that the server's thread pool or database connection pool was saturated. The system could not scale fast enough to handle the burst, causing requests to queue up.
* **User Impact:** Real users visiting during this spike would experience a page load delay of over 2 seconds, which is considered poor UX.

### 6.3 Analysis of Soak Test (20 Users)
**Graph Evidence:**
<img width="624" height="380" alt="soak test" src="https://github.com/user-attachments/assets/1dd23875-2ae8-4034-a298-6ca3756782a5" />


**Interpretation:**
* **Consistency:** Both the Request Rate (Green Line) and Response Time (Yellow/Purple Lines) remained flat and horizontal throughout the test duration.
* **No Memory Leaks:** There was no gradual increase in response time, which suggests that the application releases resources correctly after serving requests.

---

## 7.0 Discussion & Recommendations
Based on the empirical evidence gathered, specifically the latency degradation observed during the Spike Test, the following optimization strategies are recommended:

1.  **Implement Load Balancing:** To mitigate the impact of traffic spikes, a Load Balancer (e.g., NGINX or AWS ELB) should be implemented. This would distribute incoming traffic across multiple server instances.
2.  **Caching Strategy:** The high response time suggests that the server is processing every request dynamically. Implementing a caching layer (e.g., Redis) to store static content would significantly reduce the load on the backend server.
3.  **Content Delivery Network (CDN):** Offloading static assets such as images and CSS to a CDN (like Cloudflare) would reduce bandwidth usage on the origin server.

---

## 8.0 Video Presentation
A detailed walkthrough of the test execution, configuration, and result analysis can be viewed in the following video:

[**CLICK HERE TO WATCH VIDEO PRESENTATION**]([LETAK LINK YOUTUBE ANDA DI SINI])

---

## 9.0 Conclusion
In conclusion, this project successfully utilized the Locust framework to profile the performance of the Blazedemo web application. The testing confirmed the hypothesis that while the application is robust under normal conditions, it lacks the elasticity required to handle sudden traffic surges efficiently.

The assignment provided valuable hands-on experience in setting up a testing environment, writing Python-based test scripts, and critically analyzing performance data.

---
