# Product Sync & Notification Integration

**Author:** Chinmaya Swain

---

## Overview

Product Sync & Notification Integration is a MuleSoft application that fetches product data from a public API, applies business rules using Choice and DataWeave, publishes messages to a VM queue, processes them asynchronously, and upserts records into Salesforce.
The application includes retry logic, centralized error handling, correlation ID tracking, environment-based configuration, and follows best practices for reliability, scalability, and API-led connectivity.

---

##  Architecture

This project follows an **API-led layered architecture**:

### 1️⃣ Experience API

* Exposes REST endpoint to trigger product synchronization
* Handles client interaction
* Calls Process API

### 2️⃣ Process API

* Orchestrates the integration flow
* Applies business rules (Premium / Standard product classification)
* Publishes messages to VM queue
* Calls System APIs

### 3️⃣ Product Server API (System Layer)

* Connects to FakeStore public API
* Fetches product data
* Returns standardized response

### 4️ Salesforce System API

* Transforms product data to Salesforce format
* Upserts records into `External_Product__c` object
* Implements retry logic using `until-successful`
* Sends failed records to Dead Letter Queue (DLQ)

---

##  Flow Summary

1. Experience API receives request
2. Process API calls Product Server API
3. Products are fetched from FakeStore API
4. Business rules applied (Premium / Standard classification)
5. Messages published to VM queue
6. Salesforce System API consumes messages
7. Records upserted into Salesforce
8. Errors handled via global error handler and DLQ

## 🌐 Experience API

The **Experience API** is the entry point of the application. It exposes a REST endpoint to trigger product synchronization and forwards the request to the Process API.

This API follows a structured design using **four XML files**:

* `interface.xml`
* `impl.xml`
* `global.xml`
* `error-handler.xml`

---

### 📂 interface.xml
                                                                  
<img width="332" height="306" alt="image" src="https://github.com/user-attachments/assets/138aa70d-d9c3-4d9e-ae01-b176d3edc4ee" />

* Contains the **HTTP Listener**
* Logs incoming requests
* Calls the implementation flow using **Flow Reference**   

This layer handles only client interaction.

---

### 📂 impl.xml
<img width="336" height="299" alt="image" src="https://github.com/user-attachments/assets/771b36c5-a82c-4127-b6be-aed7d243056a" />


* Sends **HTTP Request** to Process API
* Logs response from Process API
* References the global error handler

This layer forwards the request and returns the response.

---

### 📂 global.xml
  <img width="347" height="222" alt="image" src="https://github.com/user-attachments/assets/1849c49d-086e-43fc-8aec-195185215dbe" />

* Contains **Configuration Properties**
* HTTP Listener configuration
* HTTP Request configuration (Process API)
* API Autodiscovery

All configurations are environment-based (local/dev).

---

### 📂 error-handler.xml
<img width="162" height="268" alt="image" src="https://github.com/user-attachments/assets/c418fbaf-8bce-402b-a94a-d7b8978916b9" />

* Defines a **Global Error Handler**
* Handles all errors (type: ANY)
* Logs error details with Correlation ID



