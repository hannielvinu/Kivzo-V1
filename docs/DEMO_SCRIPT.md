# Demo script (5 minutes)

Before judges arrive: run `python run.py`, open http://localhost:8000, run the benchmark once (Benchmark tab),
and set the browser zoom so two columns fit side by side. Everything works offline in scripted mode.

| Time | Do | Say |
|---|---|---|
| 0:00 | Title / problem slide | "EchoLeak: one email, zero clicks, internal data leaked through a URL. Filters can't fix this because attacks can be reworded endlessly." |
| 0:30 | Live demo: **T1, no attack, Undefended vs KIVZO** → Run | "Normal work flows. Every value carries a label: green is the user, red is untrusted." Point at the invoice fields labelled UNTRUSTED that still get paid: "endorsed against the PO and vendor master." |
| 1:00 | **A8 bank-detail swap** → Run | "Left: the hijacked agent pays the attacker. Right: KIVZO checks the IBAN against the vendor master, pays the real vendor and raises an alert." Click the `transfer_money` call → show OVERRIDE and the lineage graph. |
| 1:50 | Compare: **Strict IFC vs KIVZO**, **T1, no attack** | "This is the gap. Pure information-flow control is safe but refuses to pay the invoice. Our corroborated endorsement fixes that." |
| 2:20 | **A2 reworded (Hindi)**, Compare: **Prompt shield vs KIVZO** | "The detector misses wording it doesn't know. KIVZO doesn't look at wording; it looks at where the value came from." |
| 2:50 | **A5 multi-hop**, Compare: **No plan lock vs KIVZO** → click the blocked payment | "Web page → summary → saved note → payment. Three hops later the label is still UNTRUSTED. Even with the plan lock switched off, labels alone block it." |
| 3:20 | **A6 mini-EchoLeak**, then **A13 smart lock** | "The EchoLeak path: SECRET data may never leave in a URL. And the Black Hat calendar-invite attack on smart homes: untrusted data never reaches an actuator." Point at the door: LOCKED vs UNLOCKED. |
| 3:50 | Benchmark tab → **Run full benchmark** (about 15 s) | "3,255 runs, 644 attacks, regenerated live. 0% attack success, 100% task completion, policy check in a fraction of a millisecond." Show the rewording chart. |
| 4:30 | Trust model tab | "Our whole trust model on one screen, loaded from the policy file the gateway enforces." Close: "A poisoned tool can influence what the agent thinks. KIVZO decides what it's allowed to do." |

## If something goes wrong

- Port busy: `python run.py --port 8010`.
- No internet: scripted mode needs none.
- Browser issue: `python bench.py` in the terminal prints the same headline numbers, and `results/BENCHMARK_REPORT.md` has every table.
