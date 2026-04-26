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

## Create actual database using PostgreSQL

```sql
-- Table: Brand
CREATE TABLE Brand (
  brand_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(100) NOT NULL UNIQUE,
  description TEXT
);


-- Table: Model
CREATE TABLE Model (
  model_number VARCHAR(50) PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  brand_id INTEGER NOT NULL,
  FOREIGN KEY (brand_id) REFERENCES Brand(brand_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

-- Table: Dealer
-- Email is the primary key, but if email is updated, ON UPDATE CASCADE will propagate to child tables
CREATE TABLE Dealer (
  email VARCHAR(100) PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  phone_number VARCHAR(20) NOT NULL UNIQUE,
  website_url VARCHAR(200),
  street_address VARCHAR(150) NOT NULL,
  city VARCHAR(50) NOT NULL,
  suburb VARCHAR(50),
  state VARCHAR(20) NOT NULL,
  postal_code VARCHAR(20) NOT NULL,
  country_name VARCHAR(50) NOT NULL,
  country_code VARCHAR(5) NOT NULL
);

-- Table: Customer
CREATE TABLE Customer (
  customer_id SERIAL PRIMARY KEY,
  email_address VARCHAR(100),
  phone_number VARCHAR(20)
);

-- Table: Spaceship
CREATE TABLE Spaceship (
  serial_number VARCHAR(50) PRIMARY KEY,
  recommended_retail_price DECIMAL(12,2),
  manufacturing_year INTEGER NOT NULL,
  model_number VARCHAR(50) NOT NULL,
  dealer_email VARCHAR(100),
  customer_id INTEGER,
  FOREIGN KEY (model_number) REFERENCES Model(model_number) ON DELETE RESTRICT ON UPDATE CASCADE,
  FOREIGN KEY (dealer_email) REFERENCES Dealer(email) ON DELETE SET NULL ON UPDATE CASCADE,
  FOREIGN KEY (customer_id) REFERENCES Customer(customer_id) ON DELETE SET NULL ON UPDATE CASCADE
);

-- Junction table for Many-to-Many between Dealer and Customer
CREATE TABLE Customer_Dealer (
  customer_id INTEGER NOT NULL,
  dealer_email VARCHAR(100) NOT NULL,
  PRIMARY KEY (customer_id, dealer_email),
  FOREIGN KEY (customer_id) REFERENCES Customer(customer_id) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (dealer_email) REFERENCES Dealer(email) ON DELETE CASCADE ON UPDATE CASCADE
);

-- Optional: Create indexes for foreign key columns to improve join performance
CREATE INDEX idx_spaceship_model_number ON Spaceship(model_number);
CREATE INDEX idx_spaceship_dealer_email ON Spaceship(dealer_email);
CREATE INDEX idx_spaceship_customer_id ON Spaceship(customer_id);
CREATE INDEX idx_customer_dealer_customer_id ON Customer_Dealer(customer_id);
CREATE INDEX idx_customer_dealer_dealer_email ON Customer_Dealer(dealer_email);

-- I missed to add created_at and updated_at columns to the tables.
-- Hence, I added the columns as below
ALTER TABLE table_name
ADD COLUMN created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
ADD COLUMN updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW();

```

## Insert data to database.

- I addeed 10 example of brand

```sql
INSERT INTO Brand (name, description)
VALUES
  ('SpaceX', 'American aerospace manufacturer and space transportation company known for reusable rockets, Dragon spacecraft, and Starship.'),
  ('Boeing', 'American multinational corporation that designs, manufactures, and sells airplanes, satellites, and spacecraft including the CST-100 Starliner.'),
  ('Lockheed Martin', 'American aerospace and defense company specializing in advanced spacecraft, Orion crew vehicle, and interplanetary probes.'),
  ('Northrop Grumman', 'American global aerospace and defense technology company, developer of Cygnus spacecraft and James Webb Space Telescope components.'),
  ('Blue Origin', 'American private space manufacturer founded by Jeff Bezos, focusing on reusable launch vehicles and the New Shepard spacecraft.'),
  ('Virgin Galactic', 'American spaceflight company developing commercial suborbital spaceplanes for space tourism.'),
  ('Roscosmos', 'Russian state corporation responsible for space science and aerospace research, operator of Soyuz spacecraft and Progress cargo vehicles.'),
  ('ESA', 'European Space Agency, an intergovernmental organization with spacecraft like Ariane rockets, ATV, and ExoMars.'),
  ('JAXA', 'Japan Aerospace Exploration Agency, developer of H-II Transfer Vehicle (HTV) and Hayabusa asteroid probes.'),
  ('ISRO', 'Indian Space Research Organisation, known for Chandrayaan lunar missions, Mangalyaan Mars orbiter, and Gaganyaan crew vehicle.');
```

- I added 20 models for each brands, count is random.

```sql
  INSERT INTO Model (model_number, name, brand_id) VALUES
  -- SpaceX (brand_id = 1)
  ('SPX-1001', 'Falcon 9 Block 5', 1),
  ('SPX-1002', 'Falcon Heavy', 1),
  ('SPX-1003', 'Starship Cargo', 1),
  -- Boeing (brand_id = 2)
  ('BOE-CST100', 'CST-100 Starliner', 2),
  ('BOE-X37B', 'X-37B Orbital Test Vehicle', 2),
  ('BOE-702', 'Boeing 702 Satellite Bus', 2),
  -- Lockheed Martin (brand_id = 3)
  ('LM-ORION', 'Orion Multi-Purpose Crew Vehicle', 3),
  ('LM-MAVEN', 'MAVEN Mars Orbiter', 3),
  ('LM-JUNO', 'Juno Jupiter Probe', 3),
  -- Northrop Grumman (brand_id = 4)
  ('NG-CYGNUS', 'Cygnus Cargo Spacecraft', 4),
  ('NG-PEGASUS', 'Pegasus XL Rocket', 4),
  ('NG-ANTARES', 'Antares 230+', 4),
  -- Blue Origin (brand_id = 5)
  ('BO-NS4', 'New Shepard Crew Capsule', 5),
  ('BO-NG1', 'New Glenn First Stage', 5),
  -- Virgin Galactic (brand_id = 6)
  ('VG-SS3', 'SpaceShipThree', 6),
  ('VG-SS2', 'SpaceShipTwo Unity', 6),
  -- Roscosmos (brand_id = 7)
  ('RSC-SOYUZ', 'Soyuz MS', 7),
  ('RSC-PROGRESS', 'Progress MS Cargo', 7),
  -- ESA (brand_id = 8)
  ('ESA-ATV', 'Automated Transfer Vehicle', 8),
  ('ESA-EXO', 'ExoMars Trace Gas Orbiter', 8);
```

- I added 50 spaceships into spaceship table

```sql
INSERT INTO Spaceship (serial_number, recommended_retail_price, manufacturing_year, model_number, dealer_email, customer_id) VALUES
  -- SpaceX models (series SPX)
  ('SPX-F9-001', 67000000.00, 2021, 'SPX-1001', NULL, NULL),
  ('SPX-F9-002', 68500000.00, 2022, 'SPX-1001', NULL, NULL),
  ('SPX-F9-003', 70000000.00, 2023, 'SPX-1001', NULL, NULL),
  ('SPX-F9-004', 71500000.00, 2024, 'SPX-1001', NULL, NULL),
  ('SPX-F9-005', 73000000.00, 2025, 'SPX-1001', NULL, NULL),
  ('SPX-FH-001', 130000000.00, 2020, 'SPX-1002', NULL, NULL),
  ('SPX-FH-002', 135000000.00, 2022, 'SPX-1002', NULL, NULL),
  ('SPX-FH-003', 140000000.00, 2024, 'SPX-1002', NULL, NULL),
  ('SPX-STAR-001', 250000000.00, 2025, 'SPX-1003', NULL, NULL),
  ('SPX-STAR-002', 260000000.00, 2026, 'SPX-1003', NULL, NULL),

  -- Boeing models
  ('BOE-STL-001', 90000000.00, 2019, 'BOE-CST100', NULL, NULL),
  ('BOE-STL-002', 92000000.00, 2021, 'BOE-CST100', NULL, NULL),
  ('BOE-STL-003', 95000000.00, 2023, 'BOE-CST100', NULL, NULL),
  ('BOE-X37-001', 120000000.00, 2020, 'BOE-X37B', NULL, NULL),
  ('BOE-X37-002', 125000000.00, 2022, 'BOE-X37B', NULL, NULL),
  ('BOE-702-001', 50000000.00, 2018, 'BOE-702', NULL, NULL),
  ('BOE-702-002', 52000000.00, 2020, 'BOE-702', NULL, NULL),
  ('BOE-702-003', 54000000.00, 2023, 'BOE-702', NULL, NULL),

  -- Lockheed Martin models
  ('LM-ORI-001', 180000000.00, 2022, 'LM-ORION', NULL, NULL),
  ('LM-ORI-002', 185000000.00, 2024, 'LM-ORION', NULL, NULL),
  ('LM-MAV-001', 450000000.00, 2015, 'LM-MAVEN', NULL, NULL),
  ('LM-JUN-001', 1100000000.00, 2016, 'LM-JUNO', NULL, NULL),  -- Juno cost ~1.1B
  ('LM-JUN-002', 1150000000.00, 2017, 'LM-JUNO', NULL, NULL),

  -- Northrop Grumman models
  ('NG-CYG-001', 75000000.00, 2020, 'NG-CYGNUS', NULL, NULL),
  ('NG-CYG-002', 78000000.00, 2022, 'NG-CYGNUS', NULL, NULL),
  ('NG-CYG-003', 80000000.00, 2024, 'NG-CYGNUS', NULL, NULL),
  ('NG-PEG-001', 40000000.00, 2019, 'NG-PEGASUS', NULL, NULL),
  ('NG-PEG-002', 42000000.00, 2021, 'NG-PEGASUS', NULL, NULL),
  ('NG-ANT-001', 85000000.00, 2020, 'NG-ANTARES', NULL, NULL),
  ('NG-ANT-002', 88000000.00, 2023, 'NG-ANTARES', NULL, NULL),

  -- Blue Origin models
  ('BO-NS4-001', 28000000.00, 2021, 'BO-NS4', NULL, NULL),
  ('BO-NS4-002', 29500000.00, 2023, 'BO-NS4', NULL, NULL),
  ('BO-NS4-003', 31000000.00, 2025, 'BO-NS4', NULL, NULL),
  ('BO-NG1-001', 200000000.00, 2024, 'BO-NG1', NULL, NULL),
  ('BO-NG1-002', 210000000.00, 2025, 'BO-NG1', NULL, NULL),

  -- Virgin Galactic models
  ('VG-SS3-001', 450000000.00, 2025, 'VG-SS3', NULL, NULL),
  ('VG-SS2-001', 350000000.00, 2021, 'VG-SS2', NULL, NULL),
  ('VG-SS2-002', 360000000.00, 2022, 'VG-SS2', NULL, NULL),
  ('VG-SS2-003', 375000000.00, 2024, 'VG-SS2', NULL, NULL),

  -- Roscosmos models
  ('RSC-SOY-001', 55000000.00, 2018, 'RSC-SOYUZ', NULL, NULL),
  ('RSC-SOY-002', 57000000.00, 2020, 'RSC-SOYUZ', NULL, NULL),
  ('RSC-SOY-003', 59000000.00, 2022, 'RSC-SOYUZ', NULL, NULL),
  ('RSC-PRO-001', 48000000.00, 2019, 'RSC-PROGRESS', NULL, NULL),
  ('RSC-PRO-002', 50000000.00, 2021, 'RSC-PROGRESS', NULL, NULL),
  ('RSC-PRO-003', 52000000.00, 2023, 'RSC-PROGRESS', NULL, NULL),

  -- ESA models
  ('ESA-ATV-001', 450000000.00, 2015, 'ESA-ATV', NULL, NULL),
  ('ESA-EXO-001', 620000000.00, 2016, 'ESA-EXO', NULL, NULL),
  ('ESA-EXO-002', 650000000.00, 2018, 'ESA-EXO', NULL, NULL);
```

- I added 5 dealers to the dealer table

```sql
INSERT INTO Dealer (email, name, phone_number, website_url, street_address, city, suburb, state, postal_code, country_name, country_code) VALUES
  ('galactic-motors@spacedealers.com', 'Galactic Motors', '+1-555-0101', 'https://www.galacticmotors.com', '123 Space Boulevard', 'Houston', 'Westside', 'Texas', '77001', 'United States', 'US'),
  ('orion-sales@spacedealers.com', 'Orion Spacecraft Sales', '+1-555-0102', 'https://www.orionspacecraft.com', '456 Rocket Road', 'Cape Canaveral', 'Port Canaveral', 'Florida', '32920', 'United States', 'US'),
  ('stellar-enterprises@spacedealers.com', 'Stellar Enterprises', '+1-555-0103', 'https://www.stellarenterprises.com', '789 Galaxy Lane', 'Seattle', 'South Lake Union', 'Washington', '98101', 'United States', 'US'),
  ('interstellar-automotive@spacedealers.com', 'Interstellar Automotive', '+1-555-0104', 'https://www.interstellarauto.com', '101 Nebula Drive', 'Los Angeles', 'El Segundo', 'California', '90245', 'United States', 'US'),
  ('cosmic-craft@spacedealers.com', 'Cosmic Craft Distributors', '+1-555-0105', 'https://www.cosmiccraft.com', '202 Comet Circle', 'Denver', 'Aurora', 'Colorado', '80012', 'United States', 'US');
```

- Now that we've dears being added, we need to assign random delear to random spaceships
  We'll update each spaceship with a random dealer email from the dealer table using below query.

  First: Generate a random dealer index (1..5) for each spaceship
  second: Assign a row number to each dealer (1..5)
  third: Update each spaceship using the random index to pick a dealer

```sql
  WITH spaceship_random AS (
      SELECT
          serial_number,
          floor(random() * 5) + 1 AS random_index  -- 5 dealers total
      FROM spaceship
  ),
  dealer_with_index AS (
      SELECT
          email,
          row_number() OVER (ORDER BY email) AS idx
      FROM dealer
  )
  UPDATE spaceship s
  SET dealer_email = d.email
  FROM spaceship_random sr, dealer_with_index d
  WHERE s.serial_number = sr.serial_number
  AND sr.random_index = d.idx;
```

- Now, I add 15 customer to the customer table. Then I'll update the spacehisp table with random customer just for testing.

```sql
INSERT INTO Customer (email_address, phone_number)
VALUES
  ('james.wilson@stellarvoyager.com', '+1-555-1001'),
  ('sarah.connor@orbitdynamics.com', '+1-555-1002'),
  ('michael.chen@deepspace.solutions', '+1-555-1003'),
  ('emily.rodriguez@galactic.travel', '+1-555-1004'),
  ('david.kim@nebulacorp.net', '+1-555-1005'),
  ('lisa.brown@asteroidmining.co', '+1-555-1006'),
  ('robert.johnson@spacexplorer.org', '+1-555-1007'),
  ('maria.garcia@cosmicventures.com', '+1-555-1008'),
  ('thomas.martin@interstellar.sales', '+1-555-1009'),
  ('jennifer.davis@rocketrentals.com', '+1-555-1010'),
  ('daniel.lee@lunarhabitats.net', '+1-555-1011'),
  ('patricia.taylor@mars.colonization', '+1-555-1012'),
  ('christopher.white@zerog.experience', '+1-555-1013'),
  ('linda.harris@spacestation.supply', '+1-555-1014'),
  ('kevin.thompson@orionsbelt.tours', '+1-555-1015'),
  ('brian.anderson@falconflights.net', '+1-555-1016'),
  ('nancy.martinez@starfleet.supplies', '+1-555-1017'),
  ('jason.robinson@orbitalmechanics.com', '+1-555-1018'),
  ('susan.clark@rocketlab.ventures', '+1-555-1019'),
  ('matthew.lewis@cosmiccouriers.org', '+1-555-1020'),
  ('karen.walker@deepspace.logistics', '+1-555-1021'),
  ('andrew.hall@hyperdrive.systems', '+1-555-1022'),
  ('jessica.allen@astrotech.solutions', '+1-555-1023'),
  ('joshua.young@lunaroutfitters.net', '+1-555-1024'),
  ('amanda.hernandez@zerogravity.tours', '+1-555-1025'),
  ('ryan.king@martianrealty.com', '+1-555-1026'),
  ('melissa.wright@satelliteservices.co', '+1-555-1027'),
  ('kevin.lopez@rocketpropulsion.net', '+1-555-1028'),
  ('stephanie.hill@spacehabitats.org', '+1-555-1029'),
  ('brendan.scott@orbitrentals.com', '+1-555-1030');
```

Added random customer to spaceship table:

```sql
  WITH spaceship_random AS (
      SELECT
          serial_number,
          floor(random() * 30) + 1 AS random_index  -- 30 customers total
      FROM spaceship
  ),
  customer_with_index AS (
      SELECT
          customer_id,
          row_number() OVER (ORDER BY customer_id) AS idx
      FROM customer
  )
  UPDATE spaceship s
  SET customer_id = c.customer_id
  FROM spaceship_random sr, customer_with_index c
  WHERE s.serial_number = sr.serial_number
  AND sr.random_index = c.idx;
```

- Updated customer_dealer table
  We need to associate dealers with customers in Dealer and Customer tables in customer_delear table. As Dealer and Customer has many to many relationship. Hence, the dealer email in customer_dealer table and spaceship table should be relative with the customer_id in spaceship table.

  Query:

```sql
  INSERT INTO customer_dealer (customer_id, dealer_email)
  SELECT DISTINCT
      s.customer_id,
      s.dealer_email
  FROM spaceship s
  WHERE s.customer_id IS NOT NULL
    AND s.dealer_email IS NOT NULL
  ON CONFLICT (customer_id, dealer_email) DO NOTHING;
```

User this query to make sure it worked as expected.

```sql
SELECT cd.customer_id, cd.dealer_email, COUNT(s.serial_number) as ships_bought
FROM customer_dealer cd
JOIN spaceship s ON s.customer_id = cd.customer_id AND s.dealer_email = cd.dealer_email
GROUP BY cd.customer_id, cd.dealer_email;

```

## Possible data set needed for user

- List all spaceships with their model name and brand name

```sql
SELECT
    s.serial_number,
    s.manufacturing_year,
    s.recommended_retail_price,
    m.name AS model_name,
    b.name AS brand_name
FROM spaceship s
JOIN model m ON s.model_number = m.model_number
JOIN brand b ON m.brand_id = b.brand_id
ORDER BY s.manufacturing_year DESC, b.name, m.name;
```

- Show dealers with the number of spaceships they currently have in inventory (unsold)

```sql
SELECT
    d.name AS dealer_name,
    d.email,
    d.phone_number,
    COUNT(s.serial_number) AS inventory_count
FROM dealer d
LEFT JOIN spaceship s ON s.dealer_email = d.email AND s.customer_id IS NULL
GROUP BY d.email, d.name, d.phone_number
ORDER BY inventory_count DESC;
```

- List all customers along with the total number of spaceships they own

```sql

SELECT
    c.customer_id,
    c.email_address,
    c.phone_number,
    COUNT(s.serial_number) AS spaceships_owned
FROM customer c
LEFT JOIN spaceship s ON s.customer_id = c.customer_id
GROUP BY c.customer_id
ORDER BY spaceships_owned DESC;
```

- Show each dealer and which customers they have served (distinct customers)

```sql
SELECT
    d.name AS dealer_name,
    d.email AS dealer_email,
    c.email_address AS customer_email,
    c.phone_number AS customer_phone
FROM dealer d
JOIN customer_dealer cd ON cd.dealer_email = d.email
JOIN customer c ON c.customer_id = cd.customer_id
ORDER BY d.name, c.email_address;
```

- Find the most popular spaceship model (most manufactured units)

```sql
SELECT
    m.model_number,
    m.name AS model_name,
    b.name AS brand_name,
    COUNT(s.serial_number) AS total_manufactured
FROM model m
JOIN brand b ON b.brand_id = m.brand_id
LEFT JOIN spaceship s ON s.model_number = m.model_number
GROUP BY m.model_number, m.name, b.name
ORDER BY total_manufactured DESC
LIMIT 5;
```

- List all spaceships sold after a certain year (e.g., 2020) with owner and dealer details

```sql
SELECT
    s.serial_number,
    s.manufacturing_year,
    s.recommended_retail_price,
    c.email_address AS owner_email,
    d.name AS dealer_name,
    d.email AS dealer_email
FROM spaceship s
LEFT JOIN customer c ON s.customer_id = c.customer_id
LEFT JOIN dealer d ON s.dealer_email = d.email
WHERE s.manufacturing_year > 2020
  AND s.customer_id IS NOT NULL
ORDER BY s.manufacturing_year;
```

- Count how many spaceships each dealer has sold (customer assigned)

```sql
SELECT
    d.name AS dealer_name,
    COUNT(s.serial_number) AS sold_count
FROM dealer d
LEFT JOIN spaceship s ON s.dealer_email = d.email AND s.customer_id IS NOT NULL
GROUP BY d.name
ORDER BY sold_count DESC;
```

- Show customers who have bought more than one spaceship and the dealers they bought from

```sql
SELECT
    c.customer_id,
    c.email_address,
    COUNT(s.serial_number) AS total_bought,
    STRING_AGG(DISTINCT d.name, ', ') AS dealers_visited
FROM customer c
JOIN spaceship s ON s.customer_id = c.customer_id
JOIN dealer d ON d.email = s.dealer_email
GROUP BY c.customer_id, c.email_address
HAVING COUNT(s.serial_number) > 1
ORDER BY total_bought DESC;
```

- List all brands that have no models yet (if any)

```sql
SELECT b.name, b.description
FROM brand b
LEFT JOIN model m ON m.brand_id = b.brand_id
WHERE m.model_number IS NULL;
```

- Find dealers who serve the largest number of distinct customers

```sql
SELECT
    d.name AS dealer_name,
    COUNT(DISTINCT cd.customer_id) AS unique_customers
FROM dealer d
JOIN customer_dealer cd ON cd.dealer_email = d.email
GROUP BY d.email, d.name
ORDER BY unique_customers DESC;
```

- Show the total monetary value of all spaceships owned by each customer (if retail price reflects purchase price)

```sql
SELECT
    c.customer_id,
    c.email_address,
    SUM(s.recommended_retail_price) AS total_value_owned
FROM customer c
JOIN spaceship s ON s.customer_id = c.customer_id
GROUP BY c.customer_id, c.email_address
ORDER BY total_value_owned DESC;
```

- List all spaceships that are not yet sold (no customer assigned) and still with a dealer

```sql
SELECT
    s.serial_number,
    s.manufacturing_year,
    s.recommended_retail_price,
    m.name AS model_name,
    d.name AS dealer_name
FROM spaceship s
JOIN model m ON m.model_number = s.model_number
JOIN dealer d ON d.email = s.dealer_email
WHERE s.customer_id IS NULL
ORDER BY s.manufacturing_year;
```
