<div align="center">

# AEGIS402

### LLM-Powered Security Audit Agents for Smart Contracts and On-Chain Transactions

Covering the full Web3 security lifecycle — from **pre-deployment** code to **post-deployment** transactions.

<br>

![Solidity](https://img.shields.io/badge/Solidity-363636?style=for-the-badge&logo=solidity&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Hardhat](https://img.shields.io/badge/Hardhat-F0D44A?style=for-the-badge&logo=ethereum&logoColor=black)
![Ethereum](https://img.shields.io/badge/Ethereum-3C3C3D?style=for-the-badge&logo=ethereum&logoColor=white)
![LLM](https://img.shields.io/badge/LLM-gpt--oss--120b-412991?style=for-the-badge)

</div>

---

## Why AEGIS402?

Smart contract security cannot be solved by a single point-in-time check.
Before deployment, you need to catch vulnerabilities in the **code itself**.
After deployment, you need to detect anomalies in **actual transactions**.

AEGIS402 addresses both with two dedicated LLM agents.

| | **Pre-Audit Agent** | **Post-Audit Agent** |
|---|---|---|
| **Phase** | Pre-deployment | Post-deployment |
| **Input** | Solidity source code | Ethereum transaction hash |
| **Goal** | Identify code vulnerabilities | Assess executed transaction risk |
| **Model** | `gpt-oss-120b` | `gpt-oss-120b` |
| **Output** | Vulnerabilities + severity + location | Risk level + evidence-based findings |

---

## Pre-Audit Agent

> Accepts Solidity source code, forwards it to an LLM-backed analyzer, and returns a structured **vulnerability report**.

### Key Features

- **HTTP API**: a single `POST /v1/contracts/analyze` endpoint accepting JSON or raw Solidity
- **Vulnerability classification**: reentrancy, `tx.origin` auth bypass, privilege misuse, and more (OWASP / SWC categories)
- **Quantitative risk scoring**: four-axis evaluation — `risk_score` · `confidence_score` · `impact_score` · `exploitability_score`
- **Evidence-based output**: every vulnerability includes `line_start` / `line_end` / `description` for traceability

### Example Output

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

[`AEGIS402/pre-audit` →](https://github.com/AEGIS402/pre-audit)

---

## Post-Audit Agent

> Takes a single Ethereum transaction hash, **deterministically decodes** standard RPC data,
> and lets an LLM perform an *evidence-based* risk assessment.

### Design Principle — *The LLM is an auditor, not a decoder*

The LLM never performs hex decoding or decimal arithmetic. All numeric and decoding work is done by deterministic code (`src/payload.ts`); the LLM only **interprets, summarizes, and assigns risk**.

```
Collect raw RPC data
  -> deterministically normalize and decode
  -> build evidence-based audit payload
  -> ask LLM for evidence-based assessment
  -> validate output with JSON schema
```

### Pipeline Steps

| Step | Role |
|---|---|
| 1. Preserve Raw Evidence | Keep RPC responses intact, tagged with `evidence_id` |
| 2. Normalize Execution | status · gas · block · timestamp into readable form |
| 3. Decode Calldata | ERC20 `transfer/approve/transferFrom`, Uniswap V3 `exactInputSingle` |
| 4. Decode Event Logs | ERC20 `Transfer` / `Approval` |
| 5. Subject-Centric Flows | Asset flows from the audit subject's perspective |
| 6. Approval Changes | `Approval` events normalized into approval-change records |
| 7. Rule Signals | Deterministic signals — `extreme_value_imbalance`, `missing_slippage_protection`, etc. |
| 8. LLM Audit | Evidence-based assessment driven only by the structured payload |

### Two API Endpoints

```http
POST /audit/subject     # Explicit audit subject address
POST /audit/from-tx     # Use tx.from as the audit subject
```

### Example Output — Critical Case

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

### Risk Matrix

| `risk_level` | `risk_score` | Meaning |
|---|---:|---|
| `low` | 0–24 | Normal activity or informational findings only |
| `medium` | 25–59 | Suspicious activity worth user confirmation |
| `high` | 60–84 | Strong risk signal, high-value exposure, dangerous approval |
| `critical` | 85–100 | Severe loss pattern, extreme value imbalance, urgent incident candidate |

[`AEGIS402/post-audit` →](https://github.com/AEGIS402/post-audit)

---

## AI Stack

<div align="center">

| Layer | Stack |
|---|---|
| **Model** | `gpt-oss-120b` (OpenAI-compatible local endpoint) |
| **Inference** | Local / self-hosted, OpenAI-compatible API |
| **Validation** | JSON schema · evidence-ref validation · deterministic post-processing |
| **Runtime** | Hardhat 3 · TypeScript · Node.js |
| **Determinism** | All decoding, arithmetic, and normalization handled by deterministic code |

</div>

### Why *evidence-based*?

To prevent LLM hallucination, the input payload enforces the following invariants:

- Every finding must include `evidence_refs`
- Findings referencing evidence not present in the payload are stripped automatically
- `risk_level` is recomputed from `risk_score` post-hoc (the model's level is overridden)
- Output is normalized to ASCII English to block injection-style risks

---

<div align="center">

### Projects

[![pre-audit](https://img.shields.io/badge/pre--audit-Pre--Deployment-blue?style=for-the-badge)](https://github.com/AEGIS402/pre-audit)
[![post-audit](https://img.shields.io/badge/post--audit-Post--Deployment-green?style=for-the-badge)](https://github.com/AEGIS402/post-audit)

</div>
