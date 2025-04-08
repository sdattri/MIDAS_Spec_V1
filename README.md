# MIDAS (Market Informed Demand Automation Server) Technical Specifications

## Document Overview
- **Title**: Technical Specification – Market Informed Demand Automation Server (MIDAS)  
- **Version**: v1.0  
- **Date**: 2025-04-06  
- **Author(s)**: Stefanie Wayland | Shubham Attri

---

# Market Informed Demand Automation Server (MIDAS) Technical Specification

## 1. Introduction

MIDAS is a database and API system developed by the California Energy Commission, providing access to current, future, and historic time-varying rates, GHG emissions associated with electrical generation, and California Flex Alert Signals. Its primary purpose is to enable energy users to optimize their energy usage timing based on electricity price information. The system is publicly accessible via API at https://midasapi.energy.ca.gov, supporting JSON and XML formats, with registration required for access. It does not contain private information, with only login details being non-public.

### 1.1 Purpose of the Document

This document serves as a technical specification for the Market Informed Demand Automation Server (MIDAS), detailing its architecture, data structures, APIs, security measures, and operational guidelines. It aims to assist developers and IT professionals in integrating with and utilizing MIDAS effectively.

### 1.2 Overview of MIDAS

The California Energy Commission's (CEC) Market Informed Demand Automation Server (MIDAS) is a centralized database that stores current, future, and historical time-varying electricity rates, greenhouse gas (GHG) emissions associated with electrical generation, and California Flex Alert signals. It is populated by electric Load Serving Entities (LSEs), the California Independent System Operator (California ISO), and other registered entities. MIDAS provides public access through a RESTful API, supporting both XML and JSON formats, to facilitate automated demand response and load management. It’s accessible via API at https://midasapi.energy.ca.gov, requiring a simple registration process.

## 2. System Architecture

### 2.1 System Components

- **Firewall**: Protects the system from external threats, ensuring only authorized access.
- **Load Balancer**: Distributes incoming API requests to multiple servers, enhancing performance and availability.
- **API Server**: Handles RESTful API requests, processes them, and interacts with the database.
- **Database**: Stores all system data, including rates, values, historical data, holidays, and lookup tables.

### 2.2 Operational Environment

- **Hosting**: Hosted by the California Energy Commission.
- **Accessibility**: Accessible via the public internet at [https://midasapi.energy.ca.gov](https://midasapi.energy.ca.gov).
- **Data Formats Supported**: XML and JSON.
- **API Interaction**: Requires HTTP clients capable of handling RESTful requests.
  
### 2.3 Key Features and Capabilities

- **Centralized Data Repository**: Stores time-varying electricity rates, GHG emissions data, and Flex Alert signals.
- **Public API Access**: Offers RESTful API endpoints for data retrieval and submission.
- **Support for Standard Data Formats**: Provides data in XML and JSON formats.
- **Real-time and Historical Data Access**: Enables retrieval of both current and archived data.
- **Secure Data Transactions**: Implements authentication and authorization mechanisms for data integrity and security.

### 2.4 Data Flow

- User requests pass through the firewall and load balancer before reaching the API server.
- The API server queries the MIDAS database for responses and returns data in JSON or XML format.
- All connections use SSL, and authentication requires a valid token, which expires after 10 minutes.
- The architecture diagram is available in Appendix D at [Appendix D: Architecture](https://github.com/california-energy-commission/MIDAS/blob/main/appendix-d.md), showing the service architecture with request flow and security measures.


## 3. Data Model

### 3.1 Database Schema

The MIDAS database supports retrieval of electric utility price schedules, California Flex Alert signals, and marginal GHG emissions from electrical generation. Flex Alerts and GHG emissions values – both forecasted and real-time - are continually retrieved from the California ISO Flex Alert site and WattTime’s SGIP API respectively. GHG realtime and forecasted values are cached, or temporarily stored, within MIDAS until new values are available while Flex Alerts are passed through MIDAS from the California ISO website, as a user queries MIDAS. Previous real-time values are moved to the HistoricalData table automatically. A record of previously active Flex Alerts and historic GHG emissions are stored in the HistoricalData table.

Pursuant to the California Load Management Standards, the state’s largest utilities and CCAs are responsible for populating the MIDAS database Holiday table, RateInfo table, and the Value table with all time-varying rate information and values offered to customers. For upload examples, please see [Appendix A](appendix-a.md). For instructions on how to retrieve the XML upload schema, please see section 3.

The primary lookup identification (ID) for the MIDAS database is a compound key comprised of six individual fields that make up a standardized rate identification number (RIN) as shown in Figure 1. RINs are assigned at the time rate information is first uploaded by the LSE through the MIDAS API. When an LSE uploads to an existing RIN, the correct RIN must be used at the time of upload. Figure 1 illustrates the six identifiers that comprise a RIN: Country, State, Distribution, Energy, Rate, and Location. The location portion of the RIN may consist of 1 to 10 characters depending on the specified location’s requirements.

Figure 1. Rate Identification Number Structure<br>
![Rate Identification Number Specification](img/RIN-structure.png)<br>
Source: California Energy Commission

Rate Identification Numbers do not change over time. The prices and values may change, but an electricity customer's RIN should not change unless their rate components or rate modifiers change or the utility or customer changes their rate tariff.

MIDAS comprises several key tables:

- **Holiday Table**: Stores holiday schedules for LSEs.
  - Fields:
    - `HolidayID` (Primary Key)
    - `LSEID`
    - `HolidayName`
    - `Date`
    - `Description`
- **RateInfo Table**: Contains metadata about rate schedules.
  - Fields:
    - `RateID` (Primary Key)
    - `LSEID`
    - `RateName`
    - `EffectiveDate`
    - `ExpirationDate`
    - `Description`
- **Value Table**: Holds time-series data for rates, GHG emissions, and Flex Alerts.
  - Fields:
    - `ValueID` (Primary Key)
    - `RateID`
    - `Timestamp`
    - `Value`
    - `Units`

- **Lookup Tables**: Provide codes and descriptions for various entities, ensuring consistency in data. The following tables are detailed in Appendix C at [MIDAS Lookup Tables Appendix[(https://github.com/california-energy-commission/MIDAS/blob/main/appendix-c.md):

## Lookup Table Descriptions

| Table Name            | Description                                                                 | Code Details                         |
|-----------------------|-----------------------------------------------------------------------------|--------------------------------------|
| Country Table         | Contains the two-letter code for each country with the name describing each code. | Two-letter code                      |
| Day Type Table        | Contains a single number Day Type Code choice with the name, where 1 = Monday through 7 = Sunday and 8 = Holiday. | 1 to 8 (1–7 = days, 8 = Holiday)     |
| Distribution Table    | Contains the two-letter code for each distribution company in California with the name describing each code. | Two-letter code                      |
| End Use Table         | Contains the code relevant to each end use, up to 10 letters, with a description. | Up to 10 letters                     |
| Energy Table          | Contains the two-letter code for each energy company in California with the name describing each code. | Two-letter code                      |
| Holiday Table         | Contains holidays defined and uploaded by distribution and energy companies. | –                                    |
| Location Table        | Contains the 1 to 10-character code relevant to each location with a description. | 1 to 10 characters                   |
| Rate Type Table       | Contains the Rate Type Code, up to 10 characters, options applicable to each company’s rates with a description. | Up to 10 characters                  |
| Sector Table          | Contains the three-letter code for the sector relevant to each value with a description. | Three-letter code                    |
| State/Province Table  | Contains the two-letter code for each US state with the name included in the description. | Two-letter code                      |
| Unit Table            | Contains the unit code relevant to each rate, emissions, or event value, up to 50 characters with a description. | Up to 5 characters                   |

The data dictionary, available for download in MS Excel format at [MIDAS Data Dictionary](https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fmedia.githubusercontent.com%2Fmedia%2Fcalifornia-energy-commission%2FMIDAS%2Frefs%2Fheads%2Fmain%2Fsupport-docs%2FMIDAS-Data-Dictionary.xlsx&wdOrigin=BROWSELINK), provides detailed descriptions of data available through endpoints and lookup tables. Note that lookup tables in the dictionary may not align perfectly with MIDAS if changes were made by CEC staff; authoritative lookup tables are those available through the API.

### 3.2 Data Relationships

The `RateInfo` table is central, linking to the `Value` table through the `RateID`. The `Holiday` table associates with specific LSEs via the `LSEID`. This relational structure allows for efficient querying and data integrity.

## 4. API Specifications

### 4.1 Authentication Mechanism

MIDAS employs token-based authentication:

1. **Registration**: Users register via the `/api/Registration` endpoint, providing necessary credentials.
2. **Token Retrieval**: Post-registration, users obtain a token from the `/api/Token` endpoint using their credentials.
3. **Token Usage**: The token must be included in the `Authorization` header of subsequent API requests.

### 4.2 Endpoints Overview

All API requests are made over HTTPS to: [https://midasapi.energy.ca.gov/api/](https://midasapi.energy.ca.gov/api/)

### Request Format

Use headers:

- `Content-Type: application/json` or `application/xml`
- `Authorization: Bearer <token>`

---

## API Endpoints and Methods

| Endpoint           | Method | Description                                 |
|--------------------|--------|---------------------------------------------|
| `/Registration`    | POST   | Create a new user account                   |
| `/Token`           | GET    | Obtain a temporary access token             |
| `/Holiday`         | GET    | Retrieve list of holidays                   |
| `/Holiday`         | POST   | Upload new holiday entries (LSE-only)       |
| `/ValueData`       | GET    | Retrieve available rates/signals/metadata   |
| `/ValueData`       | POST   | Upload rate and value records (LSE-only)    |
| `/HistoricalList`  | GET    | List historical rates for a utility         |
| `/HistoricalData`  | GET    | Retrieve values over a historical range     |

---

## Common Requests and Responses

### Token Request

```http
GET /api/Token
Authorization: Basic amFuZWRvZTpNeVNlY3JldFBhc3MxMjM=
```

## 4.3 Detailed Endpoint Descriptions

---

### 4.3.1 Registration Endpoint

- **URL**: `/api/Registration`
- **Method**: `POST`
- **Content-Type**: `application/json`

#### Request Parameters

| Parameter     | Type     | Required | Description                          |
|---------------|----------|----------|--------------------------------------|
| `username`    | string   | Yes      | Desired unique username              |
| `password`    | string   | Yes      | Desired password                     |
| `email`       | string   | Yes      | Valid email address                  |
| `fullname`    | string   | Yes      | Full name of the user                |
| `organization`| string   | Yes      | Name of user's organization          |

#### Sample Request

```json
{
  "username": "jdoe",
  "password": "securepassword123!",
  "email": "jdoe@example.com",
  "fullname": "John Doe",
  "organization": "Energy Solutions Inc."
}
```

#### Sample Success Response

```json
{
  "message": "User account for jdoe was successfully created. A verification email has been sent to jdoe@example.com. Please click the link in the email in order to start using the API."
}
```

#### Error Codes

| Code | Meaning             | Description                                |
|------|---------------------|--------------------------------------------|
| 400  | Bad Request         | Missing or invalid parameters              |
| 409  | Conflict            | Username or email already exists           |

---

### 4.3.2 Token Endpoint

- **URL**: `/api/Token`
- **Method**: `GET`
- **Authentication**: Basic Auth (Base64 encoded `username:password`)

#### Sample Curl Request

```bash
curl -X GET https://midasapi.energy.ca.gov/api/Token \
  -H "Authorization: Basic amRvZTpzZWN1cmVwYXNzd29yZDEyMyE="
```

#### Sample Response

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 600
}
```

#### Error Codes

| Code | Meaning       | Description                     |
|------|----------------|---------------------------------|
| 401  | Unauthorized   | Invalid credentials             |

---

### 4.3.3 ValueData Endpoint

- **URL**: `/api/ValueData`
- **Methods**: `GET`, `POST`
- **Authentication**: Bearer Token required

---

#### GET Request

**Query Parameters:**

| Parameter   | Type     | Required | Description                                  |
|-------------|----------|----------|----------------------------------------------|
| `id`        | string   | Yes      | RIN of rate or signal                        |
| `queryType` | string   | No       | Type of data: `RealTime`, `Forecast`, etc.   |

**Example:**

```http
GET /api/ValueData?id=USCA-TSTS-TTOU-TEST&queryType=RealTime
Authorization: Bearer <token>
```

**Response Example:**

```json
{
  "RateInfo": {
    "RIN": "USCA-TSTS-TTOU-TEST",
    "Utility": "Pacific Gas & Electric",
    "RateName": "TOU ABC Plan"
  },
  "Value": {
    "TimeStart": "17:00",
    "TimeEnd": "20:00",
    "NumericValue": 0.30,
    "Unit": "$/kWh"
  }
}
```

---

#### POST Request

**Payload Structure:**

```json
{
  "RIN": "USCA-TSTS-TOU5-0000",
  "TimeStart": "2025-04-06T14:00:00Z",
  "TimeEnd": "2025-04-06T16:00:00Z",
  "NumericValue": 0.25,
  "Unit": "$/kWh"
}
```

**Success Response:**

```json
{
  "message": "Value data successfully posted."
}
```

**Error Codes:**

| Code | Meaning         | Scenario                             |
|------|------------------|---------------------------------------|
| 400  | Bad Request      | Invalid data                          |
| 401  | Unauthorized     | Missing or expired token              |

---

### 4.3.4 HistoricalData Endpoint

- **URL**: `/api/HistoricalData`
- **Method**: `GET`
- **Authentication**: Bearer Token required

**Query Parameters:**

| Parameter   | Type   | Required | Description                             |
|-------------|--------|----------|-----------------------------------------|
| `rin`       | string | Yes      | RIN for the rate/signal                 |
| `startDate` | string | Yes      | Start date (`YYYY-MM-DD`)               |
| `endDate`   | string | Yes      | End date (`YYYY-MM-DD`)                 |

**Example:**

```http
GET /api/HistoricalData?rin=USCA-TSTS-TTOU-TEST&startDate=2024-01-01&endDate=2024-01-07
Authorization: Bearer <token>
```

**Response:**

```json
{
  "RIN": "USCA-TSTS-TTOU-TEST",
  "Data": [
    {
      "TimeStart": "2024-01-01T17:00:00Z",
      "TimeEnd": "2024-01-01T20:00:00Z",
      "NumericValue": 0.28,
      "Unit": "$/kWh"
    },
    {
      "TimeStart": "2024-01-02T17:00:00Z",
      "TimeEnd": "2024-01-02T20:00:00Z",
      "NumericValue": 0.29,
      "Unit": "$/kWh"
    }
  ]
}
```
---

### 4.3.5 Supported `queryType` Values

| `queryType`   | Description                                                    |
|---------------|----------------------------------------------------------------|
| `RealTime`    | Live data for the current or recent period                     |
| `Forecast`    | Future predicted rates or GHG signals                          |
| `Historical`  | Archived values based on past time intervals                   |
| `Peak`        | Highest values for the specified time range (optional)         |
| `All`         | Return all available data (used cautiously for large datasets) |

---

Error messages are returned in plain text or JSON.

---

## Performance & Scaling

- Rate limit enforced per IP/token.
- GHG values cached; Flex Alerts queried live.
- 50,000 value record limit per rate.
- Token TTL: 10 minutes.
- Load-balanced stateless API.

---

## Glossary

| Term | Description                          |
|------|--------------------------------------|
| LSE  | Load Serving Entity                  |
| RIN  | Rate Identification Number           |
| GHG  | Greenhouse Gas                       |
| TOU  | Time-of-Use                          |
| API  | Application Programming Interface    |

---

## Contact

**Email:** midas@energy.ca.gov  
**Web Tool:** Accessible via MIDAS homepage  
**Support:** Mon–Fri, 9am–5pm PT

---

*End of Document*
