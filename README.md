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
