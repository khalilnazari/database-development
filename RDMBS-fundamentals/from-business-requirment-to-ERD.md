## Database Design: From Business Requirements to ER Diagram

When developing the databse for a web app the most fundamental phase the following.

---

### Step 1: Gather Business Requirements

**Goal:** Understand what the system needs to do before touching any technical design.

- Meet with stakeholders (clients, managers, end users, domain experts)
- Conduct interviews, workshops, and questionnaires
- Review existing documents — forms, reports, spreadsheets, legacy systems
- Ask key questions:
  - What data needs to be stored?
  - Who will use the system and how?
  - What business rules and constraints exist?
  - What are the inputs and outputs?
- Document everything in a **Business Requirements Document (BRD)**

**Tip:**

- Never assume — ambiguous requirements are the #1 cause of poor database design. Always validate your understanding back with stakeholders.
- Document each requirement as a clear, measurable statement (e.g., “system must track customer orders and shipments”).

---

### Step 2: Analyze and Define the Scope

**Goal:** Narrow down what falls inside and outside the database's responsibility.

- Identify the **core business processes** the database must support
- Define system boundaries — what is in scope vs. out of scope
- Prioritize requirements (must-have vs. nice-to-have)
- Resolve conflicting requirements between departments or stakeholders
- Produce a **scope document** or functional specification

**Tip:**

- A tightly scoped design is easier to model correctly. Scope creep at this stage leads to over-engineered databases.

---

### Step 3: Identify Entities

**Goal:** Extract the "things" the business needs to track.

- Read through requirements and highlight/extract **nouns** — these are candidate entities (e.g., Customer, Order, Product, Employee)
- An entity is any real-world object or concept with data worth storing
- Distinguish between:
  - **Strong entities** — exist independently (e.g., Customer)
  - **Weak entities** — depend on another entity to exist (e.g., OrderItem depends on Order)
- Eliminate duplicates and synonyms (e.g., "Client" and "Customer" may be the same entity)

**Tip:**

- If something is described by multiple pieces of data, it's likely an entity. If it's a single value, it's likely an attribute.
- Avoid ambiguous or overly broad nouns; focus on persistent, significant objects.

---

### Step 4-a: Identify Attributes

**Goal:** Define the data that describes each entity.

- For each entity, ask: _"What information do we need to store about it?"_
- List properties that describe each entity.
- Classify attributes:
  - **Simple** — atomic values (e.g., FirstName, Price)
  - **Composite** — made up of parts (e.g., FullAddress → Street, City, PostalCode)
  - **Derived** — calculated from other data (e.g., Age derived from DateOfBirth)
  - **Multi-valued** — can hold multiple values (e.g., PhoneNumbers)

**Tip:**

- Prefer simple, atomic attributes. Avoid storing composite or multi-valued attributes directly — they are often a sign of poor normalization.
- Ensure attributes are atomic (no repeating groups) and directly tied to the entity.

### Step 4-b: Determine keys

- Identify the **primary key** for each entity — a unique identifier (e.g., CustomerID, OrderID)
- Identify ohter unique identifiers (natural or surrogate) for each entity needed.
- Identify and document super key and candidate key.
- Specify the Foreign keys and correct references.

**Tip:**

- Prefer stable, minimal keys; avoid using changeable attributes (e.g., Name).

---

### Step 5: Identify Relationships

**Goal:** Define how entities are connected to each other.

- Read requirements for **verbs** — these indicate relationships (e.g., Customer _places_ Order, Employee _manages_ Department)
- For each relationship, determine:
  - Determine **Cardinality** — the numeric nature of the relationship:
    - One-to-One (1:1)
    - One-to-Many (1:N)
    - Many-to-Many (M:N)
  - **Participation** — is involvement mandatory or optional?
    - **Total participation** — every instance must participate (e.g., every OrderItem must belong to an Order)
    - **Partial participation** — involvement is optional (e.g., an Employee may or may not manage a Department)
- Name each relationship clearly and meaningfully

**Tip:**

- Many-to-Many relationships will need to be resolved into a **junction/associative entity** during logical design (e.g., Student–Course becomes an Enrollment entity).
- Use verb phrases from business rules to name relationships.

### Important notes to keep in mind

- Resolve many-to-many (M:N) relationships
- Create an associative entity (bridge table) with its own attributes (e.g., OrderProduct with Quantity).
- Name the associative entity after the business transaction (e.g., Enrollment, Assignment).
- Apply normalization (up to 3NF)
- Remove repeating groups (1NF), partial dependencies (2NF), and transitive dependencies (3NF).
- Validate each non-key attribute depends on “the key, the whole key, and nothing but the key.”

---

### Step 6: Apply Business Rules and Constraints

**Goal:** Capture rules that govern the data and its relationships.

- Document constraints such as:
  - A product must belong to at least one category
  - An order cannot exist without a customer
  - An employee can only manage one department at a time
- Identify **domain constraints** — valid values for attributes (e.g., Status can only be 'Active' or 'Inactive')
- Note **referential integrity rules** — what happens when a parent record is deleted?

**Tip:**

- Business rules that seem minor at this stage often become critical constraints in the schema. Capture all of them explicitly.

---

### Step 7: Create the Conceptual Data Model

**Goal:** Produce a high-level, technology-agnostic model of the data.

- Map out all entities, attributes, and relationships in abstract form
- No concern yet for tables, data types, or specific DBMS
- Use this model to **validate understanding with stakeholders** — it should be readable by non-technical people
- Common output: a simple entity-relationship overview or a data model diagram without technical notation

**Tip:**

- The conceptual model is your communication tool. If a business stakeholder can't understand it, simplify it before moving forward.
- Use rectangles for entities, diamonds for relationships (optional in modern notation), ovals for attributes, and lines for connections.
- Keep it high-level; show only major entities and relationships without implementation details.
- Refine the logical ER diagram; add all attributes, specify data types and nullability, mark primary and foreign keys. Resolve subtypes (if any).
- Use crow’s foot notation for clarity on cardinality and optionality.

---

### Step 8: Create the ER Diagram (Logical Data Model)

**Goal:** Translate the conceptual model into a formal, detailed ER Diagram.

- Use standard ER notation (Chen notation or Crow's Foot notation — pick one and stay consistent)
- Include in the diagram:
  - All **entities** (rectangles)
  - All **attributes** (ovals or listed inside entity boxes)
  - **Primary keys** (underlined or marked with PK)
  - All **relationships** (diamond shapes or lines with labels)
  - **Cardinality and participation** symbols on relationship lines
- Resolve Many-to-Many relationships into junction entities with their own keys and attributes
- Ensure every entity has a clearly defined primary key
- Review the diagram against the original business requirements to confirm completeness

**Tip:**

- Walk through real business scenarios (e.g., "a customer places an order with multiple items") against your ER diagram to verify it actually supports the workflow end to end.

---

### Step 9: Validate against business requirements

Walk through scenarios (e.g., “can we record a customer without an order?”) with stakeholders.

**Tip:**

- Check that every requirement has a traceable path to entities, attributes, or relationships.

---

### Quick Reference Checklist

| Step                         | Deliverable                          |
| ---------------------------- | ------------------------------------ |
| Gather Business Requirements | Business Requirements Document (BRD) |
| Analyze & Define Scope       | Scope / Functional Specification     |
| Identify Entities            | Entity list with descriptions        |
| Identify Attributes          | Attribute list per entity with keys  |
| Identify Relationships       | Relationship matrix with cardinality |
| Apply Business Rules         | Constraints and rules document       |
| Conceptual Data Model        | High-level diagram for stakeholders  |
| ER Diagram                   | Formal logical ER Diagram            |
