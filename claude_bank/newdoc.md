# BankX Multi-Agent Banking Platform — Code Flow Documentation

> **Last Updated**: February 24, 2026  
> **Platform**: Azure AI Foundry + MCP + A2A Protocol  
> **Currency**: Thai Baht (THB) | **Customers**: 10 | **Agents**: 7 (1 Supervisor + 6 Specialists)

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [System Components](#2-system-components)
3. [End-to-End Request Lifecycle](#3-end-to-end-request-lifecycle)
4. [Use Case 1 — Financial Operations (Account, Transaction, Payment)](#4-use-case-1--financial-operations)
5. [Use Case 2 — Product Information & FAQ](#5-use-case-2--product-information--faq)
6. [Use Case 3 — AI Money Coach (Personal Finance Advisory)](#6-use-case-3--ai-money-coach)
7. [Use Case 4 — Escalation & Support Tickets](#7-use-case-4--escalation--support-tickets)
8. [Cache System](#8-cache-system)
9. [Conversation Continuity & Multi-Turn Flow](#9-conversation-continuity--multi-turn-flow)
10. [Authentication & User Mapping](#10-authentication--user-mapping)
11. [Startup Order & Ports](#11-startup-order--ports)

---

## 1. Architecture Overview

BankX is a **multi-agent conversational banking system** where a central **Supervisor Agent** routes user requests to **6 specialist agents** via the **Agent-to-Agent (A2A) protocol**. Each specialist agent connects to one or more **MCP (Model Context Protocol) microservices** for data access and business logic.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        FRONTEND (React + MSAL)                      │
│                        Port 8081                                    │
│  ┌─────────┐  ┌──────────────┐  ┌────────────┐  ┌──────────────┐  │
│  │ Chat UI │  │ Thinking     │  │ Agent Map  │  │ Confirmation │  │
│  │         │  │ Panel (SSE)  │  │ Dashboard  │  │ Dialogs      │  │
│  └────┬────┘  └──────────────┘  └────────────┘  └──────────────┘  │
│       │                                                             │
└───────┼─────────────────────────────────────────────────────────────┘
        │ POST /api/chat (JWT Bearer + SSE response)
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    COPILOT BACKEND (FastAPI)                         │
│                    Port 8080                                        │
│  ┌────────────┐  ┌───────────────┐  ┌──────────────────────────┐   │
│  │ Auth/JWT   │  │ Cache Manager │  │ Conversation State Mgr   │   │
│  │ Validator  │  │ (JSON files)  │  │ (5-min TTL per customer) │   │
│  └────────────┘  └───────────────┘  └──────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              SUPERVISOR AGENT (A2A Router)                    │   │
│  │  Keyword Scoring → LLM Classification → Agent Selection      │   │
│  │  Cache Check → A2A HTTP POST → SSE Stream Response           │   │
│  └──────────┬───────────┬───────────┬───────────┬───────────┬───┘   │
└─────────────┼───────────┼───────────┼───────────┼───────────┼───────┘
              │           │           │           │           │
    ┌─────────▼──┐ ┌──────▼─────┐ ┌──▼────────┐ ┌▼─────────┐ ┌▼──────────┐
    │  Account   │ │Transaction │ │  Payment  │ │ ProdInfo │ │ AI Money  │
    │  Agent     │ │  Agent     │ │  Agent    │ │ FAQ Agent│ │ Coach     │
    │  :9001     │ │  :9002     │ │  :9003    │ │  :9004   │ │  :9005    │
    └─────┬──────┘ └─────┬──────┘ └────┬──────┘ └────┬─────┘ └─────┬────┘
          │              │             │              │             │
    ┌─────▼──────┐ ┌─────▼──────┐ ┌───▼────────┐    │             │
    │Account MCP │ │Trans. MCP  │ │Payment MCP │    │             │
    │  :8070     │ │  :8071     │ │  :8072     │    ▼             ▼
    ├────────────┤ └────────────┘ ├────────────┤  Azure AI      Azure AI
    │Limits MCP  │               │Contacts MCP│  Foundry       Foundry
    │  :8073     │               │  :8074     │  file_search   file_search
    └────────────┘               └────────────┘  (Product Docs) (Book RAG)
                                                      │             │
                                                      ▼             ▼
                                                 ┌──────────────────────┐
                                                 │  Escalation Agent    │
                                                 │  :9006               │
                                                 │  ┌────────────────┐  │
                                                 │  │Escalation MCP  │  │
                                                 │  │  :8078         │  │
                                                 │  │(Tickets+Email) │  │
                                                 │  └────────────────┘  │
                                                 └──────────────────────┘
```

---

## 2. System Components

### 2.1 MCP Microservices (Data Layer)

Each MCP server is a **FastMCP** service exposing tools over the Model Context Protocol.

| Service | Port | Tools Exposed | Data Source |
|---------|------|---------------|-------------|
| **Account MCP** | 8070 | `getAccountsByUserName`, `getAccountDetails`, `getPaymentMethodDetails` | `accounts.json`, `customers.json` |
| **Transaction MCP** | 8071 | `getLastTransactions`, `searchTransactions`, `getTransactionDetails`, `aggregateTransactions` | `transactions.json` |
| **Payment MCP** | 8072 | `processPayment` | Calls Transaction MCP internally |
| **Limits MCP** | 8073 | `checkLimits`, `getAccountLimits`, `updateLimitsAfterTransaction` | `limits.json` |
| **Contacts MCP** | 8074 | `getRegisteredBeneficiaries`, `verifyAccountNumber`, `addBeneficiary`, `removeBeneficiary` | `contacts.json` |
| **ProdInfo FAQ MCP** | 8076 | `search_documents`, `get_document_by_id`, `get_content_understanding` | Azure AI Search (vector) |
| **AI Money Coach MCP** | 8077 | `ai_search_rag_results`, `ai_foundry_content_understanding` | Azure AI Search (book index) |
| **Escalation Comms MCP** | 8078 | `create_ticket`, `get_tickets`, `update_ticket`, `close_ticket`, `send_email` | CosmosDB + Azure Comms Services |

### 2.2 A2A Specialist Agents

Each agent is a standalone FastAPI service implementing the **A2A Protocol** (`.well-known/agent.json` discovery + `/a2a/invoke` endpoint).

| Agent | Port | MCP Connections | Framework | Model |
|-------|------|-----------------|-----------|-------|
| **Account Agent** | 9001 | Account + Limits | `AzureAIClient` | gpt-5-mini |
| **Transaction Agent** | 9002 | Account + Transaction | `AzureAIClient` | gpt-5-mini |
| **Payment Agent** | 9003 | Account + Transaction + Payment + Contacts | `AzureAIClient` | gpt-5-mini |
| **ProdInfo FAQ Agent** | 9004 | None (uses Foundry `file_search`) | `AzureAIAgentClient` | gpt-5-mini |
| **AI Money Coach Agent** | 9005 | None (uses Foundry `file_search`) | `AzureAIAgentClient` | gpt-5-mini |
| **Escalation Agent** | 9006 | Escalation Comms | `AzureAIAgentClient` | gpt-5-mini |

### 2.3 Two Agent Framework Patterns

1. **MCP-Backed Agents** (Account, Transaction, Payment): Use `AzureAIClient` + `AuditedMCPTool` — tools are dynamically connected from external MCP servers at runtime.
2. **File-Search Agents** (ProdInfo, AI Coach, Escalation): Use `AzureAIAgentClient` — loads a pre-configured Azure AI Foundry agent with `file_search` capabilities over uploaded document collections.

---

## 3. End-to-End Request Lifecycle

Every user message follows this journey from browser to response:

### Step 1: Frontend → Copilot Backend

```
User types message in React Chat UI
  → Frontend sends POST /api/chat with:
     • Authorization: Bearer <MSAL JWT token>
     • Body: { messages: [...], stream: true, approach: "chat" }
  → Response: Server-Sent Events (SSE) stream
```

### Step 2: Authentication & Cache Check

```
Copilot receives POST /api/chat
  → get_current_user() dependency:
     1. Extract Bearer token from header
     2. Validate JWT against Azure AD (tenant + audience check)
     3. Extract claims: email, oid (user ID), name
     4. Map email → customer_id via customers.json
        (e.g., anan@bankxthb.onmicrosoft.com → CUST-004)
     5. Auto-trigger background cache initialization if expired/missing
  → Returns UserContext { customer_id, email, name }
```

### Step 3: Continuation Check

```
_stream_response() starts:
  → is_continuation_message(message)?
     Checks for: "yes", "confirm", "ok", "cancel", "option 1", etc.
     
     YES + active agent found for customer_id:
       → Route DIRECTLY to that agent (skip Supervisor entirely)
       → Saves ~12 seconds per follow-up
     
     NO or no active agent:
       → Proceed to Supervisor routing
```

### Step 4: Supervisor Routing (Hybrid Two-Phase)

```
SupervisorAgentA2A.processMessageStream():

  Phase A — Cache Check:
    → Skip cache for: write ops (pay/transfer), UC2, UC3
    → Check _try_cache_response() with JSON file
    → HIT: Stream response from cache → done (sub-second)
    → MISS: Continue to agent selection

  Phase B — Keyword Confidence Scoring (instant):
    → Score each agent based on keyword match count
    → Payment: "transfer", "pay", "send money", currency patterns
    → Transaction: "transaction", "history", "spent", "spending"
    → Product Info: "interest rate", "loan", "credit card", "product"
    → AI Coach: "budget", "debt", "avalanche", "snowball", "invest"
    → Account: "balance", "account", "detail" (default fallback)
    → Escalation: "ticket", "escalate", "complaint"
    
    If max_score >= 2 with single winner → HIGH confidence → agent selected
    
  Phase C — LLM Classification (1-2 second fallback):
    → gpt-4o-mini with temperature=0.0, max_tokens=20
    → Classifies into exactly one agent name
    → Default: "Account Agent"
```

### Step 5: A2A Agent Call

```
Supervisor routes to selected agent:
  → HTTP POST http://localhost:{port}/a2a/invoke
  → Body: {
      messages: [{role: "user", content: "..."}],
      thread_id: "thread_CUST-004",
      customer_id: "CUST-004",
      user_email: "anan@bankxthb.onmicrosoft.com",
      stream: false
    }
  → Timeout: 90 seconds
  → Agent processes with Azure AI Foundry + MCP tools
  → Returns: { messages: [...], thread_id, agent, version }
```

### Step 6: SSE Response Streaming

```
Copilot streams response back to frontend as SSE events:

  Event 1: {"type":"thinking","step":"checking_cache","status":"in_progress"}
  Event 2: {"type":"thinking","step":"routing","agent_name":"Account Agent"}
  Event 3: {"choices":[{"delta":{"content":"Your "}}]}     ← word-by-word
  Event 4: {"choices":[{"delta":{"content":"balance "}}]}
  ...
  Final:   {"choices":[{"delta":{"content":"..."},"message":{"content":"full response"}}],"threadId":"..."}

Frontend renders:
  → ThinkingPanel shows step-by-step agent routing
  → Chat bubble fills in word-by-word (typing effect)
  → Agent Map highlights active agent
```

---

## 4. Use Case 1 — Financial Operations

UC1 covers three sub-use-cases: **Account Inquiry**, **Transaction History**, and **Payment Transfer**.

### 4.1 Account Balance Inquiry

**Example**: *"What is my account balance?"*

```
┌──────────┐     ┌──────────┐     ┌──────────────┐
│ Frontend │────▶│ Copilot  │────▶│ Cache Check   │
└──────────┘     └──────────┘     └──────┬───────┘
                                         │
                          ┌──────────────┤
                          │ CACHE HIT    │ CACHE MISS
                          ▼              ▼
                   Return cached    ┌──────────────┐
                   balance          │Account Agent │
                   (sub-second)     │  :9001       │
                                    └──────┬───────┘
                                           │
                                    ┌──────▼───────┐
                                    │ Account MCP  │
                                    │  :8070       │
                                    │              │
                                    │ Tool called: │
                                    │ getAccounts  │
                                    │ ByUserName() │
                                    └──────────────┘
```

**Cache-Hit Path (fast, sub-second)**:
1. Supervisor receives "What is my account balance?"
2. `_try_cache_response()` loads `memory/CUST-004.json`
3. Finds `data.balance = 113400.0`
4. Returns: *"Your current account balance is **113,400.00 THB** for account 123-456-004"*
5. **No agent call made** — response is instant

**Cache-Miss Path (10-30 seconds)**:
1. Supervisor routes to Account Agent via keyword match ("balance" → Account Agent)
2. Account Agent calls `getAccountsByUserName(anan@bankxthb.onmicrosoft.com)` via Account MCP
3. Returns account details with balance
4. Response streamed back through Supervisor → Frontend

**Example Response**:
```
Your current account balance is **113,400.00 THB**.

Account Details:
- Account Number: 123-456-004
- Account Holder: Anan Chaiyaporn
- Account Type: Checking
```

---

### 4.2 Transaction History

**Example**: *"Show me my last 5 transactions"*

```
┌──────────┐     ┌──────────┐     ┌─────────────────┐     ┌──────────────┐
│ Frontend │────▶│ Copilot  │────▶│ Supervisor      │────▶│ Transaction  │
└──────────┘     └──────────┘     │ Keywords:       │     │ Agent :9002  │
                                  │ "transaction"=1 │     └──────┬───────┘
                                  │ "last"=1        │            │
                                  │ Score: 2 ✓      │     ┌──────▼───────┐
                                  └─────────────────┘     │ Account MCP  │
                                                          │ :8070        │
                                                          │ → getAccounts│
                                                          │   ByUserName │
                                                          └──────┬───────┘
                                                                 │
                                                          ┌──────▼───────┐
                                                          │Trans. MCP    │
                                                          │ :8071        │
                                                          │ → getLast    │
                                                          │   Transactions│
                                                          └──────────────┘
```

**Flow**:
1. Supervisor scores "last 5 transactions" → Transaction Agent (keywords: "transaction" + "last" → score 2)
2. Transaction Agent calls Account MCP for account lookup, then Transaction MCP for history
3. Response formatted as HTML `<table>` with columns: Date, Description, Amount, Type

**Cache-Hit path**: If asking for "last transactions" and cache has `last_5_transactions`, the Supervisor returns the cached table directly.

**Example Response**:
```html
Here are your last 5 transactions:

| Date       | Description              | Amount (THB) | Type     |
|------------|--------------------------|-------------|----------|
| 2026-02-23 | 📤 Transfer to Somchai   | -300.00     | Transfer |
| 2026-02-22 | 📥 Salary Deposit        | +45,000.00  | Income   |
| 2026-02-20 | 📤 Electric Bill Payment | -1,250.00   | Payment  |
| 2026-02-18 | 📤 Transfer to Nattaporn | -5,000.00   | Transfer |
| 2026-02-15 | 📥 Refund - Online Shop  | +890.00     | Refund   |
```

---

### 4.3 Payment Transfer (Multi-Turn with Confirmation)

**Example**: *"Transfer 300 THB to Somchai Rattanakorn"*

This is the most complex flow — it spans **multiple turns** with human-in-the-loop confirmation.

```
┌──────────────────────────────────────────────────────────────────┐
│                     PAYMENT FLOW (3 Turns)                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Turn 1: "Transfer 300 THB to Somchai Rattanakorn"              │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐               │
│  │ Frontend │───▶│Supervisor│───▶│Payment Agent │               │
│  └──────────┘    └──────────┘    │  :9003       │               │
│                                  └──────┬───────┘               │
│                                         │                        │
│              ┌──────────────────────────┤                        │
│              │                          │                        │
│       ┌──────▼──────┐           ┌──────▼───────┐                │
│       │Account MCP  │           │Contacts MCP  │                │
│       │:8070        │           │:8074         │                │
│       │getAccounts  │           │getBenefici-  │                │
│       │ByUserName() │           │aries()       │                │
│       └─────────────┘           └──────────────┘                │
│                                                                  │
│  Agent Response (Turn 1):                                        │
│  ┌────────────────────────────────────────────────────────┐      │
│  │ ⚠️ PAYMENT CONFIRMATION REQUIRED ⚠️                    │      │
│  │                                                        │      │
│  │ Amount:           300.00 THB                           │      │
│  │ Recipient:        Somchai Rattanakorn                  │      │
│  │ Account:          123-456-001                          │      │
│  │ Payment Method:   Bank Transfer                        │      │
│  │ Current Balance:  113,400.00 THB                       │      │
│  │ New Balance:      113,100.00 THB                       │      │
│  │                                                        │      │
│  │ Reply 'Yes' or 'Confirm' to proceed.                   │      │
│  └────────────────────────────────────────────────────────┘      │
│                                                                  │
│  Frontend renders: [Confirm] [Cancel] buttons                    │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Turn 2: User clicks [Confirm] → sends "Yes, confirm"          │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐               │
│  │ Frontend │───▶│Copilot   │───▶│Payment Agent │               │
│  └──────────┘    │CONTINUA- │    │  :9003       │               │
│                  │TION      │    │ (same thread)│               │
│                  │DETECTED! │    └──────┬───────┘               │
│                  └──────────┘           │                        │
│                                  ┌──────▼───────┐                │
│                                  │Payment MCP   │                │
│                                  │:8072         │                │
│                                  │processPayment│                │
│                                  │()            │                │
│                                  └──────────────┘                │
│                                                                  │
│  Agent Response (Turn 2):                                        │
│  ┌────────────────────────────────────────────────────────┐      │
│  │ ✅ Transfer completed successfully!                     │      │
│  │                                                        │      │
│  │ Transaction ID: T000139                                │      │
│  │ From: Anan Chaiyaporn (CHK-004)                        │      │
│  │ To: Somchai Rattanakorn (123-456-001)                  │      │
│  │ Amount: 300.00 THB                                     │      │
│  │                                                        │      │
│  │ Your new balance: 113,100.00 THB                       │      │
│  │ Daily limit remaining: 199,700.00 THB                  │      │
│  └────────────────────────────────────────────────────────┘      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Detailed Payment Flow**:

| Step | Action | Tool Called | MCP Server |
|------|--------|-----------|------------|
| 1 | Look up sender's account | `getAccountsByUserName(email)` | Account MCP :8070 |
| 2 | Find recipient in beneficiaries | `getRegisteredBeneficiaries(account_id)` | Contacts MCP :8074 |
| 3 | Validate transfer limits | `checkLimits(account_id, amount)` | Limits MCP :8073 |
| 4 | **STOP** — Show confirmation table | *None — agent responds with table* | — |
| 5 | *User confirms ("Yes")* | — | — |
| 6 | Execute payment | `processPayment(account_id, amount, ...)` | Payment MCP :8072 |
| 7 | Invalidate cache | Cache file deleted | Local filesystem |
| 8 | Get updated balance | `getAccountDetails(account_id)` | Account MCP :8070 |

**Continuation Mechanism (Turn 2)**:
- User sends "Yes, confirm the payment"
- Copilot's `is_continuation_message()` detects "yes" + "confirm" → **continuation**
- `ConversationStateManager.get_active_agent("CUST-004")` → finds `Payment Agent`
- **Bypasses Supervisor entirely** — routes directly to Payment Agent at `:9003`
- Payment Agent resumes existing thread (session state preserved in `thread_store`)
- Executes the payment and returns success

**Business Rules Enforced**:
- Per-transaction limit: **50,000 THB**
- Daily limit: **200,000 THB**
- Recipient must be a registered beneficiary OR verified via `verifyAccountNumber` (3 attempts max)
- Idempotent: Uses request IDs to prevent duplicate payments

---

## 5. Use Case 2 — Product Information & FAQ

**Example**: *"What is the minimum deposit for a savings account?"*

```
┌──────────┐     ┌──────────┐     ┌─────────────────┐     ┌──────────────┐
│ Frontend │────▶│ Copilot  │────▶│ Supervisor      │────▶│ ProdInfo FAQ │
└──────────┘     └──────────┘     │ Keywords:       │     │ Agent :9004  │
                                  │ "savings"=1     │     └──────┬───────┘
                                  │ "account"=1     │            │
                                  │ "deposit"=1     │     ┌──────▼───────┐
                                  │ Score: 3 ✓      │     │ Azure AI     │
                                  └─────────────────┘     │ Foundry      │
                                                          │ file_search  │
                                                          │              │
                                                          │ Searches:    │
                                                          │ • Current Acc│
                                                          │ • Savings Acc│
                                                          │ • Fixed Dep. │
                                                          │ • Banking FAQ│
                                                          └──────────────┘
```

**How It Works**:
1. Supervisor routes to ProdInfo FAQ Agent (keyword match on product terms)
2. ProdInfo Agent uses **Azure AI Foundry's native `file_search`** — NOT an MCP server
3. `file_search` searches a vector store of uploaded product documents:
   - Current Account Product Sheet
   - Savings Account Product Sheet
   - Fixed Deposit (TD Bonus 24/36 month) Product Sheet
   - Banking FAQ Document
4. Agent returns answer grounded in the document content

**Three Response Scenarios**:

| Scenario | Condition | Response |
|----------|-----------|----------|
| **Answer Found** | Document contains the information | Direct answer citing the product document |
| **Not in Knowledge Base** | No relevant document found | *"I don't have that information. Would you like me to create a support ticket?"* |
| **Irrelevant Question** | Question unrelated to banking products | *"I can only help with BankX product information."* |

**Escalation to Ticket (Multi-Turn)**:
```
User: "What is the interest rate for student loans?"
Agent: "I don't have information about student loans. Would you like me to create a support ticket?"

User: "Yes, create a ticket"
  → Continuation detected → routes back to ProdInfo Agent
  → Agent calls create_support_ticket() function
  → Function internally calls Escalation Agent at :9006 via A2A
  → Escalation Agent calls Escalation MCP → creates ticket + sends email
  → Returns: "✅ Support ticket #TKT-12345 created. You'll receive a confirmation email."
```

---

## 6. Use Case 3 — AI Money Coach

**Example**: *"What is the debt avalanche method?"*

```
┌──────────┐     ┌──────────┐     ┌─────────────────┐     ┌──────────────┐
│ Frontend │────▶│ Copilot  │────▶│ Supervisor      │────▶│ AI Money     │
└──────────┘     └──────────┘     │ Keywords:       │     │ Coach :9005  │
                                  │ "debt"=1        │     └──────┬───────┘
                                  │ "avalanche"=1   │            │
                                  │ Score: 2 ✓      │     ┌──────▼───────┐
                                  └─────────────────┘     │ Azure AI     │
                                                          │ Foundry      │
                                                          │ file_search  │
                                                          │              │
                                                          │ Book:        │
                                                          │ "Debt-Free   │
                                                          │  to Financial│
                                                          │  Freedom"    │
                                                          │ (12 chapters)│
                                                          └──────────────┘
```

**How It Works**:
1. Supervisor routes via keyword match ("debt" + "avalanche" → AI Money Coach)
2. AI Money Coach Agent uses **Azure AI Foundry `file_search`** over the book "Debt-Free to Financial Freedom"
3. Answers are **strictly grounded** in the book content — the agent will NOT give generic financial advice
4. Default answers are 2-3 lines (concise); longer explanations only if user explicitly requests

**Same Three Scenarios as UC2**:
- Answer found in book → provide grounded answer
- Not in book → offer support ticket
- Irrelevant → decline

**Grounding Validation**: The MCP server applies a 4.0/5.0 strict threshold using Azure AI Foundry Evaluation SDK to ensure responses are 100% grounded in the book text.

**Example Response**:
```
The **debt avalanche method** focuses on paying off debts with the highest 
interest rate first while making minimum payments on all other debts. 

According to the book, this method saves you the most money in interest 
over time, though it may take longer to see debts fully eliminated 
compared to the snowball method.
```

---

## 7. Use Case 4 — Escalation & Support Tickets

UC4 is triggered in two ways:
1. **Indirectly** — from ProdInfo (UC2) or AI Money Coach (UC3) when they can't answer a question
2. **Directly** — when user explicitly asks for a ticket or escalation

### 7.1 Indirect Escalation (from UC2/UC3)

```
┌──────────────────────────────────────────────────────────────────────┐
│  Turn 1: User asks ProdInfo about something not in knowledge base    │
│                                                                      │
│  ProdInfo Agent :9004                                                │
│  └──▶ "I don't have that info. Would you like a support ticket?"     │
│                                                                      │
│  Turn 2: User says "Yes, create a ticket"                           │
│                                                                      │
│  Copilot (continuation) ──▶ ProdInfo Agent :9004                     │
│  └──▶ ProdInfo internal: create_support_ticket()                     │
│       └──▶ HTTP POST http://localhost:9006/a2a/invoke                │
│            └──▶ Escalation Agent :9006                                │
│                 └──▶ Escalation MCP :8078                            │
│                      ├── create_ticket()                             │
│                      └── send_ticket_notification()                   │
│                           └──▶ Azure Communication Services          │
│                                (Email to customer + CC: support)      │
│                                                                      │
│  Response: "✅ Ticket #TKT-12345 created! Check your email."         │
└──────────────────────────────────────────────────────────────────────┘
```

### 7.2 Direct Escalation

**Example**: *"I want to escalate this issue"*

```
┌──────────┐     ┌──────────┐     ┌──────────────┐     ┌──────────────┐
│ Frontend │────▶│Supervisor│────▶│ Escalation   │────▶│Escalation    │
└──────────┘     └──────────┘     │ Agent :9006  │     │Comms MCP     │
                 Keywords:        └──────────────┘     │:8078         │
                 "escalate"=1                          │              │
                 "issue"=1                             │create_ticket │
                 Score: 2 ✓                            │send_email    │
                                                       └──────────────┘
```

**Escalation MCP Tools**:

| Tool | Description |
|------|-------------|
| `create_ticket` | Creates support ticket in CosmosDB + auto-sends confirmation email |
| `get_tickets` | List tickets for a customer by status |
| `get_ticket_details` | Full ticket detail including update history |
| `update_ticket` | Update status/priority/notes |
| `close_ticket` | Close a resolved ticket |
| `send_email` | Send email via Azure Communication Services |

**Email Flow**: When a ticket is created, the Escalation MCP automatically sends a formatted HTML email to the customer with ticket details, and CC's the support team.

---

## 8. Cache System

The cache provides **sub-second responses** for common queries (balance, transactions, account details, limits) without calling any agent.

### 8.1 Cache Architecture

```
┌──────────────────────────────────────────────────┐
│              UserCacheManager                      │
│                                                    │
│  File: memory/{CUSTOMER_ID}.json                   │
│  TTL:  300 seconds (5 minutes)                     │
│  Auto-cleanup: Files > 1 hour old deleted          │
│  Write: Atomic (write .tmp → rename)               │
│  Locking: msvcrt (Windows) / fcntl (Linux)         │
│  In-flight tracking: _initializing set             │
│                                                    │
│  Triggers:                                         │
│  1. Auto on auth (background task)                 │
│  2. Explicit POST /cache/initialize                │
│  3. On-demand if missed at query time              │
└──────────────────────────────────────────────────┘
```

### 8.2 Cache File Structure

```json
{
  "customer_id": "CUST-004",
  "cached_at": "2026-02-24T02:57:31",
  "ttl_seconds": 300,
  "data": {
    "accounts": [
      {
        "account_id": "CHK-004",
        "account_no": "123-456-004",
        "cust_name": "Anan Chaiyaporn",
        "available_balance": 113400.0
      }
    ],
    "balance": 113400.0,
    "customer_name": "Anan Chaiyaporn",
    "last_5_transactions": [
      {
        "txn_id": "T000138",
        "date": "2026-02-23",
        "description": "Transfer to Somchai",
        "amount": -300.0,
        "type": "transfer"
      }
    ],
    "beneficiaries": [
      {
        "name": "Somchai Rattanakorn",
        "account_no": "123-456-001",
        "alias": "Somchai"
      }
    ],
    "limits": {
      "per_transaction_limit": 50000,
      "daily_limit": 200000,
      "remaining_today": 199700
    }
  }
}
```

### 8.3 Cache Initialization Flow

```
get_current_user() middleware fires on every authenticated request
  │
  ├── is_cache_valid_for_customer(CUST-004)?
  │     ├── customer in _initializing set? → YES → skip (prevents duplicates)
  │     └── File exists + TTL valid? → YES → skip
  │
  └── NO → asyncio.create_task(initialize_user_cache(...))
            │
            ├── Step 1: Account MCP → getAccountsByUserName()
            │   → Gets primary account_id, balance, customer_name
            │
            └── Step 2: Parallel fetch via asyncio.gather():
                ├── Transaction MCP → getLastTransactions(account_id, limit=5)
                ├── Contacts MCP → getRegisteredBeneficiaries(account_id)
                └── Limits MCP → getAccountLimits(account_id)
                
                → Write JSON to memory/{CUST-004}.json (atomic)
```

### 8.4 In-Flight Wait Mechanism

If a chat request arrives while cache is still being initialized:

```python
# get_cached_data() waits up to 25 seconds if init is in-flight
if customer_id in self._initializing:
    for _ in range(50):           # 50 × 0.5s = 25s max wait
        await asyncio.sleep(0.5)
        if customer_id not in self._initializing:
            break                  # Init finished! Read the file now
```

### 8.5 What Gets Answered from Cache (No Agent Call)

| Query Type | Cache Field Used | Example Query |
|------------|-----------------|---------------|
| Balance | `data.balance` | "What is my balance?" |
| Account details | `data.accounts`, `data.customer_name` | "Show my account info" |
| Last transactions | `data.last_5_transactions` | "What are my recent transactions?" |
| Account limits | `data.limits` | "What are my transfer limits?" |
| **NOT cached** | — | Payments, product info, financial advice, date-filtered transactions |

---

## 9. Conversation Continuity & Multi-Turn Flow

Two systems work together for multi-turn conversations:

### 9.1 ConversationStateManager (In-Memory, Routing)

```
Purpose: Skip Supervisor re-routing for follow-up messages
Key:     customer_id (not thread_id — handles frontend's varying thread_ids)
TTL:     5 minutes
Storage: In-memory dictionary

After routing to an agent:
  state_manager.update_state(customer_id, agent_name, a2a_url, thread_id)

On next message:
  if is_continuation_message(msg):        # "yes", "confirm", "cancel", etc.
    agent = state_manager.get_active_agent(customer_id)
    if agent and not expired:
      → Route DIRECTLY to agent (bypass Supervisor)     # Saves ~12s
```

**Continuation keywords detected**:
- Confirmations: `"yes"`, `"yeah"`, `"yep"`, `"ok"`, `"confirm"`, `"proceed"`, `"go ahead"`, `"approve"`
- Negations: `"no"`, `"cancel"`, `"stop"`, `"abort"` (also treated as continuations)
- Option selections: `"option 1"`, `"choice A"`, short messages < 20 characters

### 9.2 Payment Agent Thread Persistence

The Payment Agent maintains **per-thread session state** for multi-turn payment flows:

```python
class PaymentAgentHandler:
    thread_store: dict = {}    # Class-level dictionary
    
    # Turn 1: Create new session
    session = agent.create_session(session_id=thread_id)
    # ... agent processes, calls MCP tools ...
    thread_store[thread_id] = session.to_dict()   # Save state
    
    # Turn 2: Resume session
    session = AgentSession.from_dict(thread_store[thread_id])
    # Agent remembers: previous tools called, confirmation table shown
    # Now can proceed with executeTransfer
```

### 9.3 Full Multi-Turn Payment Example

```
Message 1: "Transfer 300 THB to Somchai"
  → Supervisor: keyword "transfer" → Payment Agent
  → Payment Agent: New session created
    → Calls: getAccountsByUserName, getRegisteredBeneficiaries, checkLimits
    → Returns confirmation table
  → State saved: { customer: CUST-004, agent: Payment Agent, thread: thread_CUST-004 }

Message 2: "Yes, confirm the payment" 
  → Copilot: is_continuation("yes, confirm") = TRUE
  → state_manager.get_active_agent("CUST-004") = Payment Agent ✅
  → Bypasses Supervisor entirely
  → Routes directly to Payment Agent :9003 with full message history
  → Payment Agent: Resumes session from thread_store
    → Calls: processPayment, invalidateCache, getAccountDetails
    → Returns success message with transaction ID
```

---

## 10. Authentication & User Mapping

### 10.1 Authentication Flow

```
Frontend (MSAL.js)
  → User logs in via Azure AD (Microsoft Entra ID)
  → Acquires JWT token with:
     • Tenant ID: ed6f4727-...
     • Client ID: c37e62a7-...
     • Audience: api://c37e62a7-...
  → Sends token in Authorization header

Copilot Backend (dependencies.py)
  → TokenValidator:
     1. Fetch JWKS from Azure AD discovery endpoint
     2. Decode JWT, verify signature
     3. Check issuer, audience, expiration
  → Extract claims: preferred_username (email), oid (user ID), name
  → UserMapper: email → customer_id
```

### 10.2 Customer Mapping

Users authenticate with Azure AD emails and are mapped to banking customer IDs via `dynamic_data/customers.json`:

| Azure AD Email | Customer ID | Customer Name | Account |
|---------------|-------------|---------------|---------|
| somchai@bankxthb.onmicrosoft.com | CUST-001 | Somchai Rattanakorn | 123-456-001 |
| nattaporn@bankxthb.onmicrosoft.com | CUST-002 | Nattaporn Suksawat | 123-456-002 |
| pimchanok@bankxthb.onmicrosoft.com | CUST-003 | Pimchanok Thongchai | 123-456-003 |
| anan@bankxthb.onmicrosoft.com | CUST-004 | Anan Chaiyaporn | 123-456-004 |

### 10.3 Payment Agent Username Injection

The Supervisor **prepends the username** to all user messages sent to the Payment Agent:

```python
# Before sending to Payment Agent:
message = f"my username is {user_email}, {original_message}"

# Example:
# "my username is anan@bankxthb.onmicrosoft.com, transfer 300 THB to Somchai"
```

This is required because the Payment Agent's instructions use the email to call `getAccountsByUserName()` to identify the sender.

---

## 11. Startup Order & Ports

### Required Startup Sequence

```
Step 1: MCP Services (ports 8070-8078)
  ↓  Data layer must be available before agents connect
Step 2: A2A Specialist Agents (ports 9001-9006)
  ↓  Agents connect to MCP servers on startup
Step 3: Copilot Backend (port 8080)
  ↓  Routes to agents; auto-initializes cache from MCP
Step 4: Frontend (port 8081)
     Connects to Copilot backend
```

### Complete Port Map

| Port | Service | Type | Command |
|------|---------|------|---------|
| **8070** | Account MCP | MCP Server | `cd app\business-api\python\account; python main.py` |
| **8071** | Transaction MCP | MCP Server | `cd app\business-api\python\transaction; python main.py` |
| **8072** | Payment MCP | MCP Server | `cd app\business-api\python\payment; python main.py` |
| **8073** | Limits MCP | MCP Server | `cd app\business-api\python\limits; python main.py` |
| **8074** | Contacts MCP | MCP Server | `cd app\business-api\python\contacts; python main.py` |
| **8076** | ProdInfo FAQ MCP | MCP Server | `cd app\business-api\python\prodinfo_faq; python main.py` |
| **8077** | AI Money Coach MCP | MCP Server | `cd app\business-api\python\ai_money_coach; python main.py` |
| **8078** | Escalation Comms MCP | MCP Server | `cd app\business-api\python\escalation_comms; python main.py` |
| **9001** | Account Agent | A2A Agent | `cd app\agents\account-agent-a2a; python main.py` |
| **9002** | Transaction Agent | A2A Agent | `cd app\agents\transaction-agent-a2a; python main.py` |
| **9003** | Payment Agent | A2A Agent | `cd app\agents\payment-agent-a2a; python main.py` |
| **9004** | ProdInfo FAQ Agent | A2A Agent | `cd app\agents\prodinfo-faq-agent-a2a; python main.py` |
| **9005** | AI Money Coach Agent | A2A Agent | `cd app\agents\ai-money-coach-agent-a2a; python main.py` |
| **9006** | Escalation Agent | A2A Agent | `cd app\agents\escalation-agent-a2a; python main.py` |
| **8080** | Copilot Backend | FastAPI | `cd app\copilot; uv run uvicorn app.main:app --port 8080` |
| **8081** | Frontend | React/Vite | `cd app\frontend; npm run dev` |

---

## Appendix: Key File Reference

| File | Purpose |
|------|---------|
| `app/copilot/app/api/chat_routers.py` | Main `/api/chat` endpoint, SSE streaming, continuation logic |
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | Supervisor: keyword scoring, LLM classification, cache check, A2A routing |
| `app/copilot/app/cache/user_cache.py` | JSON file-based cache per customer, TTL, in-flight tracking |
| `app/copilot/app/api/dependencies.py` | JWT auth, email→customer mapping, cache trigger |
| `app/copilot/app/conversation_state_manager.py` | Multi-turn routing memory (5-min TTL per customer) |
| `conversations/conversation_manager.py` | Persistent conversation storage (JSON + Cosmos DB) |
| `app/agents/*/agent_handler.py` | Each agent's core logic: MCP connections, Foundry agent, message processing |
| `app/agents/*/prompts/*.md` | Agent instructions/system prompts |
| `app/agents/*/config.py` | Agent configuration (ports, model, MCP URLs) |
| `app/business-api/python/*/main.py` | MCP service implementations |
| `dynamic_data/*.json` | Banking data: customers, accounts, transactions, contacts, limits |
| `memory/*.json` | Cache files per customer |

---

# Section 12 — Detailed Code Flow Reference (File Names, Methods, A2A, MCP)

This section provides a source-code-level walkthrough of every use case, identifying every file, class, and method involved in each request. It covers the A2A protocol implementation, the Escalation Agent via Copilot Studio bridge, and exactly how MCP tools are connected and used by each agent.

---

## 12.1 A2A Protocol Implementation

### 12.1.1 What is A2A?

A2A (Agent-to-Agent) is a protocol for inter-agent communication. Each specialist agent exposes a standard HTTP interface. The Supervisor discovers agents via their agent card and calls them via a unified invoke endpoint.

### 12.1.2 Agent Card — `GET /.well-known/agent.json`

Every A2A agent main.py defines an `AGENT_CARD` dictionary and serves it at `/.well-known/agent.json`.

**File:** `app/agents/*/main.py` (all 6 agents follow this pattern)

```python
# Example from account-agent-a2a/main.py
AGENT_CARD = {
    "name": "Account Agent",
    "description": "Handles account inquiries, balance checks, and account details",
    "capabilities": ["account_balance", "account_details", "payment_methods"],
    "mcp_servers": ["account", "limits"]
}

@app.get("/.well-known/agent.json")
async def get_agent_card():
    return AGENT_CARD
```

The supervisor does NOT dynamically discover cards — it uses hardcoded A2A URLs configured via environment variables. The card endpoint exists for documentation and debugging.

### 12.1.3 Invoke Endpoint — `POST /a2a/invoke`

**File:** `app/agents/*/main.py`

All agents share the same request/response Pydantic models:

```python
class ChatMessage(BaseModel):
    role: str          # "user" or "assistant"
    content: str

class ChatRequest(BaseModel):
    messages: List[ChatMessage]
    thread_id: Optional[str] = None
    customer_id: Optional[str] = None
    user_email: Optional[str] = None
    stream: bool = False

class ChatResponse(BaseModel):
    response: str
    thread_id: Optional[str] = None
```

The `/a2a/invoke` endpoint:

```python
@app.post("/a2a/invoke")
async def invoke_agent(request: ChatRequest) -> ChatResponse:
    response = await agent_handler.process_message(
        messages=[{"role": m.role, "content": m.content} for m in request.messages],
        thread_id=request.thread_id,
        customer_id=request.customer_id,
        user_email=request.user_email
    )
    return ChatResponse(response=response, thread_id=request.thread_id)
```

### 12.1.4 How the Supervisor Calls A2A Agents

**File:** `app/copilot/app/agents/foundry/supervisor_agent_a2a.py`
**Method:** `_route_via_a2a_generic(a2a_url, messages, thread_id, customer_id, user_email, stream)`

```python
async def _route_via_a2a_generic(self, a2a_url, messages, thread_id, customer_id, user_email, stream=False):
    payload = {
        "messages": [{"role": m["role"], "content": m["content"]} for m in messages],
        "thread_id": thread_id,
        "customer_id": customer_id,
        "user_email": user_email,
        "stream": stream
    }
    response = await self.http_client.post(f"{a2a_url}/a2a/invoke", json=payload, timeout=300.0)
    data = response.json()
    return data.get("response", "")
```

Per-agent routing methods (`route_to_account_agent()`, `route_to_transaction_agent()`, etc.) check the A2A feature flag and call `_route_via_a2a_generic()`:

```python
async def route_to_account_agent(self, messages, thread_id, customer_id, user_email, stream=False):
    if self.use_account_a2a and self.account_agent_a2a_url:
        return await self._route_via_a2a_generic(
            self.account_agent_a2a_url, messages, thread_id, customer_id, user_email, stream
        )
    else:
        # Fallback: in-process agent (old path)
        return await self.account_agent.process_message(...)
```

### 12.1.5 Continuation Bypass — Direct A2A Call

**File:** `app/copilot/app/api/chat_routers.py`
**Method:** `_stream_response()`

When `is_continuation_message()` returns `True`, the chat router bypasses the Supervisor entirely and calls the A2A agent directly:

```python
if conversation_state_manager.is_continuation_message(user_message):
    active = conversation_state_manager.get_active_agent(customer_id)
    if active and active.a2a_url:
        # Direct HTTP POST to agent, skipping supervisor classification
        response = await http_client.post(f"{active.a2a_url}/a2a/invoke", json=payload)
```

### 12.1.6 Agent Lifecycle — `lifespan()` Pattern

**File:** `app/agents/*/main.py`

Every agent uses FastAPI's async lifespan context manager:

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    agent_handler = AgentHandler()
    await agent_handler.initialize()    # Creates MCP connections, loads agent from Foundry
    app.state.agent_handler = agent_handler
    yield
    # Shutdown
    await agent_handler.cleanup()       # Closes MCP connections
```

---

## 12.2 MCP Tool Architecture

### 12.2.1 MCP Services — Complete Tool Catalog

All MCP services use **FastMCP** and run as HTTP MCP servers (`mcp.run(transport="http")`).

**Account MCP — Port 8070**
File: `app/business-api/python/account/mcp_tools.py`

| Tool | Parameters | Description |
|------|-----------|-------------|
| `getAccountsByUserName` | `userName` | Get all accounts for a user (by email) |
| `getAccountDetails` | `accountId` | Get account details including balance & payment methods |
| `getPaymentMethodDetails` | `paymentMethodId` | Get payment method detail with available balance |

**Transaction MCP — Port 8071**
File: `app/business-api/python/transaction/mcp_tools.py`

| Tool | Parameters | Description |
|------|-----------|-------------|
| `getTransactionsByRecipientName` | `accountId`, `recipientName` | Get transactions by recipient name |
| `getLastTransactions` | `accountId`, `limit` (default 5) | Get last N transactions |
| `searchTransactions` | `accountId`, `fromDate`, `toDate` | Search by date range (YYYY-MM-DD) |
| `getTransactionDetails` | `accountId`, `txnId` | Get single transaction details |
| `aggregateTransactions` | `accountId`, `fromDate`, `toDate`, `metricType` | Aggregate: COUNT, SUM_IN, SUM_OUT, NET |

**Payment MCP — Port 8072**
File: `app/business-api/python/payment/mcp_tools.py`

| Tool | Parameters | Description |
|------|-----------|-------------|
| `processPayment` | `account_id`, `amount`, `description`, `payment_method_id`, `timestamp`, `recipient_name` (opt), `recipient_bank_code` (opt), `payment_type` (opt) | Submit a payment request |

**Limits MCP — Port 8073**
File: `app/business-api/python/limits/mcp_tools.py`

| Tool | Parameters | Description |
|------|-----------|-------------|
| `checkLimits` | `accountId`, `amount`, `currency` (default "THB") | Check if transaction is within limits |
| `getAccountLimits` | `accountId` | Get per-txn limit, daily limit, utilization |
| `updateLimitsAfterTransaction` | `accountId`, `amount` | Update daily limits after successful payment |

**Contacts MCP — Port 8074**
File: `app/business-api/python/contacts/mcp_tools.py`

| Tool | Parameters | Description |
|------|-----------|-------------|
| `getRegisteredBeneficiaries` | `accountId` | Get registered beneficiaries for an account |
| `verifyAccountNumber` | `accountNumber` | Verify if account number exists in the system |
| `addBeneficiary` | `accountId`, `beneficiaryAccountNumber`, `beneficiaryName`, `alias` (opt) | Add beneficiary after payment |
| `removeBeneficiary` | `accountId`, `beneficiaryAccountNumber` | Remove a beneficiary |
| `isBeneficiaryRegistered` | `accountId`, `beneficiaryAccountNumber` | Check if account is registered as beneficiary |

**Escalation Comms MCP — Port 8078**
File: `app/business-api/python/escalation_comms/mcp_tools.py` (tools via `register_tools()`)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `send_email` | `to_emails`, `subject`, `body`, `to_names` (opt), `cc_emails` (opt), `is_html` (opt) | Send email via Azure Communication Services |
| `send_ticket_notification` | `ticket_id`, `customer_email`, `customer_name`, `query`, `category` (opt) | Send ticket notification email |
| `get_tickets` | `customer_id`, `status` (opt) | Get all tickets for a customer |
| `create_ticket` | `customer_id`, `description`, `priority` (opt), `category` (opt), `customer_email` (opt), `customer_name` (opt) | Create support ticket |
| `get_ticket_details` | `ticket_id` | Get single ticket details |
| `update_ticket` | `ticket_id`, `status` (opt), `notes` (opt), `priority` (opt) | Update ticket fields |
| `close_ticket` | `ticket_id` | Close a ticket |

### 12.2.2 AuditedMCPTool — Compliance Wrapper

**File:** `app/agents/*/audited_mcp_tool.py` (copied into each agent directory; 9 copies)
**Extends:** `agent_framework.MCPStreamableHTTPTool`

`AuditedMCPTool` wraps every MCP tool call with full compliance audit logging:

```
Agent calls tool → AuditedMCPTool.call_tool()
  ├── Records start time
  ├── Calls super().call_tool() (actual MCP HTTP request)
  ├── Calculates duration
  ├── Determines compliance flags (PCI_DSS, GDPR_PERSONAL_DATA, HIGH_VALUE_TRANSACTION)
  ├── Extracts data_scope (account_data, payment_data, etc.)
  └── Writes audit JSON to observability/mcp_audit_YYYY-MM-DD.json
```

**Key Methods:**

| Method | Purpose |
|--------|---------|
| `__init__(name, url, customer_id, thread_id, mcp_server_name)` | Stores audit context; auto-extracts server name from URL |
| `call_tool(tool_name, **arguments)` | Main override — wraps `super().call_tool()` with audit logging |
| `connect()` | Connects to MCP server (HTTP transport), logs connection event |
| `_extract_server_name(url)` | Maps port/keyword in URL → server name (e.g., port 8070 → "account") |
| `_get_operation_type(tool_name)` | Returns "read", "create", "update", "delete", or "execute" |
| `_get_compliance_flags(tool_name, arguments, result)` | Returns list: `["PCI_DSS"]`, `["GDPR_PERSONAL_DATA"]`, `["HIGH_VALUE_TRANSACTION"]` |
| `_get_data_scope(tool_name)` | Returns "account_data", "transaction_history", "payment_data", "contact_data", or "general" |

### 12.2.3 How Agents Create MCP Tool Connections

**File:** `app/agents/*/agent_handler.py` — method `_create_mcp_tools()` or within `initialize()`

Each agent handler creates `AuditedMCPTool` instances for each MCP server it needs:

```python
# Example from AccountAgentHandler._create_mcp_tools()
async def _create_mcp_tools(self, customer_id=None, thread_id=None):
    account_tool = AuditedMCPTool(
        name="account_mcp",
        url=config.ACCOUNT_MCP_SERVER_URL,     # e.g., "http://localhost:8070/mcp"
        customer_id=customer_id,
        thread_id=thread_id,
        mcp_server_name="account"
    )
    await account_tool.connect()

    limits_tool = AuditedMCPTool(
        name="limits_mcp",
        url=config.LIMITS_MCP_SERVER_URL,      # e.g., "http://localhost:8073/mcp"
        customer_id=customer_id,
        thread_id=thread_id,
        mcp_server_name="limits"
    )
    await limits_tool.connect()

    return [account_tool, limits_tool]
```

**Agent → MCP Server Mapping:**

| Agent | MCP Servers Connected | Tool Count |
|-------|----------------------|------------|
| Account Agent (9001) | Account (8070) + Limits (8073) | 3 + 3 = 6 tools |
| Transaction Agent (9002) | Account (8070) + Transaction (8071) | 3 + 5 = 8 tools |
| Payment Agent (9003) | Account (8070) + Transaction (8071) + Payment (8072) + Contacts (8074) | 3 + 5 + 1 + 5 = 14 tools |
| ProdInfo FAQ Agent (9004) | None (uses Foundry file_search) | 1 custom tool (`create_support_ticket`) |
| AI Money Coach (9005) | None (uses Foundry file_search) | 1 custom tool (`create_support_ticket`) |
| Escalation Agent (9006) | Escalation Comms (8078) | 7 tools |

---

## 12.3 A2A Banking Telemetry

**File:** `app/agents/common/a2a_banking_telemetry.py`
**Class:** `A2ABankingTelemetry`

Each agent creates a singleton telemetry instance via `get_a2a_telemetry(agent_name)`. Telemetry writes NDJSON to the shared `observability/` directory.

**Key Methods:**

| Method | Output File | Data Recorded |
|--------|------------|---------------|
| `log_agent_decision(thread_id, user_query, triage_rule, ...)` | `agent_decisions_YYYY-MM-DD.json` | Which rule triggered, tools considered/invoked, duration, result |
| `log_user_message(thread_id, user_query, response_text, duration)` | `user_messages_YYYY-MM-DD.json` | Full conversation turn with timing |
| `log_tool_invocation(tool_name, parameters, result_status, ...)` | (within agent_decisions) | MCP tool call details |

**Triage Rules Used Per Agent:**

| Agent | Triage Rule Constant |
|-------|---------------------|
| Account Agent | `UC1_ACCOUNT_AGENT` |
| Transaction Agent | `UC2_TRANSACTION_AGENT` |
| Payment Agent | `UC4_PAYMENT_AGENT` |
| ProdInfo FAQ Agent | `UC2_PRODINFO_FAQ` |
| AI Money Coach | `UC3_AI_MONEY_COACH` |
| Escalation Agent | `UC6_ESCALATION_AGENT` |

---

## 12.4 Use Case 1 — Account & Balance Inquiry (Detailed Code Flow)

### Files Involved

| File | Class / Function | Role |
|------|-----------------|------|
| `app/copilot/app/api/chat_routers.py` | `_stream_response()` | Entry point: receives chat message via SSE |
| `app/copilot/app/api/dependencies.py` | `get_current_user()` | JWT auth → email → customer_id mapping |
| `app/copilot/app/cache/user_cache.py` | `UserCacheManager.get_cached_data()` | Cache check for balance/account queries |
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `SupervisorAgentA2A.processMessageStream()` | Routing: keyword scan → LLM fallback |
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `_try_cache_response()` | Returns cached balance without calling agent |
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `route_to_account_agent()` | Routes to Account Agent A2A |
| `app/agents/account-agent-a2a/main.py` | `invoke_agent()` | `/a2a/invoke` endpoint |
| `app/agents/account-agent-a2a/agent_handler.py` | `AccountAgentHandler.process_message()` | Creates Foundry agent, runs with MCP tools |
| `app/agents/account-agent-a2a/audited_mcp_tool.py` | `AuditedMCPTool.call_tool()` | Audited MCP tool calls |
| `app/business-api/python/account/mcp_tools.py` | `getAccountsByUserName()`, `getAccountDetails()` | Account data retrieval |
| `app/business-api/python/limits/mcp_tools.py` | `getAccountLimits()` | Limits data retrieval |

### Call Chain — "What is my balance?"

```
1. Frontend POST /api/chat → chat_routers._stream_response()
2. dependencies.get_current_user() → JWT decode → email="anan@bankxthb.onmicrosoft.com" → customer_id="CUST-004"
3. dependencies: triggers cache_manager.initialize_user_cache() if not valid
4. supervisor.processMessageStream(messages, thread_id, customer_id, user_email)
   4a. _try_cache_response("what is my balance?", customer_id="CUST-004")
       → Loads memory/CUST-004_cache.json
       → Detects "balance" keyword
       → Returns formatted balance string from cache.data.balance
       → ✅ SHORT-CIRCUIT: No agent call needed
5. SSE response streamed back: { "delta": "Your balance is 45,230.50 THB" }
```

### Call Chain — "Show my account details" (Cache Miss → Agent)

```
1-3. Same as above
4. supervisor.processMessageStream(...)
   4a. _try_cache_response() → No match for "account details" keyword
   4b. _classify_message_keywords("show my account details")
       → Score: "account" keyword → ACCOUNT category
   4c. route_to_account_agent(messages, thread_id, customer_id, user_email)
       → _route_via_a2a_generic("http://localhost:9001", ...)
       → HTTP POST http://localhost:9001/a2a/invoke
5. Account Agent main.py invoke_agent(request)
   → agent_handler.process_message(messages, thread_id, customer_id, user_email)
6. AccountAgentHandler.process_message():
   6a. _get_user_email(user_email) → returns email or looks up via UserMapper
   6b. _create_mcp_tools(customer_id, thread_id)
       → AuditedMCPTool("account_mcp", "http://localhost:8070/mcp") → connect()
       → AuditedMCPTool("limits_mcp", "http://localhost:8073/mcp") → connect()
   6c. get_agent() → AzureAIClient → create_agent(
           name="account-a2a",
           model="gpt-4o-mini",
           tools=[account_tool, limits_tool],
           instructions=<loaded from prompts/account_agent.md>
       )
   6d. agent.run(message="show my account details", thread=new_thread)
       → Foundry agent decides to call getAccountsByUserName(userName=email)
       → AuditedMCPTool.call_tool("getAccountsByUserName", userName=email)
           → HTTP POST to Account MCP :8070 → returns account list
       → Agent formats natural language response
   6e. telemetry.log_agent_decision(triage_rule="UC1_ACCOUNT_AGENT", ...)
7. Response returned → supervisor → SSE stream → frontend
```

---

## 12.5 Use Case 1B — Transaction History (Detailed Code Flow)

### Files Involved

| File | Class / Function | Role |
|------|-----------------|------|
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `route_to_transaction_agent()` | Routes to Transaction Agent A2A |
| `app/agents/transaction-agent-a2a/agent_handler.py` | `TransactionAgentHandler.process_message()` | Creates agent, runs with MCP tools |
| `app/agents/transaction-agent-a2a/agent_handler.py` | `TransactionAgentHandler.get_agent()` | Loads instructions, creates Foundry agent |
| `app/business-api/python/account/mcp_tools.py` | `getAccountsByUserName()` | Account lookup (needed to find account ID) |
| `app/business-api/python/transaction/mcp_tools.py` | `searchTransactions()`, `getLastTransactions()` | Transaction data |

### Call Chain — "Show my transactions from January"

```
1-3. Auth + cache check (same pattern)
4. supervisor._classify_message_keywords("show my transactions from january")
   → Score: "transaction" keyword → TRANSACTION category
   → route_to_transaction_agent()
   → _route_via_a2a_generic("http://localhost:9002", ...)
5. TransactionAgentHandler.process_message():
   5a. _create_mcp_tools() → connects to Account MCP (8070) + Transaction MCP (8071)
   5b. get_agent():
       → Loads prompts/transaction_agent.md
       → Formats {user_mail} and {current_date_time} into instructions
       → AzureAIClient.create_agent(tools=[account_tool, transaction_tool])
   5c. agent.run_stream(message, thread=new_thread)
       → Agent calls getAccountsByUserName(userName=email) → gets accountId="CHK-004"
       → Agent calls searchTransactions(accountId="CHK-004", fromDate="2025-01-01", toDate="2025-01-31")
       → Agent formats result as table
   5d. telemetry.log_agent_decision(triage_rule="UC2_TRANSACTION_AGENT")
6. Response streamed back
```

---

## 12.6 Use Case 2 — Product Information FAQ (Detailed Code Flow)

### Files Involved

| File | Class / Function | Role |
|------|-----------------|------|
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `route_to_prodinfo_agent()` | Routes to ProdInfo Agent A2A |
| `app/agents/prodinfo-faq-agent-a2a/agent_handler.py` | `ProdInfoFAQAgentHandler` | Main handler class |
| `app/agents/prodinfo-faq-agent-a2a/agent_handler.py` | `.initialize()` | Loads pre-deployed Foundry agent by name |
| `app/agents/prodinfo-faq-agent-a2a/agent_handler.py` | `.process_message()` | Runs agent with file_search, handles escalation |
| `app/agents/prodinfo-faq-agent-a2a/agent_handler.py` | `create_support_ticket_tool()` | Factory that returns callable for escalation |
| `app/agents/prodinfo-faq-agent-a2a/agent_handler.py` | `call_escalation_agent()` | HTTP POST to escalation agent's A2A endpoint |

### Key Difference: AzureAIAgentClient (not AzureAIClient)

ProdInfo FAQ and AI Money Coach use **`AzureAIAgentClient`** instead of `AzureAIClient`. The agent is pre-deployed in Azure AI Foundry with a **file_search** tool pointing to uploaded knowledge files (product PDFs, FAQ documents). The handler loads it by name rather than creating it at runtime:

```python
class ProdInfoFAQAgentHandler:
    async def initialize(self):
        self.client = AzureAIAgentClient(
            endpoint=config.AZURE_AI_PROJECT_ENDPOINT,
            credential=DefaultAzureCredential()
        )
        # Load pre-deployed agent from Foundry by name
        self.agent = await self.client.get_agent(name=config.PRODINFO_AGENT_NAME)
```

### Call Chain — "What credit cards do you offer?"

```
1-3. Auth + cache → no cache match for product queries
4. supervisor._classify_message_keywords("what credit cards do you offer")
   → Score: "credit card" / "offer" → PRODUCT category
   → route_to_prodinfo_agent()
   → _route_via_a2a_generic("http://localhost:9004", ...)
5. ProdInfoFAQAgentHandler.process_message():
   5a. Creates new thread via self.client
   5b. agent.run(message, thread=new_thread)
       → Foundry agent uses file_search tool against uploaded knowledge base
       → Returns answer about credit card offerings
   5c. telemetry.log_agent_decision(triage_rule="UC2_PRODINFO_FAQ")
6. Response returned
```

### Escalation from ProdInfo — "I need to talk to someone"

When the user asks to escalate, the ProdInfo agent uses its `create_support_ticket` tool:

```
5. ProdInfoFAQAgentHandler.process_message():
   5a. Agent determines user wants escalation
   5b. Agent calls create_support_ticket tool
       → create_support_ticket_tool(handler_instance) returns a callable
       → The callable calls call_escalation_agent(ticket_description)
   5c. call_escalation_agent():
       → HTTP POST to ESCALATION_AGENT_A2A_URL/a2a/invoke
       → JSON body: { messages: [{ role: "user", content: "Create ticket: ..." }],
                       customer_id: ..., user_email: ... }
   5d. Escalation agent creates ticket via Escalation Comms MCP
   5e. Returns ticket confirmation to ProdInfo agent
   5f. ProdInfo agent relays ticket confirmation to user
```

### Confirmation Detection in ProdInfo/AI Coach

```python
# In process_message():
if any(kw in user_msg_lower for kw in ["yes", "confirm", "create ticket", "please create"]):
    # User confirmed escalation
    # Extract original question from messages_history
    original_question = self._extract_original_question(messages)
    # Include original question in ticket description
    ticket_description = f"Customer query: {original_question}\nCustomer requested escalation."
```

---

## 12.7 Use Case 3 — AI Money Coach (Detailed Code Flow)

### Files Involved

| File | Class / Function | Role |
|------|-----------------|------|
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `route_to_ai_coach_agent()` | Routes to AI Money Coach A2A |
| `app/agents/ai-money-coach-agent-a2a/agent_handler.py` | `AIMoneyCoachAgentHandler` | Structurally identical to ProdInfo handler |

### Architecture — Identical to ProdInfo

AI Money Coach follows the **exact same pattern** as ProdInfo FAQ:
- Uses `AzureAIAgentClient` with a pre-deployed Foundry agent
- Agent has **file_search** over the "Debt-Free to Financial Freedom" book
- Has `create_support_ticket` tool for escalation
- Has `call_escalation_agent()` for A2A escalation

### Call Chain — "How should I budget my salary?"

```
1-3. Auth + cache (no cache for financial advice)
4. supervisor._classify_message_keywords("how should I budget my salary")
   → Score: "budget" / "salary" → AI_COACH category
   → route_to_ai_coach_agent()
   → _route_via_a2a_generic("http://localhost:9005", ...)
5. AIMoneyCoachAgentHandler.process_message():
   5a. Creates thread
   5b. agent.run(message, thread) → Foundry agent searches "Debt-Free to Financial Freedom"
   5c. Returns personalized budgeting advice from book content
   5d. telemetry.log_agent_decision(triage_rule="UC3_AI_MONEY_COACH")
6. Response returned
```

---

## 12.8 Use Case 4 — Payment Transfer (Detailed Code Flow)

### Files Involved

| File | Class / Function | Role |
|------|-----------------|------|
| `app/copilot/app/api/chat_routers.py` | `_stream_response()` | Prepends username to payment messages |
| `app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | `route_to_payment_agent()` | Routes to Payment Agent A2A |
| `app/copilot/app/conversation_state_manager.py` | `ConversationStateManager` | Tracks active agent for continuation |
| `app/agents/payment-agent-a2a/agent_handler.py` | `PaymentAgentHandler` | Multi-turn payment with thread persistence |
| `app/agents/payment-agent-a2a/agent_handler.py` | `PaymentAgentHandler.thread_store` | Class-level dict for session persistence |
| `app/agents/payment-agent-a2a/config.py` | Config | 4 MCP server URLs, model, agent name |
| `app/business-api/python/payment/mcp_tools.py` | `processPayment()` | Executes actual payment |
| `app/business-api/python/contacts/mcp_tools.py` | `getRegisteredBeneficiaries()`, `verifyAccountNumber()` | Beneficiary validation |
| `app/business-api/python/limits/mcp_tools.py` | `checkLimits()` | Limit validation |
| `app/business-api/python/account/mcp_tools.py` | `getAccountsByUserName()` | Sender account lookup |

### Payment Agent Thread Persistence — How It Works

**File:** `app/agents/payment-agent-a2a/agent_handler.py`
**Class:** `PaymentAgentHandler`
**Key:** `thread_store: dict = {}` — class-level variable (survives across requests)

```python
class PaymentAgentHandler:
    thread_store: dict = {}    # Persists across requests for multi-turn

    async def process_message(self, messages, thread_id, customer_id, user_email):
        # Step 1: Check if resuming existing conversation
        if thread_id and thread_id in self.thread_store:
            # Resume: Deserialize saved thread state
            current_thread = agent.get_new_thread()
            current_thread.update_from_thread_state(self.thread_store[thread_id])
        else:
            # New conversation: Create fresh thread
            current_thread = agent.get_new_thread()

        # Step 2: Run agent with thread context
        response = agent.run_stream(message, thread=current_thread)

        # Step 3: Save thread state for next turn
        self.thread_store[thread_id] = await current_thread.serialize()

        return response_text
```

### Call Chain — Turn 1: "Transfer 500 THB to Somchai"

```
1-3. Auth → customer_id="CUST-004", email="anan@bankxthb.onmicrosoft.com"
4. chat_routers._stream_response():
   → Detects payment routing
   → Prepends: "my username is anan@bankxthb.onmicrosoft.com, transfer 500 THB to Somchai"
5. supervisor._classify_message_keywords("... transfer 500 THB ...")
   → Score: "transfer" keyword → PAYMENT category
   → route_to_payment_agent()
   → _route_via_a2a_generic("http://localhost:9003", messages, thread_id="thread_CUST-004", ...)
6. PaymentAgentHandler.process_message():
   6a. thread_id="thread_CUST-004" not in thread_store → create new thread
   6b. _create_mcp_tools() → connects to 4 MCP servers:
       → AuditedMCPTool("account_mcp", "http://localhost:8070/mcp")
       → AuditedMCPTool("transaction_mcp", "http://localhost:8071/mcp")
       → AuditedMCPTool("payment_mcp", "http://localhost:8072/mcp")
       → AuditedMCPTool("contacts_mcp", "http://localhost:8074/mcp")
   6c. get_agent() → AzureAIClient.create_agent(tools=[4 MCP tools], instructions=prompts/payment_agent.md)
   6d. agent.run_stream(message, thread=current_thread)
       → Agent calls: getAccountsByUserName("anan@bankxthb.onmicrosoft.com")
           → Returns accounts including CHK-004
       → Agent calls: getRegisteredBeneficiaries("CHK-004")
           → Returns list including "Somchai" with account 123-456-001
       → Agent calls: checkLimits("CHK-004", 500.0, "THB")
           → Returns: sufficient_balance=true, within_per_txn_limit=true
       → Agent presents confirmation table:
           "Please confirm: 500 THB from CHK-004 to Somchai (123-456-001)"
   6e. thread_store["thread_CUST-004"] = await current_thread.serialize()
7. conversation_state_manager.update_state("CUST-004", "Payment Agent", "http://localhost:9003", "thread_CUST-004")
8. SSE response → confirmation table shown to user
```

### Call Chain — Turn 2: "Yes, confirm"

```
1-3. Auth → same customer
4. chat_routers._stream_response():
   → is_continuation_message("yes, confirm") = TRUE
   → conversation_state_manager.get_active_agent("CUST-004") → Payment Agent @ :9003
   → BYPASS SUPERVISOR → direct HTTP POST to http://localhost:9003/a2a/invoke
   → Payload includes full message history (Turn 1 + Turn 2)
5. PaymentAgentHandler.process_message():
   5a. thread_id="thread_CUST-004" IS in thread_store → resume thread
       → current_thread.update_from_thread_state(stored_state)
       → Agent now has context: knows the 500 THB transfer was confirmed
   5b. agent.run_stream("yes, confirm", thread=current_thread)
       → Agent calls: processPayment(account_id="CHK-004", amount=500.0, ...)
           → Payment MCP :8072 processes payment → returns {status: "ok"}
       → Agent calls: updateLimitsAfterTransaction("CHK-004", 500.0)
           → Limits MCP :8073 updates daily used amount
       → Agent calls: getAccountDetails("CHK-004")
           → Account MCP :8070 returns updated balance
       → Agent returns: "Payment successful! Txn ID: PAY-20250612-001. New balance: 44,730.50 THB"
   5c. thread_store["thread_CUST-004"] = await current_thread.serialize()
6. SSE response → success message shown to user
```

---

## 12.9 Escalation Agent — Two Paths (Detailed Code Flow)

There are **two independent implementations** of the Escalation Agent, both on port 9006 (only one runs at a time):

### 12.9.1 Path A: Escalation Agent A2A (MCP + Foundry)

**Directory:** `app/agents/escalation-agent-a2a/`

This is the primary path. It uses a Foundry-deployed agent backed by the **Escalation Comms MCP** server for ticket management and email notifications.

**Files Involved:**

| File | Class / Function | Role |
|------|-----------------|------|
| `app/agents/escalation-agent-a2a/agent_handler.py` | `EscalationAgentHandler` | Main handler |
| `app/agents/escalation-agent-a2a/agent_handler.py` | `.initialize()` | Connects to Escalation Comms MCP via AuditedMCPTool |
| `app/agents/escalation-agent-a2a/agent_handler.py` | `.process_message()` | Multi-turn message handling, MCP tool calls |
| `app/agents/escalation-agent-a2a/main.py` | `invoke_agent()` | A2A endpoint |
| `app/business-api/python/escalation_comms/mcp_tools.py` | `create_ticket()`, `send_email()` etc. | Ticket & email operations |

**Call Chain — Escalation from ProdInfo "create a ticket for me":**

```
1. ProdInfo Agent calls call_escalation_agent():
   → HTTP POST http://localhost:9006/a2a/invoke
   → Payload: { messages: [{ role: "user", content: "Create support ticket: 
                  Customer query: 'How do I apply for a gold card?'
                  Customer email: anan@bankxthb.onmicrosoft.com
                  Customer requested human assistance." }],
                customer_id: "CUST-004" }

2. EscalationAgentHandler.process_message():
   2a. _create_mcp_tools() → AuditedMCPTool to Escalation Comms MCP :8078
   2b. Injects customer_id and current_date_time into instructions
   2c. For multi-turn: combines all messages into context string
   2d. get_agent() → AzureAIClient.create_agent(
           tools=[escalation_comms_tool],
           instructions=prompts/escalation_agent.md
       )
   2e. agent.run(message, thread=new_thread)
       → Agent calls create_ticket(
             customer_id="CUST-004",
             description="Customer query about gold card application...",
             priority="normal",
             category="product_inquiry",
             customer_email="anan@bankxthb.onmicrosoft.com"
         )
       → Escalation Comms MCP: TicketService creates ticket → returns ticket_id
       → Agent calls send_ticket_notification(
             ticket_id="TKT-2025-001",
             customer_email="anan@bankxthb.onmicrosoft.com",
             customer_name="Anan Chaiyaporn",
             query="gold card application"
         )
       → Azure Communication Services sends email notification
   2f. Returns: "Support ticket TKT-2025-001 created. You'll receive email confirmation."
   2g. telemetry.log_agent_decision(triage_rule="UC6_ESCALATION_AGENT")
```

### 12.9.2 Path B: Escalation Copilot Bridge (Power Automate → Copilot Studio)

**Directory:** `app/agents/escalation-copilot-bridge/`

This is the **alternative** path that integrates with **Microsoft Copilot Studio** via Power Automate. It does NOT use an LLM agent — it programmatically parses the ticket request and sends it to a Power Automate flow.

**Files Involved:**

| File | Class / Function | Role |
|------|-----------------|------|
| `escalation-copilot-bridge/main.py` | FastAPI app | A2A endpoint + test endpoints |
| `escalation-copilot-bridge/a2a_handler.py` | `A2AHandler` | Parses ticket, calls Power Automate |
| `escalation-copilot-bridge/a2a_handler.py` | `.parse_ticket_from_message()` | Regex extraction of ticket fields |
| `escalation-copilot-bridge/a2a_handler.py` | `.create_ticket()` | Calls `PowerAutomateClient` |
| `escalation-copilot-bridge/power_automate_client.py` | `PowerAutomateClient` | HTTP POST to Power Automate flow URL |
| `escalation-copilot-bridge/power_automate_client.py` | `.create_escalation_ticket()` | Sends TicketData to flow |
| `escalation-copilot-bridge/models.py` | `TicketData`, `ExcelRow`, `EmailRecipient`, etc. | Pydantic data models |
| `escalation-copilot-bridge/config.py` | `Settings` | `POWER_AUTOMATE_FLOW_URL`, `COPILOT_BOT_NAME` |

**How It Works (No LLM):**

```
1. A2A invoke arrives at escalation-copilot-bridge :9006
   
2. A2AHandler.create_ticket(messages, customer_id):
   2a. parse_ticket_from_message(last_message_content):
       → Regex extracts: subject, description, priority, category
       → Pattern: r"subject[:\s]+(.+)" etc.
       → Falls back to entire message as description if no regex match
   
   2b. Builds TicketData:
       TicketData(
           customer_id="CUST-004",
           customer_name="Ujjwal Kumar",           # Hardcoded
           customer_email="ujjwal.kumar@microsoft.com",  # Hardcoded
           subject=extracted_subject,
           description=extracted_description,
           priority="normal",
           category="general"
       )
   
   2c. power_automate_client.create_escalation_ticket(ticket_data):
       → HTTP POST to POWER_AUTOMATE_FLOW_URL
       → Payload: ticket_data.model_dump()
       → Power Automate flow receives the JSON

3. What Power Automate Does:
   ├── Creates a row in Excel (SharePoint) with ticket details
   ├── Sends email notification to customer via Outlook
   └── Triggers Copilot Studio bot for follow-up conversation
       (Bot name configured via COPILOT_BOT_NAME setting)

4. Returns: TicketCreationResult with ticket_id, status, timestamp
```

**Key Difference Between Path A and Path B:**

| Aspect | Path A: escalation-agent-a2a | Path B: escalation-copilot-bridge |
|--------|------------------------------|-----------------------------------|
| **LLM Agent** | Yes — Foundry agent with instructions | No — purely programmatic |
| **MCP Tools** | Escalation Comms MCP (7 tools) | None |
| **Ticket Storage** | MCP TicketService (JSON files) | Power Automate → Excel (SharePoint) |
| **Email** | Azure Communication Services | Power Automate → Outlook |
| **Follow-up** | Agent-driven multi-turn | Copilot Studio bot |
| **Parsing** | LLM understands context naturally | Regex extraction from message |
| **Customer Info** | From A2A request (dynamic) | Hardcoded to ujjwal.kumar@microsoft.com |

---

## 12.10 Supervisor Agent — Routing Decision Engine (Detailed Code Flow)

### File: `app/copilot/app/agents/foundry/supervisor_agent_a2a.py`
### Class: `SupervisorAgentA2A` (1648 lines)

### 12.10.1 Constructor — `__init__()`

```python
def __init__(
    self,
    # 6 A2A URLs (one per agent)
    account_agent_a2a_url, transaction_agent_a2a_url, payment_agent_a2a_url,
    prodinfo_agent_a2a_url, ai_coach_agent_a2a_url, escalation_agent_a2a_url,
    # 6 feature flags (enable/disable A2A per agent)
    use_account_a2a, use_transaction_a2a, use_payment_a2a,
    use_prodinfo_a2a, use_ai_coach_a2a, use_escalation_a2a,
    # Fallback in-process agent references
    account_agent, transaction_agent, payment_agent,
    prodinfo_agent, ai_coach_agent, escalation_agent,
    # Shared services
    cache_manager, conversation_manager
):
    self.http_client = httpx.AsyncClient(timeout=300.0)
    self.llm_client = AsyncAzureOpenAI(...)  # For LLM classification fallback
```

### 12.10.2 Main Entry — `processMessageStream()`

```
processMessageStream(messages, thread_id, customer_id, user_email, stream)
│
├── Step 1: Check for active escalation (direct re-route)
│   → If customer has active escalation → route to escalation agent
│
├── Step 2: Try cache response
│   → _try_cache_response(user_message, customer_id)
│   → If match (balance, transactions, limits, account) → return cached data
│
├── Step 3: Keyword-based classification (instant, ~0ms)
│   → _classify_message_keywords(user_message)
│   → Scoring: Each keyword adds weight to a category
│   → Categories: ACCOUNT, TRANSACTION, PAYMENT, PRODUCT, AI_COACH, ESCALATION
│   → If score > threshold → route immediately
│
├── Step 4: LLM classification fallback (~1-3s)
│   → _classify_with_llm(user_message)
│   → Model: gpt-4o-mini, temperature=0.0, max_tokens=20
│   → Prompt: "Classify this banking query into one of: ACCOUNT, TRANSACTION,
│              PAYMENT, PRODUCT, AI_COACH, ESCALATION"
│   → Returns single category word
│
└── Step 5: Route to agent
    → route_to_account_agent() / route_to_transaction_agent() / etc.
    → Each checks A2A flag → _route_via_a2a_generic() or fallback in-process
```

### 12.10.3 Keyword Scoring — `_classify_message_keywords()`

The keyword scorer checks user message against weighted keyword lists:

```python
# Simplified keyword scoring logic:
KEYWORD_MAP = {
    "ACCOUNT": ["balance", "account", "account details", "how much"],
    "TRANSACTION": ["transaction", "history", "spent", "received", "statement"],
    "PAYMENT": ["transfer", "pay", "send money", "payment", "beneficiary"],
    "PRODUCT": ["credit card", "loan", "mortgage", "interest rate", "product"],
    "AI_COACH": ["budget", "save", "invest", "financial advice", "money tips"],
    "ESCALATION": ["complaint", "speak to someone", "escalate", "human agent"]
}
# Scores are additive — highest score wins if above threshold
```

### 12.10.4 Cache Response — `_try_cache_response()`

```python
async def _try_cache_response(self, user_message, customer_id):
    cache_data = await self.cache_manager.get_cached_data(customer_id)
    if not cache_data:
        return None
    
    msg_lower = user_message.lower()
    
    if any(kw in msg_lower for kw in ["balance", "how much"]):
        return f"Your balance is {cache_data['data']['balance']}"
    
    if any(kw in msg_lower for kw in ["last transaction", "recent transaction"]):
        txns = cache_data["data"]["last_5_transactions"]
        return format_transactions_table(txns)
    
    if any(kw in msg_lower for kw in ["limit", "daily limit"]):
        return format_limits(cache_data["data"]["limits"])
    
    return None  # No cache match → proceed to agent routing
```

---

## 12.11 Chat Router & SSE Streaming (Detailed Code Flow)

### File: `app/copilot/app/api/chat_routers.py` (690 lines)

### 12.11.1 Main Endpoint — `POST /api/chat`

```python
@router.post("/api/chat")
async def chat(request: ChatRequest, user=Depends(get_current_user)):
    return StreamingResponse(
        _stream_response(request, user),
        media_type="text/event-stream"
    )
```

### 12.11.2 SSE Streaming — `_stream_response()`

```
_stream_response(request, user):
│
├── Extract: user_email, customer_id, messages, thread_id
│
├── Payment Agent special handling:
│   → If routing to payment: prepend "my username is {email}" to user message
│
├── Continuation Check:
│   → conversation_state_manager.is_continuation_message(last_user_message)
│   → If YES:
│       → active = state_manager.get_active_agent(customer_id)
│       → If active agent exists and TTL not expired:
│           → Direct HTTP POST to active.a2a_url/a2a/invoke (BYPASS SUPERVISOR)
│           → Stream response back
│       → Else: fall through to supervisor
│
├── Normal Routing:
│   → supervisor.processMessageStream(messages, thread_id, customer_id, user_email)
│   → Returns response text
│
├── Update Conversation State:
│   → state_manager.update_state(customer_id, agent_name, a2a_url, thread_id)
│
└── SSE Format:
    → yield f"data: {json.dumps({'type': 'thinking', 'content': '...'})}\n\n"
    → yield f"data: {json.dumps({'type': 'delta', 'content': chunk})}\n\n"
    → yield f"data: {json.dumps({'type': 'final', 'content': full_response})}\n\n"
    → yield "data: [DONE]\n\n"
```

---

## 12.12 User Cache System (Detailed Code Flow)

### File: `app/copilot/app/cache/user_cache.py` (437 lines)
### Class: `UserCacheManager`

### 12.12.1 Key Attributes

```python
class UserCacheManager:
    _initializing: set = set()    # Customers currently being initialized (race protection)
    cache_dir: Path               # memory/ directory
    ttl: int = 300                # 5 minutes
```

### 12.12.2 Cache Initialization — `initialize_user_cache()`

```
initialize_user_cache(customer_id, user_email):
│
├── Add customer_id to _initializing set
│
├── Step 1 (Serial): Fetch accounts
│   → HTTP GET to Account MCP → getAccountsByUserName(email)
│   → Returns account list with IDs
│
├── Step 2 (Parallel): Fetch all data concurrently
│   → asyncio.gather(
│       fetch_transactions(account_id),    # Transaction MCP: getLastTransactions
│       fetch_contacts(account_id),        # Contacts MCP: getRegisteredBeneficiaries
│       fetch_limits(account_id)           # Limits MCP: getAccountLimits
│     )
│
├── Step 3: Build cache object
│   → { customer_id, customer_name, timestamp, ttl, data: { balance, accounts,
│       last_5_transactions, contacts, limits } }
│
├── Step 4: Atomic write
│   → Write to memory/{customer_id}_cache.json.tmp
│   → os.replace() → memory/{customer_id}_cache.json
│   → (Atomic rename prevents partial reads)
│
└── Remove customer_id from _initializing set
```

### 12.12.3 Cache Retrieval — `get_cached_data()`

```
get_cached_data(customer_id):
│
├── If customer_id in _initializing:
│   → Poll every 500ms for up to 25 seconds
│   → Wait for initialization to complete
│
├── Load memory/{customer_id}_cache.json
│
├── Check TTL:
│   → If (now - timestamp) > 300 seconds → return None (expired)
│
└── Return parsed cache object
```

### 12.12.4 Cache Validity Check — `is_cache_valid_for_customer()`

**File:** `app/copilot/app/api/dependencies.py` — called during auth

```python
# In get_current_user():
if not cache_manager.is_cache_valid_for_customer(customer_id):
    asyncio.create_task(cache_manager.initialize_user_cache(customer_id, email))

# is_cache_valid_for_customer() returns True if:
#   1. customer_id is in _initializing set (initialization in-flight), OR
#   2. Cache file exists AND TTL has not expired
```

---

## 12.13 Conversation State Manager (Detailed Code Flow)

### File: `app/copilot/app/conversation_state_manager.py` (193 lines)

### 12.13.1 ConversationState Dataclass

```python
@dataclass
class ConversationState:
    thread_id: str               # Thread ID used with the agent
    agent_name: str              # e.g., "Payment Agent"
    a2a_url: str                 # e.g., "http://localhost:9003"
    last_activity: datetime      # When last message was routed
    customer_id: str             # Key for lookup
    message_count: int = 0       # Number of messages in conversation
```

### 12.13.2 Key Methods

| Method | Signature | Purpose |
|--------|-----------|---------|
| `update_state()` | `(customer_id, agent_name, a2a_url, thread_id)` | Saves/updates active agent state |
| `get_active_agent()` | `(customer_id)` → `ConversationState or None` | Returns state if TTL (5 min) not expired |
| `is_continuation_message()` | `(message)` → `bool` | Detects short follow-ups: "yes", "confirm", "cancel", "option 1", messages < 20 chars |
| `clear_state()` | `(customer_id)` | Removes state (used after timeout or explicit reset) |

### 12.13.3 Continuation Detection Keywords

```python
CONTINUATION_KEYWORDS = [
    "yes", "yeah", "yep", "ok", "okay", "confirm", "proceed",
    "go ahead", "approve", "do it", "sure",
    "no", "cancel", "stop", "abort", "nevermind",
    "option 1", "option 2", "option 3", "choice a", "choice b"
]

def is_continuation_message(self, message: str) -> bool:
    msg_lower = message.strip().lower()
    if len(msg_lower) < 20:     # Short messages are likely continuations
        return True
    return any(kw in msg_lower for kw in CONTINUATION_KEYWORDS)
```

---

## 12.14 Authentication & Dependencies (Detailed Code Flow)

### File: `app/copilot/app/api/dependencies.py` (142 lines)

### 12.14.1 `get_current_user()` — JWT Authentication Pipeline

```
get_current_user(request: Request):
│
├── Extract Authorization header → "Bearer <jwt_token>"
│
├── TokenValidator:
│   ├── Fetch JWKS from Azure AD: https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys
│   ├── Decode JWT using matching key (kid)
│   ├── Verify: issuer, audience (api://c37e62a7-...), expiration
│   └── Extract claims: preferred_username, oid, name
│
├── UserMapper:
│   ├── Load dynamic_data/customers.json
│   ├── Map email → customer_id (e.g., "anan@bankxthb.onmicrosoft.com" → "CUST-004")
│   └── Static fallback map if dynamic lookup fails
│
├── Cache Auto-Init:
│   ├── cache_manager.is_cache_valid_for_customer(customer_id)
│   ├── If NOT valid → asyncio.create_task(cache_manager.initialize_user_cache(...))
│   └── Fire-and-forget: does not block the request
│
└── Return: User(email, customer_id, name)
```

---

## 12.15 Complete End-to-End Sequence Diagrams

### 12.15.1 Simple Query (Cache Hit)

```
User → Frontend → POST /api/chat → dependencies.get_current_user()
                                         │
                                    JWT validated
                                    Cache valid ✓
                                         │
                                    _stream_response()
                                         │
                                    is_continuation? NO
                                         │
                                    supervisor.processMessageStream()
                                         │
                                    _try_cache_response() → HIT ✓
                                         │
                                    SSE: {"type":"delta","content":"Balance: 45,230.50 THB"}
                                    SSE: [DONE]
                                         │
                                    ← Frontend renders response
```

### 12.15.2 Agent Query (Full Path)

```
User → Frontend → POST /api/chat → dependencies.get_current_user()
                                         │
                                    JWT validated
                                    Cache init triggered (async)
                                         │
                                    _stream_response()
                                         │
                                    is_continuation? NO
                                         │
                                    supervisor.processMessageStream()
                                         │
                                    _try_cache_response() → MISS
                                         │
                                    _classify_message_keywords() → TRANSACTION (score > threshold)
                                         │
                                    route_to_transaction_agent()
                                         │
                                    _route_via_a2a_generic("http://localhost:9002")
                                         │
                              HTTP POST → Transaction Agent :9002/a2a/invoke
                                         │
                              TransactionAgentHandler.process_message()
                                         │
                              _create_mcp_tools() → AuditedMCPTool → connect to MCP :8070, :8071
                                         │
                              get_agent() → Foundry create_agent(tools, instructions)
                                         │
                              agent.run() → getAccountsByUserName() → searchTransactions()
                                         │
                              AuditedMCPTool.call_tool() → audit log written
                                         │
                              ← Response text returned
                                         │
                                    conversation_state_manager.update_state()
                                         │
                                    SSE: {"type":"delta","content":"Here are your transactions..."}
                                    SSE: [DONE]
```

### 12.15.3 Multi-Turn Payment (Continuation)

```
Turn 1:
  User: "Transfer 500 THB to Somchai"
  → supervisor → keyword "transfer" → Payment Agent :9003
  → PaymentAgentHandler: new thread → 4 MCP tools → confirmation table
  → thread_store[thread_id] = serialized state
  → state_manager.update_state("CUST-004", "Payment Agent", ":9003", thread_id)

Turn 2:
  User: "Yes, confirm"
  → is_continuation("yes, confirm") = TRUE
  → state_manager.get_active_agent("CUST-004") → Payment Agent :9003
  → BYPASS SUPERVISOR → direct POST to :9003/a2a/invoke
  → PaymentAgentHandler: resume thread from thread_store
  → agent calls processPayment(), updateLimitsAfterTransaction()
  → "Payment successful!"
```

### 12.15.4 Escalation Chain (ProdInfo → Escalation Agent)

```
Turn 1:
  User: "What credit cards do you offer?"
  → supervisor → keyword "credit card" → ProdInfo Agent :9004
  → ProdInfoFAQAgentHandler: file_search → knowledge base answer

Turn 2:
  User: "I want to speak to someone"
  → is_continuation? possibly YES → direct to ProdInfo :9004
  → OR: supervisor → keyword "speak to someone" → ESCALATION
  → Either way: ProdInfo agent calls create_support_ticket tool
       → call_escalation_agent() → HTTP POST to Escalation Agent :9006
       → EscalationAgentHandler → create_ticket() via MCP → send_email() via MCP
       → Returns: "Ticket TKT-2025-001 created"
  → ProdInfo relays: "I've created ticket TKT-2025-001 for you"
```
