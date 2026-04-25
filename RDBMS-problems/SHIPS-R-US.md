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
