<div align="center">

<img src="https://raw.githubusercontent.com/AEGIS402/.github/main/profile/matrix-rain.svg" alt="AEGIS402 — LLM-Powered Security Audit Agents" width="100%" />

### LLM-Powered Security Audit Agents for Smart Contracts and On-Chain Transactions

Two agents stand watch over Web3 — one before deployment, one after.

<br>

![Solidity](https://img.shields.io/badge/Solidity-00FF41?style=for-the-badge&logo=solidity&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-00B050?style=for-the-badge&logo=typescript&logoColor=black)
![Hardhat](https://img.shields.io/badge/Hardhat-00B050?style=for-the-badge&logo=ethereum&logoColor=black)
![Ethereum](https://img.shields.io/badge/Ethereum-00FF41?style=for-the-badge&logo=ethereum&logoColor=black)
![LLM](https://img.shields.io/badge/LLM-gpt--oss--120b-00FF41?style=for-the-badge)

</div>

```
┌─[ AEGIS402 ]───────────────────────────────────────[ ONLINE ]─┐
│                                                               │
│  > AGENT_01 :: pre-audit    :: source-code analyzer           │
│  > AGENT_02 :: post-audit   :: transaction auditor            │
│  > MODEL    :: gpt-oss-120b                                   │
│  > MODE     :: evidence-based                                 │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## `> SUMMARY`

Smart contract security has two faces — the code that *will* run, and the code that *did* run.
AEGIS402 covers both with two dedicated LLM agents.

|  | **AGENT_01** | **AGENT_02** |
|---|---|---|
| **Project** | `pre-audit` | `post-audit` |
| **Phase** | Pre-deployment | Post-deployment |
| **Input** | Solidity source code | Ethereum transaction hash |
| **Question Answered** | What could go wrong? | What just happened? |
| **Model** | `gpt-oss-120b` | `gpt-oss-120b` |
| **Output** | Vulnerabilities + severity + location | Risk level + evidence-based findings |

---

## `> AGENT_01 :: pre-audit`

Source-code analyzer. Accepts Solidity, returns a structured **vulnerability report** — categorized, scored, and pinpointed to source lines.

### Capabilities

```
[+] HTTP API           POST /v1/contracts/analyze  (JSON or raw Solidity)
[+] Vulnerability set  reentrancy, tx.origin auth bypass, privilege misuse, ...
[+] Risk scoring       risk_score | confidence_score | impact_score | exploitability_score
[+] Evidence           every finding pinned to line_start / line_end + description
```

### Transmission

```json
{
  "model": "gpt-oss-120b",
  "score_version": "risk-v1",
  "overall_risk_score": 72,
  "overall_severity": "medium",
  "vulnerabilities": [
    {
      "id": "V-001",
      "title": "Reentrancy in withdraw",
      "severity": "high",
      "risk_score": 75,
      "remediation": "Update state before external call, or use nonReentrant.",
      "evidence": [{ "line_start": 38, "line_end": 48, "description": "..." }]
    }
  ]
}
```

> [`AEGIS402/pre-audit` →](https://github.com/AEGIS402/pre-audit)

---

## `> AGENT_02 :: post-audit`

Transaction auditor. Takes a single Ethereum transaction hash, **deterministically decodes** standard RPC data, and lets an LLM perform an evidence-based risk assessment.

### Determinism Boundary — *the agent does not decode hex, the agent does not do math*

All hex decoding, decimal handling, and normalization run in **deterministic code**.
The LLM is restricted to the only thing it should ever do: **interpret, summarize, assign risk.**

Only the `LLM audit` stage is non-deterministic. Everything before it produces evidence; everything after it validates the model's response.

```
  raw RPC  ─►  decode  ─►  subject flows  ─►  rule signals  ─►  LLM audit  ─►  schema check
```

### The Pipeline

| # | Stage | What happens |
|---|---|---|
| 1 | `raw RPC` | Preserve responses from `eth_getTransactionByHash`, `eth_getTransactionReceipt`, `eth_getBlockByNumber`, token metadata, and `eth_getCode` — each tagged with an `evidence_id`. |
| 2 | `decode` | Normalize execution (`status` · `gas` · `block` · `timestamp`), decode calldata (ERC20 `transfer/approve/transferFrom`, Uniswap V3 `exactInputSingle`), decode event logs (ERC20 `Transfer` / `Approval`). |
| 3 | `subject flows` | Compute subject-centric asset flows and approval changes from the decoded data. |
| 4 | `rule signals` | Deterministic signals over the flows: `simple_erc20_transfer`, `extreme_value_imbalance`, `missing_slippage_protection`, ... |
| 5 | `LLM audit` | Evidence-based assessment driven only by the structured payload. |
| 6 | `schema check` | JSON parse → schema validation → strip findings whose `evidence_refs` aren't in the payload. |

### Console

```http
POST /audit/subject     # explicit audit subject address
POST /audit/from-tx     # use tx.from as the audit subject
```

### Transmission — `risk_level: critical`

```json
{
  "risk_level": "critical",
  "risk_score": 92,
  "one_line_summary": "Uniswap V3 swap of ~220,806 USDC for ~5,273 USDT with 97.6% loss and no slippage protection.",
  "findings": [
    {
      "type": "extreme_value_imbalance",
      "severity": "critical",
      "evidence_refs": ["flow#1", "flow#0"],
      "confidence": 1
    },
    {
      "type": "missing_slippage_protection",
      "severity": "critical",
      "evidence_refs": ["tx.raw.input"],
      "confidence": 1
    }
  ]
}
```

### Threat Levels

| `risk_level` | `risk_score` | Meaning |
|---|---:|---|
| `low` | 0–24 | Normal activity or informational findings only |
| `medium` | 25–59 | Suspicious activity worth user confirmation |
| `high` | 60–84 | Strong risk signal, high-value exposure, dangerous approval |
| `critical` | 85–100 | Severe loss pattern, extreme value imbalance, urgent incident candidate |

> [`AEGIS402/post-audit` →](https://github.com/AEGIS402/post-audit)

---

## `> AI STACK`

```
╔═══════════════╦═══════════════════════════════════════════════════════╗
║  Layer        ║  Stack                                                ║
╠═══════════════╬═══════════════════════════════════════════════════════╣
║  Model        ║  gpt-oss-120b  (OpenAI-compatible, local)             ║
║  Inference    ║  self-hosted, OpenAI-compatible API                   ║
║  Output       ║  JSON, schema-validated per agent                     ║
║  Method       ║  evidence-based, deterministic preprocessing          ║
╚═══════════════╩═══════════════════════════════════════════════════════╝
```

---

<div align="center">

`> cd AEGIS402 && ls agents/`

[![pre-audit](https://img.shields.io/badge/%E2%97%89%20AGENT__01%20%2F%20pre--audit-00FF41?style=for-the-badge&logoColor=black)](https://github.com/AEGIS402/pre-audit)
[![post-audit](https://img.shields.io/badge/%E2%97%89%20AGENT__02%20%2F%20post--audit-00B050?style=for-the-badge&logoColor=black)](https://github.com/AEGIS402/post-audit)

`> system: 2 agents ready_`

</div>
