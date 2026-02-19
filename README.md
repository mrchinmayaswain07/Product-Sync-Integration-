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
*  Product Server API called
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

##  Flow Summary
1. Experience API receives request
2. Process API calls Product Server API
3. Products are fetched from FakeStore API
4. Business rules applied (Premium / Standard classification)
5. Messages published to VM queue
6. Salesforce System API consumes messages
7. Records upserted into Salesforce
8. Errors handled via global error handler and DLQ

---

##  1. Experience API
The **Experience API** is the entry point of the application. It exposes a REST endpoint to trigger product synchronization and forwards the request to the Process API.
This API follows a structured design using **four XML files**:

* `interface.xml`
* `impl.xml`
* `global.xml`
* `error-handler.xml`

### i) interface.xml
                                                                  
<img width="332" height="306" alt="image" src="https://github.com/user-attachments/assets/138aa70d-d9c3-4d9e-ae01-b176d3edc4ee" />

* Contains the **HTTP Listener**
* Logs incoming requests
* Calls the implementation flow using **Flow Reference**   
This layer handles only client interaction.

### ii) impl.xml
<img width="336" height="299" alt="image" src="https://github.com/user-attachments/assets/771b36c5-a82c-4127-b6be-aed7d243056a" />

* Sends **HTTP Request** to Process API
* Logs response from Process API
* References the global error handler

This layer forwards the request and returns the response.

### iii) global.xml
  <img width="347" height="222" alt="image" src="https://github.com/user-attachments/assets/1849c49d-086e-43fc-8aec-195185215dbe" />

* Contains **Configuration Properties**
* HTTP Listener configuration
* HTTP Request configuration (Process API)
* API Autodiscovery

All configurations are environment-based (local/dev).

### iv) error-handler.xml
<img width="162" height="268" alt="image" src="https://github.com/user-attachments/assets/c418fbaf-8bce-402b-a94a-d7b8978916b9" />

* Defines a **Global Error Handler**
* Handles all errors (type: ANY)
* Logs error details with Correlation ID

---
---

# 2. Process API

The **Process API** is responsible for orchestration and business logic execution. It connects the Experience API with the System APIs, applies business rules, and manages asynchronous processing using VM queues.

This API is structured using four XML files:

* `interface.xml`
* `impl.xml`
* `global.xml`
* `error-handler.xml`

## i) interface.xml
<img width="330" height="464" alt="image" src="https://github.com/user-attachments/assets/95a248fa-d649-4a63-a8f2-2db207541780" />


The interface layer contains:

* **HTTP Listener** – Receives requests from Experience API
* **Logger** – Logs incoming request with Correlation ID
* **Flow Reference** – Calls implementation flow
* **Scheduler Flow** – Triggers synchronization automatically at configured intervals

This layer handles request intake and scheduling, without business logic.

## ii) impl.xml
<img width="1268" height="579" alt="image" src="https://github.com/user-attachments/assets/67a2cb7b-34e5-4c80-a0d5-00654ec8c34b" />


The implementation layer performs core orchestration:

### Main Flow (`process-product-impl`)

* Calls **Product Server API** using HTTP Request
* Logs product fetch response
* Sets required variables
* Applies business rules using **Choice Router**

  * Classifies products as Premium or Standard
* Transforms data using **DataWeave**
* Publishes messages to **VM Queue** for asynchronous processing
* Returns summarized response

### VM Consumer Flow (`process-vm-consumer-flow`)

* Listens to VM queue
* Logs consumed message
* Calls **Salesforce System API**
* Processes records asynchronously

This ensures scalable, decoupled processing.

## iii) global.xml
<img width="312" height="222" alt="image" src="https://github.com/user-attachments/assets/a003f3e7-47b6-49b6-a814-613dd4403f79" />

Contains global configurations:

* Configuration Properties
* HTTP Listener Configuration
* HTTP Request Config (Server API & Salesforce API)
* VM Configuration

All configurations are environment-based using YAML property files.


## iv) error-handler.xml
<img width="165" height="317" alt="image" src="https://github.com/user-attachments/assets/819bcf2c-f1a8-4cf0-a6fe-c833765e96c3" />

Defines centralized error handling:

* Global Error Handler (`type: ANY`)
* Logs errors with Correlation ID
* Ensures consistent error management across flows

---
---

# 3)Product Server API (System Layer)

The **Product Server API** acts as the System API responsible for connecting to the external FakeStore public API. It retrieves product data and exposes it in a standardized format to the Process API.

This API is responsible only for external system connectivity and does not contain business logic.

The API is structured using four XML files:

* `interface.xml`
* `impl.xml`
* `global.xml`
* `error-handler.xml`

## i) interface.xml
<img width="227" height="342" alt="image" src="https://github.com/user-attachments/assets/5e8703c3-7a6b-41be-aba0-ae4227a301dc" />

The interface layer contains:

* **HTTP Listener** – Exposes `/system/products` endpoint
* **Flow Reference** – Calls implementation flow
* Error handling reference

This layer handles incoming requests from the Process API and forwards them to the implementation flow.

## ii) impl.xml
<img width="375" height="285" alt="image" src="https://github.com/user-attachments/assets/50126638-7c72-460d-b23b-1e61c5ab6901" />


The implementation layer performs the external API call:

* **Logger** – Logs request initiation
* **HTTP Request** – Calls FakeStore public API
* **Logger** – Logs response received
* References global error handler

This flow is responsible for retrieving product data from the external system and returning it to the Process API.

## iii) global.xml
<img width="262" height="201" alt="image" src="https://github.com/user-attachments/assets/932918c8-b377-4045-9105-2f534d5fdf39" />

Contains global configurations:

* Configuration Properties
* HTTP Listener Configuration
* HTTP Request Configuration (FakeStore API connection)

All configurations are environment-based (local/dev).


## iv) error-handler.xml
<img width="183" height="298" alt="image" src="https://github.com/user-attachments/assets/5e980aff-5d2e-41b2-beaf-86675685f960" />

Defines centralized error handling:

* Global Error Handler (`type: ANY`)
* Logs error details with Correlation ID
* Ensures consistent error logging and failure tracking

---
---


# 4. Salesforce System API (System Layer)

The **Salesforce System API** is responsible for integrating with Salesforce and upserting product records into the `External_Product__c` object.

This API handles data transformation, retry logic, error handling, and dead-letter queue processing to ensure reliable data synchronization.

The API is structured using four XML files:

* `interface.xml`
* `impl.xml`
* `global.xml`
* `error-handler.xml`

## i) interface.xml
<img width="234" height="326" alt="image" src="https://github.com/user-attachments/assets/612eb4fe-a48a-415a-ac78-9cada0de1d22" />

The interface layer contains:

* **HTTP Listener** – Exposes endpoint for upsert requests
* **Flow Reference** – Calls implementation flow
* Error handling reference

This layer receives requests from the Process API and forwards them to the implementation flow.

## ii) impl.xml
<img width="652" height="295" alt="image" src="https://github.com/user-attachments/assets/d3b656a6-f2e9-44ec-8f81-728a41a05698" />

The implementation layer performs Salesforce integration:

* **Logger** – Logs incoming request
* **Transform Message (DataWeave)** – Converts payload into Salesforce format
* **Try Scope** – Wraps Salesforce operation
* **Until Successful** – Implements retry mechanism
* **Salesforce Upsert** – Upserts record using External ID
* **Logger** – Logs successful operation
* References global error handler

This ensures reliable and fault-tolerant record synchronization.

## iii) global.xml
<img width="285" height="245" alt="image" src="https://github.com/user-attachments/assets/6c2f3ed9-5a3a-4f87-b92d-70b4b2407e03" />

Contains global configurations:

* Configuration Properties
* HTTP Listener Configuration
* Salesforce Configuration
* VM Configuration (for DLQ)
* Secure Properties Configuration

Sensitive credentials are managed using secure properties.

## iv) error-handler.xml
<img width="253" height="380" alt="image" src="https://github.com/user-attachments/assets/29b9d1af-67dc-4338-9b2b-ffbe149ba1dd" />

Defines centralized error handling:

* **On Error Propagate (MULE:RETRY_EXHAUSTED)**

  * Logs error
  * Publishes failed message to Dead Letter Queue (DLQ)

* **On Error Propagate (ANY)**

  * Logs error
  * Publishes message to DLQ

---
----


