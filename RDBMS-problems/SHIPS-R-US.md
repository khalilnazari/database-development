#Problem
Read the prolem description here: [SHIPS R US link](https://github.com/StructuredCS/awesome-database-design-problems/blob/main/problems/SHIPS-R-US.md)

## Identifying Entities and Attributes

1. Brand
   Represents spaceship brands.
   name
   description

2. Model
   Represents spaceship models.
   model_number
   name

3. Spaceship
   Represents manufactured spaceships.
   serial_number
   recommended_retail_price
   manufacturing_year

4. Dealer
   Represents dealers who distribute spaceships.
   email
   name
   phone_number
   website_url
   address
   street_address
   city
   suburb
   state
   postal_code
   country_name
   country_code

5. Customer
   Represents customers who purchase spaceships.
   customer_id
   email_address
   phone_number

## Specifying the keys

1. Brand
   id - PK
   name
   descriptoin

2. Model
   model_number - PK (unique and not null)
   name

3. Spaceship
   serial_number - PK (Unique and not null)
   recommended_retail_price
   manufacturing_year

4. Dealer
   delear_id - PK
   email - unique
   name
   phone_number
   website_url
   street_address
   city
   suburb
   state
   postal_code
   country_name
   country_code

5. Customer
   customer_id - PK(unique and not null)
   email_address
   phone_number

## Creating the Relations between entities

**One-To-Many**

- Brand (1) ──────< Model (many)
  A brand is associated with multiple models; each model belongs to exactly one brand.

  FK - Model.brand_id references Brand.brand_id

- Model (1) ──────< Spaceship (many)
  A model is associated with multiple spaceships; each spaceship belongs to exactly one model.

  FK - Spaceship.model_number references Model.model_number

- Dealer (1) ──────< Spaceship (0..1 per spaceship, but many spaceships per dealer)
  One or more spaceships are distributed to a dealer; a spaceship can be associated with at most one dealer.

  FK - Spaceship.dealer_email references Dealer.email (nullable)

- Customer (1) ──────< Spaceship (0..1 per spaceship, but many spaceships per customer)
  A customer owns one or more spaceships; a spaceship can have at most one owner (customer).

  FK - Spaceship.customer_id references Customer.customer_id (nullable)

**Many-To-Many**

- Dealer (many) ──────< Customer_Dealer >────── (many) Customer
  A dealer serves one or more customers; a customer is served by one or more dealers.

  FK - Junction table Customer_Dealer with foreign keys to Dealer.email and Customer.customer_id

  Entity Customer_Dealer (junction table) attributes:
  customer_id (foreign key to Customer)
  dealer_email (foreign key to Dealer)
  Composite primary key: (customer_id, dealer_email)

- Constraints
  - Spaceship has two optional foreign keys:
  - dealer_email → nullable (spaceship may be unsold or not yet distributed).
  - customer_id → nullable (spaceship may be owned by the dealer or unsold).
  - Customer and Dealer allow updates to email and phone number as stated, but primary keys must remain stable. Therefore, Dealer.email is the primary key (as stated unique), but if email is updatable, that would break referential integrity. To satisfy both the unique requirement and update allowance, a surrogate dealer_id should be introduced (though not explicitly asked). However, strictly following the given data, we keep email as PK and assume updates cascade or are not allowed; the interview may imply logical updates that preserve identity. For practical database design, we note this tension.
  - Customer uses a surrogate customer_id because no natural unique identifier was provided; email and phone are updatable and thus not suitable as PK.

## Normalization

First of all, let review the unormalized DB entities

SPACESHIP_RECORD
( serial_number, recommended_retail_price, manufacturing_year, model_name, model_number, brand_name, brand_description, dealer_name, dealer_email, dealer_phone, dealer_website, dealer_street, dealer_city, dealer_suburb, dealer_state, dealer_postal_code, dealer_country_name, dealer_country_code, customer_email, customer_phone )

Each spaceship has at most one dealer and at most one customer (owner).
A dealer may distribute many spaceships → dealer information repeats.
A customer may own many spaceships → customer information repeats.
No repeating groups inside a single tuple because each spaceship has single dealer and single customer. However, the many-to-many relationship between Dealer and Customer is not directly visible here; it is implied through spaceships. But for normalization, we treat this as a flat table.

### 1NF

In 1NF let's eliminate Repeating Groups, Define Primary Key
In 1NF, each attribute must be atomic, and there must be a unique identifier.

The above UNF has no repeating groups (all attributes are atomic). However, the candidate key is serial_number because each spaceship is unique. But customer_email and customer_phone can be NULL (if spaceship has no owner).
So the 1NF relation is:

SPACESHIP_1NF
Primary Key: serial_number
Attributes: all the same as UNF.

No structural change from UNF to 1NF in this case, but we explicitly define the key.

### 2NF

In 2NF let's remove Partial Dependencies
2NF requires that every non-prime attribute be fully functionally dependent on the whole primary key. Since the primary key is a single column (serial_number), there can be no partial dependencies (partial dependencies occur only with composite keys). Therefore, the relation is already in 2NF.

But to demonstrate proper design, we note that some attributes depend only on model_number (not on serial_number), e.g., model_name, brand_name, brand_description. However, because the key is a single column, these are transitive dependencies, not partial. They will be handled in 3NF.

So 2NF relation is identical to 1NF.

### 3NF

In 3NF let's remove Transitive Dependencies
3NF requires that no non-prime attribute is transitively dependent on the primary key.
In SPACESHIP_1NF, we have:

serial_number → model_number

model_number → model_name, brand_name, brand_description (and brand_name → brand_description)

Thus, model_name, brand_name, and brand_description are transitively dependent on serial_number via model_number. Similarly, dealer attributes depend on dealer_email, and customer attributes depend on customer_email (though customer has no additional attributes other than email/phone).

To achieve 3NF, we decompose into separate tables:
**Spaceship**
serial_number (PK)
recommended_retail_price
manufacturing_year
model_number (FK to Model)
dealer_email (FK to Dealer, nullable)
customer_id (FK to Customer, nullable)

**Mode**
model_number (PK)
model_name
brand_id (FK to Brand)
Table 3: Brand
brand_id (PK) – surrogate key introduced for stability
brand_name
brand_description

**Dealer**
dealer_email (PK) – as stated unique
dealer_name
dealer_phone
dealer_website
dealer_street
dealer_city
dealer_suburb
dealer_state
dealer_postal_code
dealer_country_name
dealer_country_code

**Customer**
customer_id (PK) – surrogate key (email/phone are updatable)
customer_email
customer_phone

**Customer_Dealer (junction for many-to-many)**
customer_id (FK)
dealer_email (FK)
Composite PK: (customer_id, dealer_email)

### Conclusion:

All relations are in 3NF (no partial or transitive dependencies). They also satisfy BCNF in this specific design because every determinant is a candidate key.

## Conceptual data model diagram

![SHIPS R US database caceptual data model](/RDBMS-problems/diagrams/SHIPS-R-US-data-model.png)

Below is the dbdiagram.io code to generate the conceptual data model diagram based on the 3NF design:

```dbml
  // Brand entity
  Table Brand {
    brand_id int [pk, increment] // surrogate primary key
    name varchar(100) [not null]
    description text
  }

  // Model entity
  Table Model {
    model_number varchar(50) [pk] // unique identifier as stated
    name varchar(100) [not null]
    brand_id int [ref: > Brand.brand_id, not null]
  }

  // Spaceship entity
  Table Spaceship {
  serial_number varchar(50) [pk] // unique identifier as stated
  recommended_retail_price decimal(12,2)
  manufacturing_year int
  model_number varchar(50) [ref: > Model.model_number, not null]
  dealer_email varchar(100) [ref: - Dealer.email] // optional (nullable)
  customer_id int [ref: - Customer.customer_id] // optional (nullable)
  }

  // Dealer entity
  Table Dealer {
    email varchar(100) [pk] // unique as stated
    name varchar(100) [not null]
    phone_number varchar(20) [unique, not null]
    website_url varchar(200)
    street_address varchar(150)
    city varchar(50)
    suburb varchar(50)
    state varchar(20)
    postal_code varchar(20)
    country_name varchar(50)
    country_code varchar(5)
  }

  // Customer entity
  Table Customer {
    customer_id int [pk, increment] // surrogate key (email/phone are updatable)
    email_address varchar(100)
    phone_number varchar(20)
  }

  // Junction table for many-to-many between Dealer and Customer
  Table Customer_Dealer {
    customer_id int [ref: > Customer.customer_id]
    dealer_email varchar(100) [ref: > Dealer.email]
    // composite primary key
    indexes {
      (customer_id, dealer_email) [pk]
    }
  }
```

## Logical ER diagram

![SHIPS R US database ER diagram](/RDBMS-problems/diagrams/SHIPS-R-US-full-ER-diagram.png)

```dbml
// Table: Brand
Table Brand {
  brand_id int [pk, increment, note: "Surrogate primary key"]
  name varchar(100) [not null, unique]
  description text
}

// Table: Model
Table Model {
  model_number varchar(50) [pk, note: "Unique identifier per interview"]
  name varchar(100) [not null]
  brand_id int [not null, ref: > Brand.brand_id, note: "Each model belongs to one brand"]
}

// Table: Spaceship
Table Spaceship {
  serial_number varchar(50) [pk, note: "Unique identifier per interview"]
  recommended_retail_price decimal(12,2)
  manufacturing_year int [not null]
  model_number varchar(50) [not null, ref: > Model.model_number, note: "Each spaceship has one model"]
  dealer_email varchar(100) [ref: - Dealer.email, note: "Nullable – spaceship may be unsold or not distributed"]
  customer_id int [ref: - Customer.customer_id, note: "Nullable – spaceship may have no owner yet"]
}

// Table: Dealer
Table Dealer {
  email varchar(100) [pk, note: "Primary key as per interview (unique)"]
  name varchar(100) [not null]
  phone_number varchar(20) [unique, not null]
  website_url varchar(200)
  street_address varchar(150) [not null]
  city varchar(50) [not null]
  suburb varchar(50)
  state varchar(20) [not null]
  postal_code varchar(20) [not null]
  country_name varchar(50) [not null]
  country_code varchar(5) [not null]
}

// Table: Customer
Table Customer {
  customer_id int [pk, increment, note: "Surrogate key because email/phone are updatable"]
  email_address varchar(100)
  phone_number varchar(20)
}

// Junction table for many-to-many between Dealer and Customer
Table Customer_Dealer {
  customer_id int [ref: > Customer.customer_id, note: "A customer is served by many dealers"]
  dealer_email varchar(100) [ref: > Dealer.email, note: "A dealer serves many customers"]
  // Composite primary key
  indexes {
    (customer_id, dealer_email) [pk]
  }
}
```
