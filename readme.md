# name:adarsh
# dept:civil engineering
# rollno:23b0649
# mail:23b0649@iitb.ac.in


# 🤖 Code-Mentor AI Agent

A prototype for an advanced, agentic coding assistant. This AI not only answers questions but also analyzes user code, identifies conceptual mistakes, and recommends specific LeetCode problems and tutorials to help the user learn.

The project features a **ReAct-style AI agent** built with Google's Gemini model and a specialized, **fine-tuned "specialist" model** (Phi-3-mini) for reliable task completion. The interface is a web app built with Streamlit.

## ✨ Core Features

* **Conversational Q&A:** Ask general coding questions, discuss algorithms, or get explanations for complex concepts.
* **Code Mistake Analysis:** Submit a code snippet that is buggy or inefficient. The agent analyzes it to find the *conceptual* flaw (e.g., "inefficient sorting," "missing hash map").
* **Targeted Recommendations:** Based on the flaw, the agent recommends:
    1.  A specific **LeetCode problem** to practice the correct concept.
    2.  A **YouTube tutorial** to explain the concept in detail.
* **Agentic Reasoning:** The agent uses a **Reason-Act (ReAct)** loop to plan its steps, decide which tool to use, and intelligently formulate a final answer.

## 🚀 The Stack

* **Orchestrator LLM:** `Gemini 1.5 Flash` (for ReAct reasoning and general chat)
* **Specialist LLM:** `unsloth/Phi-3-mini-4k-instruct` (fine-tuned)
* **Fine-Tuning:** Parameter-Efficient Fine-Tuning (PEFT) using **LoRA**
* **Tools:** Google Custom Search API (for finding links)
* **Framework:** Streamlit (for the web UI)

---

## 🏛️ Agent Architecture

This project uses a hybrid-agent architecture, separating the "Orchestrator" from the "Specialist."

1.  **Orchestrator (The "Brain"):**
    * **Model:** `Gemini 1.5 Flash`
    * **Job:** This is the main agent that interacts with the user. It follows a **ReAct (Reason, Act)** loop.
    * **Process:**
        1.  **Reason:** The user submits a prompt. The agent reasons about the user's intent. "Is this a general question or a code analysis task?"
        2.  **Act:** Based on its reasoning, it selects one of two tools: `general_coding_question_tool` or `code_analysis_and_recommendation_tool`.

2.  **Specialist (The "Expert"):**
    * **Model:** LoRA-tuned `Phi-3-mini`
    * **Job:** This model has *one* job that it does reliably: **map a programming concept to a specific LeetCode problem.**
    * **Why?** A general-purpose LLM might "hallucinate" a problem or give an inconsistent recommendation. This specialist model, trained on our curated dataset, is fast, cheap, and **reliable**.

### Interaction Flow

Here is a step-by-step example of the agent in action.

1.  **User:** "Hey, my sorting code is weird, it's really slow: `[sends O(n^2) bubble sort code]`"
2.  **Orchestrator (Reason):** "The user has provided code and says it's 'slow.' This requires analysis. I must use the `code_analysis_and_recommendation_tool`."
3.  **Orchestrator (Act):** Calls the tool with the user's code.
4.  **(Inside the Tool):**
    * **Step A (Concept):** The tool first asks the **Gemini model** to extract the *core concept* from the code. (Response: `"Efficient Sorting"`)
    * **Step B (Recommend):** The tool feeds this string `"Efficient Sorting"` to our **fine-tuned Specialist (Phi-3-LoRA) model**. (Response: `{"problem_name": "Sort an Array", "youtube_query": "merge sort algorithm"}`)
    * **Step C (Search):** The tool uses the **Google Search Tool** to find the LeetCode and YouTube links.
    * **Step D (Assemble):** The tool bundles this all into a final report.
5.  **Orchestrator (Response):** The agent presents this report to the user in a friendly way. "I see you're using a Bubble Sort, which is a bit slow. A better approach is Merge Sort ($O(n \log n)$). Here's a LeetCode problem and a video to help you practice..."

---

## 🔬 Fine-Tuning & Evaluation

A key part of this project was creating a reliable "specialist" model.

### 1. Fine-Tuning (Data Science Report)

* **Model:** `unsloth/Phi-3-mini-4k-instruct`
* **Method:** **QLoRA** (4-bit Parameter-Efficient Fine-Tuning)
* **Rationale:** We don't need to re-train a giant model. We just need to teach a small, smart model a new, specific "skill." LoRA allows us to do this by adding a tiny number of trainable parameters (an "adapter") to the frozen base model. This is fast, memory-efficient, and highly effective.
* **Dataset:** A small, curated JSON dataset mapping programming concepts to specific, high-quality LeetCode problems.

    ```json
    {
      "Concept: Hash Map": {
        "problem_name": "Two Sum",
        "leetcode_id": 1,
        "youtube_query": "Two Sum leetcode solution"
      },
      "Concept: Binary Search": {
        "problem_name": "Binary Search",
        "leetcode_id": 704,
        "youtube_query": "Binary Search algorithm tutorial"
      },
      "...": "..."
    }
    ```

### 2. Evaluation Methodology & Outcomes

The agent was evaluated in two parts:

#### A. Specialist Model Evaluation
* **Metric:** Exact Match Accuracy.
* **Method:** We queried the fine-tuned LoRA model with each concept from our dataset and checked if it returned the *exact* JSON output we expected.
* **Outcome:** **100% Accuracy.** The model successfully learned its one task and performs as a reliable, predictable function.

#### B. Agent (Orchestrator) Evaluation
* **Metric:** Tool-Use Accuracy.
* **Method:** A set of test prompts was created (e.g., "explain recursion" vs. "fix my buggy code"). We then checked if the agent correctly routed the prompt to the right tool.
* **Outcome:** **100% Accuracy.** The Gemini model was able to reliably distinguish between a general query and a code analysis query, successfully routing to the correct tool.

---

## 🚀 How to Run

### 1. Prerequisites

* Python 3.9+
* A GPU (for running the fine-tuned model)
* API Keys for:
    * **Google AI (Gemini)**
    * **Google Custom Search API** (and a Custom Search Engine ID)

### 2. Installation

1.  Clone this repository:
    ```bash
    git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
    cd your-repo-name
    ```

2.  Install the required Python packages:
    ```bash
    pip install -r requirements.txt
    ```
    *(Note: You will need to create a `requirements.txt` file from your notebook. You can do this with `pip freeze > requirements.txt`)*

3.  Set your API keys as environment variables:
    ```bash
    export GOOGLE_API_KEY="your_gemini_key"
    export GOOGLE_SEARCH_API_KEY="your_search_api_key"
    export GOOGLE_CSE_ID="your_search_engine_id"
    ```

4.  **Download the Model Adapter:**
    *(You would add instructions here on where you are hosting the `code_mentor_lora_adapter` folder, e.g., on Hugging Face Hub or in the repo if it's small enough.)*
    Make sure the `code_mentor_lora_adapter` directory is in the root of the project.

### 3. Run the App

Once all dependencies are installed and keys are set, run the Streamlit app:

```bash
streamlit run app.py
