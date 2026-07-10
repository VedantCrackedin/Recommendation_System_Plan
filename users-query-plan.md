# All User Queries Asked Till Now

This document contains the complete consolidated list of user queries collected so far.

## Query List

1. In one sentence, what is a hash map?

2. hi

3. hi how are you doing?

4. How do Google and Meta coding interviews differ?

5. Why do I keep getting stuck on medium problems, and how do I break through?

6. How to write the codes

7. Where I need to start first ?

8. How is CrackedIn different from just using ChatGPT for interview prep?

9. How do I sync my LeetCode profile with CrackedIn, and what do I get?

10. Show me what Google actually asks in coding interviews

11. for google's stepinternship

12. class Solution {
   public:
       int countDays(vector<int>& weights,int Cap,int n) {
           int days = 1;
           int load = 0;
           for(int i=0;i<n;i++) {
               if(load>Cap) {
                   days++;
                   load = load + weights[i];
               }
               else {
                   load+=weights[i];
               }
           }
           return days;
       }
   
   
       int shipWithinDays(vector<int>& weights, int days) {
           int n = weights.size();
           
           //finding the maximum element and also the total sum of weights
           int maxi = 0;
           int tot = 0;
           for(int i=0;i<n;i++){ 
               tot+=weights[i];
               maxi = max(maxi,weights[i]);
           }
    
   //starting the BINARY SEARCH:-
           int st = maxi;
           int end = tot;
           while(st<=end) {
               int mid = (st+end)/2;
               if(countDays(weights,mid,n)<=days) {
                   end = mid-1;
               }
               else {
                   st = mid+1;
               }
           }
           return st;
       }
   };
   whats wrong here for code of 101

13. ggoogles step internship

14. | Topic                                      | LeetCode #   | Difficulty      |
   | ------------------------------------------ | ------------ | --------------- |
   | Assign Cookies                             | LeetCode 455 | Easy            |
   | Lemonade Change                            | LeetCode 860 | Easy            |
   | Non-overlapping Intervals                  | LeetCode 435 | Medium          |
   | Merge Intervals                            | LeetCode 56  | Medium          |
   | Insert Interval                            | LeetCode 57  | Medium          |
   | Partition Labels                           | LeetCode 763 | Medium          |
   | Jump Game                                  | LeetCode 55  | Medium          |
   | Jump Game II                               | LeetCode 45  | Medium          |
   | Gas Station                                | LeetCode 134 | Medium          |
   | Queue Reconstruction by Height             | LeetCode 406 | Medium          |
   | Maximum Length of Pair Chain               | LeetCode 646 | Medium          |
   | Minimum Number of Arrows to Burst Balloons | LeetCode 452 | Medium          |
   | Candy                                      | LeetCode 135 | Hard-ish Medium |
   | Task Scheduler                             | LeetCode 621 | Medium          |
   | Reorganize String                          | LeetCode 767 | Medium          |

15. Analyze my weak areas based on my LeetCode profile

16. hi

17. a python

18. factorical of n number

19. Analyze my weak areas based on my LeetCode profile

20. job is to help me in dsa patterns build my logic strong and crack the OAS

21. help with problem 3952,
   can this be simplified, cant get intutive with this solution
   class Solution {
   public:
       long long maxTotal(vector<int>& nums, string s) {
           long long ans=0;
           int i=0;
   // simply taking sum of all nums with token=1
            while(i<nums.size() && s[i]=='1') {
                   ans+=nums[i];
                i++;
            }
           for(i;i<nums.size();i++){
               if(s[i]=='1'){
               long long csm=0;
   //csm = sum of all elements in the segment include preceding 0.
               int mn=INT_MAX;
                   csm+=nums[i-1];
                   mn=nums[i-1];
                   while(i<nums.size() && s[i]=='1'){
                       csm+=nums[i];
                       mn=min(mn,nums[i]);
                       i++;
                   }
                   ans+=csm-mn;
               }
           }
           return ans;
       }
   };
   
   //taking count of all 1s(X) and then sorting and taking the sum of those biggest X no.s
   // BELOW IS WRONG(FULL GREEDY SOLUTION(BF)):-
   // class Solution {
   // public:
   //     long long maxTotal(vector<int>& nums, string s) {
   //         int n = s.length();
   //         int cnt = 0;
   //         for(int i=0;i<n;i++) {
   //             if(s[i]=='1') {
   //                 cnt++;
   //             }
   //         }
   
   //     // now we need the max "cnt" of elements(zyada se zyda)
   //         sort(nums.begin(),nums.end());
   //         long long ans = 0;
   //         for(int i=(n-1);i>=0;i--) {
   //             if(cnt==0) {
   //                 break;
   //             }
   //             ans = ans + nums[i];
   //             cnt--;
   //         }
   //         return ans;
   //     }
   // };

22. my issue is with the 2nd while loop, why is it there, build my intution
   ans why are we doing ans+=csm-mn,
   like the ans already contains the no.s with 1s added up, and in csm also we keep on adding the the same 1s just to capture some better no.s

23. whats the role of for loop,
   tell in simple and readble language to the point

24. any similar question on leetcode

25. How is CrackedIn different from just using ChatGPT for interview prep?

26. What about Gemini

27. Analyze my weak areas based on my LeetCode profile

28. turing company how to clear

29. What should I solve next based on my progress?

30. hi how are you doing?

31. hii

32. hi lovely talking to you

33. How to prepare for interview in tech company

34. How to prepare for interview in tech company

35. hii

36. hello how are yoi doinf

37. Show me what Google actually asks in coding interviews

38. can u give me real questions asked in 2026 Online assessment of Google

39. How do I sync my LeetCode profile with CrackedIn, and what do I get?

40. epam latest coding questions asked]

41. dsa questions

42. Show me what Google actually asks in coding interviews

43. Give me google interview

44. Analyze my weak areas based on my LeetCode profile

45. Huh

46. To

47. hi what can you do for me?

48. how are you better than normal gpt?

49. Analyze my weak areas based on my LeetCode profile

50. What should I solve next based on my progress?

51. hi

52. Show me what Google actually asks in coding interviews

53. Show me what Google actually asks in coding interviews

54. Tell me the exact questions

55. How is CrackedIn different from just using ChatGPT for interview prep?

56. IBS Software

57. show me what paypay asks in backend engineer interviews

58. Hii

59. Are you like a mentor or something. Describe yourselfy

60. I get it you are like a tutor

61. Can you remember things for me?

62. hi how are you doing

63. my target company is amazon as of now but could be change

64. sde 2,

65. i don't have target date but just randomly i want to learn

66. DevOps interview question for product base company

67. hi do you know what exactly we should prepare for devops, can you check your db to get the idea from the interview experience, i am asking for my friend

68. what does the rounds look like for devops, asking for a friend just to get knowledge

69. Gcp devops

70. I am preparing for Application Support role in Lloyds Banking company

71. I am preparing for Application Support role in Lloyds banking company with Devops knowledge. Help me prepare

72. I have interview on 25th June

73. Show me what Google actually asks in coding interviews

74. Show me what Google actually asks in coding interviews

75. Aspiring full stack developer

76. give me detailed information on compute, storage, networking and security in GCP

77. preparing for interview for Application support role for Lloyds banking company

78. yes

79. gcloud run logs

80. ?

81. Cloud SQL

82. Show me what intuit ask

83. Why do I keep getting stuck on medium problems, and how do I break through?

84. Prepare for Sierra AI

85. hello

86. what about dsa

87. What are the most common DSA patterns Google asks?

88. do you have juniper square interview experiences

89. can you make a list of frameworks and stacks used primarily for interviews for ai specific role

90. i want to prepare for google se

91. explain memoization

92. can you recommend me some questions from this topic

93. explain memoization

94. can you recommend me some questions from this topic

95. i completed problem solving fundamentals

96. explain bellman ford algo

97. recommend me some questions in this topic

98. suggest me some array questions

99. have u connected to my leetcode profile ?

100. suggest me some array questions

101. suggst me some string questions

102. suggest me some hashing questions

103. explain dnf algo

104. recommend some questions for this topic

105. i understood all this, now suggest me some questions to solve for this topic

106. Analyze my weak areas based on my LeetCode profile

107. teach me bellman ford algo

108. give the code in c++, always

109. i understand, now give me some questions to practice this

110. explain me graph in dsa

111. give me some problems to practice this

112. i understand, now give me some questions to practice stack questions

113. recommend me some graph questions to practice

114. hi

115. hi

116. hi

117. Hi

118. hello

119. thanks

120. hello

121. hi

122. Hu

123. hi

124. hi

125. hi

126. can you suggest me some questions on array

127. Analyze my weak areas based on my LeetCode profile

128. Hi

129. Salesforce

130. Smts

131. recommend me some graph questions

132. give me some questions to practice stack

133. recommend me some array questions

134. recommend me some string queastions

135. some string manipulations questions pleasw

136. hi

137. hi

138. hi

139. hi

140. hi

141. lets solve leetcode problem 300

142. explain leetcode question 3500, give me constraints

143. Hi

144. Forward Deployed Engineer OpenAI

145. There is no target date

146. Yes I did both

147. Yes

148. Yes let's go

149. One by one

150. Since it's incremental in timestamp I keep a map start with first message print for first entry continue for uniques and if entry exist in map check for time constraint > 10 sex with current and prev entry this should work

151. I'm using mobile code in unnecessary since it's easy the got the approach right too

152. I thinks we are going in wrong path I'm looking for forward deployed engineer

153. How is CrackedIn different from just using ChatGPT for interview prep?

154. Show me what Google actually asks in coding interviews

155. Show me what Google actually asks in coding interviews

156. lets play stone paper scisor

157. stone

158. how many times checksum has been asked in google in past 6 months ?

159. and what about past 3 months ?





# Interview Query Types and Recommended Retrieval Methods

This document contains only the query type, example user questions, and the recommended retrieval method.

## 1. Company-Specific Interview Experience Queries

### Example questions

- What is the Google interview process?
- Show me Amazon interview experiences.
- What questions are asked at Microsoft?
- Tell me recent PayPay backend interview experiences.
- What does Flipkart ask in SDE interviews?
- Do you have Juniper Square interview experiences?

### Recommended method

**Hybrid retrieval**

Use SQL for exact company, role, level, and date filters. Use embeddings to find semantically relevant parts of interview experiences.

---

## 2. Exact Interview Question Queries

### Example questions

- Give me actual questions asked at Google.
- What coding questions were asked in Amazon interviews?
- Show me questions from Microsoft OA.
- What system design questions were asked at Uber?
- Give me exact questions asked in PayPay backend interviews.
- Which LeetCode problems appeared in Google interviews?

### Recommended method

**Hybrid retrieval over question-level events**

The retrieval unit should be an individual question rather than an entire interview post.

---

## 3. Role-Specific Interview Queries

### Example questions

- What do companies ask backend engineers?
- Show me frontend interview experiences.
- What questions are asked for DevOps roles?
- What should I prepare for an application support interview?
- Give me data engineer interview questions.
- What is asked in an ML engineer interview?

### Recommended method

**SQL role filters plus semantic role expansion**

---

## 4. Level-Specific Interview Queries

### Example questions

- Google L3 interview experience
- Amazon SDE-1 interview questions
- Microsoft SDE-2 system design questions
- Senior backend engineer interview experience
- What is asked for staff engineer roles?
- Is system design asked for freshers?

### Recommended method

**SQL-first hybrid retrieval**

Level should usually be treated as a strong or hard filter.

---

## 5. Interview Stage Queries

### Example questions

- What is asked in the online assessment?
- Show me Google phone-screen questions.
- What happens in the technical round?
- What is asked in the hiring manager round?
- Give me questions from the machine coding round.
- What happens in the bar raiser round?
- What questions are asked in HR interviews?

### Recommended method

**SQL round-type filtering plus embeddings over round content**

---

## 6. Recent and Time-Sensitive Queries

### Example questions

- What has Google asked in the last six months?
- Show me 2026 Amazon OA questions.
- What are the latest Microsoft interview questions?
- Are dynamic programming questions still asked at Google?
- What interview pattern is currently followed by Flipkart?
- Has the PayPay interview process changed recently?

### Recommended method

**SQL date filtering plus semantic retrieval and freshness ranking**

The system should distinguish the date the interview happened from the date the post was published.

---

## 7. Frequency and Trend Queries

### Example questions

- What are the most common Google coding questions?
- Which topics are frequently asked at Amazon?
- How often is dynamic programming asked at Microsoft?
- Is graph theory common in Uber interviews?
- Which system design questions repeat most often?
- What are the trending topics in recent Flipkart interviews?

### Recommended method

**SQL aggregation after semantic clustering or normalization**

---

## 8. “Was This Question Asked?” Queries

### Example questions

- Has LRU Cache been asked at Google?
- Was checksum asked in Amazon OA?
- Has LeetCode 300 appeared in Microsoft interviews?
- Do backend interviews at PayPay ask Kafka?
- Has URL Shortener been asked for SDE-2 roles?

### Recommended method

**Exact SQL match plus semantic match plus evidence verification**

---

## 9. Count-Based Queries

### Example questions

- How many times has LRU Cache been asked at Google?
- How often was checksum asked in the last six months?
- How many Amazon experiences mention graphs?
- Which company asks system design most frequently?
- How many interview posts mention Kubernetes?

### Recommended method

**SQL aggregation after semantic normalization**

The answer should clearly state what exactly was counted.

---

## 10. Topic-Specific Interview Queries

### Example questions

- Give me Google graph questions.
- What DP problems are asked at Amazon?
- Show me array questions from Microsoft interviews.
- What operating system questions appear in backend interviews?
- Give me database questions asked at product companies.
- What concurrency questions are asked in Java interviews?

### Recommended method

**SQL topic filters plus embedding-based topic expansion**

---

## 11. Problem Pattern Queries

### Example questions

- What sliding window questions are asked at Google?
- Give me two-pointer interview questions.
- Which graph patterns are common in Amazon interviews?
- Show me monotonic stack questions from real interviews.
- What cache design problems are frequently asked?

### Recommended method

**Normalized pattern tags plus semantic retrieval**

---

## 12. Similar Question Queries

### Example questions

- Show me questions similar to LRU Cache.
- What questions are like Longest Increasing Subsequence?
- Find interview questions related to cache eviction.
- Give me variations of Two Sum asked in interviews.
- What questions are similar to designing a rate limiter?

### Recommended method

**Embedding search over normalized question text**

SQL alone is not suitable for semantic similarity.

---

## 13. Question Variation Queries

### Example questions

- What variations of Two Sum are asked at Google?
- How is LRU Cache modified in interviews?
- What follow-up questions are asked after URL Shortener?
- What harder versions of BFS are asked at Amazon?
- What variations of rate limiter appear in system design interviews?

### Recommended method

**Question-family clustering plus embeddings**

---

## 14. Follow-Up Question Queries

### Example questions

- What follow-ups are asked after solving LRU Cache?
- How do interviewers extend the Two Sum problem?
- What follow-up questions come after designing a URL shortener?
- What optimization questions are asked after the first solution?
- Do they ask for time and space complexity?

### Recommended method

**Structured event-level retrieval**

---

## 15. Difficulty Queries

### Example questions

- How difficult are Google coding interviews?
- Are Amazon OA questions mostly medium or hard?
- What difficulty level should I expect for SDE-1?
- Which companies ask the hardest graph questions?
- Give me easy interview questions asked at product companies.
- Are senior interviews more focused on system design?

### Recommended method

**SQL aggregation over calibrated difficulty fields**

Different difficulty sources should be labeled separately.

---

## 16. Interview Process Queries

### Example questions

- What is the complete Google interview process?
- How many rounds does Amazon have?
- What happens after the OA?
- How long is the Microsoft interview process?
- Is there a machine coding round at Flipkart?
- Does PayPay have a system design round?

### Recommended method

**Structured aggregation from multiple interview experiences**

---

## 17. Interview Duration Queries

### Example questions

- How long is each Google interview round?
- How much time is given in Amazon OA?
- How long does the complete hiring process take?
- How many coding questions are asked in 60 minutes?
- How long is the system design round?

### Recommended method

**SQL aggregation over duration fields**

---

## 18. Location-Specific Queries

### Example questions

- Google India interview experience
- Amazon US interview process
- PayPay Japan backend interview
- Microsoft Bangalore interview questions
- Is the process different in London?
- What does Google Warsaw ask?

### Recommended method

**SQL location filtering plus hybrid retrieval**

---

## 19. Employment-Type Queries

### Example questions

- Google internship interview experience
- Amazon new graduate questions
- Contract engineer interview questions
- Off-campus interview process
- Campus placement interview experience
- Lateral hiring questions for senior engineers

### Recommended method

**SQL hard filtering**

---

## 20. Candidate Background Queries

### Example questions

- Show me interview experiences from freshers.
- What was asked to candidates from tier-3 colleges?
- Show experiences of candidates with two years of experience.
- What questions are asked during career switches?
- Give me experiences of frontend developers moving to backend.

### Recommended method

**SQL metadata filtering when reliable data is available**

This data may be sensitive, incomplete, or self-reported and should be used carefully.

---

## 21. Technology-Specific Queries

### Example questions

- What Java questions are asked at Amazon?
- Give me React interview questions from real experiences.
- What Kafka questions are asked for backend roles?
- Does Google ask Kubernetes?
- What SQL queries are asked in data engineering interviews?
- What Spring Boot questions appear in interviews?

### Recommended method

**Technology tags plus semantic retrieval**

---

## 22. Domain-Specific Interview Queries

### Example questions

- What fintech companies ask in backend interviews?
- Give me e-commerce system design questions.
- What questions are asked in trading companies?
- What security questions appear in banking interviews?
- What healthcare companies ask data engineers?

### Recommended method

**Company-domain mapping plus embeddings**

---

## 23. Coding Language Queries

### Example questions

- Can I use C++ in Google interviews?
- Are Amazon OA questions language-independent?
- Give me Java-specific interview questions.
- Do frontend interviews require JavaScript?
- Which language is preferred for PayPay backend interviews?

### Recommended method

**Structured metadata plus company guidelines and interview evidence**

A single interview experience should not be treated as official company policy.

---

## 24. Behavioral Interview Queries

### Example questions

- What behavioral questions does Amazon ask?
- Give me Google leadership questions.
- What HR questions are common?
- What questions are asked about conflict resolution?
- How should I answer “Tell me about yourself”?
- What STAR questions appeared in real interviews?

### Recommended method

**Embedding search over behavioral-question events**

---

## 25. Company Values and Principles Queries

### Example questions

- Which Amazon leadership principles are commonly tested?
- How does Google assess Googliness?
- What values does Microsoft evaluate?
- Which behavioral stories should I prepare for Uber?
- What principles are tested in a bar raiser interview?

### Recommended method

**Company-value mapping plus interview-experience retrieval**

---

## 26. System Design Queries

### Example questions

- What system design questions are asked at Google?
- Has URL Shortener been asked at Amazon?
- What design questions appear for SDE-2?
- Do freshers get system design questions?
- Give me recent backend system design experiences.
- What follow-ups are asked in rate limiter design?

### Recommended method

**Hybrid question search plus role and level filtering**

---

## 27. Low-Level Design and Machine Coding Queries

### Example questions

- What LLD questions are asked at Flipkart?
- Give me machine coding round experiences.
- Is Parking Lot still asked?
- What design patterns are tested?
- What are common follow-ups in LLD rounds?
- How much code must be completed?

### Recommended method

**Structured round filtering plus semantic question retrieval**

---

## 28. Online Assessment Queries

### Example questions

- What was asked in Google OA 2026?
- Which platform does Amazon use for OA?
- How many questions are in Microsoft OA?
- What is the cutoff for Flipkart OA?
- Are aptitude questions included?
- Was proctoring enabled?
- How difficult was the OA?

### Recommended method

**SQL aggregation plus experience evidence**

---

## 29. Interview Outcome Queries

### Example questions

- Did the candidate get selected?
- What mistakes caused rejection?
- Why was the candidate rejected after system design?
- What approaches led to selection?
- Does solving both questions guarantee selection?
- How important is communication?

### Recommended method

**Structured retrieval plus careful interpretation**

Candidate speculation should not be presented as confirmed recruiter reasoning.

---

## 30. Failure Pattern Queries

### Example questions

- Why do candidates fail Google interviews?
- What common mistakes happen in Amazon OA?
- What causes rejection in system design?
- Which behavioral mistakes are risky?
- What mistakes do candidates make in machine coding?

### Recommended method

**Semantic clustering of reflections and feedback**

---

## 31. Preparation Strategy Queries

### Example questions

- How should I prepare for Google in 30 days?
- What should I study for Amazon SDE-1?
- Create a PayPay backend preparation plan.
- Which topics should I prioritize?
- How many problems should I solve before interviewing?
- Should I focus on DSA or system design?

### Recommended method

**Company evidence plus user profile plus planning logic**

---

## 32. Personalized Company Preparation Queries

### Example questions

- Based on my LeetCode profile, am I ready for Google?
- What should I solve before my Amazon interview?
- Which interview questions match my weak topics?
- I have seven days; what should I revise?
- Which Google questions have I not solved?
- Recommend questions based on my progress.

### Recommended method

**User SQL data plus hybrid company-corpus retrieval**

---

## 33. Readiness Assessment Queries

### Example questions

- Am I ready for a Google interview?
- What are my gaps for SDE-1?
- Which topics am I weak in compared with Amazon interviews?
- How much of the Microsoft interview syllabus have I covered?
- What is my probability of clearing the OA?

### Recommended method

**User analytics plus company interview data**

---

## 34. Comparison Queries

### Example questions

- Google vs Amazon interview process
- Which is harder: Microsoft or Uber?
- Compare PayPay and Google backend interviews.
- Which company asks more DP questions?
- Amazon SDE-1 vs SDE-2 interview differences
- India vs US Google interviews

### Recommended method

**SQL aggregation plus normalized comparison**

---

## 35. Company Ranking Queries

### Example questions

- Which companies ask the most graph questions?
- Which companies focus heavily on system design?
- Which companies have the hardest OA?
- Which companies are best for frontend candidates?
- Which product companies ask medium-level DSA?

### Recommended method

**SQL aggregation across companies**

---

## 36. Cross-Company Pattern Queries

### Example questions

- What questions repeat across FAANG companies?
- Which system design questions are common across product companies?
- What topics appear in both Google and Amazon?
- Which questions are unique to Microsoft?
- What interview patterns are common in fintech?

### Recommended method

**Question clustering plus company aggregation**

---

## 37. Interview Experience Summary Queries

### Example questions

- Summarize this interview experience.
- Give me the important points.
- Extract all questions from this experience.
- Show the rounds, questions, and outcome.
- Make a timeline of this interview.
- What should I learn from this experience?

### Recommended method

**Document and event summarization**

---

## 38. Evidence and Source Queries

### Example questions

- Where did this question come from?
- Show me the original interview experience.
- How reliable is this information?
- Was this reported by multiple candidates?
- Is this a confirmed interview question?
- Give me the source and date.

### Recommended method

**SQL provenance retrieval**

---

## 39. Query by Description

### Example questions

- What is the Google question where we find the longest increasing sequence?
- Which Amazon problem involves cache eviction?
- I remember a graph question with flights and limited stops.
- Find the problem where we merge overlapping meetings.
- What question asks us to detect a cycle in a linked list?

### Recommended method

**Embedding search over problem descriptions**

This is one of the strongest reasons not to rely on SQL alone.

---

## 40. Vague Conversational Queries

### Example questions

- What does Google ask?
- Tell me about Amazon.
- Give me recent questions.
- Show me backend interviews.
- What should I prepare?
- What comes next?

### Recommended method

**Conversation-aware hybrid retrieval**

---

## 41. Multi-Constraint Queries

### Example questions

- Show Google L3 graph questions asked in India in the last year.
- Give me Amazon SDE-1 OA problems from the last six months.
- Show recent PayPay backend system design questions in Japan.
- Find medium DP questions asked to new graduates at Microsoft.
- Which unsolved problems from recent Google interviews match my weak topics?

### Recommended method

**SQL constraint engine plus embedding retrieval**

---

## 42. Negative and Exclusion Queries

### Example questions

- Show Google questions excluding dynamic programming.
- Give me backend experiences without system design.
- Show only unsolved questions.
- Exclude internship experiences.
- Do not include behavioral questions.
- Show questions not present in NeetCode 150.

### Recommended method

**SQL exclusion filters**

---


160. show me google l3 interview experiences

161. what does crackedin do?
