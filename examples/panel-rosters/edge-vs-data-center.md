# Example Panel Roster: Edge vs. Data Center

This is an example artifact for the historical ResearchOS demonstration. It
does not belong in `agents/` and must not be treated as a default roster for
future prompts. The Prompt Architect should derive a new roster from each
approved charter.

1. **Model architecture and scaling** — How do model size, sparsity,
   quantization, distillation, and test-time compute change the economics and
   feasibility of edge versus centralized inference?
2. **Inference serving systems** — What do latency, batching, networking,
   reliability, privacy, and workload shape imply for placement?
3. **Edge hardware and on-device systems** — What are the real constraints of
   memory, thermal envelope, power, silicon, accelerators, and device cost?
4. **Data-center infrastructure** — What are the constraints and advantages
   of networking, memory, cooling, power delivery, cluster utilization, and
   geographic distribution?
5. **Energy, physical constraints, and economics** — How do energy prices,
   grid availability, capex, opex, utilization, and lifecycle economics
   change the result across workloads and regions?
6. **Robotics and physical-world deployment** — Which workloads require local
   autonomy, offline operation, safety-critical latency, or sensor fusion?
7. **Open models and distribution** — How do open weights, licensing,
   compression, local deployment, cloud APIs, and developer distribution
   affect where intelligence runs?
8. **Benchmarks and workload measurement** — Which benchmarks predict actual
   deployment value, and where do public rankings and vendor metrics mislead?
9. **Industry structure and value capture** — Which layers may capture value
   under each deployment scenario, without starting from public companies?
10. **Base rates and falsification** — What comparable technology shifts,
    failed predictions, hidden assumptions, and disconfirming observations
    should constrain confidence?

Every member must investigate both the case for its assigned lens and the
strongest case against it. This roster is an example only; future projects
should create their own roster inside their project run.
