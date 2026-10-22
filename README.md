# FightOrFold

A cost-aware dispute agent that accepts, challenges or escalates every chargeback based on evidence and cost, and decides again when the bank pushes back.

**Track:** Starter Kit 04 · Revenue Protection · Dispute Response Agent
**Team:** Hackstreet Boys (Abhishek Salaria, Yuvraj, Paras Bhardwaj)

## Architecture

<img width="3200" height="2080" alt="architecture" src="https://github.com/user-attachments/assets/8f723618-f5cc-4c26-94b3-c58737bf2952" />

## How it decides

- **Minimum chance needed** = extra cost of fighting ÷ amount at stake (calculated from the merchant's fees).
- **Estimated chance** = score from a reason-code evidence checklist (always shown as an estimate).
- Clearly above the minimum → **Challenge**. Clearly below → **Accept**. Too close, conflicting, or merchant at fault → **Escalate** to a person.
- If the bank rejects the proof, the agent recalculates and decides again.

## Built on the Airwallex sandbox (no extra enablement)

| Step | Endpoint |
|---|---|
| Create and confirm payments | `POST /pa/payment_intents/create`, `/confirm` |
| Stage chargebacks, bank pushback, final result | `POST /simulation/pa/payment_disputes/create`, `/escalate`, `/resolve` |
| Read chargebacks | `GET /pa/payment_disputes` |
| Upload proof (PDF/JPG) | `POST /files/upload` on files.sandbox.airwallex.com |
| Accept or challenge | `POST /pa/payment_disputes/{id}/accept`, `/challenge` |
| Check refunds | `GET /pa/refunds` |

## Guardrails

- Amounts, fees, thresholds and state transitions live in code. The AI model only reads text, matches evidence and drafts responses through typed tools.
- One action per dispute per stage, never a separate refund on a disputed payment, a unique `request_id` per call.
- Escalations need human approval. Every decision is logged with its evidence and numbers.

## Demo highlights

- **Batch run:** 20–30 sample disputes with a live counter comparing net recovery for FightOrFold vs "fight everything" vs "accept everything".
- **Realistic evidence:** messy support emails and delivery notes where the AI model spots the missing proof.
- **Second round:** the bank rejects the proof and the agent recalculates live, explaining why it fights again or stops.

## Engineering practices

| Practice | How |
|---|---|
| Idempotency | Unique `request_id` per action; one action per dispute per stage |
| No double refunds | Accept at RFI refunds in the same call; verified with `GET /pa/refunds` |
| Webhooks | Dispute updates via webhooks, HMAC signature verified, duplicate events ignored |
| Resilience | Token refresh every 30 minutes; backoff and retry on rate limits (429) |
| Tests | Decision-engine unit tests (e.g. $12 case → accept, $480 case → challenge) |
| Swappable simulation | Sandbox simulation calls behind one interface, separate from decision logic |
| Auditability | Every decision logged with evidence, numbers and approver |

*Sandbox only. Merchant evidence is sample data. Fees are set in the app because the sandbox doesn't charge them.*
