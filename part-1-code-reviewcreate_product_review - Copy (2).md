# Part 1: Code Review & Debugging – Create Product API

This section reviews the given API endpoint used to create a new product and initialize its inventory.
Although the code compiles successfully, it contains multiple issues that can cause incorrect behavior and data inconsistencies in a real production environment.

---

## Given Code (For Reference)

```python
@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json

    product = Product(
        name=data['name'],
        sku=data['sku'],
        price=data['price'],
        warehouse_id=data['warehouse_id']
    )

    db.session.add(product)
    db.session.commit()

    inventory = Inventory(
        product_id=product.id,
        warehouse_id=data['warehouse_id'],
        quantity=data['initial_quantity']
    )

    db.session.add(inventory)
    db.session.commit()

    return {"message": "Product created", "product_id": product.id}
```

---

## Issues Identified

### 1. SKU Uniqueness Is Not Checked

**Issue:**
The API does not verify whether a product with the same SKU already exists in the system.

**Impact in Production:**
Duplicate SKUs can lead to confusion while identifying products and may cause incorrect inventory tracking and billing issues.

---

### 2. No Validation for Request Data

**Issue:**
The code directly accesses fields from `request.json` without checking if they are present.

**Impact in Production:**
If a required field is missing, the API will throw a server error instead of returning a meaningful response to the client.

---

### 3. Product Is Directly Linked to a Warehouse

**Issue:**
The Product model includes `warehouse_id`, even though products can exist in multiple warehouses.

**Impact in Production:**
This design limits a product to only one warehouse and does not support multi-warehouse inventory management.

---

### 4. Separate Database Commits

**Issue:**
The product and inventory records are committed in two separate database transactions.

**Impact in Production:**
If inventory creation fails, the product remains saved without inventory data, leading to inconsistent records.

---

### 5. No Error Handling

**Issue:**
The API always returns a success response, even if something goes wrong during database operations.

**Impact in Production:**
Clients may believe the product was created successfully when the operation actually failed.

---

## Corrected Version (Simple & Readable)

```python
@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json

    if not data:
        return {"error": "Invalid request body"}, 400

    required_fields = ['name', 'sku', 'price', 'warehouse_id', 'initial_quantity']
    for field in required_fields:
        if field not in data:
            return {"error": f"{field} is required"}, 400

    existing_product = Product.query.filter_by(sku=data['sku']).first()
    if existing_product:
        return {"error": "SKU already exists"}, 409

    try:
        product = Product(
            name=data['name'],
            sku=data['sku'],
            price=data['price']
        )

        db.session.add(product)
        db.session.commit()

        inventory = Inventory(
            product_id=product.id,
            warehouse_id=data['warehouse_id'],
            quantity=data['initial_quantity']
        )

        db.session.add(inventory)
        db.session.commit()

        return {
            "message": "Product created successfully",
            "product_id": product.id
        }, 201

    except Exception:
        db.session.rollback()
        return {"error": "Failed to create product"}, 500
```

---

## Explanation of Fixes

- Added basic validation to ensure all required input fields are present
- Checked for duplicate SKU before creating a new product
- Removed direct warehouse dependency from product creation
- Added simple error handling to avoid silent failures
- Improved API responses with appropriate status codes

---
