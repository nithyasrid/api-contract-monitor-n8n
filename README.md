

# API Contract Monitor

> An n8n-based workflow that validates REST API responses against an expected contract and alerts developers when required fields are missing.
##DEMO
> https://github.com/user-attachments/assets/813c9351-77e9-4d41-adb0-c306b3075682
 

## Overview

API Contract Monitor checks whether an API response matches the structure expected by an application.

The workflow sends a request to a REST API, validates the response using JavaScript, and determines whether the API contract is valid.

If the contract is valid, a success report is generated.

If a required field is missing, the workflow generates a contract violation alert and sends an email notification.

## Problem

Applications depend on APIs returning data in an expected structure.

For example, an application may expect:

```json
{
  "id": 1,
  "name": "John",
  "email": "john@example.com"
}
````

If a required field is removed or changed, the application consuming the API may fail.

This project detects missing required fields before they can cause downstream integration problems.

## Workflow

```text
Manual Trigger
      ↓
HTTP Request
      ↓
Contract Validator
      ↓
      IF
    ↙     ↘
 TRUE     FALSE
   ↓         ↓
Success    Contract
Report     Alert
             ↓
           Gmail
```

## How It Works

### 1. Manual Trigger

Starts the workflow manually from n8n.

### 2. HTTP Request

Sends a `GET` request to the REST API.

**Endpoint:**

```text
https://jsonplaceholder.typicode.com/users/1
```

The API returns a JSON response.

### 3. Contract Validator

The JavaScript Code node checks whether the required fields exist in the API response.

Required fields:

```text
id
name
email
```

Validation logic:

```javascript
const data = $input.first().json;

const requiredFields = [
  "id",
  "name",
  "email"
];

const missingFields = requiredFields.filter(
  field => data[field] === undefined
);

const contractValid = missingFields.length === 0;

return [
  {
    json: {
      contractValid,
      missingFields,
      endpoint: "https://jsonplaceholder.typicode.com/users/1"
    }
  }
];
```

### 4. IF Node

The IF node checks the `contractValid` value.

```text
contractValid = true
        ↓
      TRUE
```

```text
contractValid = false
        ↓
      FALSE
```

### 5. Success Report

If the API contract is valid, the TRUE branch generates a success report.

Example:

```json
{
  "status": "HEALTHY",
  "message": "API contract is valid",
  "endpoint": "https://jsonplaceholder.typicode.com/users/1",
  "checkedAt": "2026-09-07T..."
}
```

### 6. Contract Alert

If the API contract is invalid, the FALSE branch generates a contract violation report.

Example:

```text
🚨 API CONTRACT VIOLATION

Status: FAILED

Endpoint:
https://jsonplaceholder.typicode.com/users/1

Missing Fields:
age

Message:
The API response does not match the expected contract.
```

### 7. Gmail Notification

The contract violation information is automatically sent through Gmail.

Example:

```text
Subject:
🚨 API Contract Violation Detected

Status: FAILED

Endpoint:
https://jsonplaceholder.typicode.com/users/1

Missing Fields:
age
```

## Testing

### Valid Contract

The expected fields are:

```text
id
name
email
```

Result:

```text
contractValid: true
missingFields: []
```

The workflow follows the TRUE branch:

```text
API Response
     ↓
Validation
     ↓
TRUE
     ↓
Success Report
     ↓
✅ HEALTHY
```

### Contract Violation

To test the failure scenario, an additional required field such as `age` can be temporarily added:

```javascript
const requiredFields = [
  "id",
  "name",
  "email",
  "age"
];
```

Since the API does not return `age`, the validator produces:

```text
contractValid: false
missingFields: ["age"]
```

The workflow follows the FALSE branch:

```text
API Response
     ↓
Validation
     ↓
FALSE
     ↓
Contract Alert
     ↓
Gmail
     ↓
🚨 ALERT
```

After testing, restore the required fields to:

```javascript
const requiredFields = [
  "id",
  "name",
  "email"
];
```

## Tech Stack

* **n8n** — Workflow automation
* **REST API** — Data source
* **HTTP** — API communication
* **JSON** — Data format
* **JavaScript** — Response validation
* **Gmail** — Alert notification
* **GitHub** — Version control

## Project Structure

```text
api-contract-monitor/
│
├── README.md
│
├── workflow/
│   └── api-contract-monitor.json
│
└── screenshots/
    ├── healthy-response.png
    └── contract-violation.png
```

## Key Features

* REST API integration
* API response validation
* Required field checking
* Missing field detection
* Success and failure routing
* Contract violation reporting
* Automated Gmail alerts
* JavaScript-based validation

## Result

The completed workflow provides a simple automated mechanism to identify API contract violations.

```text
API Request
     ↓
JSON Response
     ↓
Contract Validation
     ↓
Valid / Invalid
   ↙       ↘
Healthy   Violation
            ↓
       Gmail Alert
```



```
```
