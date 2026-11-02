# 🤖 AI-Powered SQL Analyst | Natural Language to PostgreSQL (n8n + Gemini)

> Ask questions in plain English like *"Show me sales from Mumbai last month"* and get real answers from your PostgreSQL database. No SQL needed.

![n8n](https://img.shields.io/badge/Automation-n8n-EA4B71?logo=n8n&logoColor=white)
![Gemini](https://img.shields.io/badge/LLM-Google%20Gemini-4285F4?logo=google&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-336791?logo=postgresql&logoColor=white)
![AI Agent](https://img.shields.io/badge/Type-AI%20Agent-8A2BE2)

---

## 📑 Table of Contents

1. [About the Project](#-about-the-project)
2. [Use Case](#-use-case)
3. [Key Features](#-key-features)
4. [Workflow Architecture](#-workflow-architecture)
5. [Tech Stack](#-tech-stack)
6. [Prerequisites](#-prerequisites)
7. [Setup & Installation](#-setup--installation)
8. [Step-by-Step Build Guide](#-step-by-step-build-guide)
9. [System Message for the AI Agent](#-system-message-for-the-ai-agent)
10. [Schema Discovery Query](#-schema-discovery-query)
11. [Example Questions](#-example-questions)
12. [Testing & UAT](#-testing--uat)
13. [Safety & Best Practices](#-safety--best-practices)
14. [Project Structure](#-project-structure)
15. [Future Improvements](#-future-improvements)
16. [Skills Demonstrated](#-skills-demonstrated)
17. [Acknowledgements](#-acknowledgements)
18. [License](#-license)

---

## 📌 About the Project

This project is an **AI Agent built in n8n** that acts like a smart junior data analyst. Users chat with it in natural language, the agent understands the question, looks at the database structure, writes the right **PostgreSQL** query, runs it, and explains the result in the chat.

It uses **Google Gemini** as the reasoning model, **Simple Memory** to remember the conversation, and **Postgres tools** to read the schema and execute queries. If a question is unclear, the agent asks follow-up questions. If the data doesn't exist, it says so clearly instead of guessing.

---

## 💼 Use Case

Business teams often depend on a technical person to pull numbers from the database. This agent removes that bottleneck.

- A sales manager asks: *"Show me sales from Mumbai last month."*
- The agent converts it into SQL, runs it on Postgres, and replies with the answer.
- Teams get insights instantly, without writing a single line of SQL.

---

## ✨ Key Features

- 💬 **Chat interface** that starts the workflow whenever a message is received
- 🧠 **Natural language to SQL** conversion using Google Gemini
- 🗂️ **Schema awareness**: the agent inspects tables and columns before writing queries
- ⚡ **Direct query execution** on PostgreSQL with results returned in chat
- 🧾 **Conversation memory** so follow-up questions keep their context
- ❓ **Clarifying questions** when the request is ambiguous
- 🚫 **Graceful "no data" handling** with a clear fallback message
- 🧪 **UAT-tested** against complex, real business questions

---

## 🏗️ Workflow Architecture

```
┌───────────────────────────┐        ┌──────────────────────────────────┐
│ When chat message received│───────▶│            AI Agent              │
│      (Chat Trigger)       │        │  (Junior SQL Analyst + System    │
└───────────────────────────┘        │   Message)                       │
                                     └───┬──────────┬───────────┬───────┘
                                         │          │           │
                         ┌───────────────┘          │           └──────────────┐
                         ▼                          ▼                          ▼
              ┌────────────────────┐    ┌───────────────────┐      ┌────────────────────────┐
              │ Google Gemini      │    │ Simple Memory     │      │ Tools                  │
              │ Chat Model (brain) │    │ (conversation)    │      │  • Postgres            │
              └────────────────────┘    └───────────────────┘      │    (executeQuery)      │
                                                                   │  • Query_Executor      │
                                                                   └───────────┬────────────┘
                                                                               ▼
                                                                    ┌────────────────────┐
                                                                    │   PostgreSQL DB    │
                                                                    └────────────────────┘
```

**Flow:** User message → AI Agent understands intent → checks schema → generates SQL → executes query → returns a plain-English answer.

> 💡 Add your n8n canvas screenshot here: `![Workflow](assets/workflow.png)`

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Automation / agent framework | **n8n** (Cloud trial or self-hosted) |
| LLM | **Google Gemini Chat Model** |
| Memory | **Simple Memory** node |
| Database | **PostgreSQL** |
| Query tools | n8n **Postgres** tool (`executeQuery`) and **Query_Executor** tool |
| Trigger | n8n **Chat Trigger** (When chat message received) |
| Prompt drafting help | ChatGPT / Perplexity |

---

## ✅ Prerequisites

- An **n8n** account: [14-day cloud trial](https://app.n8n.cloud/register) or a local/self-hosted install
- A running **PostgreSQL** database with some business data (sales, customers, products, etc.)
- A **Google Gemini API key** (from Google AI Studio)
- Postgres connection details: host, port, database, user, password
- A **read-only** database user (strongly recommended)

---

## ⚙️ Setup & Installation

### 1. Set up n8n

**Cloud:** sign up at https://app.n8n.cloud/register


### 2. Create a read-only Postgres user

```sql
CREATE USER n8n_readonly WITH PASSWORD 'your_strong_password';
GRANT CONNECT ON DATABASE your_database TO n8n_readonly;
GRANT USAGE ON SCHEMA public TO n8n_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO n8n_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO n8n_readonly;
```

### 3. Add credentials in n8n

| Credential | Used by |
|---|---|
| Postgres | Postgres tool, Query_Executor |
| Google Gemini (PaLM) API | Google Gemini Chat Model |

---

## 🪜 Step-by-Step Build Guide

**Step 1: Set up your n8n account.** Use the cloud trial or a local setup.

**Step 2: Draft the automation with AI.** Use ChatGPT/Perplexity to plan the steps. Starter prompt:
> *I want to create an AI Agent in n8n that helps in writing SQL queries, executing them in my Postgres DB, and returning the answers whenever we ask questions in plain English. It should also check if there is actual data available to answer that question; otherwise, it should ask follow-up questions and then answer. If there is no data available to answer the question, it should clearly mention, 'There is no data available to answer your questions. Here are the different details I can help you with.' Draft the steps for this automation.*

**Step 3: Connect the trigger (chat).** Add the **When chat message received** node. It watches incoming messages and triggers the AI Agent.

**Step 4: Add the AI Agent node.** This is your smart junior analyst with SQL knowledge. Add the system message from the section below (or generate one with the prompt provided there).

**Step 5: Give power and life to the agent.**
- Attach a **Google Gemini Chat Model** as the brain.
- Attach **Simple Memory** so it remembers the conversation.
- Attach the **Postgres tool (`executeQuery`)** to check table structure and schema.
- Attach the **Query_Executor** tool to run the final query and fetch the data.

**Step 6: Test the agent.**
- Ask real business questions.
- Confirm it generates the correct query and returns the right answer.
- Perform UAT to validate edge cases and error handling.

---

## 🧠 System Message for the AI Agent

Generate one with this prompt in ChatGPT:
> *Draft a system message for the AI agent, which needs to act as a SQL Agent, and turn the user's plain English queries into SQL queries. It should be smart enough to understand what tables are available in our Postgres DB. If the question is not clear, it should ask follow-up questions for confirmation and then proceed with answering the questions. In case of unavailability of data to answer the particular question, it should specifically mention, 'There is no available data to answer the particular question, but I can help with the following details.'*

Example system message you can start from:

```text
You are a SQL Analyst Agent connected to a PostgreSQL database.

Your job is to turn the user's plain English questions into correct PostgreSQL queries,
run them using the provided tools, and explain the results clearly.

Rules:
1. Always check the database schema (tables and columns) before writing a query.
2. Only generate SELECT queries. Never run INSERT, UPDATE, DELETE, DROP, or ALTER.
3. If the question is ambiguous, ask a short follow-up question before querying.
4. If the data needed does not exist, reply exactly:
   "There is no available data to answer the particular question, but I can help with the following details:"
   and list the tables/metrics you can answer from.
5. Present results in a clean, readable format (short summary + table when useful).
6. Use LIMIT on large result sets and mention when results are truncated.
```

---

## 🗄️ Schema Discovery Query

Use this as the query behind the schema-checking tool so the agent can see every table and column:

```sql
SELECT
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;
```

You can generate a variation with this prompt:
> *Help me with the SQL query that can retrieve the schema details from the Postgres DB.*

---

## 💬 Example Questions

| You ask | The agent does |
|---|---|
| "Show me sales from Mumbai last month" | Finds the sales and city columns, filters by date range, returns total and breakdown |
| "Top 5 customers by revenue this year" | Joins customers and orders, aggregates, sorts, limits to 5 |
| "Which product category grew the most vs last quarter?" | Compares two periods and ranks categories by growth |
| "What was the weather in Delhi?" | Replies that no data is available and lists what it *can* answer |

---

## 🧪 Testing & UAT

- ✅ Ask simple lookups and verify the SQL and numbers manually
- ✅ Ask multi-table, multi-filter questions (joins, group by, date ranges)
- ✅ Ask vague questions and confirm it asks a follow-up
- ✅ Ask for data that doesn't exist and confirm the "no data" message
- ✅ Try follow-up questions to check that memory works
- ✅ Try destructive prompts (e.g., "delete all orders") and confirm they are refused

---

## 🛡️ Safety & Best Practices

- Use a **read-only** Postgres user so the agent can never modify data.
- Instruct the agent to generate **SELECT-only** queries in the system message.
- Add `LIMIT` to large queries to avoid heavy loads.
- Keep API keys and DB passwords in **n8n credentials**, never in the workflow JSON or this repo.
- Consider restricting the agent to specific schemas or views that contain only non-sensitive data.
- Review execution logs in the **Executions** tab to audit generated queries.

---

## 📂 Project Structure

```
ai-sql-analyst/
├── README.md
├── workflow/
│   └── sql_agent.json          # Exported n8n workflow
├── assets/
│   ├── workflow.png            # n8n canvas screenshot
│   └── chat_demo.png           # Chat test screenshot
├── prompts/
│   ├── system_message.txt
│   └── schema_query.sql
└── docs/
    └── setup_notes.md
```

---

## 🚀 Future Improvements

- 📈 Return charts and visualizations alongside text answers
- 🔐 Role-based access so different users see different tables
- 🧾 Log every question, query, and answer for auditing
- 🌐 Connect to Slack, Teams, or WhatsApp instead of the built-in chat
- 🧠 Add a vector store with column descriptions and business glossary for higher accuracy
- 🗃️ Support multiple databases (MySQL, BigQuery, Snowflake)
- ✅ Add a query validation step before execution

---

## 🎯 Skills Demonstrated

**AI & LLM**
- AI Agent design with tools, memory, and system prompts
- Natural language to SQL (text-to-SQL)
- Prompt engineering and fallback/clarification behavior
- Google Gemini integration

**Data & Databases**
- PostgreSQL querying and schema introspection (`information_schema`)
- Read-only access control and safe query design
- Translating business questions into analytical queries

**Automation**
- n8n workflow automation and agent orchestration
- Tool calling and credential management
- Testing and UAT of AI workflows

**Soft Skills**
- Problem decomposition and business-focused solution design
- Technical documentation



⭐ If you found this project useful, consider giving it a star!
