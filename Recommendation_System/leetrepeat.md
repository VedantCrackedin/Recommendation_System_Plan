# CrackedIn.io: Competitor Recommendation Strategy Directory

**Purpose:** This document lists relevant coding-interview, DSA-practice, LeetCode-tracking, spaced-repetition, roadmap, and learning-feedback platforms. It summarizes how each platform appears to guide users toward reviews, next problems, practice lists, weak areas, or learning actions.

**Important note:** This is not a product comparison. It is a competitor/reference directory for understanding public recommendation and learning-flow patterns. The platform LeetRepeat is intentionally excluded.

---

## Executive Summary

The competitor landscape around DSA practice and interview preparation shows several recurring recommendation strategies:

1. **Spaced-repetition review queues**  
   Platforms such as LeetSRS, LeetRecall, AlgoRecall, SpaceDSA, LeetCycle, LeetSpace, and SpacedSmart focus on bringing solved problems back at the right time so users do not forget patterns. Their recommendation flow is usually: solve a problem, rate confidence or performance, schedule a future review, then surface due or overdue problems.

2. **Readiness scoring and weak-area analytics**  
   HashTry is the clearest example of turning user practice behavior into a readiness score. It uses signals such as solve speed, coverage, quality, consistency, and self-assessment. This strategy helps users understand whether they are actually prepared instead of only seeing a solved-count metric.

3. **Roadmap-based sequencing**  
   NeetCode, Striver / TakeUForward, Grind 75, Blind 75 trackers, LeetDaily, and LeetTracker guide users through curated problem lists. Their recommendation style is usually based on a structured sheet, topic order, difficulty progression, and “next unsolved” navigation.

4. **Pattern-first learning**  
   AlgoMonster, NeetCode, and similar roadmap platforms organize learning around reusable problem-solving patterns. Their core recommendation idea is that users should master patterns such as sliding window, binary search, graph traversal, dynamic programming, monotonic stack, and backtracking rather than randomly solving problems.

5. **Session and habit design**  
   Platforms like LeetDaily, SpaceDSA, Codewars, and LeetReviewer use daily challenges, streaks, review cards, progress bars, ranks, honor points, and session workflows to keep users engaged.

6. **Feedback-oriented practice**  
   Exercism is not DSA-specific, but it is relevant because it uses practice plus mentoring and feedback. Its recommendation inspiration is less about “next LeetCode problem” and more about improving the learner through guided review and feedback loops.

For CrackedIn.io, the most useful strategic takeaway is to combine three recommendation layers:

- **Memory layer:** what solved problems should the user review now?
- **Roadmap layer:** what unsolved problem should the user solve next from a sheet or goal path?
- **Diagnostic layer:** what weak pattern or topic should the user repair before moving forward?

---

# Competitor Directory

## 1. LeetSRS

**Website / Source:**  
- https://leetsrs.com/  
- https://github.com/mattcdrake/LeetSRS  
- https://chromewebstore.google.com/detail/leetsrs/odgfcigkohoimpeeooifjdglncggkgko

**Summary:**  
LeetSRS is a Chrome extension that adds spaced repetition directly to LeetCode practice. It is designed to help users review solved problems before they forget the underlying pattern.

**Recommendation strategy:**  
LeetSRS recommends problems through a review queue. Users add LeetCode problems as cards, rate their recall/performance, and the system schedules future reviews. Public materials mention TS-FSRS, a spaced-repetition scheduler, as the algorithmic basis.

**Key user flow:**

1. User solves or adds a LeetCode problem.
2. Problem becomes a review card.
3. User rates the card after review.
4. Scheduler calculates the next review date.
5. Due cards are surfaced in the extension.

**Notable mechanics:**

- Embedded LeetCode workflow.
- Review queue based on spaced repetition.
- Rating-driven scheduling.
- Local-first behavior and optional sync features.
- Streaks and stats to reinforce consistency.

**Useful strategic pattern:**  
For any DSA platform, solved problems should not disappear after completion. They should become memory objects with review due dates, confidence ratings, and recall history.

---

## 2. HashTry

**Website / Source:**  
- https://hashtry.io/  
- https://chromewebstore.google.com/detail/hashtry/oigaclkkebclkjkmhdkemppdefhenknp

**Summary:**  
HashTry is a LeetCode analytics and spaced-repetition extension focused on helping users understand interview readiness. It tracks LeetCode attempts, reviews, topic coverage, quality, consistency, and self-assessment.

**Recommendation strategy:**  
HashTry uses performance analytics to guide what users should work on next. Its public website describes an Interview Readiness Score from 0 to 100 based on multiple signals, not just solved count.

**Key user flow:**

1. User practices on LeetCode.
2. HashTry tracks attempts and outcomes.
3. User receives quality or readiness signals.
4. Weak topics and stalled progress become visible.
5. Review and practice actions are guided by readiness gaps.

**Notable mechanics:**

- Interview Readiness Score.
- Topic coverage tracking.
- Solve-speed trend analysis.
- Solution quality grading.
- Consistency signal.
- Self-assessment accuracy.
- Weak-area detection.

**Useful strategic pattern:**  
A strong recommendation system should not only say “solve this next.” It should explain readiness gaps: speed, coverage, quality, consistency, and confidence.

---

## 3. LeetReviewer

**Website / Source:**  
- https://leetreviewer.com/

**Summary:**  
LeetReviewer is a smart review system for LeetCode problems. It focuses on review cards, recall sessions, and spaced repetition across mobile and web.

**Recommendation strategy:**  
LeetReviewer uses review-card workflows. The user reviews coding problems through swipeable cards and recall sessions. The strategy is to make the learner actively remember the problem-solving approach instead of passively reading old solutions.

**Key user flow:**

1. User adds or imports problems.
2. Problems become review cards.
3. User enters recall sessions.
4. Cards are surfaced based on review timing.
5. Progress is tracked through repeated recall.

**Notable mechanics:**

- Swipeable review cards.
- Recall-first sessions.
- Spaced repetition.
- Mobile and web review access.
- Review-oriented problem tracking.

**Useful strategic pattern:**  
Recommendation can be framed as a recall exercise, not just a task list. The platform can recommend a problem and ask: “Can you recall the pattern, invariant, edge cases, and complexity?”

---

## 4. LeetRecall

**Website / Source:**  
- https://chromewebstore.google.com/detail/leetrecall/lphmfhcelhlgpmijomapaodlmebogmip

**Summary:**  
LeetRecall is a Chrome extension for hands-free spaced repetition on LeetCode. It aims to help users remember problems they have already solved.

**Recommendation strategy:**  
LeetRecall recommends reviews automatically after detecting or tracking solved problems. It focuses on bringing problems back into the user’s workflow when they are due.

**Key user flow:**

1. User solves LeetCode problems.
2. Extension tracks solved items.
3. Review dates are scheduled.
4. Due problems appear in the extension/dashboard.
5. User reviews and updates proficiency.

**Notable mechanics:**

- Automatic submission or progress tracking.
- Due-today review queue.
- Proficiency-style signals.
- Offline/local-first positioning.
- Review reminders.

**Useful strategic pattern:**  
Automation matters. Users do not want to manually maintain spreadsheets. A platform should infer as much as possible and ask users only for subjective input such as confidence or difficulty felt.

---

## 5. AlgoRecall

**Website / Source:**  
- https://chromewebstore.google.com/detail/algorecall-%E2%80%94-leetcode-spa/hjfjdhpaddkdnjndeaalchgjnimioaln  
- https://github.com/Ashtam01/AlgoRecall

**Summary:**  
AlgoRecall is a browser extension for spaced revision of algorithmic problems. Public materials mention support for LeetCode and other competitive-programming sites such as Codeforces, AtCoder, and CodeChef.

**Recommendation strategy:**  
AlgoRecall recommends problems based on post-submit ratings and review schedules. After solving, the user rates the experience, and the extension schedules the next review.

**Key user flow:**

1. Solve and submit a problem.
2. Rate the problem using labels such as Again, Hard, Good, or Easy.
3. Store problem metadata, solution, tags, review history, and next review date.
4. Review when the dashboard says the problem is due.

**Notable mechanics:**

- Post-submit rating.
- Spaced-repetition scheduling.
- Multi-platform algorithm-practice support.
- Local-first behavior.
- Automatic saving of accepted solution metadata when available.

**Useful strategic pattern:**  
Post-solve rating is one of the most important recommendation signals. It converts a raw solve into a memory-strength estimate.

---

## 6. SpaceDSA

**Website / Source:**  
- https://chromewebstore.google.com/detail/spacedsa-leetcode-study-p/phdohlijhchimahmdicclcimgdlcegdd

**Summary:**  
SpaceDSA is a LeetCode study planner based on spaced repetition. It emphasizes that users can work with any problem list rather than being locked into one curated sheet.

**Recommendation strategy:**  
SpaceDSA recommends reviews from the user’s own chosen problem list. The core strategy is flexible-list scheduling: users pick or follow any list, and the system handles review timing.

**Key user flow:**

1. User chooses problems from any list.
2. Solved problems are scheduled for review.
3. The system surfaces review tasks over time.
4. User filters or works through study sessions.

**Notable mechanics:**

- Not tied to a single curated list.
- Spaced-repetition-based routine.
- Review filtering by type or difficulty.
- Study-session framing.

**Useful strategic pattern:**  
A recommendation engine should work with user-owned lists: NeetCode, Striver, Blind 75, company lists, custom sheets, or imported plans.

---

## 7. LeetCycle

**Website / Source:**  
- https://www.leetcycle.com/

**Summary:**  
LeetCycle is a practice OS for LeetCode spaced repetition. It emphasizes redoing problems and rating confidence after each review.

**Recommendation strategy:**  
LeetCycle recommends review tasks based on confidence and repetition. Every review means the user actually redoes the problem, then logs how it felt.

**Key user flow:**

1. User adds a LeetCode URL or problem name.
2. The problem enters a review loop.
3. User re-solves the problem during review.
4. User rates confidence.
5. The next review is scheduled.

**Notable mechanics:**

- Confidence-based review loop.
- Review means active re-solving.
- Streaks and progress tracking.
- Lightweight practice operating system.

**Useful strategic pattern:**  
A high-quality DSA review recommendation should require active solving, not just marking a task as complete.

---

## 8. LeetSpace

**Website / Source:**  
- https://www.leetspace.dev/

**Summary:**  
LeetSpace helps users master algorithmic problem solving through spaced repetition, progress tracking, and revision management.

**Recommendation strategy:**  
LeetSpace appears to recommend revisions using an intelligent scheduling system for LeetCode practice. The focus is revision optimization rather than broad content discovery.

**Key user flow:**

1. Track LeetCode progress.
2. Manage revision items.
3. Use scheduling to determine what to revisit.
4. Continue preparation through repeated review.

**Notable mechanics:**

- Progress tracking.
- Revision management.
- Intelligent scheduling language.
- Coding-interview preparation focus.

**Useful strategic pattern:**  
Revision should be treated as a first-class object in the product, not an afterthought below the solved-problem list.

---

## 9. SpacedSmart

**Website / Source:**  
- https://www.spacedsmart.com/

**Summary:**  
SpacedSmart positions itself as a science-backed LeetCode spaced-repetition platform. It claims to model a learner’s forgetting curve and predict when review is needed.

**Recommendation strategy:**  
SpacedSmart recommends review timing based on a personalized forgetting-curve idea. It mentions adjusting reviews sooner if the user struggled and later if the user performed well.

**Key user flow:**

1. User practices LeetCode problems.
2. System observes or receives performance signals.
3. Forgetting risk is estimated.
4. Problems are recommended for review at predicted useful times.

**Notable mechanics:**

- Forgetting-curve positioning.
- Personalized review prediction.
- Pattern-oriented DSA preparation.
- Large LeetCode problem coverage.

**Useful strategic pattern:**  
The recommendation message can be framed around memory risk: “Review now because you are likely to forget this pattern soon.”

---

## 10. SpacedLeet

**Website / Source:**  
- https://spacedleet.vercel.app/

**Summary:**  
SpacedLeet is a spaced-repetition tool for LeetCode interview preparation.

**Recommendation strategy:**  
SpacedLeet recommends review problems at increasing intervals and focuses on keeping previously solved LeetCode problems active in memory.

**Key user flow:**

1. User tracks solved LeetCode problems.
2. Problems are scheduled for future reviews.
3. Review queue tells the user what to revisit.

**Notable mechanics:**

- Spaced-repetition scheduling.
- LeetCode-focused prep.
- Simple review-loop framing.

**Useful strategic pattern:**  
Even simple interval-based scheduling creates a strong habit loop when the product clearly shows “due today.”

---

## 11. NeetCode

**Website / Source:**  
- https://neetcode.io/practice/practice/neetcode150  
- https://neetcode.io/practice

**Summary:**  
NeetCode is a structured coding-interview practice platform known for curated lists such as Blind 75 and NeetCode 150, topic-based roadmaps, and video explanations.

**Recommendation strategy:**  
NeetCode recommends practice through curated topic roadmaps. The NeetCode 150 list is organized into topic sections such as Arrays & Hashing, Two Pointers, Sliding Window, Stack, Binary Search, Linked List, Trees, Heaps, Backtracking, Graphs, Dynamic Programming, Greedy, Intervals, Math, and Bit Manipulation.

**Key user flow:**

1. User selects a curated roadmap.
2. Problems are grouped by topic and difficulty.
3. User works through topic sections.
4. Progress is tracked by solved count per list/topic.
5. Explanations and videos support learning.

**Notable mechanics:**

- Curated problem lists.
- Topic grouping.
- Difficulty distribution.
- Progress tracking.
- Free explanations and video solutions.
- Core-skills sections for algorithms and data structures.

**Useful strategic pattern:**  
A recommendation engine should know not only whether a problem is solved, but where it sits in a topic roadmap and what conceptual block it belongs to.

---

## 12. AlgoMonster

**Website / Source:**  
- https://algo.monster/dashboard  
- https://algo.monster/landing?t=1

**Summary:**  
AlgoMonster is a structured coding-interview preparation platform built around algorithmic patterns, templates, guided lessons, and company-focused preparation.

**Recommendation strategy:**  
AlgoMonster recommends learning by pattern. Instead of browsing thousands of questions randomly, users are guided through reusable patterns, lessons, templates, and progressive problem-solving modules.

**Key user flow:**

1. User follows a structured curriculum.
2. Patterns are introduced as reusable building blocks.
3. Problems reinforce those patterns.
4. AI assistance or progressive unlocking helps the user reach a solution.
5. Company question banks and speedrun-style practice support targeted prep.

**Notable mechanics:**

- Pattern-first curriculum.
- Algorithm templates.
- Progressive solution unlocking.
- AI-enabled hints and guidance.
- Company-specific question bank.
- Interview-ready study timeline.

**Useful strategic pattern:**  
Recommendations should be based on patterns and subpatterns, not just broad tags. “Sliding window with variable-size constraint” is more actionable than “Array” or “String.”

---

## 13. Striver / TakeUForward SDE Sheet

**Website / Source:**  
- https://takeuforward.org/dsa/strivers-sde-sheet-top-coding-interview-problems

**Summary:**  
Striver’s SDE Sheet is a curated list of top coding-interview problems across DSA topics. The sheet is organized by topic sections and includes overall progress, revision, random problem, difficulty, and topic grouping.

**Recommendation strategy:**  
The recommendation strategy is sheet-driven progression. Users move through handpicked interview problems organized by topic and difficulty, with revision and random-problem options.

**Key user flow:**

1. User opens the SDE Sheet.
2. Problems are grouped by topic sections.
3. User solves problems and tracks progress.
4. Revision and random modes help revisit or vary practice.

**Notable mechanics:**

- Curated interview problem sheet.
- Topic-based sections.
- Difficulty labels.
- Progress tracking.
- Revision mode.
- Random problem option.

**Useful strategic pattern:**  
A good recommendation system can combine strict sheet order with adaptive detours: continue the roadmap, but recommend revision or prerequisite problems when the user is weak.

---

## 14. Grind 75 / Tech Interview Handbook

**Website / Source:**  
- https://www.techinterviewhandbook.org/grind75/  
- https://github.com/yangshun/tech-interview-handbook

**Summary:**  
Grind 75 is a curated coding-interview question list by Yangshun Tay, positioned as an improved and customizable evolution of Blind 75.

**Recommendation strategy:**  
Grind 75 recommends problems through customization. Users can adjust available time, difficulty, topics, and other constraints to generate a suitable study plan.

**Key user flow:**

1. User chooses interview-prep constraints such as time and topics.
2. The platform generates or displays a plan.
3. Problems are ordered across weeks and difficulty levels.
4. User follows the sequence and tracks completion.

**Notable mechanics:**

- Curated core list.
- Customizable plan generation.
- Weekly sequencing.
- Topic and difficulty controls.
- Estimated time per problem.

**Useful strategic pattern:**  
Recommendations should adapt to user constraints: interview date, available hours per week, preferred topics, and difficulty tolerance.

---

## 15. LeetDaily

**Website / Source:**  
- https://leetdaily.masst.dev/  
- https://leetdaily.masst.dev/blog/blind-75-vs-neetcode-150-vs-leetcode-75

**Summary:**  
LeetDaily is a LeetCode interview-prep and tracker extension with daily challenge support and curated sheet tracking.

**Recommendation strategy:**  
LeetDaily recommends through daily prompts and next-unsolved navigation. It tracks multiple curated sheets and helps users continue from where they left off.

**Key user flow:**

1. User sees today’s LeetCode daily challenge.
2. User tracks curated sheets such as Blind 75, NeetCode 150, LeetCode 75, Namaste DSA, Fraz DSA, and Striver SDE Sheet.
3. Progress bars show completion.
4. Sequential next-unsolved navigation points users forward.
5. Focus streaks encourage consistency.

**Notable mechanics:**

- Daily challenge card.
- Multi-sheet tracking.
- Progress bars.
- Sequential next-unsolved recommendation.
- Focus streaks.
- Difficulty, acceptance, topic, and company tags.

**Useful strategic pattern:**  
“Next unsolved” is a simple but powerful recommendation. It becomes stronger when combined with multiple list memberships and streak-building.

---

## 16. LeetTracker

**Website / Source:**  
- https://www.leettracker.com/problems

**Summary:**  
LeetTracker is a free LeetCode progress tracker that organizes problems by topic, difficulty, and curated lists such as NeetCode 150, Grind 75, and Blind 75.

**Recommendation strategy:**  
LeetTracker recommends discovery through filters and curated-list organization. The user can browse problems by topic, difficulty, and sheet membership, then track completion.

**Key user flow:**

1. User opens a curated problem directory.
2. Problems are grouped by lists, topics, and difficulty.
3. User filters and marks progress.
4. The tracker visualizes strengths and streaks.

**Notable mechanics:**

- Curated problem directory.
- Topic and difficulty filters.
- Progress tracking.
- Streak support.
- Multi-list coverage.

**Useful strategic pattern:**  
Good recommendations need a clean problem catalog with metadata: topic, difficulty, list membership, and user completion state.

---

## 17. FleetCode

**Website / Source:**  
- https://www.talentd.in/fleetcode/sheets

**Summary:**  
FleetCode provides DSA sheets such as Blind 75, NeetCode 150, NeetCode 250, Striver SDE Sheet, Grind 75, and LeetCode 75.

**Recommendation strategy:**  
FleetCode recommends practice through curated sheet selection. Users can choose the sheet that matches their preparation level and follow its problem set.

**Key user flow:**

1. User selects a curated sheet.
2. The platform presents problem counts and sheet information.
3. User starts a selected path.
4. Progress is tracked against that sheet.

**Notable mechanics:**

- Multiple official and community sheets.
- Problem counts per sheet.
- Sheet-level positioning.
- User selection of preparation path.

**Useful strategic pattern:**  
A platform can recommend the right sheet before recommending the right problem. Beginners, intermediates, and interview-ready users may need different roadmaps.

---

## 18. Codewars

**Website / Source:**  
- https://docs.codewars.com/gamification/  
- https://docs.codewars.com/gamification/ranks/  
- https://docs.codewars.com/gamification/honor/

**Summary:**  
Codewars is a programming-practice platform based on kata, ranks, honor, community solutions, and gamified progression.

**Recommendation strategy:**  
Codewars guides users through difficulty-ranked kata and gamified progression. Users level up by solving kata, and the platform uses ranks and honor to represent skill and community contribution.

**Key user flow:**

1. User solves kata.
2. Solves contribute to rank progress.
3. Harder kata provide more rank progress.
4. Community actions contribute to honor.
5. Progression creates motivation to continue.

**Notable mechanics:**

- Kyu/dan rank system.
- Honor points.
- Kata difficulty progression.
- Community voting and contribution signals.
- Skill progression through completion.

**Useful strategic pattern:**  
Gamification can turn recommendations into progression goals: “solve two graph BFS problems to reach Graph Level 3” or “complete one hard review to advance your DP mastery.”

---

## 19. Exercism

**Website / Source:**  
- https://exercism.org/docs  
- https://exercism.org/docs/mentoring

**Summary:**  
Exercism is a programming-practice and mentorship platform. It supports many programming languages and emphasizes practice, feedback, and mentoring.

**Recommendation strategy:**  
Exercism recommends learning through practice tracks, exercises, mentoring, and feedback. Its strongest lesson for DSA platforms is the feedback loop: users improve when their work is reviewed and they receive guidance, not just when they solve more problems.

**Key user flow:**

1. User chooses a language track.
2. User solves exercises.
3. User may receive automated or mentor feedback.
4. Feedback helps refine understanding and code quality.
5. User progresses through more exercises.

**Notable mechanics:**

- Practice tracks.
- Exercise progression.
- Mentoring workflow.
- Feedback culture.
- Community contribution.

**Useful strategic pattern:**  
A coding platform can recommend not only problems but feedback actions: “review your previous solution,” “compare approaches,” “ask for feedback,” or “refactor for clarity.”

---

## 20. StudyCards AI

**Website / Source:**  
- https://studycardsai.com/blog/spaced-repetition-for-leetcode

**Summary:**  
StudyCards AI discusses using spaced repetition for LeetCode by turning solved problems into review cards. It frames the method as remembering patterns rather than memorizing full code.

**Recommendation strategy:**  
The recommended learning flow is card-based review. A problem becomes a prompt for recalling the high-level approach, pattern, and solution reasoning.

**Key user flow:**

1. User solves a LeetCode problem.
2. A review card is created.
3. The front prompts recall of the problem or pattern.
4. The back contains the approach and link to solution.
5. Reviews occur at increasing intervals.

**Notable mechanics:**

- Problem-to-card conversion.
- Pattern recall rather than code memorization.
- Increasing review intervals.
- Lightweight Anki-style workflow.

**Useful strategic pattern:**  
Recommendation content should ask users to remember the problem-solving idea, not the exact code.

---

# Recommendation Strategy Themes to Extract

## Theme 1: Review due/overdue solved problems

Observed in: LeetSRS, LeetRecall, AlgoRecall, SpaceDSA, LeetCycle, LeetSpace, SpacedSmart, SpacedLeet.

**Mechanism:**  
Solved problems are converted into review items. Each item has a next review date. The system surfaces due or overdue problems.

**Signals used or implied:**

- Last solved date.
- Last reviewed date.
- User confidence.
- User rating.
- Pass/fail outcome.
- Difficulty felt by the user.
- Forgetting risk.

---

## Theme 2: Ask for user rating after solving

Observed in: LeetSRS, AlgoRecall, LeetCycle, HashTry-like self-assessment workflows.

**Mechanism:**  
After a solve or review, the user rates performance using labels such as Again, Hard, Good, Easy, or confidence levels.

**Why it matters:**  
The rating turns raw completion into a mastery signal. A solved problem can still be weak if the user needed hints, took too long, or could not explain the pattern.

---

## Theme 3: Use readiness scoring

Observed in: HashTry.

**Mechanism:**  
Aggregate several practice signals into a single readiness score.

**Common score inputs:**

- Topic coverage.
- Solve speed.
- Solution quality.
- Consistency.
- Self-assessment accuracy.
- Review health.

---

## Theme 4: Use curated roadmaps

Observed in: NeetCode, Striver / TakeUForward, Grind 75, LeetDaily, LeetTracker, FleetCode.

**Mechanism:**  
Problems are grouped into known interview-prep lists. The user is guided through topic sections and next-unsolved items.

**Common roadmap signals:**

- Sheet membership.
- Topic section.
- Difficulty.
- Problem order.
- Completion state.
- User’s current progress.

---

## Theme 5: Recommend by pattern, not only topic

Observed in: AlgoMonster, NeetCode, StudyCards AI, some spaced-repetition tools.

**Mechanism:**  
The platform teaches or reviews reusable patterns. Problems are recommended because they reinforce a pattern or subpattern.

**Useful pattern examples:**

- Fixed sliding window.
- Variable sliding window.
- Prefix sum plus hashmap.
- Monotonic stack.
- Binary search on answer.
- Tree DFS recursion.
- BFS shortest path.
- Topological sort.
- 1-D dynamic programming.
- 2-D dynamic programming.
- Backtracking with pruning.

---

## Theme 6: Provide daily or session-based practice

Observed in: LeetDaily, SpaceDSA, LeetReviewer, Codewars.

**Mechanism:**  
Instead of presenting a huge problem list, the product recommends a small session: today’s challenge, due reviews, next unsolved, or a filtered practice block.

**Session examples:**

- Today’s daily challenge.
- Due reviews.
- Topic-specific session.
- Difficulty-specific session.
- Weak-area session.
- Random revision.
- Timed practice.

---

## Theme 7: Add gamified progression

Observed in: Codewars, LeetSRS streaks/stats, LeetDaily focus streaks, LeetTracker streaks.

**Mechanism:**  
The platform motivates repeated behavior through ranks, progress bars, streaks, badges, honor, or mastery levels.

**Useful gamification objects:**

- Topic XP.
- Pattern level.
- Review streak.
- Roadmap completion percentage.
- Mastery badges.
- Interview-readiness milestones.

---

## Theme 8: Recommend feedback actions, not just problems

Observed in: Exercism, AlgoMonster AI hints, LeetReviewer recall sessions.

**Mechanism:**  
The product guides the learner to reflect, get hints, review feedback, or improve a solution.

**Action examples:**

- Re-explain your approach.
- Compare with an optimal solution.
- Review missed edge cases.
- Redo without hints.
- Refactor code.
- Ask for feedback.

---

# Source Index

- LeetSRS: https://leetsrs.com/
- LeetSRS GitHub: https://github.com/mattcdrake/LeetSRS
- LeetSRS Chrome Web Store: https://chromewebstore.google.com/detail/leetsrs/odgfcigkohoimpeeooifjdglncggkgko
- HashTry: https://hashtry.io/
- HashTry Chrome Web Store: https://chromewebstore.google.com/detail/hashtry/oigaclkkebclkjkmhdkemppdefhenknp
- LeetReviewer: https://leetreviewer.com/
- LeetRecall Chrome Web Store: https://chromewebstore.google.com/detail/leetrecall/lphmfhcelhlgpmijomapaodlmebogmip
- AlgoRecall Chrome Web Store: https://chromewebstore.google.com/detail/algorecall-%E2%80%94-leetcode-spa/hjfjdhpaddkdnjndeaalchgjnimioaln
- AlgoRecall GitHub: https://github.com/Ashtam01/AlgoRecall
- SpaceDSA Chrome Web Store: https://chromewebstore.google.com/detail/spacedsa-leetcode-study-p/phdohlijhchimahmdicclcimgdlcegdd
- LeetCycle: https://www.leetcycle.com/
- LeetSpace: https://www.leetspace.dev/
- SpacedSmart: https://www.spacedsmart.com/
- SpacedLeet: https://spacedleet.vercel.app/
- NeetCode 150: https://neetcode.io/practice/practice/neetcode150
- NeetCode Practice: https://neetcode.io/practice
- AlgoMonster Dashboard: https://algo.monster/dashboard
- AlgoMonster Landing: https://algo.monster/landing?t=1
- Striver SDE Sheet / TakeUForward: https://takeuforward.org/dsa/strivers-sde-sheet-top-coding-interview-problems
- Grind 75 / Tech Interview Handbook: https://www.techinterviewhandbook.org/grind75/
- Tech Interview Handbook GitHub: https://github.com/yangshun/tech-interview-handbook
- LeetDaily: https://leetdaily.masst.dev/
- LeetDaily sheet article: https://leetdaily.masst.dev/blog/blind-75-vs-neetcode-150-vs-leetcode-75
- LeetTracker: https://www.leettracker.com/problems
- FleetCode DSA Sheets: https://www.talentd.in/fleetcode/sheets
- Codewars Gamification: https://docs.codewars.com/gamification/
- Codewars Ranks: https://docs.codewars.com/gamification/ranks/
- Codewars Honor: https://docs.codewars.com/gamification/honor/
- Exercism Docs: https://exercism.org/docs
- Exercism Mentoring Docs: https://exercism.org/docs/mentoring
- StudyCards AI LeetCode SRS article: https://studycardsai.com/blog/spaced-repetition-for-leetcode




LeetSRS — 269 Chrome users.
AlgoRecall — 123 Chrome users.
LeetRecall — 54 Chrome users.
SpaceDSA — 20 Chrome users.
HashTry — 12 Chrome users.


