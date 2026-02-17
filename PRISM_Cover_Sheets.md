### E.1 CTO Cover Sheet
**Strictly Confidential - Audience-Specific Front Matter**

**To:** CTO / VP Engineering / Head of Quant Dev  
**From:** Asymmetry Computing  
**Subject:** Deterministic Intraday Rebalancing Infrastructure

**Context**
- CPU-first overnight workflows create schedule risk at N=5,000 and above.
- Delayed solve completion reduces event-time responsiveness for tax and rebalance logic.

**What PRISM Changes**
- GPU-native convex QP solve path with deterministic run semantics.
- Real-data benchmark envelope: 9.60x-20.60x speedup vs Gurobi in planned scenarios.
- Audit artifacts generated per run: solve identity, trace hash, feasibility/KKT checks.

**Integration Surface**
- REST contract: health, solve, and audit endpoints.
- API workflow: submit solve via REST, capture `wall_ms`, retrieve audit hash.
- Hybrid routing available for small-N CPU and large-N GPU paths.

**Operational Takeaway**
- Target outcome is repeatable intraday operation with bounded tail latency and reproducible evidence.

<!-- PAGE_BREAK -->

### E.2 VC Cover Sheet
**Strictly Confidential - Audience-Specific Front Matter**

**To:** Investment Committee  
**From:** Asymmetry Computing  
**Subject:** PRISM Technical and Commercial Moat Analysis

**Market Signal**
- Direct indexing and multi-account optimization require faster and more auditable infra.
- Regulatory and resilience controls increase value of deterministic, replayable operations.

**Moat Snapshot**
1. Verified speed advantage in published real-data scenarios.
2. Engine-level optimization stack tuned for GPU hardware paths.
3. Built-in evidence packaging and audit-hash traceability.
4. Integration stickiness once intraday workflows are productionized.

**Commercial Shape**
- Value linked to latency window capture, reliability, and compliance readiness.
- Expandable from direct-indexing optimization into broader portfolio/risk workflows.
- Positioned as infrastructure layer, not a point feature.

**Investment Framing**
- Hardware-shift thesis with operational proof points and auditable outputs.

<!-- PAGE_BREAK -->

### E.3 GPU BD Cover Sheet
**Strictly Confidential - Audience-Specific Front Matter**

**To:** Strategic Partnerships / Compute Sales  
**From:** Asymmetry Computing  
**Subject:** High-Utilization Workload for Fintech Verticals

**Workload Characteristics**
- Compute-intense optimization loops with recurring production cadence.
- Memory- and bandwidth-sensitive GPU workloads with sustained throughput demand at high asset counts.
- Deterministic solve requirements under institutional SLA windows.

**Partnership Value**
- Opens a capital-markets GPU workload vertical traditionally served by CPU.
- Supports VPC/on-prem deployment models demanded by regulated institutions.
- Produces steady utilization patterns beyond burst-style inference profiles.

**Technical Fit**
- Verified hardware path references: A100, H100, L4, RTX 6000 Ada.
- Standard NVIDIA driver + container runtime environment (implementation details intentionally undisclosed).
- Containerized deployment with NVIDIA runtime support.

**Commercial Fit**
- Joint value proposition: measurable runtime advantage + auditable operational controls.
