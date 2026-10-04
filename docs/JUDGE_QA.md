# Judge Q&A: the four master questions

## 1. "Prove your security holds. How do labels survive the LLM, and how would you break it?"

- Labels are **not inferred from an LLM**. The planner never sees data; it writes a plan over variable names.
  Untrusted text is read only by the quarantined reader, which has no tools and must return one typed value.
  Plain Python runs the plan, so every value's origin is known by construction (`engine.py`).
- Labels are assigned by the gateway, stored out of band and HMAC-signed. A10 rewrites labels in memory, and the
  signature check blocks it (`labels.py`, `test_forged_label_is_detected`).
- Derived values take the most restrictive label, so summaries and notes cannot launder data. A5 travels three hops
  and is still UNTRUSTED (`test_note_keeps_its_label`).
- Branches decided by untrusted data taint the calls inside them (A12, T6).
- EchoLeak: confidentiality is a second axis and every outbound argument is a sink (A6, `test_secret_cannot_leave_in_url`).
- Rug pull: tool manifests are hash-pinned (A9). Compromised sub-agent: its output is UNTRUSTED (A11).
- **Residual risk, stated honestly:** text-only influence, poisoned systems of record, and attacks whose request
  matches the user's own intent and records.

Don't claim "100% secure". Say "0 successful attacks in our 644-attack suite, under a worst-case adversary, with these stated limits."

## 2. "If you block untrusted data, how does real work get done, and why trust your numbers?"

- Corroborated endorsement: untrusted values are admitted only into slots the plan declared, only as typed values,
  and only when a trusted record agrees. Strict IFC completes 57.1% of tasks; KIVZO completes 100%, with zero
  approvals on clean runs.
- Attacks are generated, not hand-picked: 6 goals x 7 carriers x 1-3 hops x 5 rewording levels = 630, plus 14
  named scenarios modelled on public incidents.
- The adversary is worst case (the agent always obeys), so containment comes from the architecture, not from the
  model being well-behaved.
- `python bench.py` regenerates every number in about 15 seconds. Run it live. Wilson 95% intervals are reported.
- Failures are reported, not hidden: under attack KIVZO fails 3 tasks (A7, A9, A12), by design. It refuses an
  inflated amount, a tool that changed after approval, and a 02:50 heater schedule.

## 3. "CaMeL and FIDES already exist. What is actually new?"

- Name them first. Dual-LLM planning and information-flow control are prior art, and we cite them.
- New in KIVZO: **corroborated endorsement**, the third option between "block" and "ask a human". This is the
  utility gap CaMeL's results and FIDES's own "overly conservative" note leave open. Measured as an ablation
  (S2 vs S3).
- Also combined and measured: confidentiality sinks on every outbound argument, manifest pinning, signed out-of-band
  labels, one-shot capsules, and lineage-explained decisions.
- Versus prompt shields: see the rewording chart. Detection rates depend on wording; provenance does not. A shield can
  sit in front of KIVZO, but it cannot replace the boundary.
- Users: any company whose agents act on payments, email, code or devices. Concrete value: blocking invoice and
  bank-detail fraud without stopping invoice processing.

## 4. "Would this survive production?"

- **Latency:** the policy check takes well under a millisecond (median reported live). The real cost is the extra
  planner and reader calls; the reader can be a small, cheap model because it only fills typed fields.
- **Policy authoring:** `policy.yaml` is declarative. The default "untrusted cannot reach sensitive sinks" covers
  most tools. Organisations plug in records they already own (vendor master, address book, CRM).
- **Failure:** the gateway fails closed. Approvals are denied unless a human approves. Changed tools stop until
  re-approved.
- **Integration:** run it as MCP gateway middleware or as an agent-framework tool wrapper. Enforcement sits at the
  tool boundary, so agent code barely changes. Labels and the audit log move to Postgres, and the gateway scales
  horizontally.
- **Next steps:** a policy-authoring UI, a re-approval flow for changed manifests, signed master-data records,
  and running the AgentDojo suites against KIVZO.

## Rapid fire

| Question | Answer |
|---|---|
| Is an LLM making the security decision? | No. `gateway.py` is deterministic code over labels and policy. |
| Why not fine-tune the model? | Training lowers the rate but gives no guarantee. KIVZO works with any model. |
| What if the user is the attacker? | Out of scope. KIVZO protects the user from content. |
| Why not a sandbox? | A sandbox protects the host. It doesn't stop a correctly permissioned agent from paying the wrong account. |
| Is the baseline a straw man? | S0 is the full damage of a hijacked agent. S1 uses real detector phrases, and can use the ProtectAI classifier. LLM mode runs a real agent. |
