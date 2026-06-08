# Assessment Materials

## Course

Distributed Databases and Data Warehouses  
Topic: MPP Data Processing (ClickHouse, YDB, SingleStore)

---

# 1. Homework Assignment

## Topic

Designing an MPP architecture for analytics and mixed workloads.

## Task Statement

You are working on a data platform for a delivery and city mobility service. The system must serve:

- analysts;
- a product dashboard;
- operational queries for the latest state of objects.

### Data Source

Kafka stream `service_events`:

- `event_time`
- `entity_id`
- `entity_type`
- `city`
- `metric_a`
- `metric_b`
- `status`

### Requirements

1. Access to raw data is required.
2. Queries over large time ranges are required.
3. Some queries must run near real time.
4. Some operations require transactional consistency.
5. You must explain why one system is not always better than another.

### What to submit

1. The system choice for each scenario:
   - ClickHouse;
   - YDB;
   - SingleStore.
2. A justification based on workload.
3. An architecture diagram.
4. A short description of the physical model:
   - how data is stored;
   - where columnstore is needed;
   - where rowstore is needed;
   - whether a distribution key is needed.
5. A conclusion about trade-offs.

### Submission format

- 1–2 pages;
- one diagram;
- Markdown or PDF is allowed.

## Grading Criteria

Total: 10 points

- 2 points: correct system choice per scenario;
- 2 points: correct explanation of MPP / data movement / distribution key;
- 2 points: understanding of differences between ClickHouse, YDB, and SingleStore;
- 1 point: correct storage model description;
- 1 point: architecture diagram;
- 1 point: trade-off conclusion;
- 1 point: clarity and precision of language.

## Reference Solution

### Recommended decomposition

- **ClickHouse** - for heavy analytics over raw events;
- **YDB** - for distributed SQL, consistency, and mixed workloads;
- **SingleStore** - when you need an HTAP platform with low-latency analytics and transactional queries in one place.

### Why

#### ClickHouse

- best suited for scan-heavy analytics;
- columnar storage reduces I/O;
- vectorized execution speeds up aggregations;
- good for raw events and large time-range reports.

#### YDB

- useful for distributed SQL;
- strong consistency and ACID;
- automatic sharding;
- columnar tables and MPP provide analytical capabilities;
- good when analytics and transactions must live in one database.

#### SingleStore

- HTAP approach;
- rowstore for transactional hot path;
- columnstore for analytics;
- suitable for near real-time analytics;
- good if you want fewer separate systems.

### Reference architecture

`Kafka -> streaming ingestion -> ClickHouse / YDB / SingleStore -> BI / API / app`

### Physical model

#### For ClickHouse

```sql
CREATE TABLE service_events
(
    event_time DateTime,
    entity_id UInt64,
    entity_type LowCardinality(String),
    city LowCardinality(String),
    metric_a Float64,
    metric_b Float64,
    status LowCardinality(String)
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(event_time)
ORDER BY (entity_type, city, event_time);
```

The student is expected to explain:

- why this `ORDER BY` was chosen;
- why rollup is acceptable only if raw detail is not required;
- why ClickHouse is good for large scans.

#### For YDB

- use columnar tables for analytical queries;
- account for automatic sharding;
- discuss distributed query execution;
- explicitly state that YDB is useful for mixed workloads.

#### For SingleStore

- rowstore for the transactional path;
- columnstore for analytics;
- small inserts are buffered first and then moved into columnstore;
- whether an additional distribution key is needed depends on JOIN and grouping patterns.

### What counts as a good answer

- the student does not try to force one system to solve all tasks without arguments;
- the student understands that universality and peak performance are different goals;
- the student can explain why OLAP, distributed SQL, and HTAP are not the same thing.

---

# 2. Control Test

## Format

- Duration: 45 minutes.
- Individual work.
- Only the task sheet may be used.
- Answers should be short but justified.

## Variant

### Task 1. Choosing a system for raw-event analytics

The service stores a `service_events` stream and wants to build heavy analytical reports over large time ranges. Requirements:

- large scans;
- aggregations;
- access to raw data;
- latency in seconds;
- high speed for scan-heavy workloads.

**Questions:**

1. Which system would you choose: ClickHouse, YDB, or SingleStore?
2. Why?
3. What role will `ORDER BY` play?

### Reference answer

**Choice: ClickHouse.**

Reasons:

- the system is optimized for OLAP;
- columnar storage reduces I/O;
- vectorized execution speeds up aggregations;
- `ORDER BY` defines physical sorting and helps data skipping;
- on scan-heavy workloads, ClickHouse usually wins in speed and cost.

### Task 2. Mixed workload with consistency

You need to build a service where there are:

- transactional updates of object states;
- analytical queries over those objects;
- horizontal scaling;
- strong consistency.

**Questions:**

1. Which system would you choose?
2. Why is it a better fit than the others?
3. Why is this not a pure OLAP case?

### Reference answer

**Choice: YDB.**

Expected points:

- distributed SQL;
- automatic sharding;
- ACID and strong consistency;
- columnar tables + MPP for analytics;
- the mixed workload explains why YDB is better than pure OLAP here.

### Task 3. HTAP platform

You want to reduce the number of separate systems and store:

- current states;
- transactional operations;
- near real-time dashboards.

**Questions:**

1. Which system would you choose?
2. How do rowstore and columnstore work?
3. What is the compromise of HTAP?

### Reference answer

**Choice: SingleStore.**

Expected points:

- rowstore handles the transactional path;
- columnstore handles analytics;
- small inserts are buffered first;
- HTAP is convenient, but expensive in terms of trade-offs;
- on pure OLAP, specialized ClickHouse is usually stronger.

### Task 4. Comparing architectures

Compare ClickHouse, YDB, and SingleStore on the following criteria:

- workload type;
- storage model;
- consistency;
- analytical capabilities;
- cost of universality.

### Reference answer

| Criterion | ClickHouse | YDB | SingleStore |
| --- | --- | --- | --- |
| Workload type | OLAP | distributed SQL + mixed workload | HTAP |
| Storage model | columnar | row + columnar tables | rowstore + columnstore |
| Consistency | depends on the scenario, not the main goal | strong consistency, ACID | transactional + analytical balance |
| Analytics | very strong | strong | strong, but with trade-offs |
| Cost of universality | low | medium | high |

### What counts as an excellent answer

- the student correctly matches the system to the workload;
- distinguishes OLAP, distributed SQL, and HTAP;
- can discuss trade-offs without slogans;
- does not present one platform as universally best for everything.

---

# 3. Instructor Key

## What must be present in a strong answer

- understanding of MPP as parallel processing with data movement;
- understanding that the distribution key can either accelerate or damage a query;
- correct distinction between ClickHouse, YDB, and SingleStore;
- understanding of the role of columnstore and rowstore;
- awareness that HTAP is a compromise.

## Common mistakes

- choosing ClickHouse for a scenario where the main issue is transactional consistency;
- choosing YDB as “the best ClickHouse” without considering the different goals of the systems;
- treating SingleStore as a free universal replacement for everything;
- ignoring data distribution and shuffle;
- mixing OLAP, distributed SQL, and HTAP into one category.

## Grading scale

- 9–10 points: the system choice and explanation are fully correct;
- 7–8 points: the choice is generally correct, but some terminology is slightly off;
- 5–6 points: the right system is partially chosen, but the justification is weak;
- below 5 points: the workload and system are not matched, or the trade-offs are not understood.
