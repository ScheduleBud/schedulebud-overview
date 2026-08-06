# ScheduleBud: AI-Powered Academic Scheduling for College Students

[![Production Ready](https://img.shields.io/badge/Status-Production%20Ready-green)](https://schedulebud.app/) [![React](https://img.shields.io/badge/React-18.2.0-blue)](https://reactjs.org/) [![TypeScript](https://img.shields.io/badge/TypeScript-5.8.3-blue)](https://www.typescriptlang.org/) [![Supabase](https://img.shields.io/badge/Supabase-Edge%20Functions-blue)](https://supabase.com/) [![AI Powered](https://img.shields.io/badge/AI-Gemini%202.5%20Pro-purple)](https://deepmind.google/technologies/gemini/)

**Live Application:** [**https://schedulebud.app**](https://schedulebud.app/)

## Project Overview

ScheduleBud is an AI-powered platform that automates academic scheduling for college students. It consolidates the fragmented tools a student juggles into a single interface: it syncs Canvas LMS assignments from `.ics` calendar feeds, pulls in course announcements through the Canvas API, uses Google Gemini to extract every deadline from an uploaded syllabus, and exposes an AI agent with RAG and function-calling capabilities that manages tasks and answers course-specific questions through natural language. The result is a self-healing, cost-optimized system that takes scheduling off the student's plate.

## Live Demo

A live video demo can be found here: [ScheduleBud Demo](https://youtu.be/TEmODMrIAvg)

## Tech Stack

| Frontend | Backend | AI/ML | Infrastructure | Payments | Testing |
|---|---|---|---|---|---|
| ![React](https://img.shields.io/badge/-React-61DAFB?logo=react&logoColor=white) | ![Supabase](https://img.shields.io/badge/-Supabase-3FCF8E?logo=supabase&logoColor=white) | ![Google Gemini](https://img.shields.io/badge/-Gemini%202.5%20Pro-8A2BE2?logo=google&logoColor=white) | ![Render](https://img.shields.io/badge/-Render-46E3B7?logo=render&logoColor=white) | ![Stripe](https://img.shields.io/badge/-Stripe-6772E5?logo=stripe&logoColor=white) | ![Playwright](https://img.shields.io/badge/-Playwright-2EAD33?logo=playwright&logoColor=white) |
| ![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white) | ![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-4169E1?logo=postgresql&logoColor=white) | ![Hugging Face](https://img.shields.io/badge/-Hugging%20Face-FFD000?logo=huggingface&logoColor=white) | | | |
| ![Tailwind CSS](https://img.shields.io/badge/-Tailwind%20CSS-06B6D4?logo=tailwind-css&logoColor=white) | ![Deno](https://img.shields.io/badge/-Deno-000000?logo=deno&logoColor=white) | ![pgvector](https://img.shields.io/badge/-pgvector-2F69AD?logo=postgresql&logoColor=white) | | | |

## High-Level System Design

My design for ScheduleBud was driven by five core principles, essential for a solo engineer building a production-ready application:

1.  **Serverless-First:** Eliminate infrastructure management by using serverless functions for all custom backend logic, ensuring automatic scaling and reducing operational overhead.
2.  **Maximize Velocity with Managed Services:** Leverage a Backend-as-a-Service (Supabase) for commodity components like user auth and a basic CRUD API, allowing me to focus engineering effort on unique, value-adding features.
3.  **Security at the Core:** Implement security at the lowest possible layer (the database) using Row-Level Security (RLS) and create secure boundaries for all external service interactions.
4.  **Resilience and Fallbacks:** Acknowledge that external services and complex processes can fail. The system must be designed to be self-healing, handle errors gracefully, and use fallback mechanisms wherever possible.
5.  **Cost Optimization:** Design the system to be intelligent about resource usage, especially expensive AI API calls, by implementing smart routing and caching.

Based on these principles, I architected ScheduleBud as a decoupled, multi-tier system. It consists of a React SPA frontend, a central BaaS platform, and a suite of specialized serverless microservices that handle complex, asynchronous tasks.

```mermaid
flowchart LR
    %% ---------------------------------------------------------
    %% STYLES
    %% ---------------------------------------------------------
    classDef container fill:#f9f7e8,stroke:#d3d0b8,stroke-width:2px,rx:5,ry:5
    classDef service fill:#fff,stroke:#333,stroke-width:1px
    classDef primary fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef db fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    
    %% ---------------------------------------------------------
    %% 1. USER LAYER
    %% ---------------------------------------------------------
    subgraph Client ["User Device"]
        direction TB
        A[Frontend: React SPA]:::primary
    end

    %% ---------------------------------------------------------
    %% 2. BACKEND LAYER
    %% ---------------------------------------------------------
    subgraph P1 ["Backend Platform (Supabase)"]
        direction TB
        style P1 fill:#fcfbf6,stroke:#ccc,stroke-dasharray: 5 5

        %% Core Infrastructure
        subgraph CoreInfra ["Core Services"]
            B[API Gateway]:::primary
            C[("PostgreSQL DB<br/>(RLS + pgvector)")]:::db
            E[File Storage]:::db
        end

        %% Serverless Functions
        subgraph F_Group ["Serverless Edge Functions"]
            direction TB
            style F_Group fill:#f0f0f0,stroke:#bbb,rx:5
            
            F1["ask-chatbot<br/>(AI Agent + RAG)"]:::service
            F2["embed-file<br/>(Embeddings)"]:::service
            F3["ai-analysis<br/>(Syllabus Extraction)"]:::service
            F4["canvas-sync<br/>(ICS Parsing)"]:::service
            F5["send-email-notification<br/>(Notifications)"]:::service
            F6["stripe-webhook<br/>(Payment Events)"]:::service
            F7["canvas-announcements<br/>(Announcements Sync)"]:::service
            F8["create-checkout-session<br/>(Checkout)"]:::service
            F9["create-portal-session<br/>(Billing Portal)"]:::service
            F10["parse-natural-language<br/>(Quick-Add NL Parsing)"]:::service
        end
    end

    %% ---------------------------------------------------------
    %% 3. EXTERNAL LAYER
    %% ---------------------------------------------------------
    subgraph External ["External APIs"]
        direction TB
        G["Google Gemini"]:::service
        H["Stripe API"]:::service
        J["Hugging Face"]:::service
        K["Canvas (ICS)"]:::service
        L["Resend API"]:::service
        M["Canvas API"]:::service
    end

    %% ---------------------------------------------------------
    %% CONNECTIONS
    %% ---------------------------------------------------------

    %% Flow
    A -->|API/SSE| B
    B --> CoreInfra
    B --> F_Group

    %% Function Logic
    F1 -->|Tool Call| G
    F1 -->|Query| C
    
    F2 -->|Vectorize| J
    F2 -->|Save| C
    
    F3 -->|Extract| G
    
    F4 -->|Fetch| K
    F4 -->|Sync| C
    
    F5 -->|Send| L
    
    F6 <-->|Events| H
    F6 -->|Update| C

    F7 -->|Fetch| M

    F8 <-->|Create Session| H
    F8 -->|Store Customer ID| C

    F9 <-->|Create Session| H

    F10 -->|Extract| G
```

## Key Architectural Features & Implementations

### 1. The AI Agent with RAG Pipeline & Tool Calling (Smart Assistant)
**Feature:** An intelligent AI agent that combines retrieval-augmented generation (RAG) for answering course-specific questions with function calling capabilities for task management. Users can ask questions about their course materials AND execute CRUD operations on their tasks through natural language (e.g., "Create a Biology homework task due on December 25", "Update my quiz to be due next week", "Delete all completed tasks").

**Technical Implementation:** I architected a **stateless, streaming AI agent** that intelligently routes queries and executes tools when needed. The system uses a `classifyQueryIntent` algorithm to route queries into 4 categories (`document_search`, `task_related`, `general_knowledge`, `conversational`) BEFORE making expensive API calls. This smart routing means **~70% of queries skip RAG entirely**, avoiding unnecessary HuggingFace embedding API calls and PostgreSQL vector searches.

**Key Capabilities:**

1. **RAG Pipeline for Document Questions**: Only queries explicitly about course materials trigger document retrieval via vector similarity search (pgvector). The system is self-healing: if a document search fails because files aren't processed, it auto-triggers the embedding function and retries.

2. **Function Calling for Task Management**: The AI can execute 7 task-management operations through Gemini's tool calling: `create_task`, `update_task`, `delete_task`, `search_tasks`, `search_classes`, `list_task_types`, and `create_task_type`. The system implements **two-phase execution** for the one destructive action (`delete_task`) - requiring user confirmation before execution, while non-destructive actions execute immediately.

3. **Search-to-ID Resolution**: The AI uses human-friendly search terms (task names like "homework", class names like "Biology"), and the backend automatically resolves them to database IDs through fuzzy matching. Missing classes are auto-created with an `istaskclass` flag to separate AI-managed entities from user-created entities.

4. **Streaming UX**: Server-Sent Events (SSE) deliver real-time responses with a smooth scrolling buffer that provides a typewriter effect at 60 FPS, making the interaction feel responsive even during long responses.

5. **Web Search & Clarification Tools**: Two more tools round out the agent beyond RAG and task CRUD: `web_search` for questions about current events or facts outside the student's own course materials, and `request_clarification` so the agent asks a follow-up question instead of guessing when a request is ambiguous.

6. **Quick-Add Natural Language Parsing**: A separate, lightweight edge function (`parse-natural-language`) powers a quick-add text box in the task creation modal. A user types a task in plain English (e.g., "homework due friday at 5pm"), Gemini extracts the action, date, time, and entities, and the result pre-fills the task form for the user to review before it goes through the same task-creation path as a manually typed task.

**Code Snippet (Query Intent Classification for Cost Optimization):**
```typescript
function classifyQueryIntent(
  query: string,
  hasClassContext: boolean,
  hasConversationHistory: boolean
): 'document_search' | 'task_related' | 'general_knowledge' | 'conversational' {
  const normalizedQuery = query.toLowerCase().trim();

  // Indicator keywords for different categories
  const documentIndicators = ['syllabus', 'lecture', 'reading', 'according to'];
  const taskIndicators = ['create task', 'update task', 'delete task', 'due', 'deadline'];

  // Priority scoring system to determine the most likely intent
  let documentScore = 0;
  let taskScore = 0;

  documentIndicators.forEach(ind => normalizedQuery.includes(ind) ? documentScore += 2 : null);
  taskIndicators.forEach(ind => normalizedQuery.includes(ind) ? taskScore += 2 : null);

  if (hasClassContext && documentScore === 0) documentScore += 1;
  if (hasConversationHistory && normalizedQuery.length < 30) return 'conversational';

  // Determine the highest score to select the query type
  const maxScore = Math.max(documentScore, taskScore);

  if (maxScore < 2) {
    if (hasClassContext) return 'document_search';
    return 'general_knowledge';
  }

  return documentScore > taskScore ? 'document_search' : 'task_related';
}
```

### 2. The Secure Data Ingestion & Embedding Pipeline
**Feature:** A secure pipeline to process user-uploaded syllabi (PDFs/DOCX), extract their content, and transform them into searchable vector embeddings.

**Technical Implementation:** This serverless function is designed for resilience and security. It extracts PDF text via `unpdf`, then validates and sanitizes the result against security patterns before processing; extraction failures fail the request explicitly rather than silently reporting a false success. To ensure data integrity, the function is **idempotent**, deleting any stale embeddings for a file before generating new ones.

### 3. The Event-Driven Payments System
**Feature:** A complete subscription management system integrating Stripe checkout, billing portal, and webhook-based payment synchronization.

**Technical Implementation:** I designed a three-part serverless payment architecture:

1. **Checkout Session Creation (`create-checkout-session`)**: Securely generates Stripe checkout sessions server-side, preventing client-side price manipulation and ensuring proper subscription tier validation.

2. **Billing Portal Access (`create-portal-session`)**: Provides authenticated users with secure access to Stripe's customer portal for managing subscriptions, payment methods, and billing history.

3. **Webhook Event Processing (`stripe-webhook`)**: An asynchronous, event-driven system using Stripe webhooks. This serverless function acts as a secure endpoint that **cryptographically verifies webhook signatures** before processing events. It operates as a state machine, listening for events like `invoice.payment_succeeded`, `invoice.payment_failed`, and `customer.subscription.deleted`, then updating the user's `subscription_status` in the PostgreSQL database to maintain data integrity between ScheduleBud and Stripe.

All three functions enforce strict security measures, including signature verification, authenticated user validation, and server-side payment processing to prevent tampering.

### 4. The Canvas LMS Implementation System (ICS Calendar Parsing)
**Feature:** Seamless synchronization of Canvas LMS assignments by parsing ICS calendar feeds, automatically importing due dates, assignment names, and course information into ScheduleBud.

**Technical Implementation:** I built a serverless edge function that fetches and parses Canvas ICS calendar feeds server-side, completely bypassing CORS restrictions that plague client-side implementations. The system implements intelligent duplicate detection using Canvas UIDs to ensure assignments aren't duplicated on subsequent syncs. It includes robust error handling with exponential backoff retries and supports bulk assignment processing for courses with heavy assignment loads. The parser extracts course codes, assignment details, and due dates, automatically creating tasks with proper class associations and Canvas metadata for seamless integration. A companion `canvas-announcements` function brings in course announcements the same way, so students get their deadlines and their instructor updates in one place.

### 5. The AI-Powered Document Analysis Engine
**Feature:** Intelligent extraction of structured academic metadata from user-uploaded documents, automatically detecting courses, assignments, due dates, and academic requirements from syllabi and course materials.

**Technical Implementation:** I designed a specialized serverless edge function that leverages Google Gemini 2.5 Pro's structured output capabilities to parse academic documents and extract actionable task data. The system implements sophisticated JSON truncation recovery algorithms and domain-specific parsing for academic terminology (especially nursing/healthcare: ATI, HESI, clinical engagement). It handles complex date extraction from weekly ranges, checkbox assignments, and various academic formats, then stores the structured extraction results in PostgreSQL for integration with the task management system. This is separate from the embedding pipeline and focuses purely on extracting structured academic metadata rather than creating searchable vectors.

## Key Challenge & Solution: Multi-Tenant Data Security

**Challenge:** One of the biggest challenges was designing a system where multiple users could store their personal academic data with the absolute guarantee that their information would remain private. A simple mistake in a query could potentially expose one user's data to another.

**Solution:** I solved this by making data ownership a core principle of the database schema and enforcing security at the database level with **Row-Level Security (RLS)**. Every table containing user-generated content has a `user_id` column that links to the `auth.users` table.

By implementing RLS policies on all relevant tables, I created a fundamental security model that cannot be bypassed by application-level code. This approach ensures that even if there were a bug in an API call, the database itself would still prevent unauthorized data access, effectively creating a powerful security backstop.

**Code Snippet (PostgreSQL RLS Policy):**
```sql
-- Enable Row-Level Security on the 'tasks' table
ALTER TABLE public.tasks ENABLE ROW LEVEL SECURITY;

-- Create a policy that allows users to see only their own tasks
CREATE POLICY "Users can view their own tasks"
ON public.tasks FOR SELECT
USING (auth.uid() = user_id);

-- Create a policy that allows users to insert tasks for themselves
CREATE POLICY "Users can create their own tasks"
ON public.tasks FOR INSERT
WITH CHECK (auth.uid() = user_id);
```
