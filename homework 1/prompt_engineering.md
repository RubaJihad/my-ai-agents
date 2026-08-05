# Prompt Engineering Concepts & Evaluation

In this project, three main Prompt Engineering techniques were used within the multi-agent system architecture to ensure predictable, structured, and reliable execution across specialized domain experts. Below is a detailed analysis of each concept and its effectiveness in system operations.

---

## 1. Role Prompting (Persona & System Instructions)

### Concept & Implementation

Role Prompting involves explicitly defining a specific persona, domain boundary, and functional responsibilities for the language model via the system prompt. Instead of relying on a single general-purpose agent, each expert in the system (e.g., `Database Read Expert`, `Database Write Expert`, `Content Expert`, and `Orchestrator`) is initialized with strict domain constraints and behavioral rules retrieved from the `llm_roles` table.

- **Example Configuration**:
  - **Role**: `Database Read Expert`
  - **Domain**: `SQL query generation for a resume database`
  - **Instructions**: _"Respond with a single valid SQLite SELECT query only. No markdown, no explanation — SQL only."_

### Effectiveness & Impact

- **Output Standardization:** Strictly eliminated conversational filler (e.g., _"Sure! Here is your SQL code:"_) and ensured the LLM produced clean, executable queries.
- **Hallucination Reduction:** Constrained the expert's scope strictly to schema navigation, preventing unauthorized data modification or off-topic responses.

---

## 2. Few-Shot Learning (In-Context Exemplars)

### Concept & Implementation

Few-Shot Learning provides the model with explicit input-output pairs inside the prompt before asking it to perform a task. This trains the LLM in-context to follow target schemas and syntactic expectations without requiring model fine-tuning.

- **Example Exemplar (`Database Read Expert`):**
  - **User Query:** `"How long did they work at MSU?"`
  - **Expected Output:** `SELECT p.start_date, p.end_date FROM positions p JOIN institutions i ON p.inst_id = i.inst_id WHERE i.name = 'MSU';`

### Effectiveness & Impact

- **Improved Query Accuracy:** Handled multi-table `JOIN` logic and foreign key relationships correctly on the first attempt.
- **Syntax Consistency:** Guaranteed that output structures aligned exactly with what downstream Python parsers (such as `sqlite3` execution functions) expected.

---

## 3. Task Decomposition & Orchestration (Chain-of-Thought Planning)

### Concept & Implementation

Task Decomposition breaks down complex, multi-step natural language requests into an ordered, structured execution plan. The `Orchestrator` agent analyzes multi-intent user prompts (e.g., _"Check if they have React skill, and if not, add it to their latest experience"_) and decomposes them into a sequential array of specialized agent calls.

- **Example Decomposition Plan**:
  1. Call `Database Read Expert` to check for existing skill records.
  2. Evaluate returned results.
  3. Call `Database Write Expert` to insert the new record into the SQLite database if missing.

### Effectiveness & Impact

- **Handling Complex Logic:** Solved multi-stage queries that a single zero-shot LLM call could not complete reliably in one step.
- **Modular Reliability & Fewer Crashes:** Isolated failure points by ensuring that data retrieval, data modification, and content synthesis are handled by specialized, decoupled prompt workflows.
