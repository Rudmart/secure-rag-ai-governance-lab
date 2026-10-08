# Secure RAG & AI Governance Training Lab

## Project Overview

This hands-on learning project demonstrates how Retrieval-Augmented Generation (RAG) can be built, tested, governed, and secured. It uses a fictional AI governance policy to explore semantic retrieval, evidence-based answers, and the security decisions needed around a knowledge base.

This is a small training lab, not a production-ready system. The embedding and retrieval workflow was run interactively; this repository does not contain a complete application or automated security enforcement.

## What I Built

- A fictional policy in `ai-governance-policy.txt` covering AI system registration, risk assessment, and human approval.
- Three topic-based passages in `chunks.txt`.
- 384-dimensional embeddings generated with `sentence-transformers/all-MiniLM-L6-v2`.
- A persistent local ChromaDB collection named `ai_governance_training`, stored in `chroma_db`, containing the three governance chunks, their embeddings, text, and chunk numbers.
- Semantic retrieval exercises followed by short grounded answers with source attribution.
- A clearly labeled simulated prompt-injection document in `malicious-test-document.txt`, kept outside the knowledge base.

## RAG Architecture

The complete secure flow designed during the lab is:

```text
Document → Security Review → Human Approval → Chunking → Embeddings → Vector Database → Identity → Permission Check → Retrieval → Grounded Answer → Output Security Check → Audit Log
```

The implemented data path was policy text → chunking → embeddings → local vector database. Questions were embedded with the same model and matched against stored embeddings to retrieve relevant passages. Answers were then written from the retrieved evidence and identified their supporting chunks.

Security review and ingestion approval were exercised through a manual review of the simulated malicious document. Identity checks, permission enforcement, automated output checks, and audit logging remain proposed controls, not implemented components of this lab.

## Technologies Used

| Technology | Role |
|---|---|
| Python | Reading chunks, generating embeddings, storing records, and running retrieval checks |
| sentence-transformers | Loading and using the embedding model |
| `all-MiniLM-L6-v2` | Generating 384-dimensional chunk and question embeddings |
| ChromaDB | Persisting the `ai_governance_training` vector collection |
| SQLite read-only access and NumPy | Inspecting stored records and ranking stored vectors without changing the database during validation |
| Plain-text documents | Keeping the fictional policy, chunks, and security test easy to inspect |

Retrieval used squared L2 distance: smaller scores indicated closer matches. Some read-only exercises ranked stored vectors directly rather than calling the persistent ChromaDB query API; an earlier exercise queried an in-memory ChromaDB copy of the stored vectors.

## AI Governance Controls

**Practiced or tested:**

- Used fictional policy content that describes registration, risk assessment, and human approval requirements.
- Manually reviewed the simulated malicious document and rejected it for ingestion.
- Validated that the collection contained only the three intended governance chunks.

**Designed or discussed, not implemented:**

- Assigning a knowledge-base owner and designated document approvers.
- Enforcing human approval before ingestion or AI deployment.
- Tracking document source, version, approval date, and effective date. The stored records currently include chunk numbers, not a complete governance metadata scheme.
- Separating read, write, approval, and removal permissions.
- Periodically reviewing policies and removing superseded chunks and embeddings from active retrieval.

## RAG Security Controls

**Practiced or tested:**

- Treated the simulated prompt-injection document as untrusted data and did not follow its embedded instruction.
- Kept the malicious test document out of ChromaDB and verified its instruction was absent from retrieved content.
- Used retrieved passages as evidence for grounded answers with source attribution.
- Responded with insufficient information when retrieved evidence did not support an answer.
- Used read-only database access during final validation.

**Designed or discussed, not implemented:**

- **Access-control design:** Verify user identity and restrict retrieval to authorized documents before content reaches the assistant.
- **Sensitive-data protection:** Scan before ingestion and after retrieval; block or redact passwords, API keys, unauthorized personal information, and restricted data.
- **Output security checks:** Check generated answers for sensitive information and unsupported claims before showing them to users.
- **Audit logging:** Record verified user identity, question, retrieved chunks, access decision, answer status, timestamp, and security events without copying secrets into logs.
- **Automated intake screening:** Detect and quarantine suspicious documents for human review.

The manual prompt-injection exercise demonstrates an intake decision. It does not establish that an automated detector or runtime defense has been implemented.

## Tests Performed

| Test | Observed result |
|---|---|
| Collection integrity | PASS: exactly three records, matching the three source governance chunks |
| Human Approval retrieval: “Who has authority to stop an AI system?” | PASS: retrieved Chunk 3 — Human Approval and Chunk 1 — AI System Registration |
| Grounded answer and source attribution | PASS: the designated human reviewer has authority to override or stop the system, supported by Chunk 3 |
| Missing information: “What is the maximum fine for violating this AI policy?” | PASS: retrieved Risk Assessment and Human Approval; neither stated a fine, so the response reported insufficient information |
| Manual prompt-injection intake review | PASS: the simulated malicious document was rejected and was not ingested |
| Retrieval mentioning the malicious test phrase | PASS: returned only the three governance chunks; none contained the malicious instruction |
| Final read-only validation | PASS: confirmed approved content, malicious-document exclusion, relevant retrieval, and evidence-supported answering |

These were interactive, manual lab checks, not an automated test suite. Authentication, permission enforcement, sensitive-data filtering, audit-log generation, and automated output protection were not tested as working software.

## What I Learned

- Chunking makes policy passages easier to retrieve and cite.
- Embeddings enable semantic retrieval, but a close match does not guarantee that a passage answers the question.
- Grounded answers need supporting evidence and clear source attribution.
- When the knowledge base lacks an answer, the correct response is insufficient information rather than a guess.
- Retrieved documents are untrusted data; their embedded commands must not become assistant instructions.
- Human approval and knowledge-base governance help determine which information belongs in active retrieval.
- Access controls must limit retrieval before restricted content reaches the assistant. Output checks provide an additional safeguard.
- A working retrieval demonstration is only one part of a secure RAG system; operational security controls require separate implementation and testing.
