# Part 2: Database Design – Inventory Management System

## Understanding the Given Requirements

From the problem statement, the following needs were identified:

- A company can have multiple warehouses
- Products can be stored in more than one warehouse
- Inventory quantity should be tracked
- Suppliers provide products
- Some products may be bundles of other products

The requirements are incomplete, so reasonable assumptions are made where necessary.

---

## Proposed Database Tables (Basic Design)

### 1. Company

Stores basic company information.

**Columns:**

- `id` (Primary Key)
- `name`

---

### 2. Warehouse

Represents storage locations belonging to a company.

**Columns:**

- `id` (Primary Key)
- `company_id` (Foreign Key → Company.id)
- `name`

**Reasoning:**

- One company can own multiple warehouses

---

### 3. Product

Stores product details.

**Columns:**

- `id` (Primary Key)
- `company_id` (Foreign Key → Company.id)
- `name`
- `sku`
- `price`

**Assumption:**

- SKU is unique per company

---

### 4. Inventory

Tracks how much of each product is available in each warehouse.

**Columns:**

- `id` (Primary Key)
- `product_id` (Foreign Key → Product.id)
- `warehouse_id` (Foreign Key → Warehouse.id)
- `quantity`

**Reasoning:**

- Separates product data from warehouse-specific stock

---

### 5. Supplier

Stores supplier information.

**Columns:**

- `id` (Primary Key)
- `name`
- `contact_email`

---

### 6. ProductSupplier

Defines which supplier provides which product.

**Columns:**

- `product_id` (Foreign Key → Product.id)
- `supplier_id` (Foreign Key → Supplier.id)

**Reasoning:**

- Allows flexibility if products have different suppliers

---

### 7. ProductBundle (Basic)

Used to represent products that are bundles of other products.

**Columns:**

- `bundle_product_id` (Foreign Key → Product.id)
- `child_product_id` (Foreign Key → Product.id)
- `quantity`

**Note:**

---

## Relationships Summary (Simple)

- Company → Warehouses (One-to-Many)
- Company → Products (One-to-Many)
- Product → Warehouses (Many-to-Many via Inventory)
- Product → Supplier (Many-to-Many)
- Product → Product (Bundles)

---

## Missing Requirements / Questions

To improve this design, I would ask the product team:

- Is SKU unique globally or only within a company?
- Can a product have more than one supplier?
- How should bundle inventory be calculated?
- Is inventory history tracking required?
- Are warehouse-to-warehouse transfers allowed?

---

## Design Decisions (Fresher-Level Explanation)

- Inventory table is used to support multiple warehouses
- Product and warehouse data are kept separate for clarity
- Supplier relationship is kept simple
- Advanced constraints, indexing, and performance tuning are skipped

---
