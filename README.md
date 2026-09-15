# BrightCart E-Commerce Order Support Assistant

A local NLP/conversational AI assignment project that implements an e-commerce order-support assistant using **FastAPI**, **WebSockets**, and **Ollama** with a local language model.

The assistant is designed to handle common customer-support conversations around products, orders, shipping, returns, refunds, exchanges, and cancellations while maintaining conversation state across turns.

## 1. Project Overview

**BrightCart** is an e-commerce order-support scenario in which the customer communicates with an AI assistant named **Ava**.

The system focuses on three main capabilities:

- Understanding the customer's support intent.
- Managing a multi-turn conversation through defined conversation stages.
- Generating responses using a locally hosted LLM through Ollama.

The application intentionally operates without external RAG or live order-management tools. Therefore, it does not claim to retrieve real-time order information or invent customer/order data.

## 2. Technology Stack

| Component | Technology |
|---|---|
| Backend | FastAPI |
| Communication | WebSocket |
| LLM Runtime | Ollama |
| Default LLM | `qwen2.5:1.5b-instruct-q4_K_M` |
| HTTP Client | HTTPX |
| Data Validation | Pydantic |
| Frontend | HTML, CSS, JavaScript |
| Testing | Pytest + Pytest-Asyncio |
| Server | Uvicorn |

> **Note:** Ollama is the local LLM runtime. The actual model configured by default is `qwen2.5:1.5b-instruct-q4_K_M`.

## 3. Features

### Domain Restriction

The assistant is restricted to e-commerce support topics:

1. Product questions
2. Order tracking and order status
3. Returns, refunds, exchanges, cancellations, and shipping policy

Off-topic requests are detected before being sent to the LLM and receive a controlled redirection response.

### Intent Detection

The conversation manager uses keyword-based intent detection to identify likely intents:

- `order_tracking`
- `returns_shipping`
- `product_question`

If a new message changes the active intent, the conversation stage can be updated accordingly.

### Conversation Stages

The assistant manages conversations through the following stages:

1. `greeting`
2. `intent_identification`
3. `info_gathering`
4. `resolution`
5. `confirmation`
6. `closing`

The current stage is communicated to the frontend during a response.

### Conversation Memory

Conversation history is maintained per session. A rolling summary mechanism is used when the conversation becomes long so that the prompt remains within a controlled context size.

The system also limits the active conversation window and estimates prompt size before sending the request to the local LLM.

### Streaming Responses

Responses are streamed from Ollama to the FastAPI backend and then to the browser over WebSockets.

This allows the user to see the assistant response as it is generated rather than waiting for the complete response.

### Session Management

The backend supports:

- Creating a new session
- Retrieving session information
- Resetting/deleting a session
- Maintaining separate conversation history for each session

### Latency Metrics

The LLM engine records:

- Time to First Token (TTFT)
- Total/evaluation time
- Output token count
- Tokens per second (TPS)

A benchmark script is included for measuring local model response performance.

## 4. Project Structure

```text
project/
│
├── app/
│   ├── config.py
│   ├── conversation_manager.py
│   ├── domain.py
│   ├── llm_engine.py
│   ├── main.py
│   └── schemas.py
│
├── benchmark/
│   └── benchmrk_latency.py
│
├── static/
│   ├── index.html
│   ├── style.css
│   └── app.js
│
├── tests/
│   ├── manual_stress_test.py
│   ├── test_api.py
│   └── test_conversation_manager.py
│
├── requirements.txt
├── pytest.ini
├── .gitignore
└── README.md
```

## 5. Requirements

Make sure the following are installed:

- Python 3.10+
- Ollama
- A compatible Ollama model

The project dependencies are listed in `requirements.txt`.

## 6. Installation

### Step 1 — Create a virtual environment

Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
```

Linux/macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Step 2 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 3 — Install and prepare Ollama

Install Ollama and make sure its local server is running.

Pull the default model:

```bash
ollama pull qwen2.5:1.5b-instruct-q4_K_M
```

You can verify that the model is available with:

```bash
ollama list
```

If a different model is required, set the `OLLAMA_MODEL` environment variable before starting the application.

## 7. Configuration

The main configuration is located in:

```text
app/config.py
```

Important settings include:

```text
OLLAMA_HOST
OLLAMA_MODEL
OLLAMA_TIMEOUT
MAX_TURNS_IN_WINDOW
MAX_PROMPT_TOKENS
SESSION_IDLE_TIMEOUT_SECONDS
```

The default Ollama endpoint is:

```text
http://localhost:11434
```

A different Ollama host can be configured using:

```text
OLLAMA_HOST
```

The model can be changed using:

```text
OLLAMA_MODEL
```

For example, on Windows PowerShell:

```powershell
$env:OLLAMA_MODEL="qwen2.5:1.5b-instruct-q4_K_M"
```

## 8. Running the Application

Start the FastAPI server from the project root:

```bash
uvicorn app.main:app --reload
```

The application is then available through the local server shown by Uvicorn.

Open the application in a browser and start a new conversation.

## 9. API / WebSocket Flow

The application follows this general flow:

```text
User
  │
  ▼
Web Frontend
  │
  │ WebSocket
  ▼
FastAPI
  │
  ▼
Conversation Manager
  │
  ├── Domain / Off-topic Check
  ├── Intent Detection
  ├── Stage Management
  ├── Conversation Memory
  └── Prompt Construction
          │
          ▼
      Ollama / Local LLM
          │
          ▼
     Streaming Response
          │
          ▼
       WebSocket
          │
          ▼
      Web Frontend
```

## 10. Domain Policies

The assistant is configured with the following BrightCart policies.

### Shipping

- Standard shipping: 3–5 business days
- Express shipping: 1–2 business days

### Returns

- Returns are accepted within 30 days after delivery.
- Items must be unused and in their original packaging.

### Refunds

- Refunds are issued to the original payment method.
- Processing takes approximately 5–7 business days after warehouse receipt.

### Exchanges

- Different sizes or colors can be exchanged free of charge.
- The customer pays return shipping if the original box was damaged by the customer.

### Cancellations

- Orders can be cancelled within one hour before shipping.

### Other Policies

- Gift cards are non-returnable.
- Final-sale clearance items are non-returnable.
- BrightCart does not offer price matching.

## 11. Testing

The project uses Pytest.

Run the complete test suite with:

```bash
pytest
```

The supplied tests cover areas including:

- API/session behavior
- WebSocket communication
- Malformed messages
- Off-topic requests
- Conversation-stage transitions
- Topic switching
- Conversation history
- Summarization behavior
- Session reset behavior

The project uses `pytest.ini` to automatically discover tests in the `tests/` directory and enable automatic asyncio handling.

## 12. Stress Testing

A separate manual stress/concurrency test is included:

```text
tests/manual_stress_test.py
```

It exercises scenarios such as:

- Multiple simultaneous users
- Malformed WebSocket input
- Unknown sessions
- Mid-stream client disconnection

Run it according to the script's available test entry points.

## 13. Benchmarking

The latency benchmark is located in:

```text
benchmark/benchmrk_latency.py
```

It sends representative e-commerce support prompts to the local Ollama model and records response-performance metrics.

The benchmark measures:

- TTFT
- Output tokens
- Tokens per second
- Model evaluation/response time

The benchmark is intended to provide a basic performance view of the selected local model.

## 14. Example Conversations

### Order Tracking

```text
Customer: Where is my order?

Ava: I'd be happy to help track it. Could you provide your order number?

Customer: BC-1234

Ava: Thanks. I can help with the order, but I don't have access to live order records. Please use the order-tracking information provided by BrightCart for the current shipment status.
```

### Returns

```text
Customer: Can I return an item?

Ava: Yes. BrightCart accepts returns within 30 days after delivery, provided the item is unused and in its original packaging.
```

### Off-topic Request

```text
Customer: What's the weather today?

Ava: I’m here to help with BrightCart products, orders, shipping, returns, refunds, and exchanges. What can I help you with?
```

## 15. Limitations

The current implementation has several intentional limitations:

- It does not access a real order database.
- It does not perform live shipment tracking.
- It does not use RAG.
- It does not use external customer/account tools.
- Product information is restricted to information available to the configured assistant.
- Intent detection is currently rule/keyword based.
- The LLM is locally hosted, so response speed depends on the user's hardware and selected model.
- Latency benchmark results can vary depending on whether the model is already loaded in memory.

## 16. Security / Privacy

The application is designed for local execution.

No customer conversation needs to be sent to a cloud LLM provider because generation is performed through the locally running Ollama service.

For an actual production deployment, authentication, authorization, secure WebSocket configuration, persistent storage, rate limiting, and stricter CORS configuration would be required.

## 17. Assignment Scope

This project demonstrates a conversational NLP application for an e-commerce customer-support domain. The implementation combines:

- Natural-language intent handling
- Domain restriction
- Multi-turn conversation management
- Conversation memory
- Local LLM generation
- Streaming communication
- Session management
- Automated testing
- Basic latency benchmarking

The project is intended as an academic prototype rather than a production-ready e-commerce support platform.

## 18. Quick Start

For a quick setup:

```bash
python -m venv .venv
```

Activate the environment, then:

```bash
pip install -r requirements.txt
```

Make sure Ollama is running and the model is available:

```bash
ollama pull qwen2.5:1.5b-instruct-q4_K_M
```

Run the tests:

```bash
pytest
```

Start the application:

```bash
uvicorn app.main:app --reload
```

Then open the local application in your browser.

---

**BrightCart — Local E-Commerce Order Support Assistant**
