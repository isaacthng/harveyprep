# Harvey AI — Knowledge Bases — Product Requirements Document

*Centralized institutional knowledge for legal teams—curated by admins, queryable by AI.*

Prepared by Isaac | March 2026 | Confidential

---

# 1. Problem & Opportunity

Legal teams' institutional knowledge—precedent documents, templates, playbooks—is trapped in document management systems, email threads, and individual attorneys' memories. Associates spend 20–30% of their time searching for prior work product. When attorneys leave, that knowledge walks out the door. KM spending climbed 10.5% YoY in 2025 because the status quo is unsustainable (Thomson Reuters / Georgetown 2026 Legal Market Report).

AI should solve this, but trust is the bottleneck: 83% of legal leaders now have AI access, yet only 22.1% report high trust in AI outputs (Artificial Lawyer, 2026). The core problem isn't accuracy alone—it's *verifiability*. Lawyers need every AI response to trace back to a specific, firm-vetted source document. A plausible-sounding answer without provenance is worse than no answer at all.

**Knowledge Bases** solve this by creating AI-powered, curated repositories of firm-specific knowledge—queryable through Harvey's Assistant with full citation attribution, fail-closed permissions, and ethical wall enforcement. Harvey's 100K+ lawyers and $190M ARR provide distribution. Competitors are moving (CoCounsel Knowledge Search, Luminance Institutional Memory), but Harvey's advantage is firm-specific knowledge grounded in curated internal content—not public law.

*Assumption: Harvey's existing Vault KB capabilities (96% key-term extraction) are transferable to this product. If not, the ingestion pipeline requires new parser investment, delaying P0 by 4–8 weeks.*

---

# 2. User Personas

## Admin — KM Director

**Profile:** Sarah Chen, Director of Knowledge Management at a 1,200-lawyer AmLaw 50 firm. Former M&A attorney, now manages a team of 4 PSLs and 2 KM coordinators.

**Core frustration:** Depends on attorneys voluntarily tagging documents (~30% compliance). Spends 60% of her time on manual curation. Cannot measure whether anyone uses the precedents she curates. Ethical wall enforcement is manual and error-prone.

**What success looks like:** "I stand up a KB for our restructuring practice in a day, sync it from iManage, and see within a week which precedents associates are querying."

## End User — Associate

**Profile:** James Park, 4th-year corporate associate. Works on M&A transactions. Spends 25% of his time searching for prior deal documents.

**Core frustration:** iManage returns too many results with poor relevance. Asking partners is high-friction. Doesn't trust AI answers not grounded in firm-specific sources (legal AI hallucination rates of 17–82% per Stanford HAI).

**What success looks like:** "I ask Harvey about our MAC clause approach in healthcare M&A and get a synthesized answer citing three specific precedent agreements, with links to the source documents."

---

# 3. Capabilities & Prioritization

*Accompanying visual mockups, user flow diagrams, and process flow diagrams are provided as separate HTML files.*

## P0 — Must Have (MVP)

| Flow | Persona | Job to be Done | Key Requirements |
|------|---------|----------------|------------------|
| **Create & populate KB** | Admin | Stand up a new KB for a practice area and fill it with vetted content | Name, scope, permissions in one setup flow. Three content paths: iManage DMS sync (primary—60% BigLaw market share), bulk upload, individual upload. AI auto-classifies documents and extracts metadata; admin reviews and approves. |
| **Set permissions & ethical walls** | Admin | Control who can access what, with zero tolerance for leakage | Three-layer fail-closed enforcement: KB-level → document-level → Intapp Walls sync (72% of firms use ethical walls). "Test as User" preview. LLM never sees content behind a wall—filtering happens at the retrieval layer. *Assumption: Intapp Walls integration extends to KB-level enforcement. If not, must build proprietary ethical wall system.* |
| **Query KB via Assistant** | End User | Find firm-specific precedent in natural language and trust the answer | Semantic + keyword hybrid retrieval → permission filter → multi-agent generation with citation verification (see Section 4). Synthesized answer with inline citations, source panel with clickable links, and confidence indicator. Conversational follow-up supported. |
| **Citation & source attribution** | End User | Verify that every AI claim traces to a real source | Every substantive claim includes a citation to a specific document and section. Post-generation citation verification confirms each citation exists in retrieved context. Binary pass/fail—no exceptions. |
| **Basic admin analytics** | Admin | Prove ROI of KM investment | Query volume, top queries, retrieval success rate, user feedback aggregation, content coverage gaps. |

**AI Behavioral Constraints (apply to all query responses, binary pass/fail):**
- MUST cite only documents present in retrieved context—no fabricated citations
- MUST NOT generate legal analysis beyond what source documents support
- MUST include at least one citation per substantive claim
- MUST surface a "low confidence" indicator when fewer than 2 relevant sources are found
- MUST NOT reveal the existence of documents the user is not authorized to access
- When sources conflict, MUST present both positions rather than synthesizing a false consensus

### Worked Example

> **Associate:** "What's our standard approach to MAC clauses in healthcare M&A?"
>
> **Harvey:** Returns a synthesized answer citing [1] Acme Health Merger Agreement (2024), Section 7.2 and [2] Healthcare M&A Playbook, MAC Clause Section. Source panel shows clickable links with highlighted excerpts. Confidence: High (3 corroborating sources).
>
> **Associate:** "How did we handle the pandemic carve-out in the Acme deal?" → deeper retrieval with conversational context maintained.

## P1 — Should Have (Fast Follow)

| Flow | Persona | Job to be Done | Key Requirements |
|------|---------|----------------|------------------|
| **AI-assisted classification** | Admin | Reduce manual curation burden | Auto-suggest document types, practice areas, and tags during ingestion. Reduces curation time by 40–60%. |
| **Knowledge nomination** | End User → Admin | Crowdsource KB content from attorneys | User clicks "Nominate for KB" on a useful document, adds context. Enters admin approval queue with AI-suggested metadata. |
| **KB-to-draft workflow** | End User | Use precedent directly in drafting | "Use in Draft" opens drafting workspace with source as reference. Clause adaptation, redline comparison, provenance tracking. |
| **Stale content detection** | Admin | Keep KB current and accurate | AI flags outdated documents based on age, superseding documents, and regulatory changes. |
| **NetDocuments sync** | Admin | Expand DMS coverage | ~20% DMS market share. Same sync model as iManage. |
| **KB lifecycle management** | Admin | Maintain KBs long-term | Ownership transfer, version control, archival, deprecation with redirect. |

## P2 — Future

- Content gap analysis, KB quality scoring, SharePoint/Google Drive sync, cross-KB search, personalized KB recommendations, two-way DMS sync, knowledge graph visualization

---

# 4. Technical Risks & Mitigations

*Assumption: Harvey's multi-model orchestration supports RAG over firm-specific collections without new infrastructure. If not, requires dedicated infrastructure investment.*

## Risk Overview

| Risk | Severity | Mitigation | Validation Target |
|------|----------|------------|-------------------|
| **Retrieval accuracy & hallucination** | Critical | Multi-agent verification pipeline (see below); citation verification as hard gate; confidence indicators | Precision@10 >85%, citation accuracy >99%, hallucination rate <2% |
| **Synthesis accuracy** (correct citations, wrong conclusions) | Critical | Verification Agent independently evaluates synthesis against sources; conflicting source detection; structured claim-evidence output format | User override rate <15%, attorney red-team approval >90% |
| **Permission enforcement at scale** | Critical | Three-layer fail-closed; permission cache with TTL; Intapp Walls sync <5 min propagation; immutable audit log | Zero breaches in penetration testing |
| **iManage integration complexity** | High | Start with iManage Cloud (71% of deployments) via REST API. On-prem via bridge service in Beta. Idempotent sync with checkpointing; retry with exponential backoff. *Assumption: iManage open beta provides foundation for KB sync.* | >99% sync success (Cloud), >95% (hybrid) |
| **Document parsing quality** | High | Legal-specific parser; leverage Vault's 96% key-term extraction; parse quality scoring per document | >98% text extraction, >95% structural preservation |
| **Query latency** | Medium | Streaming responses; pre-computed embeddings; retrieval caching | First token <2s, full response <8s |
| **Content curation burden** | High | AI-assisted classification; knowledge nomination workflow; bulk operations | <15 min admin time per 100 docs via DMS sync |

## Multi-Agent Verification (Key Mitigation)

A single LLM self-checking its output is unreliable in legal contexts—it confirms its own errors. Knowledge Bases uses a multi-agent pipeline where specialized agents cross-verify with no shared state between generation and verification:

| Agent | What It Does | What It Catches |
|-------|-------------|-----------------|
| **Retrieval Agent** | Finds and ranks relevant document chunks | Poor recall, irrelevant results |
| **Retrieval Evaluator** | Assesses retrieval quality before generation; triggers re-retrieval or low-confidence flag if insufficient (Corrective RAG pattern) | Prevents generation from poor context—the most dangerous hallucination source |
| **Synthesis Agent** | Generates answer with citation markers | N/A—this is where hallucination originates |
| **Verification Agent** | Independently checks if synthesis follows from sources (receives raw chunks separately, not from Synthesis Agent) | The hard problem: correct citations, wrong conclusions |
| **Citation Agent** | Confirms every citation maps to retrieved content | Fabricated or misattributed citations |

The Verification Agent is adversarial by design—its objective is to find unsupported claims, not to confirm the synthesis. Tradeoff: +1–3s latency, ~2x LLM cost per query. Justified in legal context where a single hallucinated citation can result in court sanctions ($30K+ per incident, Sixth Circuit 2025).

**Frontier technique — Semantic Entropy for confidence calibration:** Beyond simple source counting, semantic entropy clusters multiple sampled outputs by meaning to detect genuine model uncertainty. High semantic entropy on a legal question means the model is uncertain across meaningfully different conclusions—the response gets flagged for human review rather than presented with false confidence. (Farquhar et al., Nature 2024)

## Evaluation Framework

| What We Measure | Method | Threshold | Why It Matters |
|----------------|--------|-----------|----------------|
| **Retrieval precision@10** | Annotated query set (500+ queries with known-relevant docs, built with Beta firms) | >85% | Wrong docs retrieved = wrong answers downstream |
| **Citation accuracy** | Automated: verify each citation exists in retrieved chunks. Binary pass/fail. | >99% (hard gate, blocks release) | Fabricated citation in legal context = potential sanctions |
| **Synthesis faithfulness** | LLM-as-judge: separate model evaluates claim-by-claim entailment from sources | >95% entailment | Catches "correct citations, wrong conclusions" |
| **Hallucination rate** | Adversarial test suite (200+ queries designed to elicit hallucination) | <2% | Regression detection |
| **Permission enforcement** | Penetration test suite probing ethical wall boundaries and cross-KB leakage | Zero breaches (hard gate) | One breach = product credibility destroyed |
| **Legal correctness** | Practicing attorneys review 50 random responses per sprint: correct / partially correct / incorrect | >90% correct | Automated evals can't assess legal substance |
| **User override rate** | Track how often users materially change or discard AI output | <15% | Most honest signal of trust—if users don't use the output, nothing else matters |
| **Query abandonment** | User queries but doesn't click sources, copy, or draft within 5 min | <40% | Real-world adoption proxy |

**Release gates:** Alpha→Beta requires all automated evals green + zero permission breaches. Beta→GA requires human eval legal correctness >90% + user override rate <15%. GA ongoing: any automated regression blocks next release.

---

# 5. Rollout Plan

*Assumption: Timelines assume existing Harvey customers. Net-new customers add 3–6 months for enterprise legal procurement.*

| Phase | Timeline | Goal | Key Actions | Cold-Start Strategy | Success Gate |
|-------|----------|------|-------------|--------------------|--------------|
| **Alpha** | 4–6 weeks | Validate core AI pipeline | Internal testing with synthetic legal data. Manual upload only. Permission penetration testing. Parse quality benchmarking. | N/A (synthetic data) | Precision@10 >80%, citation accuracy >98%, zero permission bypasses |
| **Private Beta** | 8–12 weeks | Prove value with real firms | 2–3 partner firms (AmLaw 50 + mid-size + in-house). iManage Cloud sync. Full admin + user flows. Intapp Walls integration. Dedicated CSM with weekly feedback. Red-team by practicing attorneys. | **Min viable corpus:** 50–100 docs per KB. Quick-start templates (M&A, Litigation, Finance, Real Estate, Employment) with pre-configured taxonomy and seed guidance. "Your KB is 40% ready" progress meter. Bootstrap-from-iManage wizard for <30 min setup. | Precision@10 >85%, citation >99%, zero breaches, >30% weekly query rate, red-team approval >90% |
| **Limited GA** | 8–12 weeks | Scale to 20–30 firms | NetDocuments sync. KB-to-Draft workflow. AI classification. Knowledge nomination. Self-serve onboarding. iManage on-prem connector (beta). | Self-serve guided setup with templates. Empty-state messaging: "This KB is still being built—try broadening your query." | >30% weekly query rate, <5% feature removal requests, admin setup <4 hrs |
| **Full GA** | Ongoing | All Harvey customers | SharePoint/Google Drive sync. Advanced analytics. Cross-KB search. Full documentation and training. | Fully self-serve with help center. | Stable Tier 3 online metrics for 4+ weeks |

---

# 6. Success Metrics

## Adoption

| Metric | Target (90 days) | Baseline | Measurement |
|--------|-------------------|----------|-------------|
| KB creation rate | >2 KBs per firm | N/A (new feature) | Product analytics |
| Admin activation | >80% within 30 days | N/A | User activity logs |
| End user weekly query rate | >30% of enabled users | Enterprise tool benchmark: 20–40% | Weekly active / enabled users |
| Documents ingested | >10,000 per firm | iManage avg workspace: 50K–500K docs | Ingestion pipeline logs |

## Quality

| Metric | Target | Baseline | Measurement |
|--------|--------|----------|-------------|
| Citation accuracy | >99% | Harvey Vault: 96% key-term extraction | Automated citation verification |
| Retrieval precision@10 | >85% | Industry RAG: 70–80% | Sampled human evaluation |
| Hallucination rate | <2% | Legal AI: 17–82% (Stanford HAI) | Automated + human audit |
| User override rate | <15% | Current AI edit rate: 69.7% | Most honest trust signal |

## Efficiency

| Metric | Target | Baseline | Measurement |
|--------|--------|----------|-------------|
| Time to precedent | <60 seconds | Manual DMS search: 15–30 min | Query-to-result latency |
| Draft acceleration | >30% reduction in first-draft time | Measured during Beta | Before/after in Beta cohort |
| Admin curation efficiency | <15 min per 100 docs via DMS sync | Manual: 2–4 hours per 100 docs | Session duration analysis |

---

# 7. Competitive Positioning

| Dimension | Harvey Knowledge Bases | CoCounsel Knowledge Search |
|-----------|----------------------|---------------------------|
| **Knowledge source** | Firm-specific precedents, templates, playbooks—proprietary institutional knowledge | Federated search across DMS + Westlaw + Practical Law—primarily public legal content |
| **AI depth** | Multi-agent RAG pipeline with synthesis verification, citation verification, confidence calibration | AI re-ranking and summarization of search results |
| **Curation model** | Admin-curated + attorney-nominated + AI-classified. Content vetted before entering KB. | Searches existing DMS content as-is |
| **Trust architecture** | Four-layer model (verified source → citation → confidence → human review). Binary pass/fail constraints. | Citations grounded in Westlaw (strong for public law, not firm-specific) |
| **Ethical walls** | Fail-closed + Intapp Walls. Purpose-built for AI-era enforcement. | Relies on underlying DMS security models |
| **Competitive edge** | Makes Harvey smarter about how *this firm* practices law—firm-specific intelligence competitors can't replicate | Makes CoCounsel smarter about the law generally—valuable but commoditizable |

**Harvey's flywheel:** More firm knowledge in KBs → better AI responses → more user trust → more usage → better feedback signals → better retrieval → more admin investment in curation. Once a firm invests in curated KBs, switching costs are extremely high—the knowledge itself is firm-specific and cannot be replicated.

*Assumption: Competitive window is 6–12 months before CoCounsel and Luminance mature their offerings. If wrong, may need to cut P1 scope and accelerate GA.*

---

# 8. Open Questions & Assumptions

## Open Questions

Questions to explore with the Harvey team to sharpen the design:

1. **KB vs. Vault positioning:** Is KB an evolution of Vault, a separate product surface, or a layer on top? Affects architecture and user mental models.
2. **Existing usage data:** What does current Vault KB adoption look like? What's working and what's not?
3. **DMS sync in production:** What has the iManage open beta revealed about sync reliability and firm-specific configuration variance?
4. **Synthesis accuracy:** Has Harvey observed "correct citations, wrong conclusions" in existing products? What eval approaches exist internally?
5. **Curation burden:** Who actually curates KB content today—KM Directors, PSLs, or attorneys? What's the admin-to-content ratio?
6. **Cold-start threshold:** What's the minimum document count for useful query results? Has this been empirically tested?
7. **Competitive dynamics:** Is CoCounsel Knowledge Search already in market at Harvey's target accounts?

## Structural Assumptions

| Assumption | Impact if Wrong | How We'd Know |
|-----------|-----------------|---------------|
| KB is sold as part of the Harvey platform, not a standalone SKU | Changes GTM strategy, pricing model, and competitive positioning | Pricing discussions with sales leadership |
| Primary deployment is Harvey-hosted cloud (Azure); no on-prem for initial release | Reduces initial addressable market by ~20% (firms requiring on-prem) | Customer requirements during Beta firm selection |
| Target market is AmLaw 100 + large in-house departments with existing KM infrastructure | Mid-market requires simpler onboarding and changes the cold-start strategy | Beta firm feedback on setup complexity |

---

*Accompanying artifacts: Admin Mockup (HTML), Query Experience Mockup (HTML), Process Flow Diagram (HTML), Admin User Flow (HTML), End User Flow (HTML)*
