# KIVZO Architecture

## Components

| Component | File | Responsibility |
|---|---|---|
| Planner | `engine.py` (`make_plan`), `llm.py` (`plan`) | Builds the plan from the user's request and tool signatures only. In LLM mode Claude writes the plan as JSON; it is validated, and a verified template is used if validation fails. |
| Quarantined reader | `quarantine.py` | Turns untrusted text into one typed value (IBAN, amount, email, filename, time, location, ...). No tools. Output validated by type. |
| Interpreter | `engine.py` (`Run`) | Executes plan steps (`call`, `extract`, `compute`, conditional `when`) in plain Python, so every value's origin is known by construction. |
| Label store | `labels.py` | Creates every value, joins labels, signs label records with HMAC, answers lineage queries. |
| Gateway | `gateway.py` | Six deterministic checks per call. Loads `policy.yaml`. |
| Corroborators | `corroborate.py` | Compare untrusted values with systems of record. |
| Protected runner | `runner.py` | Issues and redeems one-shot signed capsules bound to the exact arguments; holds secrets as vault references. |
| Mock world and tools | `world.py`, `tools.py` | A small company: vendors, POs, inbox, files, CRM, web, repo, CI, calendar, smart home. Every side effect is logged. |
| Attacks | `attacks.py` | 14 named scenarios and a generator: 6 goals x 7 carriers x 1-3 hops x 5 rewording levels = 630. |
| Benchmark | `bench.py` | Runs every system on every case and writes JSON, a Markdown report and charts. |
| Console | `server.py`, `web/` | FastAPI and a static single-page console. |

## Plan language

```json
{"id": "s4", "kind": "extract", "from": "s1.text", "field": "iban", "type": "iban"}
{"id": "s7", "kind": "call", "tool": "transfer_money",
 "args": {"to_iban": {"ref": "s4"}, "amount": {"ref": "s3"}, "vendor": {"ref": "s2"}, "memo": {"lit": "INV-104"}}}
{"id": "s5", "kind": "call", "tool": "device_control", "args": {...}, "when": {"ref": "s2", "equals": "office"}}
```

A `ref` argument that points at untrusted data is a **plan-bound slot**: the user's plan declared that this
argument may be filled from that document. Attacker text can neither add slots nor add steps.

## The five systems compared

| ID | System | Difference |
|---|---|---|
| S0 | Undefended | Tools called directly |
| S1 | Prompt shield | S0 + detector on every tool output; flagged text is removed |
| S2 | Strict IFC | KIVZO gateway with endorsement disabled |
| S3 | KIVZO | Full system |
| S4 | KIVZO, no plan lock | Ablation: the agent may call any tool; labels and policy must contain attacks alone |

## Worst-case adversary simulation

When the agent reads text containing an injection, the simulation:

1. attempts every tool call the attacker asked for, with arguments labelled as coming from that text;
2. makes the quarantined reader return the attacker's values (`overrides`);
3. routes values through 1-3 hops (summarise, save and re-read a note, extract) for multi-hop attacks;
4. can forge labels in memory (A10) or replay a capsule (A14).

S0 therefore shows the full damage of a hijacked agent, and any containment by S2-S4 comes from the
architecture, not from the model resisting.

## Extending

- **New tool:** add it to `TOOLS` in `tools.py` with output provenance, then add a `policy.yaml` entry if it is sensitive.
- **New corroborator:** add a function to `corroborate.py` and reference it from `policy.yaml`.
- **New task or attack:** add it to `tasks.py` or `attacks.py`; the benchmark and console pick it up automatically.
- **Production path:** run the gateway as MCP middleware or as a tool wrapper in an agent framework. Enforcement
  sits at the tool boundary, so agent code barely changes. Move labels and the audit log to Postgres. The gateway
  is stateless per call and scales horizontally, and it fails closed.
