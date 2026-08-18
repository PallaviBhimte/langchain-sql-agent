# LangChain SQL Agent

A natural-language-to-SQL agent over the [Chinook](https://www.sqlitetutorial.net/sqlite-sample-database/)
sample database. Ask "which country generated the most revenue?" and the agent inspects
the schema, writes the SQL, validates it, runs it, and explains the result in English.

Built following the [LangChain SQL agent guide](https://docs.langchain.com/oss/python/langchain/sql-agent).
The notebook was originally written in Google Colab; this repo is set up to run it locally.

## Contents

| File | What it is |
| --- | --- |
| `Building_SQL_Agent_using_Langchain.ipynb` | The main walkthrough, building the agent step by step with commentary |
| `sql_app.py` | Streamlit app: multi-turn chat with history |
| `sql_agent_app.py` | Streamlit app: single question → answer |
| `Chinook.db` | SQLite sample database (digital media store) |
| `requirements.txt` | Pinned dependencies |

## How it works

`create_sql_agent` bundles the database into four tools and wires them to a tool-calling
model. Every question runs the same loop:

- **`sql_db_list_tables`**: the agent finds out which tables exist.
- **`sql_db_schema`**: it reads the columns and a few sample rows from the tables that
  look relevant, then drafts SQL against that schema.
- **`sql_db_query_checker`**: the draft goes back to the model for review before it runs.
- **`sql_db_query`**: the query executes and returns rows.
- The agent repeats any of those steps as needed, then answers in plain English.

`agent_type="openai-tools"` uses OpenAI function calling, so the model selects a tool
against a strict JSON schema rather than reasoning through free-form text the way the
older ReAct agent did: fewer malformed queries, and typed tool arguments.

The model is `gpt-4.1` at `temperature=0`. Swap it in `load_agent()` in either Streamlit
app, or in the model cell of the notebook.

## Setup

Requires Python 3.12 and an OpenAI API key.

```bash
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python -r requirements.txt
.venv/bin/python -m ipykernel install --user \
  --name langchain-sql-agent --display-name "Python (langchain-sql-agent)"
```

### API key

Add your OpenAI API key in the `.env` file in the format below:

```
OPENAI_API_KEY=sk-...
```

### Database

The notebook downloads `Chinook.db` on first run, or fetch it directly:

```bash
curl -o Chinook.db https://storage.googleapis.com/benchmarks-artifacts/chinook/Chinook.db
```

## Run the notebook

```bash
.venv/bin/python -m jupyter lab
```

## Run the Streamlit apps

```bash
.venv/bin/python -m streamlit run sql_app.py        # chat, keeps history
.venv/bin/python -m streamlit run sql_agent_app.py  # one question at a time
```

Both read `OPENAI_API_KEY` from the environment, falling back to an in-page password box if
it isn't set, and cache the agent with `@st.cache_resource` so it's built once per session.

## Example questions

- How many tracks are in the Chinook database?
- List 5 customers from India.
- Which genre has the longest tracks on average? Give the top 3.
- Which country generated the most revenue? Show the top 5 with totals.

## A note on safety

The default system prompt tells the agent not to run `INSERT`, `UPDATE`, `DELETE` or
`DROP`. If you ask it to delete a table, it declines before calling a single tool. This is a
prompt-level guardrail, not a database-level one. Connect as a read-only database user to
have the restriction enforced by the database itself.

## References

- [LangChain SQL agent guide](https://docs.langchain.com/oss/python/langchain/sql-agent)
- [LangChain agents](https://docs.langchain.com/oss/python/langchain/agents)
- [Chinook sample database](https://www.sqlitetutorial.net/sqlite-sample-database/)
