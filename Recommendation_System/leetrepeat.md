# LeetRepeat 🧠

**LeetRepeat** is a modern, AI-powered Spaced Repetition System (SRS) designed specifically for mastering Data Structures and Algorithms (DSA) problems. 

It combines the science of the **Forgetting Curve** with the power of **Google Gemini 2.5** to help you retain patterns, get conceptual hints without spoilers, and discover similar problems to reinforce your learning.

## ✨ Features

### 🚀 Smart Tracking (SRS)
*   **Spaced Repetition Algorithm:** Automatically schedules reviews based on mastery intervals (1, 3, 7, 14, 30 days).
*   **Visual Progress:** See exactly what is due today, what is mastered, and what is overdue.
*   **Stage System:** Problems move through 5 stages of mastery. Failing a review resets the streak, ensuring deep retention.

### 🤖 AI-Powered Assistance (Gemini 2.5 Flash)
*   **Smart Hints:** Stuck? Get a conceptual hint (strategy/data structure) in **plain text** without seeing the code solution.
*   **Auto-Fill:** Paste a LeetCode URL, and the AI automatically extracts the Title, Difficulty, and Category.
*   **Similar Problems:** The AI analyzes the current problem and suggests 3 similar problems to practice specific patterns (e.g., Sliding Window, DFS).

### 📊 Analytics & Organization
*   **Visual Dashboard:** 
    *   **Pie Charts** for Category breakdown (interactive with percentage tooltips).
    *   **Donut Charts** for Difficulty breakdown.
*   **NeetCode 150 Integration:** Built-in tracking for the famous NeetCode 150 roadmap.
*   **Filtering & Sorting:** Sort by Next Review, Difficulty, or Last Reviewed. Search by title.

### 🛠️ Developer Experience
*   **Dark Mode:** Fully supported system-aware dark mode.
*   **Local-First:** All data is stored in your browser's `localStorage`. No login required.
*   **Import/Export:** Backup your progress to a JSON file and restore it anywhere.

---

## 🛠️ Tech Stack

*   **Framework:** React 19
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS
*   **Icons:** Lucide React
*   **Charts:** Recharts
*   **AI:** Google GenAI SDK (`@google/genai`)

---

## 🚀 Getting Started

### Prerequisites
*   Node.js (v18+)
*   A Google Gemini API Key (Get it from [Google AI Studio](https://aistudio.google.com/))


## 📖 How to Use

### 1. Adding a Problem
Click **"Track Problem"**. You can either manually type the details or paste a LeetCode URL and click **"Auto-Fill"** to let the AI populate the form.

### 2. The Review Process
*   **Due Today:** Problems scheduled for review appear here.
*   **Solve the problem** on LeetCode.
*   **Mark Progress:**
    *   ✅ **Check Button:** You solved it! The interval increases (e.g., 1 day -> 3 days).
    *   🔄 **Reset Button:** You struggled. The problem resets to Stage 1 (1 day interval) to reinforce learning.

### 3. Using AI Hints
Click the **Lightbulb** icon on any problem card.
*   The AI will generate a brief, conceptual hint in plain text.
*   It avoids spoilers and Markdown code blocks to keep the focus on *thinking*.

### 4. NeetCode 150 Tab
Switch to the **NeetCode 150** tab to see the curated roadmap.
*   Track problems directly from the list.
*   Use the **Bot Icon** next to any problem (even untracked ones) to find similar problems.

---

## 🧠 The Spaced Repetition Logic

LeetRepeat uses a modified Leitner system with the following intervals:

| Stage | Interval | Status |
|-------|----------|--------|
| New   | 0 Days   | Learning |
| 1     | 1 Day    | Review Tomorrow |
| 2     | 3 Days   | Short Term |
| 3     | 7 Days   | Medium Term |
| 4     | 14 Days  | Long Term |
| 5     | 30 Days  | Mastered 🏆 |

---

## 📄 License

MIT License. Feel free to use and modify for your own learning journey.
