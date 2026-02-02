# Power Grid Simulation System - Project Overview

## 1. Scenario Summary

This project involves designing and developing a **Power Grid Simulation System** that models the interactions between power companies and their customers. The system tracks:

- Power generation and distribution infrastructure
- Power production and consumption
- Cost tracking (production, distribution, maintenance, consumption)
- Financial transactions between customers and power companies
- Equipment failures and repair operations
- Geographic locations of all entities

The primary goal is to provide comprehensive reporting on customer costs and company revenue/expenses.

---

## 2. Main Entities

### 2.1 PowerCompany
**Description**: Represents an electrical power company that generates and distributes electricity to customers.

**Attributes**:
- `longName`: String - Full company name (e.g., "Florida Power & Light Company")
- `shortName`: String - Abbreviated name (e.g., "fpl") - **Must be unique**
- `standardRate`: Decimal - Rate charged per kWh (e.g., $0.15/kWh)
- `totalCosts`: Decimal - Aggregate of all company costs
- `totalRevenue`: Decimal - Total revenue from customer billing

**Operations**:
- `addCustomer(customer)`: Register a new customer
- `addPowerPlant(plant)`: Add a power generation plant
- `calculateRevenue()`: Compute total revenue from all customers
- `calculateTotalCosts()`: Sum all operational and maintenance costs
- `generateBill(customer, billingCycle)`: Create customer bill

**Relationships**:
- Has many `Customer` (1:N)
- Has many `PowerPlant` (1:N)
- Has many `Employee` (1:N)

---

### 2.2 Customer
**Description**: Entity consuming electricity from a power company.

**Attributes**:
- `accountNumber`: String - Unique identifier assigned by power company
- `name`: String - Customer name
- `location`: Location - Geographic coordinates
- `customerType`: Enum {Residential, Commercial}
- `powerUsage`: Decimal - Total power consumed (kWh)

**Operations**:
- `recordPowerUsage(amount)`: Log power consumption
- `calculateBill(rate, billingCycle)`: Compute bill based on usage and rate
- `getLocation()`: Return customer location

**Relationships**:
- Belongs to one `PowerCompany` (N:1)
- Connected to one `Transformer` (N:1)

**Constraints**:
- Must be within maximum distance from assigned `Transformer`

---

### 2.3 PowerPlant
**Description**: Facility that generates electrical power.

**Attributes**:
- `plantID`: String - Unique identifier
- `location`: Location - Geographic coordinates
- `initialCost`: Decimal - One-time construction cost
- `recurringCost`: Decimal - Ongoing production cost (varies by power produced)
- `powerProduced`: Decimal - Total power generated (kWh)

**Operations**:
- `generatePower(amount)`: Produce power
- `calculateRecurringCost(billingCycle)`: Compute production costs based on output
- `connectSubstation(substation)`: Link to a substation

**Relationships**:
- Belongs to one `PowerCompany` (N:1)
- Connects to multiple `Substation` (1:N, max 20)

**Constraints**:
- Can support **maximum 20 substations**
- Substations must be within **maximum Manhattan distance XX**

---

### 2.4 Substation
**Description**: Intermediate facility that steps down high-voltage power for local distribution.

**Attributes**:
- `substationID`: String - Unique identifier
- `location`: Location - Geographic coordinates
- `initialCost`: Decimal - One-time construction cost
- `recurringMaintenanceCost`: Decimal - Fixed maintenance cost per billing cycle

**Operations**:
- `connectTransformer(transformer)`: Link to a transformer
- `calculateMaintenanceCost(billingCycle)`: Return fixed maintenance cost

**Relationships**:
- Receives power from one `PowerPlant` (N:1)
- Connects to multiple `Transformer` (1:N, max 10)

**Constraints**:
- Can support **maximum 10 transformers**
- Transformers must be within **maximum Manhattan distance XX**

---

### 2.5 Transformer
**Description**: Local equipment that performs final voltage step-down before customer connection.

**Attributes**:
- `transformerID`: String - Unique identifier
- `location`: Location - Geographic coordinates
- `initialCost`: Decimal - One-time installation cost
- `recurringMaintenanceCost`: Decimal - Fixed maintenance cost per billing cycle

**Operations**:
- `connectCustomer(customer)`: Link to a customer
- `calculateMaintenanceCost(billingCycle)`: Return fixed maintenance cost

**Relationships**:
- Connected to one `Substation` (N:1)
- Serves multiple `Customer` (1:N, max 5)

**Constraints**:
- Can support **maximum 5 customers**
- Customers must be within **maximum Manhattan distance XX**

---

### 2.6 Employee
**Description**: Worker employed by a power company to perform equipment repairs.

**Attributes**:
- `employeeID`: String - Unique identifier
- `name`: String - Employee name
- `startDate`: Date - Employment start date
- `hourlyWage`: Decimal - Pay rate per hour

**Operations**:
- `dispatchToIssue(issue)`: Assign employee to repair task
- `calculateRepairCost(hoursRequired)`: Compute labor cost for repair

**Relationships**:
- Works for one `PowerCompany` (N:1)
- Can be dispatched to multiple `EquipmentIssue` (1:N)

**Constraints**:
- Can only work for **one company at a time**

---

### 2.7 EquipmentIssue
**Description**: Record of equipment failure requiring repair.

**Attributes**:
- `issueID`: String - Unique identifier
- `equipmentType`: Enum {PowerPlant, Substation, Transformer}
- `equipmentID`: String - Reference to failed equipment
- `hoursRequired`: Decimal - Time needed to complete repair
- `materialCost`: Decimal - Cost of materials for repair
- `assignedEmployee`: Employee - Employee dispatched for repair
- `totalCost`: Decimal - Labor cost + material cost

**Operations**:
- `assignEmployee(employee)`: Dispatch employee to issue
- `calculateTotalCost()`: Compute (hoursRequired × employee.hourlyWage) + materialCost
- `resolveIssue()`: Mark issue as repaired

**Relationships**:
- Related to one equipment entity (PowerPlant/Substation/Transformer)
- Assigned to one `Employee` (N:1)

---

### 2.8 Location
**Description**: Two-dimensional coordinate system for geographic positioning.

**Attributes**:
- `x`: Integer - X coordinate
- `y`: Integer - Y coordinate

**Operations**:
- `calculateManhattanDistance(otherLocation)`: Return |x1 - x2| + |y1 - y2|

**Relationships**:
- Used by `Customer`, `PowerPlant`, `Substation`, `Transformer`

---

### 2.9 BillingCycle
**Description**: Represents a billing period for cost/revenue calculations.

**Attributes**:
- `cycleID`: String - Unique identifier
- `startDate`: Date - Beginning of billing period
- `endDate`: Date - End of billing period

**Operations**:
- `processCustomerBills()`: Generate bills for all customers
- `applyRecurringCosts()`: Apply maintenance and production costs
- `calculateCompanyRevenue()`: Sum all customer payments

**Relationships**:
- Associated with multiple transactions and cost records

---

## 3. Key Relationships Summary
```
PowerCompany (1) ──── (N) Customer
PowerCompany (1) ──── (N) PowerPlant
PowerCompany (1) ──── (N) Employee

PowerPlant (1) ──── (N≤20) Substation
Substation (1) ──── (N≤10) Transformer
Transformer (1) ──── (N≤5) Customer

Employee (N) ──── (N) EquipmentIssue
```

---

## 4. Constraints and Business Rules

### 4.1 Capacity Constraints
- **PowerPlant**: Maximum 20 substations
- **Substation**: Maximum 10 transformers
- **Transformer**: Maximum 5 customers

### 4.2 Distance Constraints (Manhattan Distance)
- PowerPlant ↔ Substation: ≤ XX units
- Substation ↔ Transformer: ≤ XX units
- Transformer ↔ Customer: ≤ XX units

### 4.3 Uniqueness Constraints
- PowerCompany `shortName` must be unique system-wide
- Customer `accountNumber` must be unique per company
- All equipment IDs must be unique

### 4.4 Employment Rules
- Employee can work for only **one company at a time**

### 4.5 Cost Calculation Rules
- **Customer Bill** = powerUsage × company.standardRate
- **Repair Cost** = (hoursRequired × employee.hourlyWage) + materialCost
- **Plant Recurring Cost** = Function of power produced (applied per billing cycle)
- **Substation/Transformer Maintenance** = Fixed cost per billing cycle

---

## 5. System Operations Requirements

Users must be able to:

1. **Create and Manage Entities**
   - Create power companies with components
   - Create customers and employees

2. **Simulate Operations**
   - Simulate equipment issues
   - Initiate dispatch procedures for repairs

3. **Record and Display Metrics**
   - Track power usage (customers) and production (companies)
   - Display costs for customers
   - Display costs and revenue for companies

4. **Financial Tracking**
   - Process billing cycles
   - Calculate customer bills
   - Track company revenue and total costs

---

## 6. Notes for Implementation

- **Location System**: Use integer coordinates (X, Y) with Manhattan distance calculation
- **Cost Tracking**: Distinguish between one-time (initial) and recurring costs
- **Billing**: All customer billing and recurring costs occur each billing cycle
- **Transmission Line Costs**: Included in build costs for substations and transformers
- **Customer Classification**: Support both Residential and Commercial types

---

## 7. Next Steps

1. **Design Questions**: Address any ambiguities in requirements
2. **OOA (Object-Oriented Analysis)**: 
   - Refine entity definitions
   - Identify additional attributes/operations
   - Validate relationships and constraints
3. **Class Diagram**: Create UML class diagram with all entities, attributes, operations, and relationships
4. **Sequence Diagrams**: Model key scenarios (e.g., customer billing, equipment repair dispatch)