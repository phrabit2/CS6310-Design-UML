# Object-Oriented Analysis Result: Power Grid Simulation System

## 1. Entity Identification

### 1.1 Noun Extraction and Entity Candidate Selection

| Candidate Noun | Classification | Rationale |
|----------------|----------------|-----------|
| Power Company | **Entity** | Core business object with identity, state, and behavior |
| Customer | **Entity** | Core business object with unique account |
| Power Plant | **Entity** | Physical asset with identity and costs |
| Substation | **Entity** | Physical asset in distribution hierarchy |
| Transformer | **Entity** | Physical asset connecting to customers |
| Employee | **Entity** | Human resource with payroll information |
| Equipment Issue | **Entity** | Event requiring tracking and resolution |
| Location | **Value Object** | Reusable coordinate pair without identity |
| Transaction/Bill | **Entity** | Financial record requiring persistence |
| Billing Cycle | **Entity** | Temporal boundary for financial calculations |

### 1.2 Rejected Candidates

| Noun | Reason for Rejection |
|------|---------------------|
| Power | Attribute (kWh measurement), not an entity |
| Rate | Attribute of PowerCompany |
| Voltage | Implementation detail, not tracked |
| Transmission Line | Cost included in build costs per spec |
| Materials | Attribute of EquipmentIssue |

---

## 2. Entity Specifications

### 2.1 PowerCompany

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| shortName | String | Unique identifier (e.g., "fpl") |
| longName | String | Full company name |
| ratePerKWh | Decimal | Standard charge rate per kWh |
| totalRevenue | Decimal | Accumulated revenue from customers |
| totalCosts | Decimal | Accumulated operational costs |

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| addCustomer() | customer: Customer | void | Register a new customer |
| removeCustomer() | accountNumber: String | void | Remove customer by account |
| addPowerPlant() | plant: PowerPlant | void | Add generation facility |
| addEmployee() | employee: Employee | void | Hire an employee |
| calculateTotalCosts() | - | Decimal | Sum all operational costs |
| calculateProfit() | - | Decimal | Revenue minus costs |
| processBillingCycle() | cycle: BillingCycle | void | Execute billing for all customers |

---

### 2.2 Customer

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| accountNumber | String | Unique identifier assigned by company |
| name | String | Customer name |
| location | Location | Physical address as coordinates |
| customerType | CustomerType | RESIDENTIAL or COMMERCIAL |
| powerUsage | Decimal | Current cycle usage in kWh |
| totalAmountDue | Decimal | Outstanding balance |

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| recordPowerUsage() | kWh: Decimal | void | Add power consumption |
| resetPowerUsage() | - | void | Reset usage for new cycle |
| calculateBill() | rate: Decimal | Decimal | Compute bill based on usage and rate |
| makePayment() | amount: Decimal | void | Process payment |

**Enumeration - CustomerType:**
- RESIDENTIAL
- COMMERCIAL

---

### 2.3 PowerPlant

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| plantId | String | Unique plant identifier |
| location | Location | Physical coordinates |
| buildCost | Decimal | Initial construction cost |
| productionCostPerKWh | Decimal | Recurring cost per kWh produced |
| powerProduced | Decimal | Current cycle production in kWh |

**Constraints:**
- Maximum 20 substations per plant

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| addSubstation() | substation: Substation | Boolean | Connect substation if under limit |
| removeSubstation() | substationId: String | void | Disconnect substation |
| getSubstationCount() | - | Integer | Current substation count |
| canAddSubstation() | - | Boolean | Check if under 20 limit |
| recordPowerProduction() | kWh: Decimal | void | Record power generated |
| calculateProductionCost() | - | Decimal | powerProduced × productionCostPerKWh |
| resetProduction() | - | void | Reset for new billing cycle |

---

### 2.4 Substation

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| substationId | String | Unique substation identifier |
| location | Location | Physical coordinates |
| buildCost | Decimal | Initial construction cost |
| maintenanceCost | Decimal | Fixed recurring cost per cycle |

**Constraints:**
- Maximum 10 transformers per substation
- Must be within MAX_PLANT_DISTANCE from connected PowerPlant

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| addTransformer() | transformer: Transformer | Boolean | Connect transformer if under limit |
| removeTransformer() | transformerId: String | void | Disconnect transformer |
| getTransformerCount() | - | Integer | Current transformer count |
| canAddTransformer() | - | Boolean | Check if under 10 limit |
| getMaintenanceCost() | - | Decimal | Return fixed maintenance cost |

---

### 2.5 Transformer

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| transformerId | String | Unique transformer identifier |
| location | Location | Physical coordinates |
| installationCost | Decimal | Initial installation cost |
| maintenanceCost | Decimal | Fixed recurring cost per cycle |

**Constraints:**
- Maximum 5 customers per transformer
- Must be within MAX_SUBSTATION_DISTANCE from connected Substation

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| addCustomer() | customer: Customer | Boolean | Connect customer if under limit |
| removeCustomer() | accountNumber: String | void | Disconnect customer |
| getCustomerCount() | - | Integer | Current customer count |
| canAddCustomer() | - | Boolean | Check if under 5 limit |
| getMaintenanceCost() | - | Decimal | Return fixed maintenance cost |

---

### 2.6 Employee

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| employeeId | String | Unique employee identifier |
| name | String | Employee full name |
| startDate | Date | Employment start date |
| hourlyWage | Decimal | Pay rate per hour |
| isAvailable | Boolean | Availability for dispatch |

**Constraints:**
- Employee can work for only one company at a time

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| dispatch() | issue: EquipmentIssue | void | Assign to equipment issue |
| completeWork() | - | void | Mark as available after repair |
| calculateLaborCost() | hours: Decimal | Decimal | hours × hourlyWage |

---

### 2.7 EquipmentIssue

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| issueId | String | Unique issue identifier |
| equipmentType | EquipmentType | Type of affected equipment |
| equipmentId | String | ID of affected equipment |
| hoursRequired | Decimal | Estimated hours to complete |
| materialsCost | Decimal | Cost of replacement parts |
| status | IssueStatus | Current issue status |
| reportedDate | Date | When issue was reported |
| resolvedDate | Date | When issue was resolved (nullable) |

**Enumeration - EquipmentType:**
- POWER_PLANT
- SUBSTATION
- TRANSFORMER

**Enumeration - IssueStatus:**
- REPORTED
- DISPATCHED
- IN_PROGRESS
- RESOLVED

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| assignEmployee() | employee: Employee | void | Dispatch employee to issue |
| calculateTotalCost() | - | Decimal | (hoursRequired × employee.hourlyWage) + materialsCost |
| markResolved() | - | void | Update status and set resolved date |
| getAssignedEmployee() | - | Employee | Return dispatched employee |

---

### 2.8 Location (Value Object)

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| x | Integer | X coordinate |
| y | Integer | Y coordinate |

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| manhattanDistance() | other: Location | Integer | |x1-x2| + |y1-y2| |
| isWithinDistance() | other: Location, maxDist: Integer | Boolean | Check if within max distance |

---

### 2.9 BillingCycle

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| cycleId | String | Unique cycle identifier |
| startDate | Date | Cycle start date |
| endDate | Date | Cycle end date |
| status | CycleStatus | Current cycle status |

**Enumeration - CycleStatus:**
- ACTIVE
- CLOSED
- PROCESSING

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| closeCycle() | - | void | End current billing cycle |
| isActive() | - | Boolean | Check if cycle is active |

---

### 2.10 Transaction

**Attributes:**
| Name | Type | Description |
|------|------|-------------|
| transactionId | String | Unique transaction identifier |
| transactionDate | Date | Date of transaction |
| amount | Decimal | Transaction amount |
| transactionType | TransactionType | Type of transaction |
| description | String | Transaction details |

**Enumeration - TransactionType:**
- CUSTOMER_BILL
- CUSTOMER_PAYMENT
- MAINTENANCE_COST
- PRODUCTION_COST
- BUILD_COST
- REPAIR_COST

**Operations:**
| Name | Parameters | Return | Description |
|------|------------|--------|-------------|
| isRevenue() | - | Boolean | Check if transaction is revenue |
| isCost() | - | Boolean | Check if transaction is cost |

---

## 3. Relationships

### 3.1 Association Summary

| From | To | Relationship Type | From Multiplicity | To Multiplicity | Description |
|------|----|-------------------|-------------------|-----------------|-------------|
| PowerCompany | Customer | Composition | 1 | 0..* | Company has many customers |
| PowerCompany | PowerPlant | Composition | 1 | 1..* | Company owns plants |
| PowerCompany | Employee | Composition | 1 | 1..* | Company employs workers |
| PowerCompany | BillingCycle | Association | 1 | 0..* | Company runs billing cycles |
| PowerCompany | Transaction | Composition | 1 | 0..* | Company records transactions |
| PowerCompany | EquipmentIssue | Composition | 1 | 0..* | Company tracks equipment issues |
| PowerPlant | Substation | Aggregation | 1 | 0..20 | Plant feeds substations |
| Substation | Transformer | Aggregation | 1 | 0..10 | Substation feeds transformers |
| Transformer | Customer | Association | 1 | 0..5 | Transformer serves customers |
| Employee | EquipmentIssue | Association | 0..1 | 0..1 | Employee assigned to issue |
| EquipmentIssue | PowerPlant | Association | 0..* | 0..1 | Issue affects plant |
| EquipmentIssue | Substation | Association | 0..* | 0..1 | Issue affects substation |
| EquipmentIssue | Transformer | Association | 0..* | 0..1 | Issue affects transformer |
| Customer | Transaction | Association | 1 | 0..* | Customer has billing transactions |

### 3.2 Relationship Details (PlantUML Compatible Notation)

```
PowerCompany "1" *-- "0..*" Customer : customers
PowerCompany "1" *-- "1..*" PowerPlant : powerPlants
PowerCompany "1" *-- "1..*" Employee : employees
PowerCompany "1" *-- "0..*" Transaction : transactions
PowerCompany "1" *-- "0..*" EquipmentIssue : equipmentIssues
PowerCompany "1" -- "0..*" BillingCycle : billingCycles

PowerPlant "1" o-- "0..20" Substation : substations
Substation "1" o-- "0..10" Transformer : transformers
Transformer "1" -- "0..5" Customer : customers

Employee "0..1" -- "0..1" EquipmentIssue : assignedIssue
EquipmentIssue "0..*" -- "0..1" PowerPlant : affectedPlant
EquipmentIssue "0..*" -- "0..1" Substation : affectedSubstation
EquipmentIssue "0..*" -- "0..1" Transformer : affectedTransformer

Customer "1" -- "0..*" Transaction : transactions
```

### 3.3 Bidirectional Navigation

| Relationship | Forward Navigation | Reverse Navigation |
|--------------|-------------------|-------------------|
| PowerCompany-Customer | company.getCustomers() | customer.getCompany() |
| PowerCompany-PowerPlant | company.getPowerPlants() | plant.getCompany() |
| PowerPlant-Substation | plant.getSubstations() | substation.getPowerPlant() |
| Substation-Transformer | substation.getTransformers() | transformer.getSubstation() |
| Transformer-Customer | transformer.getCustomers() | customer.getTransformer() |
| Employee-EquipmentIssue | employee.getAssignedIssue() | issue.getAssignedEmployee() |

---

## 4. Constraints Summary

### 4.1 Business Rules

| ID | Constraint | Entity |
|----|------------|--------|
| C1 | shortName must be unique within the system | PowerCompany |
| C2 | accountNumber must be unique within a company | Customer |
| C3 | Maximum 20 substations per power plant | PowerPlant |
| C4 | Maximum 10 transformers per substation | Substation |
| C5 | Maximum 5 customers per transformer | Transformer |
| C6 | Employee can work for only one company | Employee |
| C7 | Employee can be assigned to at most one active issue | Employee |

### 4.2 Distance Constraints

| ID | Constraint | Affected Entities |
|----|------------|-------------------|
| D1 | Substation must be within MAX_PLANT_DISTANCE from PowerPlant | PowerPlant, Substation |
| D2 | Transformer must be within MAX_SUBSTATION_DISTANCE from Substation | Substation, Transformer |
| D3 | Customer must be within MAX_TRANSFORMER_DISTANCE from Transformer | Transformer, Customer |

### 4.3 Referential Integrity

| ID | Constraint |
|----|------------|
| R1 | Customer must be connected to exactly one Transformer |
| R2 | Transformer must be connected to exactly one Substation |
| R3 | Substation must be connected to exactly one PowerPlant |
| R4 | PowerPlant must belong to exactly one PowerCompany |
| R5 | EquipmentIssue must reference exactly one equipment (Plant, Substation, or Transformer) |

---

## 5. System Constants

| Constant | Type | Description |
|----------|------|-------------|
| MAX_SUBSTATIONS_PER_PLANT | Integer | 20 |
| MAX_TRANSFORMERS_PER_SUBSTATION | Integer | 10 |
| MAX_CUSTOMERS_PER_TRANSFORMER | Integer | 5 |
| MAX_PLANT_TO_SUBSTATION_DISTANCE | Integer | TBD |
| MAX_SUBSTATION_TO_TRANSFORMER_DISTANCE | Integer | TBD |
| MAX_TRANSFORMER_TO_CUSTOMER_DISTANCE | Integer | TBD |

---

## 6. Derived Attributes and Calculations

### 6.1 PowerCompany Calculations

```
totalRevenue = SUM(transactions WHERE type = CUSTOMER_PAYMENT)
totalCosts = SUM(transactions WHERE type IN (MAINTENANCE_COST, PRODUCTION_COST, BUILD_COST, REPAIR_COST))
profit = totalRevenue - totalCosts
```

### 6.2 Customer Bill Calculation

```
billAmount = powerUsage × company.ratePerKWh
```

### 6.3 Equipment Issue Cost Calculation

```
totalRepairCost = (hoursRequired × assignedEmployee.hourlyWage) + materialsCost
```

### 6.4 Power Plant Production Cost

```
cycleCost = powerProduced × productionCostPerKWh
```

---

## 7. Interface Definitions (Abstract Types)

### 7.1 Equipment (Interface)

Shared interface for PowerPlant, Substation, and Transformer.

**Attributes:**
| Name | Type |
|------|------|
| id | String |
| location | Location |
| buildCost | Decimal |

**Operations:**
| Name | Return |
|------|--------|
| getId() | String |
| getLocation() | Location |
| getBuildCost() | Decimal |

### 7.2 Maintainable (Interface)

For entities with recurring maintenance costs.

**Operations:**
| Name | Return |
|------|--------|
| getMaintenanceCost() | Decimal |
