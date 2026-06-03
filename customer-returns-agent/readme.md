## **Customer Returns Agent**

This workflow implements an AI-powered customer returns assistant in n8n. It handles live chat conversations, verifies customers and orders against data tables, retrieves product details, and creates return requests. It also includes an evaluation pipeline for testing multi-turn conversations and measuring correctness and tool usage before publishing.

### **Workflow Components**

1. **Customer Service Agent Core**

The `Customer Service Agent` node orchestrates the full return flow: collecting customer info, validating order context, asking for return reason, and producing a structured response.

2. **Chat Entry and Memory**

`When chat message received` starts live conversations. `Chat Memory` and `Chat Memory Manager` persist context so multi-turn interactions stay coherent.

3. **Lookup and Return Tools**

`Lookup Customer`, `Lookup Order`, and `Lookup Product` query data tables used by the agent as tools. `Process Return` writes a return request record.

4. **Structured Output Layer**

`Structured Output` enforces a JSON response shape containing fields like `response`, `customerFound`, `orderFound`, `returnProcessed`, and `nextStep`.

5. **LLM Nodes (Gemini)**

Gemini chat model nodes power both agent reasoning and evaluation/scoring steps.

6. **Sample Data Bootstrap**

`Customer Sample Data`, `Order Sample Data`, and `Product Sample Data` with split/insert nodes populate customer, order, and product tables for testing.

7. **Evaluation Pipeline**

`When fetching a dataset row` feeds test cases into the agent. `Eval Correctness` and `Eval - Tool Use` score response quality and expected tool behavior, then write outputs through `Evaluation1`.

### **Setup Instructions**

1. **Create Data Tables**

Create and connect data tables for:
- Customers
- Orders
- Products
- Returns
- Evaluation dataset
- Evaluation output

2. **Map Data Table Nodes**

Open each Data Table/Data Table Tool node and point it to the correct table in your n8n project. Verify table schemas match the node mappings.

3. **Configure Google Gemini Credentials**

Open each Gemini chat model node and set your Google Gemini (PaLM) API credentials.

4. **Configure Chat Trigger**

Open `When chat message received` and confirm webhook/chat settings for your workspace or deployment target.

5. **Initialize Sample Data (Optional but Recommended)**

Run the sample data branch once to seed customers, orders, and products before live testing.

6. **Prepare Evaluation Dataset**

Set up an evaluation dataset table with conversation test rows and expected outcomes (including deterministic tool-use expectations if you use that metric).

7. **Validate Structured Output Contract**

Confirm the `Structured Output` schema matches your downstream app requirements.

### **Execution Details**

**Regular / Automatic Execution**

For production chat usage, activate the workflow and route user messages through the chat trigger. The agent retrieves memory and calls lookup/process tools as needed to complete return requests.

**Evaluation Execution**

Use the evaluation trigger to run test rows through the same agent logic. Review correctness and tool-use metrics in evaluation outputs to identify regressions before release.

**How the Return Flow Works at Runtime**

1. User opens chat and asks about a return.
2. Agent asks for missing identifiers (email or order number).
3. Agent verifies customer and order via lookup tools.
4. Agent fetches product details for context.
5. Agent collects return reason and writes return request via `Process Return`.
6. Agent responds with clear next steps, formatted using the structured output parser.

**Deployment / Embedding**

After validation, publish the workflow's chat endpoint/widget from n8n and embed it in your site or app. Keep data table credentials scoped to least privilege and monitor evaluation scores over time for drift.
