# SmartTech Data Model (MongoDB)

## 1. Overview

This document defines the MongoDB data model for the SmartTech SaaS marketplace platform. It includes all collections, schemas, relationships, and embedding vs referencing decisions.

It is prepared as part of SDS Section 4 (Data Model) to ensure the database design is fully specified before implementation.

---

## 2. Collections

customers, products, categories, specifications, orders, inventory, payments, reviews, compatibility_rules

---

## 3. Data Schemas

### customers
{
  "_id": "ObjectId",
  "name": "string",
  "email": "string",
  "passwordHash": "string",
  "role": "buyer | seller | admin",
  "createdAt": "Date"
}

---

### products
{
  "_id": "ObjectId",
  "name": "string",
  "description": "string",
  "price": "number",
  "categoryId": "ObjectId (ref categories)",
  "sellerId": "ObjectId (ref customers)",
  "specifications": [
    {
      "key": "string",
      "value": "string"
    }
  ],
  "createdAt": "Date"
}

---

### categories
{
  "_id": "ObjectId",
  "name": "string",
  "parentCategoryId": "ObjectId (nullable)"
}

---

### specifications
{
  "_id": "ObjectId",
  "productId": "ObjectId (ref products)",
  "cpu": "string",
  "ram": "string",
  "storage": "string",
  "battery": "string",
  "gpu": "string",
  "os": "string"
}

---

### orders
{
  "_id": "ObjectId",
  "customerId": "ObjectId (ref customers)",
  "items": [
    {
      "productId": "ObjectId (ref products)",
      "quantity": "number",
      "priceAtPurchase": "number"
    }
  ],
  "totalAmount": "number",
  "status": "pending | paid | shipped | delivered | cancelled",
  "createdAt": "Date"
}

---

### inventory
{
  "_id": "ObjectId",
  "productId": "ObjectId (ref products)",
  "stock": "number",
  "warehouseLocation": "string",
  "updatedAt": "Date"
}

---

### payments
{
  "_id": "ObjectId",
  "orderId": "ObjectId (ref orders)",
  "customerId": "ObjectId (ref customers)",
  "amount": "number",
  "method": "card | paypal | cod",
  "status": "pending | success | failed",
  "transactionId": "string",
  "paidAt": "Date"
}

---

### reviews
{
  "_id": "ObjectId",
  "productId": "ObjectId (ref products)",
  "customerId": "ObjectId (ref customers)",
  "rating": "number (1-5)",
  "comment": "string",
  "createdAt": "Date"
}

---

### compatibility_rules
{
  "_id": "ObjectId",
  "deviceType": "string",
  "category": "string",
  "rules": [
    {
      "attribute": "string",
      "condition": "string",
      "value": "string"
    }
  ]
}

---

## 4. Relationships

- customers → orders (1-to-many)
- customers → reviews (1-to-many)
- sellers → products (1-to-many)
- products → categories (many-to-1)
- products → inventory (1-to-1)
- orders → items (embedded)
- orders → payments (1-to-1)
- products → reviews (1-to-many)

---

## 5. Embedding vs Referencing Decisions

Embedded:
- Order items inside orders (fast read performance)
- Product specifications inside products (faster product loading)

Referenced:
- Customers (identity consistency)
- Products (central entity)
- Categories (shared structure)
- Reviews (scalable feedback)
- Inventory (stock control)
- Payments (financial tracking)

---

## 6. Key Design Decisions

- Orders store priceAtPurchase to preserve history
- Products are the central entity
- Reviews are fully decoupled
- Inventory is separate for real-time stock updates
- Compatibility rules are isolated for future expansion

---

## 7. Compatibility Rules

- Device-based matching (Laptop, Phone, Tablet)
- Category-based constraints
- Attribute rule engine

Example:
Gaming laptop requires:
- RAM ≥ 16GB
- Dedicated GPU
- CPU ≥ i5 equivalent

---

## 8. Summary

This data model ensures:
- Scalable MongoDB structure
- Clear relationships
- Optimized performance
- Proper normalization and embedding balance