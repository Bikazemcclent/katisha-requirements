# Katisha Online Operator Software - API Endpoint Specification

This document outlines the REST API endpoints designed for the Katisha Online Operator Software. It supports functionalities such as authentication, ticket management, trip scheduling (both single and recurring trips with dynamic bus assignment), payment processing (via MTN Mobile Money and Airtel Money), EBM integration, reporting, logging, and user/authorization management.

**Note:**  In this specification, ticket templates (available trip models) are distinct from purchased tickets (records of completed transactions).

---

## 1. Authentication Endpoints

### 1.1 User Login

* **Method:** `POST`
* **URL:** `/api/auth/login`
* **Description:** Authenticates the user and returns a JSON Web Token (JWT) for session management.
* **Request Body:**

    ```json
    {
      "username": "string",
      "password": "string"
    }
    ```

* **Response:**

    ```json
    {
      "token": "string",
      "userRole": "string"  // e.g., "operator" or "admin"
    }
    ```

### 1.2 User Logout

* **Method:** `POST`
* **URL:** `/api/auth/logout`
* **Headers:** `Authorization: Bearer`
* **Description:** Logs out the user by invalidating the JWT.
* **Response:**

    ```json
    {
      "message": "Successfully logged out"
    }
    ```

---

## 2. Ticketing Endpoints

### 2.1 Ticket Templates

Ticket templates represent the available trip models (predefined trip configurations) from which clients can select and then purchase a ticket.

#### 2.1.1 List Ticket Templates

* **Method:** `GET`
* **URL:** `/api/ticket-templates`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves a list of ticket templates available for purchase. These include details such as origin, destination, departure time, available seats (derived from the assigned bus), and price.
* **Response:**

    ```json
    [
      {
        "templateId": "string",
        "origin": "string",
        "destination": "string",
        "departureTime": "ISO8601_timestamp",
        "assignedBus": {
          "busId": "string",
          "plateNumber": "string",
          "availableSeats": "integer"
        },
        "price": "number"
      }
    ]
    ```

#### 2.1.2 Retrieve Ticket Template Details

* **Method:** `GET`
* **URL:** `/api/ticket-templates/{templateId}`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves details for a specific ticket template.
* **Response:**

    ```json
    {
      "templateId": "string",
      "origin": "string",
      "destination": "string",
      "departureTime": "ISO8601_timestamp",
      "assignedBus": {
        "busId": "string",
        "plateNumber": "string",
        "availableSeats": "integer"
      },
      "price": "number",
      "intermediateStops": ["string"]
    }
    ```

### 2.2 Purchased Tickets

Purchased tickets are the finalized records once a client has completed the purchase process. They are kept for reference and logging.

#### 2.2.1 Issue Purchased Ticket

* **Method:** `POST`
* **URL:** `/api/tickets`
* **Headers:** `Authorization: Bearer`
* **Description:** Issues a purchased ticket to a passenger based on a selected ticket template.
* **Request Body:**

    ```json
    {
      "ticketTemplateId": "string",  // Reference to the ticket template
      "passengerName": "string",
      "phoneNumber": "string",
      "seatNumber": "integer",         // Must be validated against available seats
      "paymentMethod": "string"        // e.g., "MTN", "Airtel"
    }
    ```

* **Response:**

    ```json
    {
      "ticketId": "string",
      "status": "string",  // e.g., "pending", "confirmed"
      "paymentRequestId": "string"
    }
    ```

* **Quirks:**
  * The ticket issuance process triggers a USSD payment authorization window.
  * The seat number chosen must be available on the bus assigned in the ticket template.

#### 2.2.2 Retrieve Purchased Ticket Details

* **Method:** `GET`
* **URL:** `/api/tickets/{ticketId}`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves detailed information about a purchased ticket.
* **Response:**

    ```json
    {
      "ticketId": "string",
      "ticketTemplateId": "string",
      "passengerName": "string",
      "phoneNumber": "string",
      "seatNumber": "integer",
      "origin": "string",
      "destination": "string",
      "departureTime": "ISO8601_timestamp",
      "status": "string"
    }
    ```

#### 2.2.3 Search and Filter Purchased Tickets

* **Method:** `GET`
* **URL:** `/api/tickets`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves a list of purchased tickets filtered by departure time, origin, and destination.
* **Query Parameters:**
  * `departureTime` (optional): ISO8601 timestamp for filtering.
  * `origin` (optional): Filter by origin location.
  * `destination` (optional): Filter by destination.
* **Response:**

    ```json
    [
      {
        "ticketId": "string",
        "passengerName": "string",
        "seatNumber": "integer",
        "origin": "string",
        "destination": "string",
        "departureTime": "ISO8601_timestamp",
        "status": "string"
      }
    ]
    ```

#### 2.2.4 Send Mobile Receipt for Purchased Ticket

* **Method:** `POST`
* **URL:** `/api/tickets/{ticketId}/send-receipt`
* **Headers:** `Authorization: Bearer`
* **Description:** Sends an SMS receipt (invoice) to the passenger’s phone number for a purchased ticket.
* **Response:**

    ```json
    {
      "message": "Receipt sent successfully"
    }
    ```

#### 2.2.5 Print Passenger List for a Trip

* **Method:** `GET`
* **URL:** `/api/tickets/print`
* **Headers:** `Authorization: Bearer`
* **Query Parameter:** `tripId` – Identifier of the trip.
* **Description:** Retrieves a formatted list of passengers (purchased tickets) for a specific trip, intended for driver verification.
* **Response:**

    ```json
    {
      "tripId": "string",
      "passengers": [
        {
          "ticketId": "string",
          "passengerName": "string",
          "seatNumber": "integer"
        }
      ]
    }
    ```

---

## 3. Scheduling Endpoints

### 3.1 Create Trip Schedule

* **Method:** `POST`
* **URL:** `/api/schedules`
* **Headers:** `Authorization: Bearer`
* **Description:** Creates a new trip schedule. Supports:
  * Single-trip scheduling: A one-off trip.
  * Multiple-trip scheduling: Recurring trips separated by a fixed interval.

    The operator may specify one or more buses (via plate numbers) for the trip. If the `assignRandomBus` flag is true, the system will randomly assign a bus from the provided list to each trip, read its seat capacity, and update the ticket template’s available seats.
* **Request Body:**

    ```json
    {
      "origin": "string",
      "destination": "string",
      "departureTime": "ISO8601_timestamp", // For single-trip scheduling
      "plateNumbers": ["string"],            // Optional list of buses available
      "intermediateStops": ["string"],
      "interval": "integer",                 // In minutes; if provided, creates recurring trips
      "assignRandomBus": "boolean"           // If true, system randomly assigns a bus per trip
    }
    ```

* **Response:**

    ```json
    {
      "scheduleId": "string",
      "ticketTemplates": [
        {
          "templateId": "string",
          "origin": "string",
          "destination": "string",
          "departureTime": "ISO8601_timestamp",
          "assignedBus": {
            "busId": "string",
            "plateNumber": "string",
            "availableSeats": "integer"
          },
          "price": "number"
        }
      ],
      "message": "Schedule created successfully"
    }
    ```

* **Quirks:**
  * For recurring trips, the system generates separate ticket templates for each departure time.
  * When using `assignRandomBus`, the endpoint shuffles the bus list and assigns one randomly to each ticket template.
  * Validation must ensure each selected bus has sufficient capacity.

### 3.2 Edit Trip Schedule

* **Method:** `PUT`
* **URL:** `/api/schedules/{scheduleId}`
* **Headers:** `Authorization: Bearer`
* **Description:** Edits an existing schedule. Edits are only allowed if no purchased tickets exist for that schedule.
* **Request Body:**

    ```json
    {
      "origin": "string",
      "destination": "string",
      "departureTime": "ISO8601_timestamp",
      "plateNumbers": ["string"],
      "intermediateStops": ["string"],
      "interval": "integer",
      "assignRandomBus": "boolean"
    }
    ```

* **Response:**

    ```json
    {
      "message": "Schedule updated successfully"
    }
    ```

* **Quirks:**
  * A check is performed to ensure no purchased tickets are linked to the schedule before permitting changes.

### 3.3 List Schedules

* **Method:** `GET`
* **URL:** `/api/schedules`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves trip schedules with optional filters (e.g., date, origin, destination).
* **Query Parameters (optional):**
  * `date`: ISO8601 date to filter.
  * `origin`: Filter by origin.
  * `destination`: Filter by destination.
* **Response:**

    ```json
    [
      {
        "scheduleId": "string",
        "origin": "string",
        "destination": "string",
        "departureTime": "ISO8601_timestamp",
        "ticketTemplate": {
          "templateId": "string",
          "assignedBus": {
            "busId": "string",
            "plateNumber": "string",
            "availableSeats": "integer"
          }
        }
      }
    ]
    ```

---

## 4. Bus Management Endpoints

### 4.1 List Available Buses

* **Method:** `GET`
* **URL:** `/api/buses`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves a list of buses including their plate numbers and seating capacities.
* **Response:**

    ```json
    [
      {
        "busId": "string",
        "plateNumber": "string",
        "totalSeats": "integer",
        "availableSeats": "integer"
      }
    ]
    ```

### 4.2 Assign Bus to Schedule (Optional)

* **Method:** `POST`
* **URL:** `/api/schedules/{scheduleId}/assign-bus`
* **Headers:** `Authorization: Bearer`
* **Description:** Manually assigns a specific bus to an existing schedule if random assignment was not used.
* **Request Body:**

    ```json
    {
      "busId": "string"
    }
    ```

* **Response:**

    ```json
    {
      "message": "Bus assigned successfully",
      "assignedBus": {
        "busId": "string",
        "plateNumber": "string",
        "availableSeats": "integer"
      }
    }
    ```

---

## 5. Payment Endpoints

### 5.1 Initiate Payment

* **Method:** `POST`
* **URL:** `/api/payments`
* **Headers:** `Authorization: Bearer`
* **Description:** Initiates the payment process for a purchased ticket via MTN Mobile Money or Airtel Money. Triggers a USSD authorization prompt.
* **Request Body:**

    ```json
    {
      "ticketId": "string",
      "paymentMethod": "string",  // "MTN" or "Airtel"
      "amount": "number"
    }
    ```

* **Response:**

    ```json
    {
      "paymentId": "string",
      "status": "pending",
      "USSDCode": "string"  // USSD code for mobile payment confirmation
    }
    ```

* **Quirks:**
  * Must support asynchronous callbacks from the mobile money provider to update payment status.

### 5.2 Check Payment Status

* **Method:** `GET`
* **URL:** `/api/payments/{paymentId}`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves the current payment status.
* **Response:**

    ```json
    {
      "paymentId": "string",
      "status": "string",  // e.g., "pending", "successful", "failed"
      "transactionDetails": {}
    }
    ```

---

## 6. EBM Integration Endpoints

### 6.1 Generate EBM Invoice

* **Method:** `POST`
* **URL:** `/api/ebm/invoice`
* **Headers:** `Authorization: Bearer`
* **Description:** Generates an electronic invoice via the Rwanda Revenue Authority’s EBM system for a completed ticket sale.
* **Request Body:**

    ```json
    {
      "ticketId": "string",
      "amount": "number",
      "taxpayerId": "string"
    }
    ```

* **Response:**

    ```json
    {
      "invoiceId": "string",
      "ebmCode": "string",
      "message": "Invoice generated successfully"
    }
    ```

---

## 7. Reporting and Logging Endpoints

### 7.1 Generate Sales Report

* **Method:** `GET`
* **URL:** `/api/reports/sales`
* **Headers:** `Authorization: Bearer`
* **Description:** Generates a report of purchased ticket sales over a specified period.
* **Query Parameters:**
  * `startDate`: ISO8601 date.
  * `endDate`: ISO8601 date.
* **Response:**

    ```json
    {
      "totalTicketsSold": "integer",
      "totalRevenue": "number",
      "reportId": "string"
    }
    ```

### 7.2 Generate Scheduling Report

* **Method:** `GET`
* **URL:** `/api/reports/schedules`
* **Headers:** `Authorization: Bearer`
* **Description:** Provides a report on scheduled trips, including details on assigned buses and current available seats.
* **Query Parameters:** (optional filters such as date range or route)
* **Response:**

    ```json
    {
      "report": [
        {
          "scheduleId": "string",
          "origin": "string",
          "destination": "string",
          "departureTime": "ISO8601_timestamp",
          "assignedBus": {
            "busId": "string",
            "plateNumber": "string",
            "availableSeats": "integer"
          }
        }
      ]
    }
    ```

### 7.3 Workbench Dashboard (Operator)

* **Method:** `GET`
* **URL:** `/api/dashboard/workbench`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves statistics for the workbench in use by an operator (e.g., tickets sold, revenue, routes, trip times).
* **Response:**

    ```json
    {
      "ticketsSold": "integer",
      "totalRevenue": "number",
      "routes": ["string"],
      "tripTimes": ["ISO8601_timestamp"]
    }
    ```

### 7.4 Admin Dashboard

* **Method:** `GET`
* **URL:** `/api/dashboard/admin`
* **Headers:** `Authorization: Bearer`
* **Description:** Provides a centralized dashboard for company administrators, with data aggregated from all branches and workbenches.
* **Response:**

    ```json
    {
      "totalTicketsSold": "integer",
      "totalRevenue": "number",
      "branchData": [
        {
          "branchId": "string",
          "ticketsSold": "integer",
          "revenue": "number"
        }
      ]
    }
    ```

### 7.5 Logging Events

* **Method:** `POST`
* **URL:** `/api/logs`
* **Headers:** `Authorization: Bearer`
* **Description:** Logs events such as ticket sales and schedule modifications for auditing.
* **Request Body:** (Example for a ticket sale)

    ```json
    {
      "eventType": "ticketSale",
      "ticketId": "string",
      "workerId": "string",
      "timestamp": "ISO8601_timestamp"
    }
    ```

* **Response:**

    ```json
    {
      "message": "Log entry created successfully"
    }
    ```

---

## 8. User and Authorization Management Endpoints

### 8.1 List Users (Admin Only)

* **Method:** `GET`
* **URL:** `/api/users`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves a list of all users, including roles and permissions.
* **Response:**

    ```json
    [
      {
        "userId": "string",
        "username": "string",
        "role": "string",   // e.g., "operator", "admin"
        "permissions": {
          "canSchedule": true,
          "canSellTickets": true
        }
      }
    ]
    ```

### 8.2 Update User Permissions (Admin Only)

* **Method:** `PUT`
* **URL:** `/api/users/{userId}/permissions`
* **Headers:** `Authorization: Bearer`
* **Description:** Updates permissions (e.g., scheduling, ticket selling) for a given user.
* **Request Body:**

    ```json
    {
      "canSchedule": "boolean",
      "canSellTickets": "boolean"
    }
    ```

* **Response:**

    ```json
    {
      "message": "User permissions updated successfully"
    }
    ```

---

## 9. Additional Utility Endpoints

### 9.1 Retrieve Location Lists

* **Method:** `GET`
* **URL:** `/api/locations`
* **Headers:** `Authorization: Bearer`
* **Description:** Retrieves available origins and destinations.
* **Response:**

    ```json
    {
      "origins": ["string"],
      "destinations": ["string"]
    }
    ```

### 9.2 Fetch Route Price

* **Method:** `GET`
* **URL:** `/api/prices`
* **Headers:** `Authorization: Bearer`
* **Description:** Returns the price for a given route (based on origin and destination).
* **Query Parameters:**
  * `origin`: string
  * `destination`: string
* **Response:**

    ```json
    {
      "price": "number"
    }
    ```

---

## Summary of Key Considerations

* **Ticket Differentiation:**
  * **Ticket Templates:** Represent available trip models (with dynamic available seats read from the assigned bus). These are used as a basis for purchase.
  * **Purchased Tickets:** Finalized records of completed transactions, stored for reference and logging.
* **Scheduling Flexibility:** Supports both single and recurring trips.
* **Payment Options:**  Integrates with MTN Mobile Money and Airtel Money, using USSD authorization.
* **EBM Compliance:**  Generates electronic invoices for the Rwanda Revenue Authority.
* **Comprehensive Reporting:** Includes sales, scheduling, and dashboard reports for both operators and administrators.
