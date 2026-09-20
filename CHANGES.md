# AI Business Assistant Experiment Plan

## Experiment Overview

ProStore will be used as the foundation for an AI Business Assistant experiment focused on LLM tool calling, intentional retrieval, and retrieval-augmented generation (RAG). The central question is whether a model can decide when external information is needed and select the appropriate capability to obtain it.

Retrieval is not a default action. A conversational message such as "Hey!" should receive a natural response without querying business data or retrieving knowledge-base content. A business question should allow the model to choose an appropriately scoped capability, while the application remains responsible for authorising, validating, and executing that capability.

The experiment will progress incrementally from chat, through structured tool calling, to knowledge retrieval and eventual multi-source synthesis. It will not begin as an autonomous agent.

## Why ProStore

The README describes ProStore as a full-featured ecommerce website built with Next.js, TypeScript, PostgreSQL, and Prisma. It provides a realistic business domain with operational data and workflows rather than a purpose-built mock dataset.

The documented features include product, order, and user management; an interactive checkout process; payment through Stripe, PayPal, and cash on delivery; ratings and reviews; search, sorting, filtering, and pagination; user profiles and order history; and an admin area with statistics and charts. The README also documents Next Auth authentication, a PostgreSQL database configuration, Prisma Studio, and database seeding.

Those documented capabilities make questions about customers/users, products, orders, carts, reviews, and operational policies meaningful. They let the experiment evaluate whether an LLM selects retrieval based on the information need, rather than merely generating plausible ecommerce answers over invented data.

## Current Foundation

The following foundation is established by the README:

- A Next.js and TypeScript ecommerce application.
- PostgreSQL as the documented database and Prisma as the documented data layer/tooling, including Prisma Studio and a seed command.
- Next Auth authentication.
- An admin area with statistics and charts.
- Order, product, and user management.
- A user area with profile and order access.
- An interactive checkout process with Stripe, PayPal, and cash-on-delivery payment options.
- Featured products, multi-image support through Uploadthing, ratings and reviews, and customer/admin search.
- Sorting, filtering, pagination, light/dark mode, and a deployed demo.
- External service configuration for payments, uploads, and Resend email.

The README does not describe an existing chatbot, LLM provider, vector database, embedding pipeline, knowledge base, or tool-calling contract. These are experimental capabilities to introduce deliberately, not existing functionality to assume.

## Green Signals

- **Established ecommerce domain:** Products, orders, users, checkout, payments, reviews, and search are documented business concerns.
- **Relational business data:** PostgreSQL and Prisma provide a documented foundation for structured information retrieval.
- **Business workflows already exist:** Product/order/user management and the documented checkout and payment options provide concrete operations to query and explain.
- **Authentication and user roles:** Next Auth, user-facing account functionality, and an admin area indicate existing identity-sensitive workflows.
- **Administrative context:** The admin area includes management functionality, statistics, and charts, making business questions more realistic than in a storefront-only prototype.
- **Operational integrations:** Payment, image-upload, and email service configuration demonstrate an application that already interacts with external services.
- **Runnable, seedable application:** The README documents development, build, production, Prisma Studio, and database-seeding workflows.

These signals mean the experiment can focus on retrieval decisions and grounded answers rather than rebuilding an ecommerce domain, core entities, database setup, authentication, or basic business workflows.

## Target Architecture

The intended architecture evolves in stages:

```text
Chat
	↓
Tool calling
	↓
Structured retrieval
	↓
RAG
	↓
Multi-tool retrieval
	↓
Evaluation
```

At every stage, the model determines whether an available capability is relevant. The application owns access control, argument validation, execution, result shaping, and observability. The model must not have unrestricted database access.

## Phase 1 — Chat

Introduce a conversational interface where a user can submit a message and receive an LLM response:

```text
User
 ↓
LLM
 ↓
Response
```

No retrieval is automatic in this phase. The initial implementation should establish the interaction boundary, identity and access expectations, response handling, and a way to observe the conversation without adding product, order, database, or knowledge-base retrieval.

## Phase 2 — Tool Picking

Give the LLM a small, explicit set of capability definitions. The model decides whether no tool, one tool, or eventually more than one tool is needed. Tool definitions should be scoped to documented ProStore business domains:

- **Product tools:** Eventually retrieve product information relevant to catalog or inventory-style questions. The README documents product management and featured products, but does not define a specific inventory schema; tool contracts must be designed from the application data model during implementation.
- **Order tools:** Eventually retrieve order information for operational or purchase-history questions. The README documents order management and user order access.
- **Customer/user tools:** Eventually retrieve authorised user/customer information needed for support or administrative questions. The README documents user management, profiles, orders, authentication, and an admin area.
- **Cart tools:** Eventually retrieve authorised cart information when questions concern shopping-cart activity. The README documents ecommerce and an interactive checkout experience; the exact tool surface must follow the implemented data model.
- **Review tools:** Eventually retrieve ratings and review information. The README documents a ratings and reviews system.

The governing contract is: **the LLM chooses a tool; the application executes and validates the tool call.** Each tool should expose a narrowly defined purpose, typed/validated arguments, authorisation requirements, bounded results, and clear failure behaviour. Do not expose general SQL execution or unrestricted data access to the model.

## Phase 3 — Structured Retrieval

Connect approved, selected tools to the existing PostgreSQL-backed application:

```text
User
 ↓
LLM
 ↓
Tool call
 ↓
Application
 ↓
Service layer
 ↓
PostgreSQL
 ↓
Tool result
 ↓
LLM
 ↓
Response
```

The service layer shown here is an intended boundary for the experiment: it should centralise validation, authorisation, and retrieval behaviour before database access. Its concrete placement and integration should be decided only after reviewing the application code.

Structured business data, such as customers/users, products, orders, staff/users, and reviews, should be retrieved through application/database tools. This differs from traditional RAG: the system should query structured records using purpose-built, authorised capabilities rather than blindly embedding relational records into a vector store. Tool results should be minimal and task-specific so the LLM receives only the information necessary to answer.

## Phase 4 — RAG

After structured tool calling is working and observable, introduce a knowledge base for unstructured business information. Candidate content can include policies, product documentation, internal procedures, FAQs, business rules, and other relevant documents.

Introduce a retrieval capability such as:

```text
searchKnowledgeBase(query)
```

The LLM should select this capability only when unstructured knowledge is necessary, for example for a return, refund, or warranty policy question. The specific vector store, embedding model, ingestion process, metadata strategy, and document sources are not established by the README and must be selected in a later implementation design.

```text
Structured business data
				↓
Database/API tools

Unstructured knowledge
				↓
RAG / knowledge retrieval
```

## Phase 5 — Multi-Source Retrieval & Synthesis

Eventually support questions that require more than one source, such as identifying customers who purchased a product and then applying the current return-policy context.

The target workflow is:

1. Understand the user's intent.
2. Determine the information required.
3. Select the relevant tool or tools.
4. Call one or more tools through the application-controlled execution layer.
5. Inspect tool results.
6. Decide whether additional retrieval is necessary.
7. Synthesize the grounded information.
8. Produce the final response.

For example:

```text
User
 ↓
LLM
 ↓
Customer/order/product tools
 ↓
Structured database results
 ↓
Knowledge-base retrieval
 ↓
Policy context
 ↓
LLM synthesis
 ↓
Final response
```

This is a future target architecture, not a requirement to implement autonomous planning or unrestricted action execution in the first iteration.

## Architectural Principles

1. **No unnecessary retrieval.** Greetings, thanks, jokes, and similar conversational requests should not trigger database queries or RAG.
2. **Tool selection is model-driven.** The LLM determines whether an available capability is relevant to the user's request.
3. **Tool execution is application-controlled.** The model never receives unrestricted database access. The application validates, authorises, executes, and bounds every tool call.
4. **Structured and unstructured retrieval are different.** Use database/application tools for structured business data and RAG for unstructured knowledge.
5. **Retrieval is observable.** Preserve an inspectable record of the user query, selected tool, tool arguments, tool result, additional tool calls, and final response. This supports analysis of retrieval decisions, not only answer quality.
6. **Build incrementally.** Progress from chat to tool calling, structured retrieval, RAG, multi-tool retrieval, and evaluation. Do not jump directly to an autonomous agent.
7. **Grounded responses.** When a response depends on retrieved information, the system should make the LLM's answer traceable to authorised tool output or retrieved knowledge context.

## Evaluation Strategy

The eventual evaluation should measure whether the model selected the appropriate retrieval path, in addition to whether its response is correct, useful, and appropriately bounded by authorisation.

### No Retrieval Required

Examples:

```text
"Hi"
"Thanks"
"Tell me a joke"
```

Expected result:

```text
No tool call
```

### Structured Retrieval Required

Examples:

```text
"How many products are in stock?"
"Show me the orders for customer X."
"What did customer Y purchase?"
```

Expected result:

```text
Database/application tool
```

The availability and semantics of stock-specific retrieval must be confirmed from the application data model before this example becomes an executable test.

### RAG Required

Examples:

```text
"What is our refund policy?"
"What does our warranty policy say?"
```

Expected result:

```text
Knowledge retrieval
```

### Multiple Retrieval Sources

Example:

```text
"Which customers bought product X and are affected by our return policy?"
```

Expected result:

```text
Structured retrieval
+
Knowledge retrieval
```

Evaluation records should capture the query category, expected retrieval decision, selected tools and arguments, execution and authorisation outcome, result sufficiency, final response quality, and unnecessary-retrieval rate. Build a labelled evaluation set gradually from representative ProStore workflows and approved knowledge-base documents.

## Next Immediate Steps

1. Review the existing application code and data model to map the documented product, order, user, cart, and review capabilities to their actual authorisation and data-access boundaries.
2. Define the Phase 1 conversational interface and its identity/access behaviour without adding retrieval.
3. Select and configure the LLM integration appropriate for the application, including a server-side request boundary and basic conversation observability.
4. Draft narrow tool contracts for the first structured business questions, including input schemas, authorisation rules, result limits, error behaviour, and audit fields.
5. Implement the minimum tool-calling loop with mock or non-sensitive controlled results before connecting to live structured data.
6. Connect the first validated tools to PostgreSQL through application-controlled access paths, then add focused tests for no-tool and correct-tool decisions.
7. Define the knowledge-base document sources, ownership, ingestion process, and retrieval evaluation criteria only after structured retrieval is working.
8. Add RAG as a separately selectable capability, then evaluate single-source and multi-source retrieval decisions with observable traces.
