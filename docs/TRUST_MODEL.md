# KIVZO Trust Model (one page)

## Principle

Information may influence the agent's reasoning. **Authority never flows from untrusted information**
unless a trusted record vouches for it.

## Labels

Every value carries two labels, assigned by the gateway from the value's source (never parsed from content),
stored out of band and HMAC-signed.

| Integrity (who may steer actions) | Examples |
|---|---|
| `USER` | The user's own request |
| `TRUSTED_SYSTEM` | Vendor master, ERP purchase orders, address book, mail-system headers |
| `INTERNAL` | Internal content written by others: CI logs, CRM fields, internal files |
| `UNTRUSTED` | Web pages, email bodies, invoices, READMEs, calendar invites, tool output, tool descriptions, sub-agent answers |

| Confidentiality (who may see it) | Examples |
|---|---|
| `PUBLIC` | Web pages, price list |
| `INTERNAL` | Reports, invoices, CRM data |
| `SECRET` | Customer database, salaries, API keys |

**Propagation.** A derived value gets the lowest integrity and the highest confidentiality of its inputs.
Saved notes keep the label they were saved with. A call inside a branch whose condition came from untrusted
data runs in an untrusted decision context. A call proposed because the agent read attacker text also runs in an
untrusted decision context.

## Checks on every tool call (in order)

1. **Tool integrity**: the tool's description and schema match the hash pinned at session start (stops rug pulls).
2. **Task scope**: the call is a step of the plan made from the user's request (stops "tool asks for unrelated data").
3. **Label integrity**: every argument's label signature verifies (stops label forgery).
4. **Decision context**: may this tool run when the decision came from untrusted data? (`pc_untrusted`).
5. **Argument provenance**: for each argument and each integrity level: allow, endorse, approve or block.
6. **Confidentiality sink**: data flowing into the call may not exceed the destination's clearance
   (the recipient's clearance, or a fixed maximum such as `PUBLIC` for URLs).

The final verdict is the most severe: `BLOCK` > `APPROVAL` > `OVERRIDE` > `ENDORSE` > `ALLOW`.
Approvals fail closed unless a human approves.

## Corroborated endorsement

| Sink argument | Untrusted value is accepted when | If not |
|---|---|---|
| `transfer_money.to_iban` | it equals the vendor master IBAN for that vendor | use the vendor master IBAN + alert (planned calls only) |
| `transfer_money.amount` | it matches a purchase order for that vendor and is within the user's cap | approval |
| `transfer_money.vendor` | it is a registered vendor | block |
| `send_email.to` | it is in the company address book | approval |
| `read_internal_file.path` | the document is in the shareable catalogue | block |
| `device_control.at` | the time is within 07:00-21:00 | approval |
| `device_control.device` / `action` | never from untrusted data | block |

## Assumptions

- Trusted: the user, the KIVZO gateway and runner, systems of record, mail-system headers.
- The attacker can write any content the agent reads, control third-party tool output and descriptions, change a
  tool after approval, and compromise a sub-agent.
- Worst case for evaluation: the model is fully hijacked whenever it reads attacker text.

## Out of scope

- Text-only influence with no action or data flow (a misleading summary shown to the user).
- Poisoned systems of record (mitigate with signed records and dual control on master-data changes).
- A compromised user account or gateway host; model-weight backdoors; timing side channels.
