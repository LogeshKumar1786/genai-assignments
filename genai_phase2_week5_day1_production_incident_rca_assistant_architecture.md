# Production Incident & Root Cause Analysis Assistant

## 1. Overview

The **Production Incident & Root Cause Analysis Assistant** is an
AI-powered incident triage solution designed to reduce the manual effort
required to investigate production P1 incidents.

When a **P1 alert is triggered in DataDog**, DataDog sends the alert
information and application logs to the assistant. The assistant also
retrieves the relevant **scheduler logs**. It then searches historical
RCA knowledge using a hybrid retrieval approach, identifies similar
incidents, correlates current evidence with historical patterns,
determines a probable root cause, and presents the analysis in a React
dashboard.

The solution is designed to assist engineers with investigation and
troubleshooting. The final root cause and recommended actions remain
subject to engineering verification.

------------------------------------------------------------------------

## 2. Problem Statement

During a production/API failure, QA, support, and engineering teams
typically need to manually inspect:

-   Error messages
-   Application/API logs
-   Scheduler logs
-   DataDog logs and alerts
-   Historical incidents
-   RCA documents
-   Jira records
-   Runbooks
-   Previously known solutions

This process can be time-consuming, especially for P1 incidents where
rapid triage is required.

### Objective

Build an AI-assisted incident analysis system that:

1.  Automatically receives P1 incidents from DataDog.
2.  Analyzes application/API logs.
3.  Retrieves and analyzes scheduler logs.
4.  Searches historical RCA knowledge.
5.  Identifies genuinely similar incidents.
6.  Correlates current evidence with historical patterns and rules.
7.  Generates a probable root cause with measurable confidence.
8.  Provides evidence and recommended troubleshooting actions.
9.  Displays the complete investigation in a dashboard.
10. Updates the RCA knowledge base when a genuinely new error is
    identified.

------------------------------------------------------------------------

# 3. Scope

## In Scope

-   DataDog P1 alert integration
-   Application/API log analysis
-   Scheduler log analysis
-   Historical RCA ingestion
-   RAG-based RCA retrieval
-   Hybrid Vector + BM25 search
-   Query preprocessing
-   Reranking for P1 incidents
-   LLM-based similarity evaluation
-   Root-cause synthesis
-   Evidence correlation
-   Confidence calculation
-   Recommended troubleshooting actions
-   RCA duplicate detection
-   Automatic insertion of genuinely new RCAs
-   React dashboard
-   Investigation and action tracking
-   REST APIs
-   DataDog webhook/event endpoint
-   Configurable LLM provider
-   Configurable embedding provider
-   AWS containerized deployment

## Out of Scope

-   Automatic production remediation without engineer approval
-   Storing every production log permanently in the RAG/vector database
-   Replacing DataDog as the monitoring platform
-   Replacing human engineering verification
-   Automatic execution of destructive production actions

------------------------------------------------------------------------

# 4. High-Level Architecture

``` text
                         +----------------------+
                         |      DataDog         |
                         |   P1 Alert Trigger   |
                         +----------+-----------+
                                    |
                                    | P1 Webhook/Event
                                    v
                    +---------------+----------------+
                    | Production Incident Assistant |
                    |       Node.js / TypeScript    |
                    +---------------+----------------+
                                    |
                    +---------------+----------------+
                    |                                |
                    v                                v
          +-------------------+             +-------------------+
          | Application/API   |             | Scheduler Logs    |
          | Logs from DataDog |             | Log Retrieval     |
          +---------+---------+             +---------+---------+
                    |                                 |
                    +---------------+-----------------+
                                    |
                                    v
                         +----------------------+
                         | Incident Processing  |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | Query Preprocessing  |
                         | - Normalization      |
                         | - Abbreviation       |
                         | - Synonym expansion  |
                         +----------+-----------+
                                    |
                                    v
                    +---------------+----------------+
                    |       Hybrid Retrieval         |
                    |                                |
                    | Vector Search + BM25 Search   |
                    +---------------+----------------+
                                    |
                                    v
                         +----------------------+
                         | P1 Reranking Layer   |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | Top 3 Similar RCAs   |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         |         LLM          |
                         | Similarity + RCA      |
                         | Synthesis             |
                         +----------+-----------+
                                    |
                                    v
                    +---------------+----------------+
                    | Structured Incident Analysis |
                    +---------------+----------------+
                                    |
                     +--------------+---------------+
                     |                              |
                     v                              v
          +----------------------+       +----------------------+
          | React Dashboard      |       | RCA Knowledge Update |
          | Investigation UI     |       | New Error Detection  |
          +----------------------+       +----------------------+
                                                   |
                                                   v
                                          +----------------------+
                                          | MongoDB Vector Store |
                                          +----------------------+
```

------------------------------------------------------------------------

# 5. End-to-End Incident Flow

## Step 1 --- P1 Alert

A P1 alert is triggered in DataDog.

DataDog sends:

-   Incident ID
-   Alert title
-   Error message
-   Service/component information
-   Environment
-   Timestamp
-   Application/API logs associated with the alert

The DataDog event is received through:

``` text
POST /api/webhooks/datadog/p1
```

------------------------------------------------------------------------

## Step 2 --- Incident Creation

The backend creates or updates the incident record.

The system supports both:

-   Manual incident creation from the React dashboard
-   API-based incident creation

Example:

``` text
POST /api/incidents
```

A minimal incident can also be accepted, such as:

``` text
Incident ID
Error message
```

The system can enrich missing information from available operational
data.

------------------------------------------------------------------------

# 6. Log Analysis

## 6.1 Application/API Logs

For a P1 incident, DataDog provides application/API logs.

The assistant analyzes these logs to identify:

-   HTTP status codes
-   Exception messages
-   Timeout patterns
-   API failures
-   Service/component failures
-   Error frequency
-   Failure timestamps
-   Transaction identifiers
-   Dependency failures

Application logs are treated as **incident-specific evidence**.

They are not automatically inserted into the historical RCA vector
database.

------------------------------------------------------------------------

## 6.2 Scheduler Logs

Scheduler logs are retrieved and analyzed for every P1 incident.

This applies regardless of whether the incident initially appears to be:

-   A realtime API issue
-   A scheduler issue

This allows the assistant to identify relationships such as:

``` text
Realtime API Failure
        |
        +---- Scheduler Failure
        |
        +---- Dependency Failure
```

Scheduler evidence can help determine whether a scheduler triggered,
retried, failed, or propagated the incident.

------------------------------------------------------------------------

# 7. Historical RCA Knowledge Base

Historical RCA documents are ingested into the RAG system during the
initial setup.

Typical sources include:

-   Historical RCA documents
-   Incident records
-   Known resolutions
-   Troubleshooting guides
-   Runbooks
-   Relevant Jira documentation

The historical RCA knowledge base is persistent and is used for future
incident retrieval.

------------------------------------------------------------------------

# 8. RAG Ingestion Pipeline

``` text
Historical RCA Documents
          |
          v
      Document Parser
          |
          v
       Chunking
          |
          v
      Text Cleaning
          |
          v
       Embedding
          |
          v
 MongoDB Vector Store
```

Each RCA document is converted into searchable chunks.

Each chunk contains metadata such as:

``` json
{
  "rcaId": "RCA-102",
  "incidentId": "INC-9871",
  "errorCode": "500",
  "service": "Lead Processing",
  "source": "Historical RCA",
  "content": "Salesforce API timeout during lead processing..."
}
```

------------------------------------------------------------------------

# 9. Query Preprocessing

Before historical RCA retrieval, the incident query is normalized.

The selected approach is:

**Normalization + Abbreviation Expansion + Synonym Expansion**

Example:

``` text
Original:
SF API timeout while processing leads

Normalized:
Salesforce API request timeout during lead processing
```

Processing may include:

### Normalization

-   Remove unnecessary noise
-   Standardize casing
-   Normalize error formats

### Abbreviation Expansion

Examples:

``` text
SF  -> Salesforce
API -> Application Programming Interface
DB  -> Database
```

### Synonym Expansion

Examples:

``` text
timeout
request timeout
response timeout
API timeout
```

This improves retrieval quality.

------------------------------------------------------------------------

# 10. Hybrid Retrieval

Historical RCA retrieval uses:

``` text
Vector Search + BM25 Search
```

## Vector Search

Vector search identifies semantically similar incidents.

It is useful when the wording is different but the underlying failure
pattern is similar.

Example:

``` text
"Salesforce request exceeded response limit"
```

may retrieve:

``` text
"Salesforce API request timed out"
```

------------------------------------------------------------------------

## BM25 Search

BM25 provides keyword-based matching.

It is particularly useful for:

-   Error codes
-   Exception names
-   API names
-   Service names
-   Scheduler names
-   Specific technical terms

Example:

``` text
HTTP 500
Salesforce
TimeoutException
Lead Scheduler
```

------------------------------------------------------------------------

## Hybrid Search Flow

``` text
Incident Query
      |
      +------------------+
      |                  |
      v                  v
Vector Search         BM25 Search
      |                  |
      +--------+---------+
               |
               v
        Combined Results
```

------------------------------------------------------------------------

# 11. Reranking

Reranking is applied for P1 incidents.

The hybrid retrieval layer produces candidate historical RCAs.

The reranker evaluates relevance using factors such as:

-   Error type
-   Error message
-   Error code
-   Service/component
-   API
-   Scheduler
-   Failure pattern
-   Environment
-   Historical resolution

The output is the most relevant candidate set.

------------------------------------------------------------------------

# 12. Top 3 RCA Selection

The reranking layer provides the top three candidate incidents.

The LLM then evaluates these candidates rather than blindly accepting
all three.

The LLM compares:

-   Current error
-   Current logs
-   Current service/component
-   API/scheduler behavior
-   Historical error
-   Historical failure pattern
-   Historical resolution

The LLM determines which candidate incidents are genuinely similar.

``` text
Hybrid Search
      |
      v
Reranking
      |
      v
Top 3 RCAs
      |
      v
LLM Similarity Evaluation
      |
      v
Genuine Similar Incidents
```

------------------------------------------------------------------------

# 13. Root Cause Determination

Root cause determination uses a combination of:

1.  Rules
2.  Historical RCA similarity
3.  Current application/API logs
4.  Current scheduler logs
5.  Known failure patterns
6.  Evidence correlation
7.  LLM synthesis

The flow is:

``` text
P1 Alert
   |
   +---- Application/API Logs
   |
   +---- Scheduler Logs
   |
   +---- Historical RCA
   |
   +---- Known Rules/Patterns
   |
   v
Evidence Correlation
   |
   v
LLM Synthesis
   |
   v
Probable Root Cause
```

The system should clearly identify the result as a **probable root
cause** unless verification has confirmed it.

------------------------------------------------------------------------

# 14. Confidence Calculation

Confidence should not be an arbitrary value generated by the LLM.

A hybrid confidence mechanism is used.

Potential confidence factors include:

-   Historical RCA similarity
-   Error code match
-   API/service match
-   Scheduler match
-   Supporting log evidence
-   Known rule/pattern match
-   Number and quality of supporting evidence sources
-   Conflicting evidence

Example conceptual calculation:

``` text
Historical RCA Match       -> 25%
Error/Exception Match      -> 20%
Service/API Match          -> 15%
Scheduler Evidence         -> 15%
Current Log Evidence       -> 15%
Known Rule Match           -> 10%
```

The exact weights can be configured and validated during implementation.

The LLM interprets the evidence and explains the root cause, while the
confidence engine provides a measurable confidence value.

------------------------------------------------------------------------

# 15. Evidence Management

Every important conclusion should be traceable to evidence.

Example:

``` json
{
  "source": "Application Logs",
  "detail": "Salesforce API returned HTTP 500 during lead processing",
  "reference": "datadog://incident/INC-10245/log/12345"
}
```

Other evidence sources may include:

``` text
Application Logs
Scheduler Logs
Historical RCA
Runbook
Known Rule
DataDog Incident
```

This improves traceability and allows engineers to validate the
AI-generated analysis.

------------------------------------------------------------------------

# 16. Recommended Actions

Recommended actions use two sources.

## Historical Actions

If a similar historical RCA contains a proven troubleshooting or
resolution procedure, those actions can be reused where applicable.

Example:

``` text
1. Check Salesforce API availability.
2. Review API response latency.
3. Identify failed transactions.
4. Retry failed transactions after service recovery.
```

## AI-Generated Current Actions

The LLM may generate additional actions based on the current incident
evidence.

For example:

``` text
1. Check whether the timeout started after a deployment.
2. Compare current API latency against normal baseline.
3. Verify scheduler execution status.
4. Identify the affected transaction range.
```

The dashboard allows engineers to track these actions.

------------------------------------------------------------------------

# 17. Verification Logic

The output includes:

``` json
{
  "verificationRequired": true
}
```

Verification is required when:

-   Confidence is below the configured threshold
-   Evidence is insufficient
-   Evidence is conflicting
-   No sufficiently similar historical RCA exists
-   The root cause cannot be established from available evidence

The system should not present an uncertain hypothesis as a confirmed
production root cause.

------------------------------------------------------------------------

# 18. RCA Knowledge Base Update

After incident analysis, the generated RCA is compared with existing RCA
knowledge.

The refined lifecycle is:

``` text
AI Generated RCA
       |
       v
Compare with Existing RCA Knowledge
       |
       +-----------------------+
       |                       |
       v                       v
Existing Error            New Error
       |                       |
       v                       v
Do Not Duplicate          Insert New RCA
                               |
                               v
                       MongoDB Vector Store
```

A new RCA is added only when the error is determined to be genuinely
new.

This avoids unnecessary duplication of historical RCAs.

------------------------------------------------------------------------

# 19. Structured Output

The assistant produces a structured incident analysis.

Example:

``` json
{
  "incidentId": "INC-10245",
  "summary": "Salesforce API timeout during lead processing",
  "impact": {
    "environment": "PROD",
    "affectedRecords": 127,
    "affectedService": "Lead Processing"
  },
  "errorAnalysis": {
    "errorCode": "500",
    "errorType": "Salesforce API Timeout",
    "component": "Lead Processing Scheduler"
  },
  "similarIncidents": [
    {
      "incidentId": "INC-9871",
      "similarity": 0.91,
      "resolution": "Retry failed transactions after API recovery"
    }
  ],
  "probableRootCause": {
    "description": "Salesforce API response latency caused request timeouts",
    "confidence": 0.87,
    "verificationRequired": true
  },
  "recommendedActions": [
    "Check Salesforce API availability",
    "Review API response latency",
    "Verify scheduler logs",
    "Identify failed transactions",
    "Retry failed transactions after recovery"
  ],
  "evidence": [
    "Production API logs",
    "Scheduler logs",
    "INC-9871",
    "Salesforce API failure runbook"
  ],
  "status": "TRIAGE_COMPLETED"
}
```

------------------------------------------------------------------------

# 20. React Dashboard

The React dashboard is the primary interface for engineers.

The dashboard should display:

## Incident Summary

-   Incident ID
-   Severity
-   Environment
-   Timestamp
-   Affected service
-   Impact

## Error Analysis

-   Error code
-   Error type
-   Exception
-   Component
-   API
-   Scheduler

## Application/API Logs

Display relevant log entries received from DataDog.

## Scheduler Logs

Display scheduler execution and failure details.

## Similar Incidents

Show:

-   Incident ID
-   Similarity
-   Historical resolution
-   Source reference

## Probable Root Cause

Show:

-   Root cause description
-   Confidence
-   Verification status

## Evidence

Show each evidence item with its source/reference.

## Recommended Actions

Engineers can:

-   View actions
-   Mark actions as completed
-   Update action status
-   Track investigation progress

------------------------------------------------------------------------

# 21. API Architecture

The API architecture is hybrid.

## REST APIs

Used for normal application functionality.

### Incident APIs

``` text
POST /api/incidents
POST /api/incidents/analyze

GET /api/incidents/{incidentId}
GET /api/incidents/{incidentId}/similar
GET /api/incidents/{incidentId}/evidence
GET /api/incidents/{incidentId}/logs
```

### Action APIs

``` text
GET   /api/incidents/{incidentId}/actions
PATCH /api/incidents/{incidentId}/actions/{actionId}
```

### RCA APIs

``` text
POST /api/rca
GET  /api/rca/{rcaId}
```

### DataDog Integration

``` text
POST /api/webhooks/datadog/p1
```

This endpoint receives P1 alert events from DataDog.

------------------------------------------------------------------------

# 22. Example Incident API Flow

``` text
DataDog
   |
   | POST /api/webhooks/datadog/p1
   v
Incident Controller
   |
   v
Incident Service
   |
   +---- Store Incident
   |
   +---- Analyze Application Logs
   |
   +---- Retrieve Scheduler Logs
   |
   v
Query Preprocessor
   |
   v
Hybrid Retrieval
   |
   v
Reranker
   |
   v
Top 3 Historical RCAs
   |
   v
LLM
   |
   v
Root Cause + Evidence + Actions
   |
   v
Dashboard
```

------------------------------------------------------------------------

# 23. Technology Stack

  Layer                Technology
  -------------------- ---------------------------------
  Frontend             React.js
  Backend              Node.js / TypeScript
  API                  REST + DataDog Webhook
  Vector/Data Store    MongoDB
  LLM                  Configurable OpenAI / Groq
  Embeddings           Configurable Mistral / OpenAI
  Monitoring           DataDog
  Deployment           AWS Containerized Deployment
  RAG                  Vector Search + BM25
  Search Enhancement   Query preprocessing + Reranking

------------------------------------------------------------------------

# 24. LLM Provider Architecture

The LLM provider should be abstracted.

Supported providers:

``` text
OpenAI
Groq
```

Conceptual interface:

``` typescript
interface LLMProvider {
  generateResponse(
    prompt: string,
    context: string
  ): Promise<LLMResponse>;
}
```

Provider selection can be configuration-driven.

This allows the application to switch between providers without changing
the core incident-processing workflow.

------------------------------------------------------------------------

# 25. Embedding Provider Architecture

Embedding generation should also be provider-independent.

Supported providers:

``` text
Mistral
OpenAI
```

Conceptual interface:

``` typescript
interface EmbeddingProvider {
  generateEmbedding(
    text: string
  ): Promise<number[]>;
}
```

This keeps embedding infrastructure independent from the selected LLM
provider.

------------------------------------------------------------------------

# 26. MongoDB Data Model

A conceptual incident document:

``` json
{
  "incidentId": "INC-10245",
  "severity": "P1",
  "environment": "PROD",
  "summary": "Salesforce API timeout",
  "errorAnalysis": {},
  "logs": {
    "application": [],
    "scheduler": []
  },
  "similarIncidents": [],
  "rootCause": {},
  "recommendedActions": [],
  "evidence": [],
  "verificationRequired": true,
  "status": "TRIAGE_COMPLETED",
  "createdAt": "2026-09-27T00:00:00Z"
}
```

Historical RCA documents can be stored with vector embeddings and
searchable metadata.

------------------------------------------------------------------------

# 27. Data Separation

The architecture separates persistent knowledge from incident-specific
evidence.

## Persistent RAG Knowledge

Stored for future retrieval:

``` text
Historical RCA
Runbooks
Known Solutions
Historical Incident Information
```

## Incident-Specific Evidence

Used during current P1 investigation:

``` text
Current DataDog Application Logs
Current Scheduler Logs
Current Alert Details
Current Incident Metadata
```

This prevents the vector knowledge base from becoming polluted with
every operational log entry.

------------------------------------------------------------------------

# 28. Security Considerations

The production implementation should include:

-   Authentication and authorization
-   DataDog webhook authentication
-   API input validation
-   Secure secret management
-   Encryption in transit
-   Encryption at rest
-   Role-based dashboard access
-   Audit logging
-   PII/secret masking in logs
-   Prompt-injection protection for retrieved documents
-   Output validation before displaying AI results

The LLM should not receive unnecessary secrets, credentials, tokens, or
sensitive production information.

------------------------------------------------------------------------

# 29. Reliability and Guardrails

The system should implement:

### Evidence Grounding

The LLM should base RCA explanations on supplied evidence.

### Structured Output Validation

Validate the LLM response against a defined JSON schema.

### Confidence Threshold

Low-confidence results should require engineer verification.

### Source Traceability

Each important conclusion should have supporting evidence references.

### Duplicate Prevention

Before inserting a generated RCA, compare it against existing RCA
knowledge.

### Failure Handling

If DataDog logs, scheduler logs, retrieval, or the LLM are unavailable,
the dashboard should clearly show the unavailable component rather than
inventing results.

------------------------------------------------------------------------

# 30. Deployment Architecture

The application is deployed using containers on AWS.

Conceptual deployment:

``` text
                 AWS
                  |
        +---------+---------+
        |                   |
        v                   v
 React Container      Node.js Container
        |                   |
        |                   +----------------+
        |                   |                |
        |                   v                v
        |              MongoDB          LLM Provider
        |                                 |
        |                         +-------+-------+
        |                         |               |
        |                       OpenAI          Groq
        |
        +------------------------------------+
                                             |
                                             v
                                          DataDog
```

The exact AWS container service can be selected during implementation.

------------------------------------------------------------------------

# 31. Complete P1 Incident Flow

``` text
1. P1 alert triggered in DataDog
             |
             v
2. DataDog sends alert + application logs
             |
             v
3. Assistant receives P1 event
             |
             v
4. Incident created
             |
             v
5. Scheduler logs retrieved
             |
             v
6. Application + Scheduler logs analyzed
             |
             v
7. Incident query normalized
             |
             v
8. Abbreviations expanded
             |
             v
9. Synonyms expanded
             |
             v
10. Hybrid search
      |              |
      v              v
 Vector Search     BM25
      |              |
      +------+-------+
             |
             v
11. Combined retrieval results
             |
             v
12. P1 reranking
             |
             v
13. Top 3 historical RCAs
             |
             v
14. LLM evaluates genuine similarity
             |
             v
15. Evidence correlation
             |
             v
16. Rules + historical RCA + current logs
             |
             v
17. LLM generates probable RCA
             |
             v
18. Confidence calculated
             |
             v
19. Verification requirement determined
             |
             v
20. Recommended actions generated
             |
             v
21. Structured response created
             |
             v
22. React dashboard displays analysis
             |
             v
23. Engineer verifies and tracks actions
             |
             v
24. Generated RCA compared with existing RCA
             |
       +-----+-----+
       |           |
       v           v
 Existing       New Error
       |           |
       v           v
 No Duplicate   Insert RCA
                   |
                   v
             RAG Knowledge Base
```

------------------------------------------------------------------------

# 32. Example Final Dashboard View

``` text
------------------------------------------------------------
PRODUCTION INCIDENT ANALYSIS
------------------------------------------------------------

Incident ID       : INC-10245
Severity          : P1
Environment       : PROD
Status            : TRIAGE_COMPLETED

------------------------------------------------------------
INCIDENT SUMMARY
------------------------------------------------------------
Salesforce API timeout during lead processing.

------------------------------------------------------------
ERROR ANALYSIS
------------------------------------------------------------
Error Code        : HTTP 500
Error Type        : Salesforce API Timeout
Component         : Lead Processing Scheduler

------------------------------------------------------------
IMPACT
------------------------------------------------------------
Affected Records  : 127
Affected Service  : Lead Processing

------------------------------------------------------------
SIMILAR INCIDENTS
------------------------------------------------------------
INC-9871
Similarity        : 0.91
Resolution        : Retry failed transactions after API recovery

------------------------------------------------------------
PROBABLE ROOT CAUSE
------------------------------------------------------------
Salesforce API response latency caused request timeouts.

Confidence        : 0.87
Verification      : Required

------------------------------------------------------------
EVIDENCE
------------------------------------------------------------
1. DataDog Application Logs
2. Scheduler Logs
3. INC-9871 Historical RCA
4. Salesforce API Failure Runbook

------------------------------------------------------------
RECOMMENDED ACTIONS
------------------------------------------------------------
[ ] Check Salesforce API availability
[ ] Review API response latency
[ ] Verify scheduler execution
[ ] Identify failed transactions
[ ] Retry failed transactions after recovery

------------------------------------------------------------
```

------------------------------------------------------------------------

# 33. Key Architectural Decisions

  -----------------------------------------------------------------------
  Area                                Selected Approach
  ----------------------------------- -----------------------------------
  Incident Input                      Manual UI + API

  Minimal Incident Input              Supported

  P1 Trigger                          DataDog

  Application Logs                    DataDog P1 payload

  Scheduler Logs                      Retrieved and analyzed for every P1

  Historical RCA Retrieval            Hybrid Vector + BM25

  Query Preprocessing                 Normalization + Abbreviation +
                                      Synonym expansion

  Reranking                           P1 incidents

  Similar Incident Selection          Top 3 reranked candidates evaluated
                                      by LLM

  Root Cause                          Rules + Historical RCA + Current
                                      Logs + LLM synthesis

  Confidence                          Hybrid measurable confidence

  Evidence                            Source + detail + reference

  Recommended Actions                 Historical RCA + LLM-generated
                                      current actions

  Verification                        Required when confidence is low or
                                      evidence is
                                      insufficient/conflicting

  RCA Update                          Insert only genuinely new errors

  LLM                                 Configurable OpenAI / Groq

  Embeddings                          Configurable Mistral / OpenAI

  Dashboard                           React

  API Architecture                    Hybrid REST + DataDog webhook

  Deployment                          Containerized AWS
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 34. Expected Benefits

The architecture is intended to provide:

-   Faster P1 incident triage
-   Reduced manual RCA investigation
-   Reuse of historical troubleshooting knowledge
-   Better evidence traceability
-   Consistent incident analysis
-   Faster identification of similar incidents
-   Centralized incident investigation dashboard
-   Improved knowledge reuse through automatic RCA updates
-   Clear separation between current incident evidence and persistent
    RAG knowledge

------------------------------------------------------------------------

# 35. Final Architecture Summary

The core architecture can be summarized as:

``` text
DataDog P1
    |
    v
Incident + Application Logs
    |
    +---- Scheduler Logs
    |
    v
Incident Processing
    |
    v
Query Preprocessing
    |
    v
Hybrid RAG
(Vector Search + BM25)
    |
    v
P1 Reranking
    |
    v
Top 3 Historical RCAs
    |
    v
LLM Similarity Evaluation
    |
    v
Evidence Correlation
    |
    v
Rules + Logs + Historical RCA
    |
    v
LLM RCA Synthesis
    |
    v
Confidence + Verification
    |
    v
Recommended Actions
    |
    v
Structured Incident Analysis
    |
    v
React Dashboard
    |
    v
Engineer Verification / Action Tracking
    |
    v
New Error Detection
    |
    v
RCA Knowledge Base Update
```

The resulting solution provides an automated, evidence-driven workflow
from **DataDog P1 alert → log analysis → RAG retrieval → RCA synthesis →
dashboard investigation → knowledge-base improvement**, while keeping
human engineers responsible for production verification and final
remediation decisions.
