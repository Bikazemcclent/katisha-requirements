# Katisha Online Bus Operator - Structured Requirements Gathering Process

## 1. Introduction

### 1.1 Purpose
This document defines a **bottom-up** approach for gathering functional requirements for the **Katisha Online Bus Operator Software**. Rather than listing final requirements, it details a **step-by-step protocol** to ensure all system features are captured efficiently, reducing confusion and ambiguity.

### 1.2 Overview
The process follows these structured steps:
1. **Identify smallest system components**: User actions and system events.
2. **Group these into functional requirements** (features).
3. **Expand into use cases and data flows**.
4. **Document sequence diagrams for deeper interactions**.

Each step builds upon the previous one, ensuring a logical, structured process.

---
## 2. Bottom-Up Steps for Requirements Gathering

### Step 1: Identify User Actions & System Events

**Goal:** Capture every possible action the **Bus Agency Operator** performs and how the system responds.

| **User Action** | **System Event (Response)** |
|----------------|-----------------------------|
| Operator logs in | System validates credentials, grants/denies access |
| Operator adds a bus schedule | System stores schedule and confirms addition |
| Operator updates a route | System updates the database and confirms change |
| Operator cancels a booking | System removes the booking, updates availability |
| Operator generates a report | System fetches and displays report data |

📌 *This is the smallest unit of requirement gathering. Every feature originates from these interactions.*

---
### Step 2: Group Actions into Functional Features

Each set of related **user actions + system events** forms a **feature**:

1. **User Management** (Login, Logout, Password Reset)
2. **Bus Schedule Management** (Add, Edit, Remove Schedules)
3. **Route Management** (Create, Update, Delete Routes)
4. **Booking Management** (View, Modify, Cancel Bookings)
5. **Reporting** (Generate, Export Operational Reports)

📌 *Now we have an organized list of **features**, each linked to the smallest user actions and system responses.*

---
### Step 3: Expand Features into Use Cases

Each **feature** is now expanded into detailed **Use Cases**:

#### Example: "Manage Bus Schedules" Use Case

- **Actor:** Bus Agency Operator
- **Precondition:** Operator is logged in.
- **Main Flow:**
  1. Operator selects "Bus Schedules."
  2. System displays existing schedules.
  3. Operator clicks "Add New Schedule."
  4. System prompts for details (bus number, route, time, etc.).
  5. Operator submits the schedule.
  6. System validates and saves the data.
  7. System confirms success.
- **Alternative Flow:** If data is invalid, system displays errors for correction.

📌 *We now have structured, repeatable steps for every feature.*

---
### Step 4: Create User Flow Diagrams

We take the **Use Cases** and represent them visually.

#### Example: "Adding a Bus Schedule" Flow

1️⃣ Operator logs in → 2️⃣ Navigates to "Schedules" → 3️⃣ Clicks "Add Schedule"
→ 4️⃣ Enters details → 5️⃣ Clicks "Submit" → 6️⃣ System validates & saves

🛑 If data is incorrect → Operator gets error message → Corrects data → Resubmits

📌 *This step helps to catch gaps and ensure completeness.*

---
### Step 5: Document System Interaction with Sequence Diagrams

#### Example: "Processing a Ticket Sale" Sequence Diagram

1. Operator initiates ticket sale.
2. UI sends sale request to Ticketing System.
3. Ticketing System verifies availability.
4. Ticketing System processes payment.
5. System updates database and confirms sale.
6. Operator receives success confirmation.

📌 *Now, we know **how** features interact with the system, ensuring no missing pieces.*

---
## 3. Finalizing Requirements

After following **Steps 1-5**, we now have:
✅ A full list of **functional requirements**
✅ Use cases with detailed **step-by-step flows**
✅ **Diagrams** showing user interactions and system behavior

This ensures:
- **Nothing is missed** (since we started from user actions and system responses).
- **Developers can implement clearly defined features**.
- **Stakeholders understand how the system works**.

---
## 4. Summary

📌 **Bottom-Up Process Recap:**
1️⃣ **List user actions & system events** → Smallest building blocks
2️⃣ **Group into functional features** → High-level structure
3️⃣ **Expand into detailed use cases** → Clear steps for each function
4️⃣ **Create user flows** → Visual clarity & validation
5️⃣ **Build sequence diagrams** → Deep system interaction mapping

This structured method ensures **a complete, error-free requirements document** for the Katisha Online Bus Operator software.

---
## 5. Next Steps
- 🛠 Conduct stakeholder review for validation
- 🔍 Iterate and refine based on feedback
- 📖 Prepare final Software Requirements Specification (SRS)

