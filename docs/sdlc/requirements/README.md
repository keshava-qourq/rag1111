# Product Requirements

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start narrative -->
## Summary

### REQ-TECH-STACK · approved · source: requirements:tech-stack

Product owner approved an exact technology stack: FastAPI 0.115 backend, AWS cloud, PostgreSQL 16 database, React 19 frontend. Designs/builds must use exactly these, no substitutions.

Backend: FastAPI 0.115 -- chosen because the RAG pipeline (upload parsing, LLM calls, embedding/retrieval) is Python-native and FastAPI natively supports chat streaming. Cloud: AWS -- hosts the app, file storage, and database for a small internal team; S3 pairs with document upload; scales down cheaply with room to grow. Database: PostgreSQL 16 -- stores users, roles, documents, and private chat history, and via pgvector also provides semantic search, keeping structured data and vector search in one system; deemed ample for a few hundred documents and 50 users. Frontend: React 19 -- builds the responsive chat web app with streaming answers, markdown rendering, and citation links.

**Related:** EPIC-001, EPIC-002, EPIC-003, EPIC-004, EPIC-005

### REQ-SERVICES · approved · source: requirements:runtime-needs

Product owner approved exact runtime services: pgvector for vector search, OpenAI embeddings, Amazon S3 for file storage, Redis as a background queue, and SMTP for account emails.

Vector store: pgvector (in PostgreSQL, known) -- semantic search over document chunks without an extra system. Embeddings: OpenAI embeddings (saas, env OPENAI_API_KEY) -- used to embed documents and questions for retrieval. File storage: Amazon S3 (saas, env AWS_ACCESS_KEY_ID/AWS_SECRET_ACCESS_KEY/AWS_REGION/S3_BUCKET) -- durable storage of original uploaded files, re-processable if indexing changes. Queue: Redis (container redis:7-alpine) -- background processing so uploads return immediately while indexing happens asynchronously. Email: SMTP server (saas, env SMTP_HOST/SMTP_PORT/SMTP_USER/SMTP_PASSWORD) -- sends invitation and password-setup emails. Full required API keys/env vars: AWS_ACCESS_KEY_ID, AWS_REGION, AWS_SECRET_ACCESS_KEY, OPENAI_API_KEY, REDIS_URL, S3_BUCKET, SMTP_HOST, SMTP_PASSWORD, SMTP_PORT, SMTP_USER.

**Related:** EPIC-001, EPIC-002, EPIC-005, US-002, US-010

- ⚠ Discrepancy: the out-of-scope list explicitly states 'AI provider (the suggested OpenAI was removed from the stack)', and EPIC-005 states the LLM/embeddings provider was removed and requirements are written provider-agnostic (admin configures whichever hosted provider via API key) -- yet the approved services list and capabilities list still name 'OpenAI embeddings' with env var OPENAI_API_KEY as an included, approved service. This is a direct conflict between 'OpenAI removed' and 'OpenAI embeddings approved' and needs product-owner clarification on whether OpenAI specifically, or a generic hosted provider, is actually to be used.

### REQ-US-001 · draft · source: requirements:US-001

Admins upload PDF, Word, or plain-text documents into the knowledge base; unsupported formats are rejected with a clear message; staff without admin rights cannot access upload.

AC-001: Signed-in admin uploads a PDF, Word (.doc/.docx), or plain-text file -> file is accepted, stored, and queued for indexing with a visible status. AC-002: Admin attempts to upload an unsupported format -> upload rejected, message lists supported formats (PDF, Word, plain text). AC-003: A staff user without admin rights tries to access upload -> access denied, upload capability not available. Related NFRs: admin-only restriction (NFR-002), background processing so upload returns immediately (NFR-004), durable original-file storage outside the database for re-indexing (NFR-007).

**Related:** EPIC-001, AC-001, AC-002, AC-003, NFR-002, NFR-004, NFR-007

### REQ-US-002 · draft · source: requirements:US-002

Uploaded documents are text-extracted, split, embedded, and added to the search index; failures are marked with a reason; indexing requires a valid provider API key.

AC-004: Indexing extracts text, splits, embeds, and adds content to the search index; status shows 'indexed' when complete. AC-005: A corrupt/unreadable document is marked failed/error with a reason and is not treated as indexed. AC-006: If no valid embeddings-provider API key is configured, indexing must not silently succeed -- the admin is shown that provider configuration is required. Background processing per NFR-004 so large uploads don't hang the app.

**Related:** EPIC-001, AC-004, AC-005, AC-006, NFR-004

### REQ-US-003 · draft · source: requirements:US-003

Admins can view a list of all documents in the knowledge base and delete outdated ones, which also removes them from the search index.

AC-007: Document library shows each document with at least name, format, upload date, and indexing status. AC-008: Deleting a document removes it from the library and from the search index so future answers no longer use it. Restricted to admins per NFR-002.

**Related:** EPIC-001, AC-007, AC-008, NFR-002

### REQ-US-004 · draft · source: requirements:US-004

Staff type a question in chat and receive a text answer grounded in indexed document content; all authenticated staff can query the whole knowledge base; interaction is text-only.

AC-009: Signed-in staff user submits a question with relevant content in the knowledge base -> receives a text answer derived from indexed content. AC-010: Any authenticated user's query runs against the entire knowledge base with no per-document or per-team restriction (explicit assumption/accepted risk, see NFR-009). AC-011: Chat is text-based only, no voice input/output. Answers stream into the chat as generated (NFR-008).

**Related:** EPIC-002, AC-009, AC-010, AC-011, NFR-008, NFR-009

### REQ-US-005 · draft · source: requirements:US-005

Every chatbot answer cites the source document(s) it was drawn from, including all contributing documents when more than one is used.

AC-012: Displayed answer shows a citation identifying the source document(s). AC-013: When an answer draws on more than one document, all contributing documents are cited.

**Related:** EPIC-002, AC-012, AC-013

### REQ-US-006 · draft · source: requirements:US-006

When the knowledge base doesn't contain the answer, the chatbot clearly says it does not know instead of guessing or citing unrelated sources.

AC-014: If a question's answer is not present in any indexed document, the chatbot states it does not know / the information isn't in the knowledge base, and does not cite unrelated sources.

**Related:** EPIC-002, AC-014

### REQ-US-007 · draft · source: requirements:US-007

Each user's chat conversations are saved and can be revisited after signing back in; conversations are private to the owning user.

AC-015: Signed-in user opens chat history -> sees past conversations and can open one to view its questions and answers. AC-016: A user viewing chat history only sees their own conversations, never another user's (private, per NFR-001).

**Related:** EPIC-003, AC-015, AC-016, NFR-001

### REQ-US-008 · draft · source: requirements:US-008

Staff sign in with email and password; correct credentials grant access, incorrect credentials are rejected; unauthenticated users cannot reach any knowledge-base content.

AC-017: Correct email/password for a provisioned account -> signed in, can access chat. AC-018: Incorrect email or password -> not signed in, error message shown. AC-019: Not signed in -> redirected to sign-in, cannot access chat or document features (NFR-003). No SSO; in-app email/password accounts only.

**Related:** EPIC-004, AC-017, AC-018, AC-019, NFR-003

### REQ-US-009 · draft · source: requirements:US-009

Admins invite/create user accounts, assign admin or staff roles, and can remove or deactivate users to revoke access.

AC-020: Admin invites/creates a user -> granted access with admin role (docs + users) or staff role (chat only). AC-021: Admin removes/deactivates a user -> that user can no longer sign in or access the knowledge base. AC-022: Staff user without admin rights is denied access to user management (NFR-002).

**Related:** EPIC-004, AC-020, AC-021, AC-022, NFR-002

### REQ-US-010 · draft · source: requirements:US-010

Admins configure the hosted LLM provider's API key, which is required for indexing and answer generation to function at all.

AC-023: Admin provides a valid provider API key -> indexing and answer generation use that key and function correctly. AC-024: No valid API key configured -> system reports the LLM provider is not configured when a user asks a question or an admin tries to index, rather than failing silently. Note: the specific provider was originally suggested as OpenAI but was removed from the stack; requirements are written provider-agnostic (see REQ-SERVICES open_gap for the resulting discrepancy with the approved services list).

**Related:** EPIC-005, AC-023, AC-024

### REQ-NFR-SYSTEM · draft · source: requirements:NFR-005,NFR-006

System-wide non-functional requirements: sized for a small internal team (~50 staff, a few hundred documents) and delivered solely as a responsive web app.

NFR-005 [scalability]: Must comfortably handle up to ~50 staff users and a few hundred documents. NFR-006 [compatibility]: Delivered as a single responsive web application adapting to different screen sizes; no native mobile or desktop app (confirmed out of scope).

**Related:** NFR-005, NFR-006
<!-- sdlc:auto:end narrative -->

<!-- sdlc:auto:start summary -->
**Requirements version:** v1  
**Delivery scope:** lightweight  
**Approved:** 2026-11-04

**Capabilities in scope:** Document-grounded question answering in chat, Source document citations on each answer, "I don't know" response when the answer isn't in the knowledge base, Document upload and indexing (PDF, Word, plain text), Staff chat interface for asking questions, Persistent, private, revisitable chat history, User sign-in, Document library listing, OpenAI embeddings for indexing and search (requires an API key)

**Approved technology stack:**

| Layer | Choice | Rationale |
|---|---|---|
| backend | FastAPI | The heart of this product is the RAG pipeline — parsing uploads, calling the LLM, embedding and retrieving document chunks. That work lives in Python, where the AI and document-handling libraries are strongest. FastAPI lets your retrieval and chat endpoints sit in the same language as that pipeline, and it natively supports the streaming responses a chat interface expects. |
| cloud | AWS | You need somewhere to run the app, hold the uploaded files, and keep the database for a small internal team. AWS covers all of it, and its S3 storage pairs naturally with the document-upload feature at the centre of this product. It scales down to a small footprint cheaply and gives you room to grow without changing providers. |
| database | PostgreSQL | You need to store users, roles, documents and each user's private chat history reliably — and you also need vector search over document chunks. PostgreSQL covers the structured data and, via the pgvector extension, the semantic search too, so your whole knowledge base lives in one system instead of two. For a few hundred documents and 50 users that is ample headroom. |
| frontend | React | Your product is a responsive web chat app where staff type questions, watch answers stream in, and revisit past conversations. React handles exactly this kind of live, interactive interface well, and its large ecosystem means chat UI, markdown rendering of answers, and citation links are all well-trodden ground — which helps you reach the answer-quality bar you set without reinventing the shell around it. |
<!-- sdlc:auto:end summary -->

<!-- sdlc:auto:start epics -->
## EPIC-001: Document Upload, Indexing & Library

Admins build and maintain the knowledge base by uploading company documents (PDF, Word, plain text). Uploaded documents are indexed (text extracted and embedded) so staff questions can be answered from their content. Admins can view and manage the document library. Scope is a small team: a few hundred documents.

### US-001: Upload documents to the knowledge base

**Priority:** critical

> As a admin, I want to upload PDF, Word, and plain text documents into the app, so that their content becomes part of the knowledge base staff can query.

| ID | Given | When | Then |
|---|---|---|---|
| AC-001 | I am signed in as an admin | I upload a file in PDF, Word (.doc/.docx), or plain text format | the file is accepted, stored, and queued for indexing with a visible status |
| AC-002 | I am signed in as an admin | I attempt to upload a file in an unsupported format | the upload is rejected and I see a message listing the supported formats (PDF, Word, plain text) |
| AC-003 | I am a staff user without admin rights | I try to access the document upload function | access is denied and the upload capability is not available to me |

### US-002: Index uploaded documents for search

**Priority:** critical

> As a admin, I want uploaded documents to have their text extracted and embedded into a searchable index, so that the chatbot can retrieve relevant passages when answering questions.

| ID | Given | When | Then |
|---|---|---|---|
| AC-004 | a supported document has been uploaded | indexing runs | the document's text is extracted, split, embedded, and added to the search index, and its status shows as indexed when complete |
| AC-005 | a document fails to index (e.g. it is corrupt or unreadable) | indexing finishes | the document is marked with a failed/error status and a reason, and it is not treated as indexed |
| AC-006 | indexing uses embeddings from the configured provider | no valid provider API key is configured | indexing does not silently succeed and the admin is shown that a provider configuration is required (assumption: embeddings/answer generation use a hosted LLM provider requiring an API key) |

### US-003: View and manage the document library

**Priority:** high

> As a admin, I want to see a list of all documents in the knowledge base and remove ones that are outdated, so that I can keep the knowledge base accurate and relevant.

| ID | Given | When | Then |
|---|---|---|---|
| AC-007 | documents have been uploaded | I open the document library | I see each document listed with at least its name, format, upload date, and indexing status |
| AC-008 | I am viewing the document library as an admin | I delete a document | it is removed from the library and its content is removed from the search index so future answers no longer use it |

## EPIC-002: Document-Grounded Question Answering

Staff ask questions in a text chat interface. The chatbot answers using only the content of the indexed documents, grounds its answers in the source material, cites which document(s) each answer came from, and explicitly says it doesn't know when the answer isn't in the knowledge base rather than guessing. All authenticated staff can query the entire knowledge base with no per-document or per-team restrictions. Interaction is text-based only — no voice.

### US-004: Ask a question and get a grounded answer

**Priority:** critical

> As a staff user, I want to type a question in the chat and receive an answer based on the company documents, so that I can quickly find information from company documents, policies, and procedures.

| ID | Given | When | Then |
|---|---|---|---|
| AC-009 | I am signed in as a staff user and relevant content exists in the knowledge base | I submit a question in the chat | I receive a text answer derived from the content of the indexed documents |
| AC-010 | I am any authenticated user | I ask a question | my query runs against the entire knowledge base with no per-document or per-team access restriction (assumption: all authenticated staff can query everything) |
| AC-011 | I am in the chat interface | I use it | interaction is text-based only, with no voice input or output option |

### US-005: See source document citations on answers

**Priority:** critical

> As a staff user, I want each answer to cite the document(s) it came from, so that I can trust the answer and verify it against the original source.

| ID | Given | When | Then |
|---|---|---|---|
| AC-012 | the chatbot answers a question from the knowledge base | the answer is displayed | it shows a citation identifying the source document(s) the answer was drawn from |
| AC-013 | an answer draws on more than one document | the answer is displayed | all contributing source documents are cited |

### US-006: Decline when the answer is not in the knowledge base

**Priority:** critical

> As a staff user, I want the chatbot to tell me it doesn't know when the documents don't contain the answer, so that I am not misled by a fabricated or ungrounded answer.

| ID | Given | When | Then |
|---|---|---|---|
| AC-014 | I ask a question whose answer is not present in any indexed document | the chatbot responds | it clearly states it does not know / the information isn't in the knowledge base rather than guessing, and does not cite unrelated sources |

## EPIC-003: Private Persistent Chat History

Each user has their own persistent, private chat history they can revisit. Conversations are not shared between users.

### US-007: Revisit my past conversations

**Priority:** high

> As a staff user, I want my chat conversations to be saved so I can return to them later, so that I can refer back to previous answers without re-asking.

| ID | Given | When | Then |
|---|---|---|---|
| AC-015 | I have had previous conversations in the chat | I sign in again and open my chat history | I see my past conversations and can open one to view its questions and answers |
| AC-016 | I am a signed-in user | I view chat history | I only see my own conversations and cannot see any other user's conversations (assumption: chat history is private and not shared between users) |

## EPIC-004: User Sign-In, Roles & Provisioning

Staff authenticate with in-app email/password accounts. Two roles exist: admins (upload/manage documents and manage users) and staff (chat only). Admins invite or provision users. SSO is not included. This is an internal tool with no payment, billing, or subscription features.

### US-008: Sign in to the application

**Priority:** critical

> As a staff user, I want to sign in with my email and password, so that only authorised staff can access the knowledge base and my chat history.

| ID | Given | When | Then |
|---|---|---|---|
| AC-017 | I have an account provisioned in the app | I enter my correct email and password | I am signed in and can access the chat interface |
| AC-018 | I enter an incorrect email or password | I attempt to sign in | I am not signed in and I see an error message (assumption: in-app email/password accounts; SSO not included) |
| AC-019 | I am not signed in | I try to access the chat or document features | I am redirected to sign in and cannot access any knowledge-base content |

### US-009: Provision and manage user accounts

**Priority:** high

> As a admin, I want to invite or create user accounts and assign the admin or staff role, so that the right people can access the tool with the right permissions.

| ID | Given | When | Then |
|---|---|---|---|
| AC-020 | I am signed in as an admin | I invite or create a new user | the user is granted access and assigned either the admin role (upload/manage documents and users) or the staff role (chat only) |
| AC-021 | I am signed in as an admin | I remove or deactivate a user | that user can no longer sign in or access the knowledge base |
| AC-022 | I am a staff user without admin rights | I try to access user management | access is denied (assumption: two roles — admins manage docs and users, staff chat only) |

## EPIC-005: LLM Provider Configuration

The app relies on a hosted LLM provider for embeddings (indexing/search) and answer generation, configured via an API key. Note: the originally suggested OpenAI provider was removed from the stack, so requirements are written provider-agnostic — an admin configures whichever hosted provider is used via its API key. Surfaced as an assumption because the specific provider was not fixed.

### US-010: Configure the hosted LLM provider API key

**Priority:** high

> As a admin, I want to configure the API key for the hosted LLM provider used for embeddings and answer generation, so that indexing and question answering can function.

| ID | Given | When | Then |
|---|---|---|---|
| AC-023 | I am an admin | I provide a valid provider API key in configuration | indexing and answer generation use that key and function correctly |
| AC-024 | no valid provider API key is configured | a user tries to ask a question or an admin tries to index a document | the system reports that the LLM provider is not configured rather than failing silently (assumption: a hosted LLM provider with an API key is required; the specific provider, originally suggested as OpenAI, was removed from the stack and is left configurable) |
<!-- sdlc:auto:end epics -->
