# 📬 Mail Agent (Human-in-the-Loop Email Assistant)

An intelligent, interactive email generation and review agent built with **LangGraph**, **LangChain**, and **Anthropic Claude**. 

Mail Agent translates natural language requests into structured email drafts and leverages **Human-in-the-Loop (HITL)** workflows via LangGraph interrupts, giving the user full oversight to approve or request revisions before sending.

---

## 🚀 Features

- **Entity & Intent Extraction**: Parses the user's natural language request into structured fields (`recipient_name`, `recipient_email`, `mail_reason`) using Pydantic schemas.
- **Context-Aware Drafting**: Generates professional email subjects (≤ 40 words) and concise bodies (≤ 200 words).
- **Human-in-the-Loop (HITL) Review**: Pauses execution using LangGraph `interrupt()` to allow user inspection and feedback.
- **Iterative Refinement**: Revises drafts dynamically according to user critique.
- **Safety Guardrail / Feedback Cap**: Limits revisions to a maximum of 3 feedback iterations (`feedback_count > 2`) before automatically canceling to avoid endless loops.
- **Thread Persistence**: Maintains execution checkpoints using `InMemorySaver` to enable smooth workflow pausing and resumption.

---

## 🔄 Workflow Architecture

The agent runs as a cyclical state machine powered by **LangGraph**:

```mermaid
flowchart TD
    START([Start]) --> retriever[Retriever Node<br/><i>Extracts recipient & reason</i>]
    retriever --> draft[Draft Node<br/><i>Generates or revises email</i>]
    draft --> review_node{Review Node<br/><i>interrupt() for user review</i>}
    
    review_node -->|"Approved ('yes')"| send[Send Node<br/><i>Dispatches email</i>]
    review_node -->|"Feedback provided (Count <= 2)"| draft
    review_node -->|"Feedback limit exceeded (> 2)"| cancel[Cancel Node<br/><i>Terminates without sending</i>]
    
    send --> END([End])
    cancel --> END([End])
```

### State & Execution Nodes

| Node | Description |
| :--- | :--- |
| **`retriever`** | Uses structured output (`UserDetails`) to extract recipient info and intent from the initial query. |
| **`draft`** | Generates a new draft or updates the existing draft using user-provided feedback. |
| **`review_node`** | Triggers a LangGraph interrupt, waiting for human approval (`yes`) or feedback. |
| **`router`** | Evaluates the review result and routes state to `send`, `draft` (retry), or `cancel`. |
| **`send`** | Simulates sending the finalized email and sets response status to `"Email Sent"`. |
| **`cancel`** | Halts the process if the maximum allowed feedback attempts are reached. |

---

## 📁 Project Structure

```text
MAIL-AGENT/
├── agent.py            # Main application script & LangGraph definition
├── pyproject.toml      # Project configuration and dependency definitions
├── requirements.txt    # Pip dependency requirements
├── uv.lock             # uv lockfile for deterministic dependency resolution
├── .env                # Environment configuration (API keys)
└── README.md           # Documentation
```

---

## 🛠️ Prerequisites & Installation

### 1. Prerequisites
- **Python 3.12+**
- Anthropic API Key

### 2. Clone the Repository
```bash
git clone https://github.com/VikasSingh7221/MAIL-AGENT.git
cd MAIL-AGENT
```

### 3. Set Up Virtual Environment

Using **uv** (recommended):
```bash
uv venv
source .venv/bin/activate
uv sync
```

Or using standard **pip**:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 4. Configure Environment Variables

Create a `.env` file in the root directory:
```env
ANTHROPIC_API_KEY="your-anthropic-api-key-here"
```

---

## 💻 Usage

Run the agent in terminal:

```bash
python agent.py
```

### Example Interaction

```text
User: Send an email to Alice at alice@example.com asking for project updates for Q3

------------------------------------------------------------
Subject:  Quick Catch-up: Q3 Project Updates
Body:  Hi Alice,

Hope you're having a productive week. Could you please share the latest status updates on our Q3 deliverables when you get a chance? 

Looking forward to hearing from you.

Best regards
------------------------------------------------------------
Approve to send the mail or provide the feedback: Make it sound more urgent and set a deadline for tomorrow 5 PM

------------------------------------------------------------
Subject:  URGENT: Q3 Project Updates Required by Tomorrow 5 PM
Body:  Hi Alice,

I hope this finds you well. We are finalizing our review and urgently need the latest updates on Q3 deliverables. 

Please ensure your updates are shared by tomorrow at 5:00 PM.

Thank you for your prompt attention to this matter.

Best regards
------------------------------------------------------------
Approve to send the mail or provide the feedback: yes

mail is sending: Alice URGENT: Q3 Project Updates Required by Tomorrow 5 PM ...
AI:  Email Sent
```

Type `quit` at the prompt to exit the loop.

---

## ⚙️ Configuration & Customization

- **Change LLM Model**: In [agent.py](file:///Users/bot/MAIL-AGENT/agent.py#L15), modify `ChatAnthropic(model_name='claude-sonnet-5')` or swap with another provider (e.g., `ChatOpenAI`, `ChatGroq`).
- **Adjust Feedback Limit**: In [agent.py](file:///Users/bot/MAIL-AGENT/agent.py#L115), adjust the threshold `if state.feedback_count > 2:` to allow more or fewer refinement rounds.
- **Email Delivery Integration**: Connect a real email transport (such as SMTP, SendGrid, Resend, or Gmail API) inside the [`send_node`](file:///Users/bot/MAIL-AGENT/agent.py#L127-L133).

---

## 📄 License

This project is licensed under the MIT License.