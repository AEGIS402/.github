<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=28&duration=2500&pause=1500&color=00FF41&center=true&vCenter=true&width=720&lines=AEGIS402+%2F%2F+ONLINE;Initializing+audit+agents...;Scanning+the+chain..." alt="typing" />

# `> AEGIS402_`

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
████████████████████████████████████████████████████████████████
█  ┌─[ AEGIS402 ]─────────────────────────────────────[ ONLINE ]┐
█  │ > AGENT_01 :: pre-audit    :: source-code analyzer         │
█  │ > AGENT_02 :: post-audit   :: transaction auditor          │
█  │ > MODEL    :: gpt-oss-120b                                 │
█  │ > MODE     :: evidence-based                               │
█  └────────────────────────────────────────────────────────────┘
████████████████████████████████████████████████████████████████
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

### Prime Directive — *the agent does not decode hex, the agent does not do math*

All hex decoding, decimal handling, and normalization run in **deterministic code**.
The LLM is restricted to the only thing it should ever do: **interpret, summarize, assign risk.**

```
  Collect raw RPC ──► Normalize ──► Decode ──► Subject-centric flows
                                                        │
                                                        ▼
                                               Rule signals
                                                        │
                                                        ▼
                                              ┌─────────────┐
                                              │  LLM AUDIT  │
                                              └──────┬──────┘
                                                     │
                                                     ▼
                                            JSON schema validate
```

### The Pipeline

| # | Step | Role |
|---|---|---|
| 1 | Preserve Raw Evidence | Keep RPC responses intact, tagged with `evidence_id` |
| 2 | Normalize Execution | status · gas · block · timestamp into readable form |
| 3 | Decode Calldata | ERC20 `transfer/approve/transferFrom`, Uniswap V3 `exactInputSingle` |
| 4 | Decode Event Logs | ERC20 `Transfer` / `Approval` |
| 5 | Subject-Centric Flows | Asset flows from the audit subject's perspective |
| 6 | Approval Changes | `Approval` events normalized into approval-change records |
| 7 | Rule Signals | `extreme_value_imbalance`, `missing_slippage_protection`, ... |
| 8 | LLM Audit | Evidence-based assessment driven only by the structured payload |

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

## `> THE CONSTRUCT`  &nbsp; *AI Stack*

```
╔═══════════════╦══════════════════════════════════════════════════════╗
║  Layer        ║  Stack                                                ║
╠═══════════════╬══════════════════════════════════════════════════════╣
║  Model        ║  gpt-oss-120b  (OpenAI-compatible, local)             ║
║  Inference    ║  self-hosted, OpenAI-compatible API                   ║
║  Validation   ║  JSON schema · evidence-ref check · post-processing   ║
║  Runtime      ║  Hardhat 3 · TypeScript · Node.js                     ║
║  Determinism  ║  all decoding/arithmetic/normalization in code        ║
╚═══════════════╩══════════════════════════════════════════════════════╝
```

### Why **evidence-based**?

LLMs hallucinate. The pipeline makes it impossible.

```
  ▸ Every finding MUST carry evidence_refs
  ▸ Findings referencing absent evidence are auto-stripped
  ▸ risk_level is recomputed from risk_score (model's value is overridden)
  ▸ Output normalized to ASCII English (blocks injection vectors)
```

---

<div align="center">

### `> PROJECTS`

[![pre-audit](https://img.shields.io/badge/AGENT_01-pre--audit-00FF41?style=for-the-badge&logo=solidity&logoColor=black)](https://github.com/AEGIS402/pre-audit)
[![post-audit](https://img.shields.io/badge/AGENT_02-post--audit-00B050?style=for-the-badge&logo=ethereum&logoColor=black)](https://github.com/AEGIS402/post-audit)

<br>

`▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮`

</div>
