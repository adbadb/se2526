# Assessment Material

## Databases for ML: Feature Stores, RAG, and MLOps

## Variant 1

### Task 1. Conceptual understanding

Define the following terms in the context of ML systems:

1. Feature store
2. Offline store
3. Online store
4. Point-in-time correctness
5. RAG
6. MLOps

Then explain in 5-7 sentences why these concepts are related but not identical.

### Task 2. Feature engineering and leakage

A company wants to predict whether a customer will churn in the next 30 days. The available operational tables are:

- `customers(customer_id, signup_date, segment)`
- `subscriptions(subscription_id, customer_id, start_date, end_date, status)`
- `payments(payment_id, customer_id, payment_date, amount, status)`
- `support_tickets(ticket_id, customer_id, created_at, category, closed_at)`

For a training example with prediction time `2025-06-01`, define three features that would be useful for churn prediction and explain how to compute them without data leakage.

### Task 3. RAG data modeling

Design a minimal data model for a RAG system that stores documents, chunks, embeddings, and retrieval logs.

Your answer must include:

1. The main entities
2. Their key fields
3. The relationships between them
4. One important metadata field for governance or traceability

### Task 4. SQL task

Write a PostgreSQL query that returns, for each customer, the number of support tickets created in the last 30 days before `2025-06-01`.

Use the `support_tickets` table from Task 2.

The result must include:

- `customer_id`
- `ticket_count`

Only return customers who have at least 2 tickets in that period.

### Task 5. MLOps short answer

Explain two risks that appear when an ML model is deployed without proper MLOps support. Your answer should mention at least one issue related to reproducibility, monitoring, or governance.

---

## Solutions

### Solution to Task 1

1. **Feature store**: a managed data layer for defining, computing, versioning, and serving ML features consistently across training and inference.
2. **Offline store**: the historical storage layer optimized for training, backfills, and analytical reconstruction of feature values.
3. **Online store**: the low-latency serving layer used at inference time to fetch the latest features quickly.
4. **Point-in-time correctness**: a guarantee that only information available at the prediction timestamp is used when building a training example.
5. **RAG**: Retrieval-Augmented Generation, an architecture in which the model retrieves relevant external information before generating an answer.
6. **MLOps**: the engineering discipline for managing the ML lifecycle, including data, features, experiments, model versions, deployment, monitoring, and retraining.

These concepts are related because they all belong to the infrastructure of modern AI systems. However, they are not identical. A feature store focuses on structured predictive features, while RAG focuses on retrieving knowledge documents or chunks for generation. Offline and online stores serve different latency and consistency requirements. Point-in-time correctness is a data quality property, not a storage system by itself. MLOps is the broader operational framework that manages the whole lifecycle. Confusing these concepts leads to weak system design and incorrect expectations about what each component should do.

### Solution to Task 2

Possible useful features for churn prediction:

1. **Number of successful payments in the last 30 days**
2. **Number of support tickets opened in the last 30 days**
3. **Current subscription status or days since subscription start**

#### How to compute them without leakage

For prediction time `2025-06-01`, each feature must use only records with timestamps earlier than or equal to that date.

- For payments, count only payments with `payment_date < '2025-06-01'` and a successful status.
- For support tickets, count only tickets with `created_at < '2025-06-01'` and within the 30-day window starting `2025-05-02`.
- For subscription status, use the subscription record as it existed at prediction time, not future cancellations or renewals.

The key point is that no future events after `2025-06-01` may be used. Otherwise, the model would see information that was not available at prediction time, causing leakage.

### Solution to Task 3

#### 1. Main entities

- `documents`
- `chunks`
- `embeddings` or embedding fields inside `chunks`
- `retrieval_logs`

#### 2. Suggested schema

```sql
documents (
    document_id,
    source_uri,
    title,
    doc_type,
    version,
    checksum,
    access_level,
    created_at,
    valid_from,
    valid_to
)

chunks (
    chunk_id,
    document_id,
    section_path,
    chunk_text,
    token_count,
    embedding_vector,
    embedding_model,
    embedding_version,
    created_at
)

retrieval_logs (
    retrieval_id,
    query_text,
    user_id,
    retrieved_chunk_id,
    score,
    rerank_score,
    retrieved_at
)
```

#### 3. Relationships

- One document has many chunks.
- One chunk belongs to exactly one document.
- One retrieval event may reference one or more chunks.
- Each retrieval log stores the query and the selected context.

#### 4. Important metadata field

An important governance field is `access_level`, because it helps ensure that only permitted content is retrieved or exposed to the user. Another important traceability field is `checksum`, which supports version control and change detection.

### Solution to Task 4

```sql
SELECT
    customer_id,
    COUNT(*) AS ticket_count
FROM support_tickets
WHERE created_at >= DATE '2025-05-02'
  AND created_at < DATE '2025-06-01'
GROUP BY customer_id
HAVING COUNT(*) >= 2
ORDER BY ticket_count DESC, customer_id;
```

#### Explanation

- The query counts support tickets in the 30-day window before `2025-06-01`.
- The lower bound is inclusive: `2025-05-02`.
- The upper bound is exclusive: `2025-06-01`.
- `GROUP BY customer_id` computes the count per customer.
- `HAVING COUNT(*) >= 2` keeps only customers with at least two tickets.

### Solution to Task 5

Two risks of deploying an ML model without proper MLOps support are:

1. **Poor reproducibility**: if dataset versions, feature definitions, or model artifacts are not tracked, it becomes impossible to reproduce training results or debug performance changes.
2. **Weak monitoring and governance**: without monitoring, the model may drift silently in production, and without governance, there may be no audit trail for who deployed the model, which data it used, or whether it complied with policy.

Other valid risks include deployment errors, untracked experiment changes, feature skew between training and inference, and the inability to rollback a bad model version.

---

## Grading Guide

- Task 1: 20 points
- Task 2: 25 points
- Task 3: 25 points
- Task 4: 15 points
- Task 5: 15 points

**Total:** 100 points

