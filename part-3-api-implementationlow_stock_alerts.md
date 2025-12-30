# Part 3: Low Stock Alerts API – Basic Implementation

## Endpoint Specification

```
GET /api/companies/{company_id}/alerts/low-stock
```

The endpoint returns products whose inventory is below a defined threshold and need attention.

---

## Assumptions Made

Due to incomplete requirements, the following assumptions were made:

- Low stock threshold is stored in the Product table
- Recent sales means products that had at least one sale in the last 30 days
- Each product has only one supplier
- Inventory quantity is checked per warehouse
- Basic calculation is sufficient for alerts

---

## High-Level Approach

1. Fetch all warehouses belonging to the given company
2. Fetch inventory records for those warehouses
3. Check if inventory quantity is below the product’s low-stock threshold
4. Include supplier details for reordering
5. Return alerts in the expected response format

---

## Sample Implementation (Flask)

```python
@app.route('/api/companies/<int:company_id>/alerts/low-stock', methods=['GET'])
def get_low_stock_alerts(company_id):
    alerts = []

    inventories = Inventory.query \
        .join(Warehouse) \
        .filter(Warehouse.company_id == company_id) \
        .all()

    for inventory in inventories:
        product = Product.query.get(inventory.product_id)
        supplier = Supplier.query.first()

        if inventory.quantity < product.low_stock_threshold:
            alerts.append({
                "product_id": product.id,
                "product_name": product.name,
                "sku": product.sku,
                "warehouse_id": inventory.warehouse_id,
                "warehouse_name": inventory.warehouse.name,
                "current_stock": inventory.quantity,
                "threshold": product.low_stock_threshold,
                "supplier": {
                    "id": supplier.id,
                    "name": supplier.name,
                    "contact_email": supplier.contact_email
                }
            })

    return {
        "alerts": alerts,
        "total_alerts": len(alerts)
    }
```

---

## Edge Cases Considered (Basic)

- If no inventory is below threshold, an empty list is returned
- Handles multiple warehouses under one company
- Avoids crashing when no alerts are found

---
