# Interview Experience Embedding Strategy

## 1. Purpose of This Document

This document explains how embeddings should be used in the interview-experience search system. The goal is not to replace the existing database, but to add a smart semantic search layer on top of it.

The current database already has a useful structured shape:

```text
interview_posts
    ↓
interview_rounds
    ↓
round_problems
    ↓
problems
```

In this structure:

- `interview_posts` stores scraped interview experiences.
- `interview_rounds` stores rounds inside each experience.
- `round_problems` stores questions/problems asked in each round.
- `problems` stores LeetCode problem metadata.

Embeddings should help users search this data even when their query does not exactly match the text stored in the database.

---

## 2. Core Idea

Embeddings are a semantic search index on top of the structured database.

They should not be treated as a new database or as the final source of truth.

The normal database remains responsible for factual data:

```text
company
role
round type
post date
problem description
source URL
quality score
LeetCode problem link
```

Embeddings are used to find the most relevant database rows when the user's words are fuzzy, incomplete, or different from the stored text.

Example:

```text
User query:
Amazon cache eviction question
```

The database may contain:

```text
Design a data structure that supports get and put in O(1), and removes the least recently used item when capacity is full.
```

A normal SQL keyword search may not match this perfectly. Embeddings can understand that:

```text
cache eviction ≈ least recently used removal ≈ LRU Cache
```

So embeddings help route the query to the right row.

---

## 3. What Problem Embeddings Actually Solve

SQL works well when the user query matches exact fields.

For example:

```text
Amazon SDE-1 DSA round after 2024
```

This can be handled with SQL filters:

```sql
WHERE company_normalized = 'amazon'
AND role_normalized = 'sde-1'
AND round_type = 'dsa'
AND post_date >= '2024-01-01'
```

But SQL struggles with semantic queries such as:

```text
question where stack is used to find next greater element
cache replacement policy problem
tree problem where longest path is required
design system to send alerts
problem similar to LRU but with frequency
```

Embeddings help because they compare meaning, not only exact words.

---

## 4. Why Embedding Only the User Query Is Not Enough

Embedding only the user query is not useful by itself.

To compare meaning, both sides need vectors:

```text
User query text  → query vector
DB row text      → stored vector
```

Then the system compares them:

```text
similarity(query vector, DB row vector)
```

So the database rows also need embeddings.

A simple analogy:

```text
Query vector = user's location on a map
DB vectors   = restaurant locations on the map
Search       = find nearest restaurants
```

If only the user's location exists, there is nothing to compare it with.

---

## 5. What Should Be Embedded?

Do not embed the whole database.

Do not embed user tables, logs, sessions, passwords, analytics, or sync data.

### Do Not Embed

```text
users
password_hash
user_solved_problems
user_lc_profiles
user_lc_topics
chat_messages
chat_sessions
api_logs
sync_log
user_read_posts
```

These tables are useful for filtering, personalization, analytics, or user state, but not for semantic interview-question search.

### Embed First

The first and most useful embedding target is:

```text
round_problems
```

But do not embed `round_problems.problem_description` alone. Build a clean searchable text by joining it with parent context from:

```text
round_problems
+ interview_rounds
+ interview_posts
+ problems
```

This gives the embedding enough context.

---

## 6. Why Start With Problem-Level Embedding?

Most useful user queries are question-focused.

Examples:

```text
Amazon cache eviction question
Google tree recursion problem
Flipkart DP question
questions similar to LRU Cache
```

These queries are best answered by searching at the `round_problems` level.

So the practical MVP should be:

```text
one embedding per round_problems row
```

This is the simplest and highest-value implementation.

---

## 7. What About Post, Round, and Cluster Embeddings?

Earlier, four levels of embeddings were discussed:

```text
1. Post-level embedding
2. Round-level embedding
3. Event/question-level embedding
4. Canonical problem/cluster embedding
```

These are not all required at the beginning.

### 7.1 Problem-Level Embedding

This finds the right question/problem.

Useful for:

```text
LRU cache question
cache eviction problem
tree diameter style question
sliding window interview question
```

This should be implemented first.

### 7.2 Round-Level Embedding

This finds the right interview round.

Useful for:

```text
Flipkart machine coding round
Amazon DSA round experience
Google behavioral round questions
```

This is not required in MVP unless users frequently search for round-level experience.

### 7.3 Post-Level Embedding

This finds the right full interview experience.

Useful for:

```text
Amazon SDE-1 selected candidate experience
full Flipkart backend interview experience
interview experience with 3 rounds and system design
```

This is optional and can be added later.

### 7.4 Canonical Cluster Embedding

This groups similar questions into patterns.

Example:

```text
LRU Cache
LFU Cache
Cache with TTL
Design Redis cache
```

These can be grouped into:

```text
Cache design / cache eviction pattern
```

This is useful for recommendations and trend detection, but not required for the first search MVP.

---

## 8. Final Decision on Embedding Levels

For the current stage, do not implement all four levels.

Use this order:

```text
Phase 1: Problem-level embedding only
Phase 2: Add post-level embedding only if users search full experiences
Phase 3: Add round-level embedding only if users search round-wise experiences
Phase 4: Add cluster-level embedding only for recommendations and trends
```

The most practical starting point is:

```text
one embedding per round_problems row
```

---

## 9. How Embeddings Are Created Practically

Embeddings are created offline or in a background job.

They should not be created for all database rows every time a user searches.

### Offline Embedding Flow

```text
1. Read round_problems rows.
2. Join each row with interview_rounds, interview_posts, and problems.
3. Build clean embedding text.
4. Generate embedding vector.
5. Store the vector in an embedding table.
6. Link the embedding back to the original round_problem_id.
```

### Query-Time Flow

```text
1. User enters a query.
2. Embed only the user query.
3. Compare query embedding with stored row embeddings.
4. Get the most relevant round_problem_id values.
5. Fetch real data from the structured database.
6. Return search results or generate an answer from those rows.
```

---

## 10. Example Embedding Text

For each `round_problems` row, create text like this:

```text
Company: Amazon
Role: SDE-1
Round type: DSA
Round label: Technical Round 1
Question: Design a data structure that supports get and put in O(1), and removes the least recently used item when capacity is full.
Topics: HashMap, Linked List, Cache
Linked LeetCode problem: LRU Cache
Difficulty: Medium
Post date: 2024-05-12
Source: LeetCode
```

This is much better than embedding only:

```text
Design a data structure that supports get and put in O(1)
```

because the embedding now understands the company, role, round, topic, and linked problem context.

---

## 11. Where Embeddings Are Stored

For an easy MVP with SQLite, store embeddings in a separate table.

```sql
CREATE TABLE IF NOT EXISTS round_problem_embeddings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,

    round_problem_id INTEGER NOT NULL REFERENCES round_problems(id) ON DELETE CASCADE,

    embedded_text TEXT NOT NULL,
    embedded_text_hash TEXT NOT NULL,

    embedding_model TEXT NOT NULL,
    embedding_json TEXT NOT NULL,

    created_at TEXT DEFAULT (datetime('now')),

    UNIQUE(round_problem_id, embedding_model, embedded_text_hash)
);
```

This table directly links each embedding to the original DB row:

```text
round_problem_embeddings.round_problem_id → round_problems.id
```

For MVP, storing the vector as JSON in SQLite is acceptable if the dataset is small or medium.

Later, if scale increases, the vectors can move to:

```text
Postgres + pgvector
LanceDB
Chroma
FAISS
```

But these are not required at the beginning.

---

## 12. How the Original DB Links to the Embedding Store

The link is created through a stable row ID.

Example:

```text
Original DB:
round_problems.id = 125

Embedding table:
round_problem_id = 125
embedding_json = [0.12, -0.44, 0.87, ...]
```

So the original database and embedding table are connected like this:

```text
round_problems.id ↔ round_problem_embeddings.round_problem_id
```

If using an external vector store later, store a `vector_id`:

```sql
CREATE TABLE IF NOT EXISTS embedding_index (
    id INTEGER PRIMARY KEY AUTOINCREMENT,

    entity_type TEXT NOT NULL,
    entity_id INTEGER NOT NULL,

    vector_id TEXT NOT NULL,

    embedding_model TEXT NOT NULL,
    embedded_text TEXT NOT NULL,
    embedded_text_hash TEXT NOT NULL,

    created_at TEXT DEFAULT (datetime('now')),

    UNIQUE(entity_type, entity_id, embedding_model, embedded_text_hash)
);
```

Example:

```text
entity_type = round_problem
entity_id = 125
vector_id = round_problem_125_v1
```

This bridge table connects the original DB to the vector store.

---

## 13. How the Vector Store Links Back to the Original DB

When the user searches, vector search returns matching IDs.

Example result:

```json
[
  {
    "entity_type": "round_problem",
    "entity_id": 125,
    "score": 0.89
  },
  {
    "entity_type": "round_problem",
    "entity_id": 340,
    "score": 0.83
  }
]
```

Then the backend uses those IDs to fetch the actual database rows:

```sql
SELECT
    rp.id,
    rp.problem_description,
    rp.topics,
    r.round_type,
    r.round_label,
    p.company,
    p.role,
    p.post_date,
    p.source_url,
    p.quality_score,
    pr.name AS leetcode_name,
    pr.url AS leetcode_url
FROM round_problems rp
JOIN interview_rounds r ON rp.round_id = r.id
JOIN interview_posts p ON r.interview_post_id = p.id
LEFT JOIN problems pr ON rp.problem_id = pr.id
WHERE rp.id IN (125, 340);
```

So the embedding layer finds relevant row IDs, and the database provides the factual answer.

---

## 14. Does Embedding Generate the Final Answer?

No.

Embeddings mostly route the query to the correct database rows.

The final answer should come from the database.

```text
Embedding layer = semantic row finder
Database layer  = source of truth
Answer layer    = display or summarize DB rows
```

For a search UI, the system can directly display the matched rows.

For a chat UI, the system can fetch the rows and then summarize them using an LLM.

But the LLM should summarize only the fetched rows, not invent new data.

---

## 15. Practical Query Flow

User query:

```text
Amazon cache eviction follow-up
```

### Step 1: Extract filters

```json
{
  "company": "amazon",
  "semantic_query": "cache eviction follow-up"
}
```

### Step 2: SQL filter candidates

```sql
SELECT
    rp.id,
    e.embedding_json
FROM round_problem_embeddings e
JOIN round_problems rp ON e.round_problem_id = rp.id
JOIN interview_rounds r ON rp.round_id = r.id
JOIN interview_posts p ON r.interview_post_id = p.id
WHERE p.company_normalized = 'amazon'
  AND p.dedup_status = 'unique'
  AND p.hydrated = 1;
```

### Step 3: Embed user query

```text
cache eviction follow-up → query vector
```

### Step 4: Compare vectors

Use cosine similarity between the query vector and stored row vectors.

Example output:

```text
round_problem_id = 125, score = 0.89
round_problem_id = 340, score = 0.84
round_problem_id = 882, score = 0.78
```

### Step 5: Fetch real DB rows

```sql
SELECT
    rp.problem_description,
    rp.topics,
    r.round_type,
    r.round_label,
    p.company,
    p.role,
    p.post_date,
    p.source_url,
    p.quality_score,
    pr.name AS leetcode_name,
    pr.url AS leetcode_url
FROM round_problems rp
JOIN interview_rounds r ON rp.round_id = r.id
JOIN interview_posts p ON r.interview_post_id = p.id
LEFT JOIN problems pr ON rp.problem_id = pr.id
WHERE rp.id IN (125, 340, 882);
```

### Step 6: Return answer

Example:

```text
Found 3 Amazon cache-related interview questions:

1. LRU Cache / cache eviction
   Round: DSA
   Topics: HashMap, Linked List, Cache

2. Cache with TTL follow-up
   Round: LLD
   Topics: Cache, Expiry, System Design

3. Distributed cache discussion
   Round: System Design
   Topics: Redis, Consistency, Scaling
```

---

## 16. Use SQL Before Vector Comparison

Do not compare the query against every vector if it is not needed.

Bad approach:

```text
Compare query with every embedding in the DB every time.
```

Better approach:

```text
1. Use SQL filters first.
2. Narrow candidates by company, role, round type, dedup status, quality, or date.
3. Run vector similarity only on those candidates.
```

Example:

```text
User query: Amazon cache question
```

First filter:

```text
company_normalized = amazon
```

Then run vector similarity only on Amazon-related rows.

This keeps the system simpler and more efficient.

---

## 17. Why Not Embed the Whole DB?

Embedding the whole DB is wasteful and noisy.

Many tables are not semantic search content.

For example:

```text
users
sync_log
api_logs
chat_messages
user_read_posts
```

These do not need embeddings.

The useful searchable unit is the interview question/problem.

So the MVP should embed:

```text
round_problems + parent context
```

Not everything.

---

## 18. Edge Cases and How to Handle Them

### 18.1 Duplicate Posts

The schema already contains deduplication fields such as `content_hash`, `structural_fingerprint`, and `dedup_status`.

Avoid embedding duplicates when possible.

Rule:

```text
Only embed dedup_status = 'unique'
```

### 18.2 Stub or Non-Hydrated Posts

Some posts may have `hydrated = 0`, meaning raw content may be missing.

Rule:

```text
Prefer embedding only hydrated = 1 rows
```

### 18.3 Empty Problem Descriptions

Some `round_problems.problem_description` values may be too short or empty.

Rule:

```text
If problem_description is empty, fallback to linked LeetCode problem name, topics, and round notes.
```

### 18.4 Updated Rows

If a row changes, the old embedding may become stale.

Use `embedded_text_hash`.

Flow:

```text
1. Build embedding text.
2. Compute hash.
3. If hash already exists, skip.
4. If hash changed, regenerate embedding.
```

### 18.5 Deleted Rows

If a `round_problems` row is deleted, its embedding should also be deleted.

With SQLite foreign keys and `ON DELETE CASCADE`, this is simpler when embeddings are stored in SQLite.

### 18.6 Stale Vector Results

If using an external vector store later, sometimes vector search may return an ID that no longer exists in the DB.

Handle this safely:

```text
1. Try to fetch the DB row.
2. If missing, skip the result.
3. Optionally clean up the stale vector.
```

---

## 19. Recommended MVP Plan

### Phase 1: Keep Existing Schema

Do not break the current DB.

Use the existing structure:

```text
interview_posts
interview_rounds
round_problems
problems
```

### Phase 2: Add One Embedding Table

Add:

```text
round_problem_embeddings
```

### Phase 3: Build Embedding Script

Create a background script:

```text
scripts/embed_round_problems.py
```

The script should:

```text
1. Read round_problems.
2. Join with interview_rounds, interview_posts, and problems.
3. Build clean embedding text.
4. Generate embedding.
5. Store it in round_problem_embeddings.
```

### Phase 4: Build Search Endpoint

Example endpoint:

```text
GET /api/search/interview?q=cache eviction&company=amazon
```

It should:

```text
1. Extract filters from query.
2. Embed query.
3. SQL-filter candidate embeddings.
4. Compare vectors.
5. Fetch real DB rows.
6. Return results.
```

### Phase 5: Add Higher-Level Embeddings Later

Only add these if needed:

```text
post-level embeddings
round-level embeddings
canonical cluster embeddings
```

Do not build them before the problem-level search works.

---

## 20. Simple Human Explanation

Use embeddings as a smart search layer on top of the existing database, not as a replacement. The main data still stays in the normal tables: interview posts, rounds, round problems, and LeetCode problems. For each interview question/problem, create a clean text containing company, role, round type, question, topics, and linked LeetCode problem, then generate its embedding once and store it with the original `round_problem_id`. When a user searches something like “Amazon cache eviction question,” embed that query, compare it with stored embeddings, find the closest matching question IDs, and then fetch the real details from the database. SQL is still used for exact filters like company, role, date, and round, while embeddings help when the user's words do not exactly match the database text, like matching “cache eviction” with “LRU Cache.” For now, keep it simple: do not embed the whole database, just start with one embedding per interview question/problem row.

---

## 21. Final Recommendation

For the current stage, implement only this:

```text
one embedding per round_problems row
```

Use the embedding to route user queries to the right DB rows.

Use SQL and joins to fetch the final answer.

Do not start with complex infrastructure or all four embedding levels.

The best practical architecture is:

```text
SQLite structured DB
    ↓
round_problem_embeddings table
    ↓
query embedding
    ↓
cosine similarity search
    ↓
matched round_problem_id values
    ↓
SQL joins back to real DB rows
    ↓
answer/search result
```

This is simple, useful, and directly fits the current database.
