# Assessment Material

## Databases Course
### Topic: AI Assistants, Copilots, Agents, Harnesses, Skills, Tools, and MCP for Database Design and Querying

## Variant 1

### Task 1. Terminology and conceptual distinction

Define the following terms in the context of database systems:

1. AI assistant
2. Copilot
3. Agent
4. Harness
5. Skill
6. Tool
7. MCP

Then explain in 5-7 sentences why these concepts should not be used interchangeably.

### Task 2. Database design from business requirements

A small online course platform has the following requirements:

- Each student may enroll in many courses.
- Each course is taught by exactly one instructor.
- A course may have many students.
- Each enrollment stores the enrollment date and the final grade.
- Each instructor has a unique email address.

Design a relational schema for this system.

Your answer must include:

1. The main entities
2. The relationships
3. Primary keys
4. Foreign keys
5. At least one integrity constraint that should be enforced

### Task 3. SQL generation

Using the schema from Task 2, write a SQL query that returns:

- the name of each course
- the number of enrolled students

Only include courses with at least 10 students. Order the result by the number of students in descending order.

### Task 4. Query debugging

Consider the following query:

```sql
SELECT c.course_name, COUNT(s.student_id) AS student_count
FROM courses c
JOIN enrollments e ON c.course_id = e.course_id
JOIN students s ON e.student_id = s.student_id
WHERE student_count >= 10
ORDER BY student_count DESC;
```

Explain what is wrong with the query and provide a corrected version.

### Task 5. Short analytical question

Explain two benefits and two risks of using AI agents for database querying and schema design. Your answer should be based on the lecture topic and should mention at least one governance or safety concern.

---

## Solutions

### Solution to Task 1

1. **AI assistant**: a general conversational system that helps users answer questions, generate text, or use tools. In databases, it may explain SQL, summarize results, or help interpret schemas.
2. **Copilot**: an assistant embedded in a specific workflow environment, such as a SQL editor or BI tool. It supports the user while they work.
3. **Agent**: a more autonomous system that can plan and execute multiple steps toward a goal, choosing tools and revising its actions based on intermediate results.
4. **Harness**: the orchestration layer that manages prompts, tools, context, permissions, retries, logging, and state.
5. **Skill**: a reusable workflow or capability, such as SQL generation, schema normalization, or execution-plan analysis.
6. **Tool**: an external executable capability, such as query execution, schema introspection, or catalog search.
7. **MCP**: Model Context Protocol, a standard for connecting models to external tools and context sources in a modular way.

These terms should not be used interchangeably because they refer to different layers of the system. An assistant is a general interface, while a copilot is tied to a workflow. An agent can act over multiple steps, whereas a harness controls the system around the model. Skills represent reusable behavior, tools perform real actions, and MCP standardizes external integration. Confusing these concepts leads to unrealistic expectations, weak system design, and potential safety problems. In practice, a high-quality database AI solution usually combines all of them in a structured architecture.

### Solution to Task 2

#### 1. Main entities

- `students`
- `courses`
- `instructors`
- `enrollments`

#### 2. Relationships

- One instructor teaches many courses.
- One student can enroll in many courses.
- One course can have many students.
- The `enrollments` table resolves the many-to-many relationship between students and courses.

#### 3. Suggested schema

```sql
CREATE TABLE instructors (
    instructor_id SERIAL PRIMARY KEY,
    full_name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    full_name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    course_name TEXT NOT NULL,
    instructor_id INT NOT NULL REFERENCES instructors(instructor_id)
);

CREATE TABLE enrollments (
    student_id INT NOT NULL REFERENCES students(student_id),
    course_id INT NOT NULL REFERENCES courses(course_id),
    enrollment_date DATE NOT NULL,
    final_grade NUMERIC(5,2),
    PRIMARY KEY (student_id, course_id)
);
```

#### 4. Integrity constraints

Examples:

- `instructors.email` must be unique
- `students.email` must be unique
- `enrollments.student_id` and `enrollments.course_id` must refer to existing students and courses
- `final_grade` should be constrained to a valid range if grading rules are known, for example with `CHECK (final_grade BETWEEN 0 AND 100)`

#### 5. Why this design is appropriate

The design separates core entities and uses the `enrollments` table to model the many-to-many relationship between students and courses. It also preserves referential integrity through foreign keys and ensures that each course is linked to exactly one instructor.

### Solution to Task 3

```sql
SELECT
    c.course_name,
    COUNT(e.student_id) AS student_count
FROM courses c
JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name
HAVING COUNT(e.student_id) >= 10
ORDER BY student_count DESC;
```

#### Explanation

- The query joins `courses` with `enrollments`.
- It groups by course.
- It counts the number of students per course.
- It filters groups with `HAVING COUNT(...) >= 10`.
- It sorts the result in descending order by the count.

### Solution to Task 4

#### What is wrong with the query

There are two main problems:

1. `student_count` is an alias for an aggregate expression, so it cannot be used in the `WHERE` clause. Aggregate filters must be written in `HAVING`.
2. The query is missing a `GROUP BY` clause, even though it uses `COUNT(...)` together with `c.course_name`.

#### Corrected version

```sql
SELECT
    c.course_name,
    COUNT(s.student_id) AS student_count
FROM courses c
JOIN enrollments e ON c.course_id = e.course_id
JOIN students s ON e.student_id = s.student_id
GROUP BY c.course_id, c.course_name
HAVING COUNT(s.student_id) >= 10
ORDER BY student_count DESC;
```

#### Optional improvement

Because the count is based on enrollments, the `students` table is not strictly necessary if the enrollment table already contains valid `student_id` references. A simpler version is:

```sql
SELECT
    c.course_name,
    COUNT(e.student_id) AS student_count
FROM courses c
JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name
HAVING COUNT(e.student_id) >= 10
ORDER BY student_count DESC;
```

### Solution to Task 5

#### Two benefits

1. **Higher productivity**: AI agents can automate repetitive work such as query drafting, schema suggestions, and query debugging.
2. **Better accessibility**: they can help non-experts interact with databases through natural language and reduce the barrier to SQL usage.

#### Two risks

1. **Incorrect or hallucinated output**: an agent may invent tables, columns, or relationships that do not exist, which can lead to wrong results.
2. **Security and governance issues**: if an agent can access sensitive data or execute queries, it may violate access control, privacy, or auditing requirements.

#### Example of a governance concern

An AI agent should not be allowed to run potentially destructive SQL on production data without permission, logging, and role-based access control. The harness must enforce these policies.

---

## Grading Guide

- Task 1: 20 points
- Task 2: 25 points
- Task 3: 15 points
- Task 4: 20 points
- Task 5: 20 points

**Total:** 100 points
