# Plan for Uploading Standard Problems and Checking Solved Status

## 1. Existing Relationship in the Database

The database already has these two important tables:

```text
problems(problem_id, leetcode_no)
user_solved_problems(user_id, problem_id, status)
```

### Explanation

The `problems` table stores the LeetCode problem details.

Example:

```text
problems
---------
id
leetcode_number
name
difficulty
topics
url
```

The `user_solved_problems` table stores whether a user has solved a particular problem.

Example:

```text
user_solved_problems
--------------------
user_id
problem_id
status
```

Here, `problem_id` connects both tables.

```text
problems.id = user_solved_problems.problem_id
```

So, if a row exists in `user_solved_problems` for a particular `user_id` and `problem_id` with status `solved`, then that user has solved the problem.

---

## 2. New Table for Standard Problems

This table stores only important standard problems.

It does not store problem name, difficulty, topic, URL, or description because those details are already present in the `problems` table.

```sql
CREATE TABLE standard_important_problems (
    id SERIAL PRIMARY KEY,
    problem_id INTEGER NOT NULL REFERENCES problems(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (problem_id)
);
```

### Purpose of this table

This table keeps a curated list of important LeetCode problems.

It only stores `problem_id`, which points to the existing `problems.id`.

```text
standard_important_problems.problem_id
        ↓
problems.id
```

---

## 3. Upload the Standard Sheet Manually

The standard sheet will contain LeetCode problem numbers.

Example:

```text
54, 1, 121, 200, 70, 53
```

Instead of manually entering internal `problem_id`, we use `leetcode_number` from the `problems` table.

```sql
INSERT INTO standard_important_problems (problem_id)
SELECT p.id
FROM problems p
WHERE p.leetcode_number IN (54, 1, 121, 200, 70, 53)
ON CONFLICT (problem_id) DO NOTHING;
```

### How this works

Suppose the `problems` table has:

| id | leetcode_number | name |
|---:|---:|---|
| 10 | 54 | Spiral Matrix |
| 11 | 1 | Two Sum |
| 12 | 200 | Number of Islands |

After running the insert query, the `standard_important_problems` table will store:

| id | problem_id |
|---:|---:|
| 1 | 10 |
| 2 | 11 |
| 3 | 12 |

So the uploaded LeetCode numbers are converted into internal `problem_id` values.

---

## 4. How to Check Whether a Particular Problem Is Solved or Not

The relationship is:

```text
standard_important_problems.problem_id
        ↓
points to problems.id
        ↓
same problem_id is checked in user_solved_problems
        ↓
if row exists with status = 'solved' → solved
if row does not exist → not solved
```

---

## 5. SQL Query to Check Solved Status

```sql
SELECT
    p.id AS problem_id,
    p.leetcode_number,
    p.name,
    p.difficulty,
    p.topics,
    p.url,
    CASE
        WHEN usp.id IS NULL THEN false
        ELSE true
    END AS is_solved
FROM standard_important_problems sip
JOIN problems p
    ON p.id = sip.problem_id
LEFT JOIN user_solved_problems usp
    ON usp.problem_id = p.id
    AND usp.user_id = :user_id
    AND usp.status = 'solved';
```

### Result meaning

```text
is_solved = true   → user has solved the problem
is_solved = false  → user has not solved the problem
```

---

## 6. Query to Get Only Unsolved Standard Problems

These are the problems that can be recommended to the user.

```sql
SELECT
    p.id AS problem_id,
    p.leetcode_number,
    p.name,
    p.difficulty,
    p.topics,
    p.url
FROM standard_important_problems sip
JOIN problems p
    ON p.id = sip.problem_id
LEFT JOIN user_solved_problems usp
    ON usp.problem_id = p.id
    AND usp.user_id = :user_id
    AND usp.status = 'solved'
WHERE usp.id IS NULL;
```

---

## 7. Final Flow

```text
1. Upload important LeetCode problem numbers manually.
   Example: 54, 1, 121, 200

2. PostgreSQL finds those problems in the problems table using problems.leetcode_number.

3. PostgreSQL inserts the matching problems.id into standard_important_problems.problem_id.

4. To check solved status, compare:
   standard_important_problems.problem_id
   with
   user_solved_problems.problem_id

5. If a matching row exists in user_solved_problems with status = 'solved':
   the problem is solved.

6. If no matching row exists:
   the problem is unsolved and can be recommended.
```

