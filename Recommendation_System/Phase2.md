# Phase 2: Recommendation System for Standard LeetCode Problems

## 1. Goal of Phase 2

Phase 2 builds a recommendation system that suggests important LeetCode problems to a user based on:

```text
1. Standard important problems
2. Problems the user has not solved yet
3. Problem difficulty priority
4. User weak topics
5. Previously recommended, dismissed, or solved recommendations
```

The recommendation system should not simply show random standard problems.

It should rank problems using a scoring system and recommend the most useful problems first.

---

## 2. Phase 1 Database Setup

The database setup from Phase 1 is already done.

The existing Phase 1 tables are:

```text
problems
user_solved_problems
standard_important_problems
```

### Existing relationship

```text
standard_important_problems.problem_id
        ↓
problems.id
        ↓
user_solved_problems.problem_id
```

So, for a particular user:

```text
If a problem exists in user_solved_problems with status = 'solved'
→ the user has solved that problem.

If no such row exists
→ the problem is unsolved and can be recommended.
```

---

## 3. What the Recommendation System Should Do

The recommendation system is basically a ranking system.

For every standard important problem, it should check:

```text
1. Is this problem important?
2. Has the user already solved it?
3. Is this problem from a weak topic for the user?
4. Has this problem already been recommended before?
5. Did the user dismiss this recommendation?
6. Did the user already solve this recommendation?
```

Only problems that are useful and not already handled should be recommended.

---

## 4. Create User Problem Recommendations Table

Recommendations should not be calculated again and again from scratch every time.

Instead, generate recommendations and store them in a table.

```sql
CREATE TABLE user_problem_recommendations (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    problem_id INTEGER NOT NULL REFERENCES problems(id) ON DELETE CASCADE,

    reason TEXT,
    score NUMERIC(6, 3) DEFAULT 0,

    status TEXT DEFAULT 'pending',

    recommended_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    clicked_at TIMESTAMP,
    solved_at TIMESTAMP,
    dismissed_at TIMESTAMP,
    expires_at TIMESTAMP,

    UNIQUE (user_id, problem_id)
);
```

### Purpose of this table

This table stores generated recommendations for each user.

Example:

| id | user_id | problem_id | score | status | reason |
|---:|---:|---:|---:|---|---|
| 1 | 5 | 10 | 120 | pending | Recommended because it is from one of your weakest topics |
| 2 | 5 | 12 | 100 | shown | Recommended because it is an important medium-level problem |
| 3 | 5 | 18 | 90 | dismissed | User dismissed this recommendation |

### Status meaning

| status | Meaning |
|---|---|
| pending | Recommendation was generated but not shown yet |
| shown | Recommendation was shown to the user |
| clicked | User clicked/opened the problem |
| solved | User solved the recommended problem |
| dismissed | User dismissed the recommendation |

---

## 5. Create Recommendation Events Table

This table stores the history of user interactions with recommendations.

```sql
CREATE TABLE recommendation_events (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    recommendation_id INTEGER REFERENCES user_problem_recommendations(id) ON DELETE CASCADE,
    problem_id INTEGER REFERENCES problems(id) ON DELETE CASCADE,

    event_type TEXT NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Purpose of this table

This table is useful for tracking events such as:

```text
recommendation_created
recommendation_shown
recommendation_clicked
recommendation_dismissed
recommendation_solved
```

Example:

| id | user_id | problem_id | event_type |
|---:|---:|---:|---|
| 1 | 5 | 10 | recommendation_created |
| 2 | 5 | 10 | recommendation_shown |
| 3 | 5 | 10 | recommendation_clicked |
| 4 | 5 | 10 | recommendation_solved |

---

## 6. Basic Query: Get Standard Problems Not Solved by User

First, find all standard important problems that the user has not solved.

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

### Explanation

```text
1. Get all problems from standard_important_problems.
2. Join with problems to get problem details.
3. Left join with user_solved_problems.
4. If usp.id is NULL, the user has not solved the problem.
```

These unsolved standard problems become candidate problems for recommendation.

---

## 7. Ranking Logic

After finding unsolved standard problems, rank them using a score.

```text
final_score = base_score + difficulty_score + topic_weakness_score
```

### Base score

Every standard important problem gets a base score:

```text
base_score = 50
```

This means every problem in the standard list is already considered important.

---

## 8. Difficulty Score

Difficulty score gives priority based on interview usefulness.

```text
Easy   = +10
Medium = +30
Hard   = +20
```

### Why Medium gets the highest score

Medium problems are usually the most common in coding interviews.

So medium-level standard problems should be recommended first.

---

## 9. User Topic Weakness Score

The table `user_lc_topics` is already present in the database.

Example structure:

```text
user_lc_topics
--------------
user_id
tag_name
problems_solved
```

This table tells how many problems a user has solved in each topic.

### Topic weakness scoring

```text
0-5 solved in topic      = +40
6-15 solved in topic     = +25
16-30 solved in topic    = +10
30+ solved in topic      = +0
Unknown topic data       = +20
```

### Meaning

```text
If the user has solved very few problems in a topic,
that topic is weak for the user.

Problems from weak topics should get a higher recommendation score.
```

---

## 10. Query to Rank Candidate Problems

This query finds standard unsolved problems and calculates their recommendation score.

```sql
WITH candidate_problems AS (
    SELECT
        p.id AS problem_id,
        p.leetcode_number,
        p.name,
        p.slug,
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
    LEFT JOIN user_problem_recommendations upr
        ON upr.problem_id = p.id
        AND upr.user_id = :user_id
        AND upr.status IN ('dismissed', 'solved')
    WHERE usp.id IS NULL
      AND upr.id IS NULL
),

topic_scores AS (
    SELECT
        cp.problem_id,
        MAX(
            CASE
                WHEN ult.problems_solved IS NULL THEN 20
                WHEN ult.problems_solved <= 5 THEN 40
                WHEN ult.problems_solved <= 15 THEN 25
                WHEN ult.problems_solved <= 30 THEN 10
                ELSE 0
            END
        ) AS topic_weakness_score
    FROM candidate_problems cp
    LEFT JOIN user_lc_topics ult
        ON ult.user_id = :user_id
        AND LOWER(cp.topics) LIKE '%' || LOWER(ult.tag_name) || '%'
    GROUP BY cp.problem_id
)

SELECT
    cp.problem_id,
    cp.leetcode_number,
    cp.name,
    cp.slug,
    cp.difficulty,
    cp.topics,
    cp.url,

    50 AS base_score,

    CASE
        WHEN cp.difficulty = 'Medium' THEN 30
        WHEN cp.difficulty = 'Hard' THEN 20
        WHEN cp.difficulty = 'Easy' THEN 10
        ELSE 0
    END AS difficulty_score,

    COALESCE(ts.topic_weakness_score, 20) AS topic_weakness_score,

    (
        50
        + CASE
            WHEN cp.difficulty = 'Medium' THEN 30
            WHEN cp.difficulty = 'Hard' THEN 20
            WHEN cp.difficulty = 'Easy' THEN 10
            ELSE 0
          END
        + COALESCE(ts.topic_weakness_score, 20)
    ) AS final_score

FROM candidate_problems cp
LEFT JOIN topic_scores ts
    ON ts.problem_id = cp.problem_id
ORDER BY final_score DESC, cp.leetcode_number ASC;
```

---

## 11. How the Ranking Query Works

### Step 1: candidate_problems

This CTE selects problems that satisfy these conditions:

```text
1. Problem is in standard_important_problems.
2. Problem exists in problems table.
3. User has not solved the problem.
4. User has not already dismissed the recommendation.
5. User has not already solved the recommendation.
```

```sql
WHERE usp.id IS NULL
  AND upr.id IS NULL
```

This prevents recommending solved or dismissed problems again.

---

### Step 2: topic_scores

This CTE calculates how weak the user is in the topic of each problem.

```sql
LEFT JOIN user_lc_topics ult
    ON ult.user_id = :user_id
    AND LOWER(cp.topics) LIKE '%' || LOWER(ult.tag_name) || '%'
```

If the problem topic matches the user's topic data, the system uses `problems_solved` to calculate weakness.

If no topic data exists, a default score of `20` is used.

---

### Step 3: final_score

The final score is calculated as:

```text
final_score = 50 + difficulty_score + topic_weakness_score
```

Higher score means higher priority.

---

## 12. Insert Recommendations into user_problem_recommendations

Once the ranking query works, insert top recommendations into the `user_problem_recommendations` table.

```sql
WITH candidate_problems AS (
    SELECT
        p.id AS problem_id,
        p.leetcode_number,
        p.name,
        p.slug,
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
    LEFT JOIN user_problem_recommendations old_upr
        ON old_upr.problem_id = p.id
        AND old_upr.user_id = :user_id
        AND old_upr.status IN ('dismissed', 'solved')
    LEFT JOIN user_problem_recommendations active_upr
        ON active_upr.problem_id = p.id
        AND active_upr.user_id = :user_id
        AND active_upr.status IN ('pending', 'shown', 'clicked')
        AND (
            active_upr.expires_at IS NULL
            OR active_upr.expires_at > NOW()
        )
    WHERE usp.id IS NULL
      AND old_upr.id IS NULL
      AND active_upr.id IS NULL
),

topic_scores AS (
    SELECT
        cp.problem_id,
        MAX(
            CASE
                WHEN ult.problems_solved IS NULL THEN 20
                WHEN ult.problems_solved <= 5 THEN 40
                WHEN ult.problems_solved <= 15 THEN 25
                WHEN ult.problems_solved <= 30 THEN 10
                ELSE 0
            END
        ) AS topic_weakness_score
    FROM candidate_problems cp
    LEFT JOIN user_lc_topics ult
        ON ult.user_id = :user_id
        AND LOWER(cp.topics) LIKE '%' || LOWER(ult.tag_name) || '%'
    GROUP BY cp.problem_id
),

scored_problems AS (
    SELECT
        cp.problem_id,

        50 AS base_score,

        CASE
            WHEN cp.difficulty = 'Medium' THEN 30
            WHEN cp.difficulty = 'Hard' THEN 20
            WHEN cp.difficulty = 'Easy' THEN 10
            ELSE 0
        END AS difficulty_score,

        COALESCE(ts.topic_weakness_score, 20) AS topic_weakness_score,

        (
            50
            + CASE
                WHEN cp.difficulty = 'Medium' THEN 30
                WHEN cp.difficulty = 'Hard' THEN 20
                WHEN cp.difficulty = 'Easy' THEN 10
                ELSE 0
              END
            + COALESCE(ts.topic_weakness_score, 20)
        ) AS final_score,

        CASE
            WHEN COALESCE(ts.topic_weakness_score, 20) >= 40
                THEN 'Recommended because it is an important problem from one of your weakest topics'
            WHEN cp.difficulty = 'Medium'
                THEN 'Recommended because it is an important medium-level interview problem'
            WHEN cp.difficulty = 'Hard'
                THEN 'Recommended because it is an important advanced interview problem'
            WHEN cp.difficulty = 'Easy'
                THEN 'Recommended because it builds an important foundation'
            ELSE 'Recommended because it is an important standard problem'
        END AS reason

    FROM candidate_problems cp
    LEFT JOIN topic_scores ts
        ON ts.problem_id = cp.problem_id
)

INSERT INTO user_problem_recommendations (
    user_id,
    problem_id,
    reason,
    score,
    status,
    expires_at
)
SELECT
    :user_id,
    problem_id,
    reason,
    final_score,
    'pending',
    NOW() + INTERVAL '7 days'
FROM scored_problems
ORDER BY final_score DESC
LIMIT 20
ON CONFLICT (user_id, problem_id) DO NOTHING;
```

---

## 13. Why active_upr Is Needed

The insert query checks both old recommendations and active recommendations.

### old_upr

```sql
old_upr.status IN ('dismissed', 'solved')
```

This prevents recommending a problem again if the user already dismissed it or solved it through recommendation flow.

### active_upr

```sql
active_upr.status IN ('pending', 'shown', 'clicked')
```

This prevents creating duplicate active recommendations.

A recommendation is considered active when:

```text
status is pending, shown, or clicked
and
expires_at is NULL or expires_at is in the future
```

---

## 14. Recommendation Expiry

Each recommendation is inserted with an expiry time:

```sql
NOW() + INTERVAL '7 days'
```

This means the recommendation is valid for 7 days.

After expiry, the system can choose to generate fresh recommendations again.

---

## 15. Tracking Recommendation Events

Whenever the system creates, shows, clicks, dismisses, or solves a recommendation, insert an event into `recommendation_events`.

### Example: recommendation created

```sql
INSERT INTO recommendation_events (
    user_id,
    recommendation_id,
    problem_id,
    event_type,
    metadata
)
VALUES (
    :user_id,
    :recommendation_id,
    :problem_id,
    'recommendation_created',
    '{}'::jsonb
);
```

### Example: recommendation clicked

```sql
UPDATE user_problem_recommendations
SET
    status = 'clicked',
    clicked_at = NOW()
WHERE id = :recommendation_id
  AND user_id = :user_id;

INSERT INTO recommendation_events (
    user_id,
    recommendation_id,
    problem_id,
    event_type,
    metadata
)
VALUES (
    :user_id,
    :recommendation_id,
    :problem_id,
    'recommendation_clicked',
    '{}'::jsonb
);
```

### Example: recommendation dismissed

```sql
UPDATE user_problem_recommendations
SET
    status = 'dismissed',
    dismissed_at = NOW()
WHERE id = :recommendation_id
  AND user_id = :user_id;

INSERT INTO recommendation_events (
    user_id,
    recommendation_id,
    problem_id,
    event_type,
    metadata
)
VALUES (
    :user_id,
    :recommendation_id,
    :problem_id,
    'recommendation_dismissed',
    '{}'::jsonb
);
```

### Example: recommendation solved

```sql
UPDATE user_problem_recommendations
SET
    status = 'solved',
    solved_at = NOW()
WHERE user_id = :user_id
  AND problem_id = :problem_id
  AND status IN ('pending', 'shown', 'clicked');

INSERT INTO recommendation_events (
    user_id,
    recommendation_id,
    problem_id,
    event_type,
    metadata
)
SELECT
    user_id,
    id,
    problem_id,
    'recommendation_solved',
    '{}'::jsonb
FROM user_problem_recommendations
WHERE user_id = :user_id
  AND problem_id = :problem_id
ORDER BY recommended_at DESC
LIMIT 1;
```

---

## 16. Query to Fetch Recommendations for User

Use this query to show active recommendations to the user.

```sql
SELECT
    upr.id AS recommendation_id,
    upr.problem_id,
    p.leetcode_number,
    p.name,
    p.slug,
    p.difficulty,
    p.topics,
    p.url,
    upr.reason,
    upr.score,
    upr.status,
    upr.recommended_at,
    upr.expires_at
FROM user_problem_recommendations upr
JOIN problems p
    ON p.id = upr.problem_id
WHERE upr.user_id = :user_id
  AND upr.status IN ('pending', 'shown', 'clicked')
  AND (
      upr.expires_at IS NULL
      OR upr.expires_at > NOW()
  )
ORDER BY upr.score DESC, upr.recommended_at DESC;
```

---

## 17. Final Flow

```text
1. Phase 1 stores standard important problems in standard_important_problems.

2. Phase 2 finds standard problems that the user has not solved.

3. The system removes problems that were already dismissed, solved, or actively recommended.

4. Remaining candidate problems are scored using:
   - base_score
   - difficulty_score
   - topic_weakness_score

5. The highest scoring problems are inserted into user_problem_recommendations.

6. Recommendations are stored with:
   - reason
   - score
   - status
   - recommended_at
   - expires_at

7. User actions are tracked in recommendation_events.

8. When the user clicks, dismisses, or solves a recommendation:
   - update user_problem_recommendations
   - insert an event into recommendation_events

9. The frontend fetches active recommendations from user_problem_recommendations.

10. The user receives important unsolved problems ranked by usefulness.
```

---

## 18. Summary

Phase 2 adds a proper recommendation system on top of the standard important problems from Phase 1.

The system recommends problems that are:

```text
Important
Unsolved
Useful for the user's weak topics
Prioritized by difficulty
Not already dismissed
Not already actively recommended
```

The final output is a ranked list of LeetCode problems that can be recommended to the user.