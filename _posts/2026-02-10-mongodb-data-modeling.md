---
layout: post
title: "MongoDB Data Modeling notes #1"
date: 2026-02-10 12:00:00 +0200
categories: [technology, notes]
---

## The Core Principle

> What is used together in the application is stored together.

This is the guiding philosophy behind MongoDB's document model. Unlike relational databases that normalize data across many tables, MongoDB encourages you to shape your data around how your application actually uses it.

---

## MongoDB at a Glance

MongoDB ships with several built-in capabilities worth knowing:

- **ACID transactions** — native, SQL-like transaction support.
- **Encryption key management** — built into the database engine.
- **Data federation & archiving** — archived and live data managed as a single collection.
- **Lucene-based search** — full-text search engine integrated directly.

---

## Workload-Driven Data Modeling

In relational databases you aim for third-normal form. In MongoDB, **the optimal grouping of objects into collections is determined by the workload**.

A workload is described by three dimensions:

1. **Size** — how large is the data?
2. **Quantity** — how many operations occur?
3. **Quality** — what kind of operations are they (reads vs. writes)?

Start by identifying the highest-velocity operations — these drive your schema design. Then identify relationships between entities and quantify them. The central question becomes: **should you embed or reference?**

---

## Referencing (Linking Collections)

Referencing means storing related data in **separate collections** and linking them through a stored identifier. MongoDB doesn't perform joins natively; instead, you use the **aggregation pipeline** with `$lookup` to combine data across collections.

```json
// users collection
{
  "_id": "user123",
  "name": "John Doe",
  "email": "john@example.com"
}

// orders collection
{
  "_id": "order456",
  "userId": "user123",
  "total": 150.00
}
```

**When to use referencing:**

1. The "many" side is very large (e.g., 10,000+ followers).
2. You need write-integrity on many-to-many relationships.
3. One part of the data is accessed frequently while the other is not, and memory is a concern.

---

## Embedding (Single Collection)

Embedding means nesting related data **inside a single document**. An array field naturally models one-to-many relationships.

**When to use embedding:**

1. You need read-integrity — the document model is ACID at the single-document level.
2. You need write-integrity for one-to-one or one-to-many relationships (e.g., cascading deletes).
3. The data is always deleted or archived together.
4. **By default** — embedding is the recommended starting point.

> The data model is defined at the application level. The way you use your data dictates how you store your data.

Before embedding, ask yourself three questions:

1. How often does the embedded information get accessed?
2. Is the data queried using the embedded information?
3. Does the embedded information change often?

---

## Common Design Patterns

### Extended Reference Pattern

Instead of embedding an entire related document, store only the most frequently used fields alongside the reference. For example, an order document might embed a customer's name and phone number while still holding a `customerId` reference for the full profile.

### Computed Pattern

Pre-calculate values instead of computing them on every read. For instance, store the total number of comments directly on a blog post document rather than counting them on each query.

> Never recompute what you can precompute.

### Subset Pattern

Embed only a small, representative slice of related data (e.g., the latest 5 reviews). When the user needs more, fetch the full set from a separate collection. This keeps working-set documents small while still providing a fast default view.

### Schema Versioning Pattern

Add a `schema_version` field to each document. This allows different versions of a document schema to coexist in the same collection, making it possible to migrate documents incrementally rather than all at once. This leverages MongoDB's natural support for polymorphic documents.

### Bucket Pattern

Group related, time-ordered data into "buckets" instead of storing one document per event. For example, rather than creating a new document for every sensor reading, create one document **per time window per sensor** and push readings into an array.

This is ideal for **time-series data, real-time analytics, and IoT applications**. A banking analogy: group transactions by account and date rather than storing each transaction as a standalone document. Use `upsert` operations to create or update buckets efficiently.

**Pros:**
- Reduces the overall number of documents in a collection.
- Improves index performance.
- Simplifies data access through pre-aggregation.

### Approximation Pattern

When expensive calculations run frequently and **exact precision is not critical**, use statistical approximations to reduce write load — for example, incrementing a page-view counter only on every 10th visit and multiplying by 10.

**Pros:**
- Fewer writes to the database.
- Maintains statistically valid numbers.

**Cons:**
- Exact numbers aren't represented.
- Logic must be implemented in the application.

### Attribute Pattern

When documents contain many similar fields (e.g., `release_date_us`, `release_date_eu`, `release_date_jp`) and you need to sort or query across that subset, restructure them into an array of key-value pairs. This allows a single index to cover all variants.

**Useful when:**
- Documents have a large number of similar fields.
- Only a small subset of documents contain the fields you need to sort on.
- Both conditions apply simultaneously.
