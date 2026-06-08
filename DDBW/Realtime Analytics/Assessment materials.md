# Assessment Materials

## Course

Distributed Databases and Data Stores  
Topic: Real-time Analytics, Druid, Pinot, and In-memory Systems

---

# 1. Homework Assignment

## Topic

Designing a user-facing analytics architecture for a high-load product.

## Task Statement

You are working on analytics for a food delivery service. In the restaurant dashboard, the system must show order statistics and menu views in real time.

### Data Source

Kafka stream `restaurant_events` with the following fields:

- `event_time`
- `restaurant_id`
- `user_id`
- `country`
- `event_type`
- `menu_item_id`
- `revenue`

### Requirements

1. The dashboard must open in less than 1 second.
2. Thousands of customers may use analytics at the same time.
3. Queries are repetitive: filter by `restaurant_id`, time range, group by hour or day.
4. One of the charts must show the number of unique users.
5. Storing raw events is not required if that makes the system faster and cheaper.

### What to submit

1. The choice of the main analytical database: ClickHouse or Druid / Pinot.
2. A short justification based on the requirements.
3. An architecture diagram from Kafka to the frontend.
4. Table design:
   - if ClickHouse is chosen, specify `ENGINE`, `PARTITION BY`, and `ORDER BY`;
   - if Druid / Pinot is chosen, describe ingestion, dimensions, metrics, and granularity.
5. Explain whether rollup will be used and why.
6. For unique users, explain whether the count is:
   - exact;
   - or approximate.
7. A short conclusion on trade-offs.

### Submission format

- 1–2 pages of text;
- one architecture diagram;
- SQL or pseudo-configuration if needed;
- Markdown or PDF is allowed.

## Grading Criteria

Total: 10 points

- 2 points: correct system choice;
- 2 points: justification based on latency and concurrency;
- 2 points: correct storage and ingestion design;
- 1 point: correct solution for distinct users;
- 1 point: architecture diagram;
- 1 point: rollup explanation / justification for not using it;
- 1 point: clarity and precision of language.

## Reference Solution

### Recommended choice

**Druid or Pinot.**

### Why

- This is a user-facing analytics workload.
- Very low latency is required.
- High concurrency is required.
- The queries are repetitive and templated.
- Rollup and specialized indexes provide a strong advantage.

### Why not ClickHouse as the default answer

ClickHouse can also solve the task, but:

- it often requires more raw data to be stored;
- materialized views and indexes must be carefully designed;
- for very high concurrency, Druid / Pinot is usually more natural as the serving layer.

### Acceptable alternative

An answer such as “ClickHouse + materialized views” should also be accepted if the student:

- correctly explains how latency is achieved;
- shows awareness of the storage cost of raw data;
- does not claim that ClickHouse and Druid / Pinot are equivalent for this workload.

### Reference architecture

`Kafka -> streaming ingestion job -> Druid / Pinot -> backend API -> web frontend`

### Reference design

#### If Druid is chosen

- `granularity`: `MINUTE` or `HOUR` depending on the query profile;
- dimensions: `restaurant_id`, `country`, `event_type`, possibly `menu_item_id`;
- metrics: `COUNT(*)`, `SUM(revenue)`, sketch for distinct users;
- rollup: yes, if losing raw detail is acceptable.

#### If Pinot is chosen

- real-time table;
- dimensions: `restaurant_id`, `country`, `event_type`;
- metrics: `count`, `sum(revenue)`;
- distinct users: `DISTINCTCOUNTHLL`, `DISTINCTCOUNTHLLPLUS`, `DISTINCTCOUNT`, or another sketch-based method;
- rollup: yes, if the queries are templated and raw rows are not required.

### Reference ClickHouse table

```sql
CREATE TABLE restaurant_events
(
    event_time DateTime,
    restaurant_id UInt64,
    user_id UInt64,
    country LowCardinality(String),
    event_type LowCardinality(String),
    menu_item_id UInt64,
    revenue Float64
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(event_time)
ORDER BY (restaurant_id, event_time);
```

### How to count unique users

Two variants are acceptable:

- **exact** — if high precision is required and the volume allows it;
- **approximate** — if latency and cost are more important.

For Druid / Pinot, an approximate sketch-based distinct count is preferred.

### What counts as a strong answer

- the student correctly connects requirements to architecture;
- the student does not confuse Druid / Pinot with ClickHouse;
- the student understands that rollup is not always beneficial;
- the student is aware of the cost of distinct counting on raw data.

---

# 2. Control Test Variant

## Format

- Duration: 45 minutes.
- Individual work.
- Only the handout may be used.
- Answers should be short but justified.

## Variant 1

### Task 1. System choice for internal analytics

An analytics team wants to build reports on a raw `ad_clicks` stream. Requirements:

- complex ad-hoc SQL queries are needed;
- drill-down to each record is needed;
- 5–10 second latency is acceptable;
- no more than 20 analysts work concurrently.

**Questions:**

1. Which system would you choose: ClickHouse or Druid / Pinot?
2. Why?
3. Is rollup needed?

### Reference answer

**Choice: ClickHouse.**

Reasons:

- raw data access is needed;
- ad-hoc queries and JOINs are important;
- concurrency is moderate;
- 5–10 second latency is acceptable for ClickHouse;
- rollup is not needed because it would interfere with drill-down.

### Task 2. User-facing dashboard

A customer dashboard must satisfy the following:

- load time under 500 ms;
- thousands of concurrent users;
- repetitive queries;
- data may be pre-aggregated.

**Questions:**

1. Which system would you choose?
2. What data would you aggregate during ingestion?
3. How would you count unique users?

### Reference answer

**Choice: Druid / Pinot.**

Expected points:

- rollup or pre-aggregation;
- dimensions for the main filters;
- metrics for counters and sums;
- distinct users via sketch-based methods rather than exact counting if speed matters.

### Task 3. Hybrid architecture

There is a `vehicle_positions` stream with the following fields:

- `vehicle_id`
- `route_id`
- `timestamp`
- `lat`
- `lon`
- `passengers_estimated`

Requirements:

- analysts want raw tracks;
- users want average route load over the last 5 minutes;
- users want the last known vehicle position.

**Questions:**

1. Which three storage systems would you choose?
2. What should be stored in each one?
3. Why is one database not enough?

### Reference answer

- **ClickHouse** - raw GPS tracks for analysts;
- **Druid / Pinot** - route- and time-based aggregates;
- **Redis / Tarantool** - latest position by `vehicle_id`.

Why one database is not enough:

- the latency and query type requirements are different;
- analytics and key-value lookup have different optimal stores;
- putting everything into one system would worsen either speed, cost, or flexibility.

---

# 3. Instructor Notes and Answer Key

## What must be present in a strong answer

- the correct connection between workload and database choice;
- understanding of the difference between raw analytics and user-facing analytics;
- understanding of when rollup is appropriate;
- correct use of the term `distinct`;
- no confusion between Druid and Pinot components;
- a reasonable explanation of hybrid architecture.

## Common mistakes

- choosing Druid / Pinot for a task where raw data access and arbitrary SQL are critical;
- using rollup where drill-down is required;
- trying to compute exact distinct counts on a huge stream without discussing the cost;
- mixing up Druid Coordinator and Pinot Controller;
- claiming that an in-memory layer can replace the main analytical database.

## Grading scale

- 9–10 points: the choice and justification are fully correct, the architecture is realistic;
- 7–8 points: the choice is generally correct, but terminology or implementation details are slightly off;
- 5–6 points: the choice is partly correct, but the justification is weak or ingestion has mistakes;
- below 5 points: the system choice is wrong or the trade-offs are not understood.

