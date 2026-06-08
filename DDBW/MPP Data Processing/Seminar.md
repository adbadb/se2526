# Seminar 13. MPP Data Processing

## Topic

ClickHouse, YDB, SingleStore: how to choose a system for a given MPP workload and how to design data for that workload.

## Seminar Goal

The seminar aims to help students:

- distinguish pure OLAP from distributed SQL and HTAP;
- choose a system by workload, not popularity;
- design a physical data model for MPP;
- understand the role of `distribution key`, `ORDER BY`, `columnstore`, `rowstore`, and `shuffle`.

## Tools

- draw.io / Excalidraw / Miro for diagrams;
- SQL client for discussing schemas and queries;
- optional: ClickHouse in Docker, YDB in-memory in Docker.

## What is assessed

- correctness of the system choice;
- understanding of MPP constraints and trade-offs;
- quality of argumentation;
- adequacy of the physical data design;
- understanding of data movement and skew;
- ability to explain why one system is more appropriate than another for this workload.

---

## Running case

You are designing a data platform and operational analytics solution for a city mobility service called `MetroPulse`.

### Data source

Kafka stream `vehicle_events`:

- `event_time`
- `vehicle_id`
- `route_id`
- `driver_id`
- `lat`
- `lon`
- `speed`
- `passengers_estimated`
- `city`

### Business tasks

1. Internal analysts need heavy queries over raw data.
2. The product team needs a near real-time dashboard by route.
3. The operational system needs to show the last position of a vehicle by `vehicle_id`.

---

## Task 1. ClickHouse for heavy analytics

### Scenario

Analytics wants reports over raw `vehicle_events`:

- trips by route;
- average speed;
- passenger flow;
- city comparison;
- time filtering.

### Requirements

1. Access to raw events is required.
2. Ad-hoc queries are required.
3. Responses in a few seconds are acceptable.
4. The table may become very large.

### Assignment

1. Choose a system.
2. Design a table in ClickHouse.
3. Explain which `ORDER BY` you would choose and why.
4. Is rollup needed?
5. Which query types will be accelerated well, and which will not?

### Expected result

Students are expected to conclude that:

- ClickHouse is the best fit;
- `ORDER BY` should reflect the typical filters, usually time plus filter keys;
- rollup is undesirable if drill-down to raw events is required;
- the strengths are scan-heavy analytics, aggregations, and wide tables.

### Discussion prompt

Ask:

- what is faster: `GROUP BY city` on a wide raw dataset or on a pre-aggregated layer?
- what happens if you choose `ORDER BY (city, event_time)` instead of `ORDER BY (event_time, city)`?

---

## Task 2. YDB as a distributed SQL MPP platform

### Scenario

The system must support:

- transactional operations on current object states;
- analytical queries over large tables;
- high availability;
- automatic sharding;
- strong consistency.

### Requirements

1. The system must handle load growth without manual sharding.
2. ACID transactions are required.
3. Analytical queries over large volumes are required.
4. Explain why distributed SQL can be better than pure OLAP here.

### Assignment

1. Why is YDB a reasonable choice here?
2. What properties make it closer to distributed SQL than to pure OLAP?
3. What does columnar storage inside YDB provide?
4. Why is YDB useful for mixed workloads?

### Expected result

Students are expected to explain:

- YDB combines horizontal scaling and ACID;
- columnar tables + MPP are useful for heavy analytical queries;
- automatic sharding reduces operational complexity;
- YDB is especially good when analytics and transactions live in the same product.

### Understanding check

Why should YDB not automatically be considered a replacement for ClickHouse when the task is only scan-heavy BI?

---

## Task 3. SingleStore as an HTAP solution

### Scenario

A product needs one layer that can:

- accept fresh events;
- serve transactional requests;
- provide analytical dashboards in near real time.

### Requirements

1. Both transactional and analytical workloads are needed.
2. Low-latency analytics is needed.
3. The number of separate systems should be minimized.
4. A trade-off in peak performance is acceptable if universality is gained.

### Assignment

1. Why is SingleStore suitable here?
2. What roles do rowstore and columnstore play?
3. Why is HTAP always a compromise?
4. In which cases can SingleStore be more expensive than a specialized ClickHouse deployment?

### Expected result

Students are expected to state:

- SingleStore is useful as a universal platform;
- rowstore serves the hot transactional path;
- columnstore serves analytics;
- small inserts are buffered first and then moved into columnstore;
- universality is more expensive than specialized OLAP.

---

## Task 4. Architecture choice

### Scenario

The `MetroPulse` product needs an architecture decision:

- analytics over raw events;
- near real-time reporting;
- latest-state lookup;
- strong consistency for part of the operations.

### Assignment

Choose a combination of the three systems:

- ClickHouse;
- YDB;
- SingleStore;

and explain the role of each system.

### Expected result

A strong answer:

- ClickHouse for heavy analytics over raw data;
- YDB for distributed SQL and mixed workloads;
- SingleStore if you want to combine transactional and analytical scenarios in one platform.

---

## Homework mini-task after the seminar

### Statement

Take one of the scenarios above and describe:

1. the physical data model;
2. the expected queries;
3. the main bottleneck;
4. how the solution changes if the load increases 10x.

### Success criterion

The answer should show that the student can connect workload, storage model, and architectural trade-offs.
