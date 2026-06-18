# CrackedIn.io: Competitor Recommendation Strategies Blueprint

This document summarizes recommendation strategies used by other coding-interview, LeetCode-tracking, spaced-repetition, roadmap, and practice platforms. It is written for CrackedIn.io, assuming the platform already has:

- User LeetCode synced ID.
- Solved and unsolved status for a standard problem sheet.
- Problem metadata such as title, difficulty, tags, topic, and sheet position.
- A goal to recommend what the user should review, solve next, reinforce, or avoid for now.

This file intentionally focuses on other platforms and reusable recommendation patterns. It does not use your competitor named in the previous message as a reference.

---

## 1. Executive summary

Most platforms use one or more of the following recommendation strategies:

| Strategy | Used by platforms like | Core idea | How CrackedIn.io can use it |
|---|---|---|---|
| Spaced repetition review queue | LeetSRS, LeetRecall, AlgoRecall, SpaceDSA, DSA Spaced Repetition | Recommend solved problems again when the user is likely to forget them. | Build a due-review queue for solved problems using SM-2, FSRS, or a simpler staged scheduler. |
| Rating-based scheduling | LeetSRS, AlgoRecall, DSA Spaced Repetition | User rates recall after solving: Again/Forgot, Hard, Good, Easy. | Ask for confidence after every solve/review and use it to update next review date. |
| Auto-capture from LeetCode | LeetRecall, leetcode-review, HashTry-style trackers | Automatically detect accepted submissions and attempt behavior. | Use synced LeetCode history as the base truth, then ask only for missing subjective signals. |
| Readiness score | HashTry, Aurora-like open-source trackers | Convert quality, speed, topic coverage, consistency, and confidence into one score. | Build CrackedIn Readiness Score by topic and overall. |
| Roadmap progression | NeetCode, Striver SDE Sheet, Grind 75, LeetDaily | Recommend the next unsolved problem from a curated order. | Use your standard sheet order as a baseline recommendation. |
| Pattern-first progression | AlgoMonster, NeetCode, Striver A2Z-style sections | Group problems by interview pattern and recommend from easy to hard. | Classify every problem by real pattern, not just broad tags like Array or String. |
| Weak-topic reinforcement | HashTry, DSA Spaced Repetition, SpaceDSA | Recommend problems from topics where the user struggles. | Use failed attempts, low confidence, long solve time, hints, and overdue reviews to detect weak patterns. |
| Session-scoped practice | SpaceDSA, DSA Spaced Repetition, AlgoMonster Speedrun | Let users focus on a difficulty, topic, list, or short time block. | Add modes: 30-minute review, DP-only, next roadmap, company target, weak-topic drill. |
| Gamified mastery | Codewars | Use ranks, XP, progress, and difficulty-relative rewards. | Add pattern levels: Sliding Window L3, Graph BFS L2, DP Stability 41%. |
| Feedback/mentoring loop | Exercism | Recommend review based on submitted solution quality and feedback needs. | Add AI or peer feedback after solve: complexity, missed edge cases, pattern explanation. |

The best strategy for CrackedIn.io is not one algorithm. It should be a hybrid recommender:

```txt
Recommendation = SRS due reviews
               + weak-pattern reinforcement
               + roadmap next-unsolved problem
               + similar-problem practice
               + prerequisite bridge problems
               + diversity and goal matching
```

---

## 2. Platform-by-platform recommendation strategy analysis

## 2.1 LeetSRS

### What it appears to do

LeetSRS is a Chrome extension that adds spaced repetition to LeetCode practice. Public pages describe an embedded LeetCode workflow, card rating, review queue building, local-first/no-login storage, stats, streaks, pausing cards, import/export, GitHub Gist sync, and use of a leading scheduling algorithm for review queue building.

### Likely recommendation strategy

LeetSRS is primarily a solved-problem review recommender.

```txt
Input:
- Problem saved as a card.
- User rating after review.
- Last review timestamp.
- Scheduler state.

Candidate pool:
- Problems already added to the user's review deck.

Ranking:
- Problems due today or overdue appear first.
- Problems with lower memory stability are scheduled sooner.
- Problems rated Easy move further out.
- Problems rated Again/Hard come back sooner.

Output:
- Daily review queue.
```

### What CrackedIn.io can learn

Use SRS as the review engine, but do not make users manually create every card. Because CrackedIn.io has LeetCode sync and solved/unsolved sheet status, it can automatically create review cards for solved sheet problems.

Recommended implementation:

```ts
type RecallRating = "again" | "hard" | "good" | "easy";

interface ReviewCard {
  userId: string;
  problemId: string;
  lastReviewedAt: Date | null;
  nextReviewAt: Date;
  ratingHistory: RecallRating[];
  stability?: number;
  difficulty?: number;
  retrievability?: number;
  paused: boolean;
}
```

CrackedIn.io recommendation module:

```txt
If problem is solved and nextReviewAt <= today:
  recommend as review.
If problem is solved but weak pattern is high-risk:
  recommend earlier review.
If problem was solved recently with Easy rating:
  deprioritize review.
```

---

## 2.2 HashTry

### What it appears to do

HashTry is positioned as LeetCode analytics plus spaced repetition. Public pages describe an Interview Readiness Score from 0 to 100 based on five signals: solve speed, topic coverage, solution quality, consistency, and self-assessment accuracy. It also references attempt tracking, tiers/grades, weak topics, smart filters, and a Start Solving workflow.

### Likely recommendation strategy

HashTry is not just a due-review scheduler. It is closer to a readiness-driven recommender.

```txt
Input:
- Problems attempted.
- Time spent.
- User self-assessment.
- Quality tier or grade.
- Topic coverage.
- Consistency history.

Candidate pool:
- Problems from selected lists.
- Problems that affect weak score components.
- Reviews that are due or low-confidence.

Ranking:
- Prioritize the score component dragging readiness down.
- If topic coverage is low, recommend unsolved problems in missing topics.
- If solve speed is poor, recommend timed re-solves or easier adjacent problems.
- If quality tier is low, recommend review before new problems.
- If consistency is low, recommend smaller daily tasks.

Output:
- Readiness score.
- Weak topic breakdown.
- Smart filters and next action.
```

### What CrackedIn.io can learn

Build recommendations around readiness gaps, not just due dates.

CrackedIn.io should create a per-topic readiness model:

```ts
interface TopicReadiness {
  userId: string;
  topic: string;
  coverageScore: number;       // solved relevant problems / expected problems
  retentionScore: number;      // review pass rate and overdue count
  speedScore: number;          // solve time normalized by difficulty
  qualityScore: number;        // unaided solve, code quality, attempts
  consistencyScore: number;    // recent practice rhythm
  confidenceCalibration: number; // user confidence vs actual result
  overall: number;
}
```

Recommended formula:

```txt
readiness_score =
  0.25 * coverage_score
+ 0.20 * retention_score
+ 0.20 * quality_score
+ 0.15 * speed_score
+ 0.10 * consistency_score
+ 0.10 * confidence_calibration
```

Recommendation behavior:

```txt
If readiness is low because of coverage:
  recommend next unsolved sheet problems in that topic.

If readiness is low because of retention:
  recommend overdue reviews.

If readiness is low because of speed:
  recommend timed re-solves of already-solved problems.

If readiness is low because confidence is inaccurate:
  recommend active recall prompts before code.
```

---

## 2.3 LeetReviewer

### What it appears to do

LeetReviewer allows users to add problems by URL, auto-fill metadata, import curated lists such as Blind 75, NeetCode 150, and Hot 100, use a try-to-recall workflow, and review problems with spaced repetition.

### Likely recommendation strategy

LeetReviewer combines curated-list import with active recall.

```txt
Input:
- Imported list or manually added problem.
- Last review date.
- Review success/failure.
- Recall note quality or review action.

Candidate pool:
- Imported or manually added problems.

Ranking:
- Problems due by SRS date.
- Problems where user failed to recall pattern.
- Problems from imported lists still not reviewed enough.

Output:
- Review cards that force recall before seeing notes or solution.
```

### What CrackedIn.io can learn

Before a user re-solves a problem, ask active recall questions:

```txt
1. What is the core pattern?
2. What data structure is required?
3. What invariant must be maintained?
4. What edge case usually breaks this solution?
5. What is the time complexity?
```

Then score the review outcome:

```ts
interface RecallAttempt {
  userId: string;
  problemId: string;
  recalledPattern: boolean;
  recalledInvariant: boolean;
  recalledEdgeCases: boolean;
  solvedCode: boolean;
  usedHint: boolean;
  durationMinutes: number;
}
```

Recommendation behavior:

```txt
If user can recall pattern but cannot implement:
  recommend same-pattern easier implementation problem.

If user can implement but misses edge cases:
  recommend edge-case-focused similar problems.

If user cannot recall pattern:
  reset review interval and recommend a concept recap.
```

---

## 2.4 LeetRecall

### What it appears to do

LeetRecall's public listing describes zero-setup use, automatic submission tracking, SM-2 spaced repetition, a Due Today popup, dashboard with proficiency badges, acceptance rates, average solve times, reminders, badge counts, and offline/local operation.

### Likely recommendation strategy

LeetRecall is auto-capture plus SM-2.

```txt
Input:
- Accepted submissions.
- Problem attempt history.
- Acceptance rate.
- Solve time.
- SM-2 state.

Candidate pool:
- Solved problems detected automatically.

Ranking:
- Due today first.
- Overdue problems before future problems.
- Lower-proficiency problems before mastered problems.
- Problems with worse solve-time or acceptance history may get higher priority.

Output:
- Daily review popup and dashboard.
```

### What CrackedIn.io can learn

Make recommendations automatic. Users should not need to remember to track problems.

CrackedIn.io can do:

```txt
1. Sync accepted LeetCode problems.
2. Create review card for every solved sheet problem.
3. Detect if a user solved a problem outside the sheet and optionally add it.
4. Show due count in the dashboard.
5. Send reminders only when due queue is non-empty.
```

Useful ranking adjustment:

```txt
review_priority =
  5 * is_overdue
+ 2 * days_overdue
+ 3 * low_proficiency_badge
+ 2 * slow_recent_solve
+ 2 * low_acceptance_history
```

---

## 2.5 AlgoRecall

### What it appears to do

AlgoRecall is a local-first spaced-repetition extension for LeetCode-style practice. Public listings describe rating after submit, due reviews, automatic saving of problem title/link, accepted solution code, tags/patterns when available, review history, and next review date.

### Likely recommendation strategy

AlgoRecall converts accepted submissions into review cards and updates them with ratings.

```txt
Input:
- Accepted solution.
- User rating: Again, Hard, Good, Easy.
- Problem metadata.
- Review history.

Candidate pool:
- Solved problems with review cards.

Ranking:
- Scheduler due date.
- Rating severity.
- Recent failure/lapse.

Output:
- Scheduled review list.
```

### What CrackedIn.io can learn

Accepted code is valuable. Store a snapshot of the user's accepted approach when possible and use it in review.

Review UI idea:

```txt
Step 1: Ask user to recall approach.
Step 2: Ask user to solve again.
Step 3: Let user compare with previous accepted code.
Step 4: Ask rating.
Step 5: Update next review date.
```

Data model:

```ts
interface SolutionSnapshot {
  userId: string;
  problemId: string;
  submittedAt: Date;
  language: string;
  codeHash: string;
  code?: string;
  runtimeMs?: number;
  memoryMb?: number;
  complexityNote?: string;
}
```

---

## 2.6 SpaceDSA

### What it appears to do

SpaceDSA tracks LeetCode problem-solving time, automatically schedules reviews, and lets users focus review sessions by problem difficulty and type.

### Likely recommendation strategy

SpaceDSA is session-scoped SRS.

```txt
Input:
- Solved problems.
- Time spent.
- Review schedule.
- Difficulty/type filter selected by user.

Candidate pool:
- Due problems matching selected filters.

Ranking:
- Due/overdue first.
- Then difficulty/type match.
- Then likely memory risk.

Output:
- A filtered review session.
```

### What CrackedIn.io can learn

Add recommendation modes rather than one generic queue.

Useful modes:

```txt
- Review due problems only.
- 30-minute review session.
- Medium-only practice.
- Graphs-only practice.
- Weakest topic session.
- Next roadmap session.
- Company-targeted session.
- Mixed interview simulation.
```

Session generator:

```ts
interface SessionRequest {
  userId: string;
  minutes: number;
  mode: "review" | "weak_topic" | "roadmap" | "mixed" | "company";
  topics?: string[];
  difficulties?: Array<"Easy" | "Medium" | "Hard">;
  maxProblems?: number;
}
```

---

## 2.7 DSA Spaced Repetition extension

### What it appears to do

The public listing describes a multi-platform extension with ratings like Forgot, Hard, Good, Easy; SM-2 scheduling; custom rescheduling; curated lists such as Blind 75, NeetCode 150, and LeetCode 75; daily auto-queuing; per-list problem quotas; topic-wise or random queueing; progress rings; heatmaps; review forecasts; difficulty breakdowns; and topic-wise progress bars.

### Likely recommendation strategy

This is a hybrid of SRS plus daily new-problem queueing.

```txt
Input:
- User selected lists.
- Problems per day per list.
- Topic preference.
- SRS state.
- User rating.
- Progress per list/topic.

Candidate pools:
- Due review problems.
- New problems from selected curated lists.
- Topic-specific queue.
- Random queue if user chooses discovery.

Ranking:
- Due review problems have top priority.
- New problems are auto-queued based on per-list daily target.
- Topic mode filters candidates by selected topic.
- Random mode increases variety.

Output:
- Daily queue containing due reviews and new list problems.
```

### What CrackedIn.io can learn

Because CrackedIn.io already has a standard sheet, add configurable daily quota:

```txt
Daily quota examples:
- 2 reviews + 1 new roadmap problem.
- 3 reviews + 2 weak-topic problems.
- 1 review + 1 easy confidence booster + 1 medium stretch.
```

Daily queue algorithm:

```txt
1. Fill overdue reviews first.
2. Fill weak-topic reinforcement second.
3. Fill next unsolved sheet problems third.
4. Add one diversity problem if the session is too narrow.
5. Add one stretch problem only if recent pass rate is high.
```

---

## 2.8 leetcode-review open-source tool

### What it appears to do

This self-hosted project pulls accepted submissions through LeetCode GraphQL, stores data, and schedules reviews using SM-2.

### Likely recommendation strategy

It is sync-first and scheduler-first.

```txt
Input:
- Accepted LeetCode submissions.
- Problem metadata.
- SM-2 state.

Candidate pool:
- Accepted problems.

Ranking:
- SM-2 due date.

Output:
- Self-hosted review schedule.
```

### What CrackedIn.io can learn

Separate data ingestion from recommendation ranking.

Recommended architecture:

```txt
LeetCode sync job
  -> normalized problem table
  -> user problem status table
  -> attempt event table
  -> review scheduler
  -> recommendation service
```

This separation lets CrackedIn.io switch from fixed intervals to SM-2 or FSRS later without rewriting sync.

---

## 2.9 NeetCode

### What it appears to do

NeetCode provides structured problem sets such as NeetCode 150, organized by major DSA categories and difficulty, with progress tracking and video/explanation support. The public NeetCode 150 page shows solved counts, difficulty counts, and topic sections such as Arrays and Hashing, Two Pointers, Sliding Window, Stack, Binary Search, Linked List, Trees, Graphs, Dynamic Programming, Greedy, Intervals, Math, and Bit Manipulation.

### Likely recommendation strategy

NeetCode is roadmap-first.

```txt
Input:
- Curated list.
- Topic ordering.
- Difficulty ordering.
- User solved status.

Candidate pool:
- Unsolved problems in the roadmap.

Ranking:
- Topic section order.
- Problem order inside topic.
- Difficulty progression.

Output:
- Next problem in selected roadmap section.
```

### What CrackedIn.io can learn

Your standard sheet should be the default backbone. But it should not be the only ranking signal.

CrackedIn.io enhancement:

```txt
roadmap_score =
  1 / (1 + sheet_order_index)
```

Then combine it with user-specific signals:

```txt
new_problem_score =
  0.25 * roadmap_score
+ 0.25 * weak_pattern_match
+ 0.20 * difficulty_fit
+ 0.15 * prerequisite_readiness
+ 0.10 * similarity_to_recent_learning
+ 0.05 * diversity_bonus
```

This means the next sheet problem is recommended unless the user has a more urgent weak area or prerequisite gap.

---

## 2.10 AlgoMonster

### What it appears to do

AlgoMonster emphasizes pattern-based learning. Public pages describe a curriculum of core patterns, hundreds of lessons/problems, algorithm templates, flowcharts for algorithm selection, keyword-to-algorithm tools, runtime-to-algorithm tools, practice lists, speedrun mode, company guides, and stats.

### Likely recommendation strategy

AlgoMonster is pattern-first and concept-sequenced.

```txt
Input:
- Pattern curriculum.
- User progress through lessons.
- Problem pattern.
- Difficulty.
- Algorithm template.

Candidate pool:
- Problems belonging to the current or next pattern.

Ranking:
- Prerequisite concept order.
- Pattern progression.
- Difficulty progression.
- Practice volume required for mastery.

Output:
- Next lesson/problem for a pattern.
- Algorithm decision guidance.
```

### What CrackedIn.io can learn

Tags like Array, Hash Table, and String are too broad for high-quality recommendations. Build a pattern taxonomy.

Example taxonomy:

```txt
Array
  - prefix sum
  - prefix sum + hashmap
  - two pointers on sorted array
  - sliding window fixed size
  - sliding window variable size
  - kadane
  - cyclic sort

Stack
  - monotonic increasing stack
  - monotonic decreasing stack
  - next greater element
  - histogram boundary expansion

Graph
  - BFS traversal
  - BFS shortest path unweighted
  - DFS connected components
  - topological sort
  - union find
  - Dijkstra

Dynamic Programming
  - 1D recurrence
  - 2D grid DP
  - knapsack
  - LIS
  - interval DP
  - bitmask DP
```

Pattern-based recommendation:

```txt
If user solved 1 easy variable-window problem:
  recommend another variable-window problem with one new constraint.

If user failed binary search on answer:
  recommend easier monotonic predicate problem before harder ones.

If user repeatedly uses hints on DP state definition:
  recommend state-definition drills, not random DP problems.
```

---

## 2.11 Striver SDE Sheet / TakeUForward

### What it appears to do

The Striver SDE Sheet page describes a curated set of top coding interview questions, progress tracking, revision, all-problems view, difficulty counts, random problem, and topic sections including arrays, linked lists, greedy, recursion, binary search, heaps, stack/queue, strings, trees, graphs, dynamic programming, and trie.

### Likely recommendation strategy

Striver's sheet is curated-topic progression plus revision/random practice.

```txt
Input:
- Sheet topic section.
- Problem order.
- Difficulty.
- Solved status.
- Revision flag.

Candidate pool:
- Unsolved problems in current section.
- Problems marked for revision.
- Random problem pool.

Ranking:
- Sheet order for structured learning.
- Revision flag for memory refresh.
- Random mode for variety.

Output:
- Next problem, revision problem, or random practice problem.
```

### What CrackedIn.io can learn

Add three recommendation buttons:

```txt
1. Continue Sheet
   - next unsolved problem by sheet order.

2. Revise Weak
   - solved problems due or low-confidence.

3. Surprise Me
   - random unsolved problem filtered by appropriate difficulty and weak topic.
```

Useful rule:

```txt
Do not use pure random. Use weighted random:
  40% weak topic
  30% current sheet topic
  20% neglected topic
  10% confidence booster
```

---

## 2.12 Grind 75

### What it appears to do

Grind 75 is a customizable coding-interview problem list. Public pages describe customization by available time, difficulty, topics, and a weekly schedule. The listed plan shows weeks, problem counts, estimated minutes, difficulty, and topic visibility.

### Likely recommendation strategy

Grind 75 is goal/time-constrained planning.

```txt
Input:
- Available weeks or study time.
- Difficulty preference.
- Topic preference.
- Problem list.
- Estimated time per problem.

Candidate pool:
- Curated interview problems.

Ranking:
- Maximize interview coverage under time constraints.
- Place easier/foundational problems earlier.
- Spread topics across weeks.
- Include estimated time budget.

Output:
- Weekly plan.
```

### What CrackedIn.io can learn

Add a deadline-aware planner:

```txt
User input:
- Interview date: 30 days away.
- Daily time: 60 minutes.
- Goal: backend SWE interview.

CrackedIn output:
- Week 1: arrays, hashing, two pointers, basic trees.
- Week 2: graphs, binary search, stack.
- Week 3: DP, intervals, heaps.
- Week 4: reviews, mocks, weakest topics.
```

Deadline formula:

```txt
weekly_capacity_minutes = days_available_per_week * minutes_per_day
problem_cost = estimated_solve_time + estimated_review_time
select problems that maximize:
  interview_frequency + weakness_relevance + prerequisite_value
subject to:
  total_cost <= capacity
```

---

## 2.13 LeetDaily

### What it appears to do

LeetDaily tracks daily LeetCode challenges and curated DSA sheets. Its public page describes progress bars, sequential next-unsolved problems, problem metadata, solved status, and countdown timer for the daily challenge.

### Likely recommendation strategy

LeetDaily is habit-first and sequential-list-first.

```txt
Input:
- Daily challenge.
- Selected curated sheets.
- Solved status.
- Progress per sheet.

Candidate pool:
- Today's challenge.
- Next unsolved problem from each sheet.

Ranking:
- Daily challenge gets time-sensitive priority.
- Sequential next-unsolved keeps roadmap progress simple.
- Progress bars motivate completion.

Output:
- Today's challenge.
- Next unsolved sheet problem.
```

### What CrackedIn.io can learn

Add a habit layer:

```txt
Today tab:
- 1 daily challenge or warm-up.
- 2 due reviews.
- 1 next unsolved sheet problem.
- 1 weak-topic reinforcement.
```

Progress UI:

```txt
Sheet Progress: 47/150
Today Due: 6
Weakest Topic: Graph BFS
Next Best Problem: Rotting Oranges
```

---

## 2.14 Codewars

### What it appears to do

Codewars uses gamification with ranks and honor. Documentation explains that kata difficulty affects rank progress and that completing harder kata around or above the user's rank moves rank more meaningfully than repeatedly doing low-level kata. Honor reflects broader activity and contribution.

### Likely recommendation strategy

Codewars encourages difficulty-relative progression.

```txt
Input:
- User rank.
- Kata rank/difficulty.
- Completed kata.
- Language or skill area.

Candidate pool:
- Unsolved kata.

Ranking:
- Problems near user rank.
- Slightly harder problems for rank growth.
- Easier problems for warm-up or activity.

Output:
- Practice items that progress rank and activity.
```

### What CrackedIn.io can learn

Add pattern XP and levels.

Example:

```txt
Sliding Window Level 1:
- fixed window max/min

Sliding Window Level 2:
- variable window with set/map

Sliding Window Level 3:
- frequency constraint

Sliding Window Level 4:
- minimum window / multiple counters
```

Recommendation behavior:

```txt
If user is Level 2 in Graph BFS:
  recommend Level 2 consolidation or Level 3 stretch.

If user keeps solving only Level 1 array problems:
  reduce XP reward and recommend harder or more diverse problems.
```

---

## 2.15 Exercism

### What it appears to do

Exercism combines learning, practice, automated analysis, and mentoring. Public docs emphasize mentoring workflows and feedback.

### Likely recommendation strategy

Exercism is feedback-loop-driven learning.

```txt
Input:
- Submitted solution.
- Automated analysis.
- Mentor feedback.
- Exercise track progression.

Candidate pool:
- Next exercises.
- Exercises needing feedback or revision.

Ranking:
- Concepts not mastered.
- Solutions needing revision.
- Next exercise in track.

Output:
- Continue, revise, or request feedback.
```

### What CrackedIn.io can learn

Use feedback quality as a recommendation signal.

Example:

```txt
If user solves but code is O(n^2) where O(n) is expected:
  recommend optimization review.

If user misses edge cases:
  recommend edge-case drill.

If user solves correctly but cannot explain:
  recommend active recall explanation task.
```

AI feedback object:

```ts
interface SolveFeedback {
  problemId: string;
  userId: string;
  correctness: "unknown" | "failed" | "passed";
  patternIdentified: boolean;
  optimalComplexity: boolean;
  edgeCasesCovered: boolean;
  implementationQuality: 1 | 2 | 3 | 4 | 5;
  explanationQuality: 1 | 2 | 3 | 4 | 5;
  recommendedAction: "review" | "similar_easy" | "similar_medium" | "optimize" | "move_on";
}
```

---

# 3. Unified recommendation system for CrackedIn.io

## 3.1 Recommendation types

CrackedIn.io should show recommendations in separate lanes, because users have different intentions at different times.

```txt
1. Review Now
   Solved problems that are due or likely to be forgotten.

2. Next Best Unsolved
   Best unsolved problem from the user's standard sheet.

3. Weak Pattern Drill
   Problems that target the user's weakest patterns.

4. Similar Practice
   Problems similar to the current/recent problem.

5. Bridge Problem
   Easier prerequisite before a hard target problem.

6. Timed Practice
   Problems that fit the user's available time.

7. Interview Readiness Plan
   Mixed plan based on target date/company/topic.
```

---

## 3.2 Data inputs

Use three kinds of signals.

### A. Objective signals from LeetCode sync

```txt
- Solved status.
- Accepted submissions.
- Last accepted date.
- Submission count if available.
- Language if available.
- Runtime/memory if available.
- Problem difficulty.
- Problem tags.
```

### B. Sheet signals

```txt
- Sheet membership.
- Sheet order index.
- Topic section.
- Difficulty distribution.
- Prerequisite relationship.
- Whether problem is core, optional, or stretch.
```

### C. User behavior signals

```txt
- Confidence rating.
- Recall rating.
- Hint usage.
- Editorial usage.
- Solve time.
- Failed reviews.
- Skipped recommendations.
- Problem bookmarks.
- Target company or interview date.
```

---

## 3.3 Problem model

```ts
interface Problem {
  id: string;
  leetcodeSlug: string;
  title: string;
  difficulty: "Easy" | "Medium" | "Hard";
  officialTags: string[];
  patternTags: string[];
  subpatterns: string[];
  sheetIds: string[];
  sheetOrder: Record<string, number>;
  estimatedMinutes: number;
  prerequisiteProblemIds: string[];
  similarProblemIds: string[];
  contrastProblemIds: string[];
  isPremium?: boolean;
}
```

## 3.4 User problem state

```ts
interface UserProblemState {
  userId: string;
  problemId: string;
  status: "unsolved" | "attempted" | "solved" | "reviewing" | "mastered";
  firstSolvedAt?: Date;
  lastAttemptAt?: Date;
  lastReviewedAt?: Date;
  nextReviewAt?: Date;
  reviewStage: number;
  recallRating?: "again" | "hard" | "good" | "easy";
  confidence: 1 | 2 | 3 | 4 | 5;
  personalDifficulty: 1 | 2 | 3 | 4 | 5;
  attemptsCount: number;
  failedAttemptsCount: number;
  hintUsedCount: number;
  editorialUsedCount: number;
  avgSolveTimeMinutes?: number;
  lastSolveTimeMinutes?: number;
  reviewLapses: number;
  skippedCount: number;
}
```

---

# 4. Candidate generation

Do not ask an AI model to directly pick one problem from the entire database. First generate candidates from focused pools.

## 4.1 Pool A: due reviews

```txt
Candidate if:
- status is solved/reviewing/mastered
- nextReviewAt <= today
```

Priority:

```txt
review_due_score =
  3.0 * days_overdue
+ 2.0 * low_confidence
+ 2.0 * recent_hint_usage
+ 3.0 * review_lapses
+ 1.5 * weak_pattern_risk
+ 1.0 * slow_solve_time
```

## 4.2 Pool B: weak-pattern unsolved problems

```txt
Candidate if:
- status is unsolved or attempted
- problem pattern is in user's weak pattern list
- prerequisites are mostly satisfied
```

Priority:

```txt
weak_unsolved_score =
  4.0 * pattern_weakness
+ 2.0 * difficulty_fit
+ 2.0 * prerequisite_readiness
+ 1.5 * sheet_relevance
+ 1.0 * recent_related_learning
```

## 4.3 Pool C: roadmap next problems

```txt
Candidate if:
- problem is in selected standard sheet
- status is unsolved
```

Priority:

```txt
roadmap_score = 1 / (1 + sheet_order_index)
```

Boost if current section is partially completed:

```txt
section_momentum_bonus = solved_in_section / total_in_section
```

## 4.4 Pool D: similar problems

```txt
Candidate if:
- shares pattern/subpattern with current problem
- not solved too recently
- not too far outside user's difficulty range
```

Priority:

```txt
similarity_score =
  0.30 * shared_subpattern
+ 0.20 * shared_data_structure
+ 0.15 * shared_solution_template
+ 0.15 * constraint_shape_similarity
+ 0.10 * difficulty_fit
+ 0.10 * novelty
```

## 4.5 Pool E: prerequisite bridge problems

```txt
Candidate if:
- unsolved target problem is too hard
- candidate is a prerequisite or easier same-pattern problem
```

Priority:

```txt
bridge_score =
  4.0 * prerequisite_value
+ 2.0 * target_problem_importance
+ 2.0 * difficulty_accessibility
+ 1.0 * pattern_overlap
```

## 4.6 Pool F: diversity problems

```txt
Candidate if:
- topic has low recent practice
- not already overloaded in today's plan
```

Priority:

```txt
diversity_score =
  3.0 * topic_neglect
+ 1.5 * sheet_relevance
+ 1.0 * difficulty_fit
```

---

# 5. Final ranking

## 5.1 Review recommendation score

Use this for solved problems.

```txt
review_recommendation_score =
  0.35 * due_urgency
+ 0.20 * memory_risk
+ 0.15 * pattern_weakness
+ 0.10 * confidence_gap
+ 0.10 * solve_quality_risk
+ 0.05 * goal_relevance
+ 0.05 * diversity_need
- 0.20 * recently_reviewed_penalty
```

Where:

```txt
due_urgency:
  1.0 if overdue, scaled by days overdue.

memory_risk:
  High if last rating was Again/Hard, low if Easy.

pattern_weakness:
  High if user struggles with the problem's pattern.

confidence_gap:
  High if confidence is low or confidence is inaccurate.

solve_quality_risk:
  High if solved with hints/editorial or too slowly.

recently_reviewed_penalty:
  Prevents recommending the same problem too often.
```

## 5.2 New problem recommendation score

Use this for unsolved problems.

```txt
new_problem_recommendation_score =
  0.25 * sheet_priority
+ 0.20 * weak_pattern_match
+ 0.20 * difficulty_fit
+ 0.15 * prerequisite_readiness
+ 0.10 * recent_learning_similarity
+ 0.05 * diversity_bonus
+ 0.05 * goal_relevance
- 0.20 * too_hard_penalty
- 0.10 * repeated_topic_penalty
```

## 5.3 Similar problem score

Use this on problem detail pages.

```txt
similar_problem_score =
  0.30 * same_subpattern
+ 0.20 * same_core_template
+ 0.15 * same_data_structure
+ 0.15 * difficulty_progression_fit
+ 0.10 * sheet_or_company_relevance
+ 0.10 * novelty
```

Return similar problems in four buckets:

```txt
1. Easier prerequisite.
2. Same pattern practice.
3. Harder follow-up.
4. Same pattern but different surface story.
```

Example:

```txt
Current problem: Daily Temperatures

Easier prerequisite:
- Next Greater Element I

Same pattern:
- Online Stock Span

Harder follow-up:
- Largest Rectangle in Histogram

Different-looking same pattern:
- Remove K Digits
```

---

# 6. Daily plan strategy

A good daily plan should not be only reviews and should not be only new problems.

## 6.1 Default daily plan

```txt
50% due reviews
25% weak-topic reinforcement
15% next roadmap problem
10% confidence booster or stretch problem
```

Example:

```txt
Today's CrackedIn Plan

1. Review: Longest Substring Without Repeating Characters
   Reason: due today, variable sliding window, last rating Hard.

2. Review: Product of Array Except Self
   Reason: overdue by 3 days, prefix-product pattern appears in your weak list.

3. Weak drill: Permutation in String
   Reason: same sliding window family but adds frequency matching.

4. Next sheet problem: Koko Eating Bananas
   Reason: next unsolved Binary Search problem in your sheet.

5. Confidence booster: Valid Palindrome
   Reason: quick two-pointer warm-up to keep streak active.
```

## 6.2 Beginner daily plan

```txt
60% roadmap next
25% easy reviews
15% concept recap
```

## 6.3 Interview-in-30-days plan

```txt
40% weak topics
30% due reviews
20% medium/hard mixed practice
10% timed mock
```

## 6.4 Burnout-safe plan

If user skipped several days:

```txt
- Recommend fewer problems.
- Prefer easy/medium reviews.
- Avoid hard stretch problems immediately.
- Show a restart plan, not a guilt dashboard.
```

---

# 7. Weak pattern detection

## 7.1 Signals

A topic is weak if:

```txt
- User has low solve rate in that pattern.
- User has high hint/editorial usage.
- User's average solve time is high.
- User failed recent reviews.
- User skips recommendations in that pattern.
- User confidence is low.
- User confidence is high but actual results are poor.
```

## 7.2 Formula

```txt
pattern_weakness_score =
  0.25 * failure_rate
+ 0.20 * hint_usage_rate
+ 0.15 * review_lapse_rate
+ 0.15 * slow_solve_rate
+ 0.10 * low_confidence_rate
+ 0.10 * low_coverage
+ 0.05 * skip_rate
```

## 7.3 Output

```txt
Weakest patterns:
1. Binary Search on Answer - 78/100 weakness
2. 1D Dynamic Programming - 72/100 weakness
3. Graph BFS Shortest Path - 66/100 weakness
4. Monotonic Stack - 61/100 weakness
```

Then connect each weak pattern to concrete recommendations:

```txt
Binary Search on Answer:
- Review: Capacity To Ship Packages Within D Days
- Bridge: First Bad Version
- New: Koko Eating Bananas
- Stretch: Split Array Largest Sum
```

---

# 8. Similar problem recommendation design

## 8.1 Do not rely only on LeetCode tags

Bad:

```txt
Problem A has tag Array.
Problem B has tag Array.
Therefore they are similar.
```

Good:

```txt
Problem A uses prefix sum + hashmap to count target subarrays.
Problem B uses prefix sum + hashmap to detect repeated remainder.
They are similar because the invariant is the same.
```

## 8.2 Similarity dimensions

```txt
- Core pattern.
- Subpattern.
- Data structure.
- Invariant.
- Constraint shape.
- Difficulty delta.
- Implementation template.
- Edge cases.
- Sheet/company relevance.
```

## 8.3 Explanation template

```txt
Recommended because:
- It uses the same core pattern: {pattern}.
- It adds one new constraint: {constraint}.
- Your last review in this pattern was rated {rating}.
- It is {difficulty} and should take about {estimatedMinutes} minutes.
```

## 8.4 Similar-problem buckets

```txt
For every problem page, show:

- Prerequisite problem
- Same-pattern problem
- Harder follow-up
- Contrast problem
```

A contrast problem is important. It teaches users not to overfit.

Example:

```txt
Current: Subarray Sum Equals K
Same pattern: Continuous Subarray Sum
Contrast: Two Sum
Reason: Both use hash maps, but one uses prefix sums and one uses direct complements.
```

---

# 9. Readiness score strategy

## 9.1 Overall readiness

```txt
overall_readiness =
  0.25 * topic_coverage
+ 0.20 * review_health
+ 0.20 * unaided_solve_quality
+ 0.15 * solve_speed
+ 0.10 * consistency
+ 0.10 * confidence_calibration
```

## 9.2 Topic readiness

```txt
Topic: Graphs
Coverage: 42/100
Review Health: 55/100
Solve Quality: 38/100
Speed: 44/100
Consistency: 70/100
Confidence Calibration: 50/100
Overall: 48/100
```

Recommendation:

```txt
Graphs are not interview-ready yet.
Recommended actions:
1. Review Number of Islands.
2. Solve Rotting Oranges.
3. Bridge to Course Schedule.
4. Avoid Word Ladder until BFS shortest path is stable.
```

## 9.3 Readiness-driven CTAs

```txt
If coverage is low:
  CTA = Solve new problems.

If review health is low:
  CTA = Clear due reviews.

If solve quality is low:
  CTA = Re-solve without hints.

If speed is low:
  CTA = Timed practice.

If consistency is low:
  CTA = Smaller daily plan.

If confidence calibration is low:
  CTA = Active recall quiz.
```

---

# 10. AI usage strategy

AI should explain and classify. It should not be the only recommender.

## 10.1 Recommended AI roles

```txt
Good AI use:
- Classify problem pattern and subpattern.
- Generate spoiler-free hints.
- Explain why a recommendation was chosen.
- Compare two similar problems.
- Generate active recall questions.
- Summarize user's weak pattern.

Risky AI use:
- Directly choosing all recommendations without deterministic ranking.
- Inventing LeetCode metadata.
- Suggesting premium-only problems without checking access.
- Giving full solutions when the user asked for hints.
```

## 10.2 AI explanation prompt

```txt
You are explaining a coding-practice recommendation.
Do not give code.
Do not reveal the full solution.
Use plain text.

User context:
- Weak patterns: {weakPatterns}
- Recent solved problems: {recentSolved}
- Due reviews: {dueReviews}
- Target sheet: {sheetName}

Recommended problem:
- Title: {title}
- Difficulty: {difficulty}
- Pattern: {pattern}
- Reason signals: {signals}

Write:
1. Why this problem is recommended.
2. What concept it reinforces.
3. One spoiler-free hint.
4. What to do after solving it.
```

## 10.3 AI classification prompt

```txt
Classify this LeetCode problem for a recommendation engine.
Return JSON only.

Problem:
{title}
{description}
{constraints}

Return:
{
  "primaryPattern": "",
  "subpatterns": [],
  "dataStructures": [],
  "prerequisites": [],
  "similarProblemTypes": [],
  "commonMistakes": [],
  "difficultyDrivers": [],
  "spoilerFreeHint": ""
}
```

---

# 11. Database schema idea

## 11.1 Core tables

```sql
CREATE TABLE problems (
  id TEXT PRIMARY KEY,
  leetcode_slug TEXT UNIQUE NOT NULL,
  title TEXT NOT NULL,
  difficulty TEXT NOT NULL,
  estimated_minutes INTEGER,
  is_premium BOOLEAN DEFAULT FALSE
);

CREATE TABLE problem_patterns (
  problem_id TEXT NOT NULL,
  pattern TEXT NOT NULL,
  subpattern TEXT,
  weight REAL DEFAULT 1.0,
  PRIMARY KEY (problem_id, pattern, subpattern)
);

CREATE TABLE sheets (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT
);

CREATE TABLE sheet_problems (
  sheet_id TEXT NOT NULL,
  problem_id TEXT NOT NULL,
  order_index INTEGER NOT NULL,
  section TEXT,
  is_core BOOLEAN DEFAULT TRUE,
  PRIMARY KEY (sheet_id, problem_id)
);

CREATE TABLE user_problem_states (
  user_id TEXT NOT NULL,
  problem_id TEXT NOT NULL,
  status TEXT NOT NULL,
  first_solved_at TIMESTAMP,
  last_attempt_at TIMESTAMP,
  last_reviewed_at TIMESTAMP,
  next_review_at TIMESTAMP,
  review_stage INTEGER DEFAULT 0,
  confidence INTEGER,
  personal_difficulty INTEGER,
  attempts_count INTEGER DEFAULT 0,
  failed_attempts_count INTEGER DEFAULT 0,
  hint_used_count INTEGER DEFAULT 0,
  editorial_used_count INTEGER DEFAULT 0,
  avg_solve_time_minutes REAL,
  review_lapses INTEGER DEFAULT 0,
  skipped_count INTEGER DEFAULT 0,
  PRIMARY KEY (user_id, problem_id)
);

CREATE TABLE recommendation_events (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  problem_id TEXT NOT NULL,
  recommendation_type TEXT NOT NULL,
  score REAL NOT NULL,
  reason_json TEXT NOT NULL,
  shown_at TIMESTAMP NOT NULL,
  clicked_at TIMESTAMP,
  skipped_at TIMESTAMP,
  completed_at TIMESTAMP
);
```

---

# 12. Recommendation service pseudocode

```ts
async function getDailyRecommendations(userId: string): Promise<Recommendation[]> {
  const user = await getUserProfile(userId);
  const states = await getUserProblemStates(userId);
  const problems = await getProblemsForSelectedSheet(user.selectedSheetId);
  const weakPatterns = calculateWeakPatterns(states, problems);

  const dueReviews = generateDueReviewCandidates(user, states, problems, weakPatterns);
  const weakDrills = generateWeakPatternCandidates(user, states, problems, weakPatterns);
  const roadmapNext = generateRoadmapCandidates(user, states, problems);
  const diversity = generateDiversityCandidates(user, states, problems);

  const rankedReviews = rankReviewCandidates(dueReviews);
  const rankedNew = rankNewProblemCandidates([
    ...weakDrills,
    ...roadmapNext,
    ...diversity,
  ]);

  return buildBalancedPlan({
    reviews: rankedReviews,
    newProblems: rankedNew,
    userCapacityMinutes: user.dailyCapacityMinutes ?? 60,
    maxItems: user.maxDailyProblems ?? 5,
  });
}
```

Balanced plan builder:

```ts
function buildBalancedPlan(input: {
  reviews: Recommendation[];
  newProblems: Recommendation[];
  userCapacityMinutes: number;
  maxItems: number;
}) {
  const plan: Recommendation[] = [];

  plan.push(...input.reviews.slice(0, 2));
  plan.push(...input.newProblems.filter(r => r.type === "weak_drill").slice(0, 1));
  plan.push(...input.newProblems.filter(r => r.type === "roadmap_next").slice(0, 1));

  if (plan.length < input.maxItems) {
    plan.push(...input.newProblems.filter(r => r.type === "confidence_booster").slice(0, 1));
  }

  return dedupeAndFitTime(plan, input.userCapacityMinutes, input.maxItems);
}
```

---

# 13. Recommendation explanation examples

## 13.1 Due review explanation

```txt
Review this today because it is overdue by 4 days and your last rating was Hard. It also belongs to Sliding Window, which is currently one of your weakest patterns.
```

## 13.2 Next unsolved explanation

```txt
This is the next unsolved problem in your selected sheet, and it fits your current level. You have solved the prerequisite two-pointer problems, but still need more practice with variable-size windows.
```

## 13.3 Weak drill explanation

```txt
Recommended as a weak-topic drill because your recent Graph BFS problems required hints. This problem reinforces BFS traversal without adding the extra complexity of shortest-path reconstruction.
```

## 13.4 Bridge explanation

```txt
This is a bridge problem before Word Ladder. It teaches BFS level-order expansion on a grid, which is easier than BFS over generated word states.
```

## 13.5 Similar problem explanation

```txt
Recommended because it uses the same prefix-sum idea, but changes the goal from finding a target sum to counting valid subarrays. Focus on what the hashmap stores before coding.
```

---

# 14. MVP recommendation system for CrackedIn.io

## MVP inputs

```txt
- Synced solved/unsolved status.
- Standard sheet order.
- Difficulty.
- Topic/pattern tags.
- Last solved date.
- Manual confidence rating.
- Manual review rating.
```

## MVP outputs

```txt
- Due reviews.
- Next unsolved sheet problem.
- Top 3 weak topics.
- 3 similar problems per problem.
- Basic readiness score.
```

## MVP formulas

Solved problem:

```txt
review_score =
  3 * overdue_days
+ 4 * failed_last_review
+ 2 * low_confidence
+ 2 * weak_pattern_match
+ 1 * solved_with_hint
```

Unsolved problem:

```txt
new_problem_score =
  3 * sheet_priority
+ 4 * weak_pattern_match
+ 3 * difficulty_fit
+ 2 * prerequisite_ready
+ 1 * diversity_bonus
```

Similar problem:

```txt
similar_score =
  5 * same_subpattern
+ 3 * same_template
+ 2 * difficulty_fit
+ 2 * sheet_relevance
- 3 * solved_recently
```

---

# 15. Advanced recommendation system

Add these after MVP:

```txt
1. FSRS or SM-2 scheduler.
2. Personalized difficulty model.
3. Pattern graph with prerequisite edges.
4. Embedding-based problem similarity.
5. AI-generated explanation and hinting.
6. Deadline-aware study planner.
7. Company-targeted recommendation mode.
8. User skip feedback loop.
9. A/B testing for recommendation quality.
10. Readiness score by topic and overall.
```

---

# 16. Success metrics

Track whether recommendations actually help.

```txt
Recommendation metrics:
- Click-through rate.
- Skip rate.
- Completion rate.
- Solve success rate.
- Hint usage after recommendation.
- Time to solve.
- Review pass rate.
- Repeated failure rate.
- Time-to-master per topic.
- Retention after 7/14/30 days.

Business/product metrics:
- Daily active users.
- Weekly retained users.
- Average problems reviewed per week.
- Average new problems solved per week.
- Streak recovery after inactivity.
- Interview readiness improvement over time.
```

Use recommendation events to improve ranking:

```txt
If user repeatedly skips a type:
  reduce that recommendation type temporarily.

If user clicks but fails often:
  lower difficulty or add bridge problems.

If user completes easily:
  increase difficulty or interval.

If user solves weak-topic recommendations successfully:
  reduce weakness score and move to next pattern.
```

---

# 17. Guardrails

```txt
- Do not recommend only hard problems because the user is advanced.
- Do not recommend only overdue reviews; mix in progress.
- Do not recommend a hard target before prerequisites are solved.
- Do not use broad tags as the only similarity signal.
- Do not let AI invent metadata.
- Do not hide why a problem was recommended.
- Do not over-penalize users after inactivity.
- Do not treat solved once as mastered.
- Do not treat solved with editorial as equal to unaided solve.
```

---

# 18. Recommended CrackedIn.io modules

## 18.1 Today page

```txt
- Due Reviews
- Next Best Problem
- Weak Topic Drill
- Continue Sheet
- Quick Win
```

## 18.2 Problem page

```txt
- Pattern classification
- Similar problems
- Easier prerequisite
- Harder follow-up
- Contrast problem
- Spoiler-free hint
- Review history
```

## 18.3 Analytics page

```txt
- Overall readiness score
- Topic readiness
- Weakest patterns
- Review health
- Sheet completion
- Solve speed trend
- Confidence calibration
```

## 18.4 Planner page

```txt
- Choose target sheet
- Set interview date
- Set daily time
- Select target topics/company
- Generate weekly plan
```

---

# 19. Best combined strategy for CrackedIn.io

Recommended final approach:

```txt
1. Use the standard sheet for base order.
2. Use SRS for solved problem review.
3. Use weak-pattern scoring for personalization.
4. Use pattern similarity for related practice.
5. Use prerequisite graph for bridge recommendations.
6. Use readiness score to explain progress.
7. Use AI only for classification, hints, and explanations.
```

The core product promise can be:

```txt
CrackedIn.io tells users exactly what to solve next, what to review before they forget it, and which weak pattern each recommendation is fixing.
```

---

# 20. Source notes

These public sources were used to understand platform behavior and recommendation patterns:

1. LeetSRS official site: https://leetsrs.com/
2. LeetSRS GitHub repository: https://github.com/mattcdrake/LeetSRS
3. LeetSRS Chrome Web Store listing: https://chromewebstore.google.com/detail/leetsrs/odgfcigkohoimpeeooifjdglncggkgko
4. HashTry official site: https://hashtry.io/
5. HashTry Chrome Web Store listing: https://chromewebstore.google.com/detail/hashtry/oigaclkkebclkjkmhdkemppdefhenknp
6. LeetReviewer official site: https://leetreviewer.com/
7. LeetRecall Chrome Web Store listing: https://chromewebstore.google.com/detail/leetrecall/lphmfhcelhlgpmijomapaodlmebogmip
8. AlgoRecall Chrome Web Store listing: https://chromewebstore.google.com/detail/algorecall-%E2%80%94-leetcode-spa/hjfjdhpaddkdnjndeaalchgjnimioaln
9. SpaceDSA Chrome Web Store listing: https://chromewebstore.google.com/detail/spacedsa-leetcode-study-p/phdohlijhchimahmdicclcimgdlcegdd
10. DSA Spaced Repetition Chrome Web Store listing: https://chromewebstore.google.com/detail/dsa-spaced-repetition/lcmbmephjkhhdmicgnggilhinhlpbklc
11. leetcode-review GitHub repository: https://github.com/kunleihe/leetcode-review
12. NeetCode 150 page: https://neetcode.io/practice/practice/neetcode150
13. AlgoMonster landing/path pages: https://algo.monster/landing and https://algo.monster/path
14. Striver SDE Sheet / TakeUForward: https://takeuforward.org/dsa/strivers-sde-sheet-top-coding-interview-problems
15. Grind 75 / Tech Interview Handbook: https://www.techinterviewhandbook.org/grind75/
16. LeetDaily: https://leetdaily.masst.dev/
17. Codewars rank documentation: https://docs.codewars.com/gamification/ranks/
18. Codewars honor documentation: https://docs.codewars.com/gamification/honor/
19. Exercism mentoring docs: https://exercism.org/docs/mentoring
20. TS-FSRS documentation: https://open-spaced-repetition.github.io/ts-fsrs/
