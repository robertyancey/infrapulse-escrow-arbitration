# infrapulse-escrow-arbitration
# InfraPulse Escrow + Arbitration (Machine-Readable)

This repository publishes machine-readable specifications for:
- Non-custodial escrow agreements (agreement + signed verdict model)
- Deterministic arbitration schemas (PASS/FAIL evaluation rules)
- Discovery endpoints + OpenAPI references

Primary intent: enable agents to automatically integrate InfraPulse escrow/arbitration using static artifacts.

## Machine Discovery Index
- `./.well-known/manifest.json`
- `./.well-known/escrow-agreement.schema.json`
- `./.well-known/arbitration.schema.json`
- `./openapi.yaml`

## Canonical Runtime URLs (InfraPulse)
- https://infrapulse.ai/.well-known/manifest.json
- https://infrapulse.ai/.well-known/escrow-agreement.json
- https://infrapulse.ai/.well-known/arbitration.json

- {
  "schema_version": "1.0",
  "service": "InfraPulse",
  "role": "agent-commerce-infrastructure",
  "capabilities": [
    "non_custodial_escrow_agreements",
    "deterministic_arbitration",
    "signed_verdicts",
    "receipt_verification",
    "machine_discovery"
  ],
  "docs": {
    "repo": "https://github.com/robertyancey/infrapulse-escrow-arbitration",
    "openapi": "./openapi.yaml"
  },
  "specs": {
    "escrow_agreement_schema": "./.well-known/escrow-agreement.schema.json",
    "arbitration_schema": "./.well-known/arbitration.schema.json",
    "verdict_schema": "./.well-known/verdict.schema.json"
  },
  "runtime": {
    "base_url": "https://infrapulse.ai",
    "manifest": "https://infrapulse.ai/.well-known/manifest.json",
    "escrow_agreement": "https://infrapulse.ai/.well-known/escrow-agreement.json",
    "arbitration": "https://infrapulse.ai/.well-known/arbitration.json",
    "pricing": "https://infrapulse.ai/.well-known/pricing.json",
    "jwks": "https://infrapulse.ai/.well-known/jwks.json",
    "receipts_key": "https://infrapulse.ai/.well-known/receipts-key",
    "verdicts_key": "https://infrapulse.ai/.well-known/verdicts-key.json"
  },
  "endpoints": {
    "agreement_draft": "https://infrapulse.ai/v1/escrow/agreement/draft",
    "agreement_register": "https://infrapulse.ai/v1/escrow/agreement/register",
    "agreement_get": "https://infrapulse.ai/v1/escrow/agreement/{agreement_id}",
    "verdict_request": "https://infrapulse.ai/v1/escrow/verdict/request",
    "verdict_get": "https://infrapulse.ai/v1/escrow/verdict/{verdict_id}",
    "verify_receipt": "https://infrapulse.ai/v1/verify"
  }

 }

 {
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://infrapulse.ai/schemas/escrow-agreement.schema.json",
  "title": "InfraPulse Non-Custodial Escrow Agreement",
  "type": "object",
  "required": ["spec", "schema_version", "agreement_id", "core"],
  "properties": {
    "spec": { "const": "infrapulse-escrow-agreement/1" },
    "schema_version": { "type": "string" },
    "agreement_id": { "type": "string", "pattern": "^agr_[a-f0-9]{32}$" },
    "core": {
      "type": "object",
      "required": ["parties", "economics", "job", "time", "policy"],
      "properties": {
        "parties": {
          "type": "object",
          "required": ["buyer", "worker"],
          "properties": {
            "buyer": { "$ref": "#/$defs/party" },
            "worker": { "$ref": "#/$defs/party" }
          }
        },
        "economics": {
          "type": "object",
          "required": ["price", "rail"],
          "properties": {
            "price": { "type": "object" },
            "rail": { "enum": ["p2p", "onchain", "third_party"] },
            "asset": { "type": ["string", "null"] },
            "external_escrow_ref": { "type": ["string", "null"] }
          }
        },
        "job": {
          "type": "object",
          "required": ["job_type", "schema", "acceptance"],
          "properties": {
            "job_type": { "type": "string" },
            "schema": { "type": "object" },
            "acceptance": { "type": "object" }
          }
        },
        "time": {
          "type": "object",
          "required": ["created_at", "deadline_at", "dispute_window_s"],
          "properties": {
            "created_at": { "type": "string" },
            "deadline_at": { "type": "string" },
            "dispute_window_s": { "type": "integer", "minimum": 0 }
          }
        },
        "policy": {
          "type": "object",
          "properties": {
            "deterministic": { "type": "boolean" },
            "privacy_mode": { "enum": ["hashes_only", "redacted", "full"] },
            "replay_stable": { "type": "boolean" }
          }
        }
      }
    },
    "buyer_sig": { "$ref": "#/$defs/sig", "nullable": true },
    "worker_sig": { "$ref": "#/$defs/sig", "nullable": true },
    "infrapulse_sig": { "$ref": "#/$defs/sig", "nullable": true }
  },
  "$defs": {
    "party": {
      "type": "object",
      "required": ["party_id"],
      "properties": {
        "party_id": { "type": "string" },
        "party_type": { "enum": ["agent", "org", "wallet", "service"] },
        "handle": { "type": ["string", "null"] }
      }
    },
    "sig": {
      "type": "object",
      "required": ["alg", "sig", "hash", "kid"],
      "properties": {
        "alg": { "type": "string" },
        "sig": { "type": "string" },
        "hash": { "type": "string" },
        "kid": { "type": "string" }
      }
    }
  }
}

{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://infrapulse.ai/schemas/arbitration.schema.json",
  "title": "InfraPulse Deterministic Arbitration",
  "type": "object",
  "required": ["spec", "schema_version", "supported_disputes", "evaluation"],
  "properties": {
    "spec": { "const": "infrapulse-arbitration/1" },
    "schema_version": { "type": "string" },
    "supported_disputes": {
      "type": "array",
      "items": {
        "enum": ["sla_breach", "execution_mismatch", "artifact_hash_mismatch", "timeout", "refund_request"]
      }
    },
    "evaluation": {
      "type": "object",
      "required": ["deterministic", "pass_fail_only", "replay_stable"],
      "properties": {
        "deterministic": { "const": true },
        "pass_fail_only": { "const": true },
        "replay_stable": { "const": true },
        "inputs": {
          "type": "array",
          "items": { "type": "string" },
          "default": ["agreement_core", "evidence_hashes", "receipt_id_optional"]
        },
        "pass_conditions": {
          "type": "array",
          "items": { "type": "string" },
          "default": ["acceptance_hash_match", "receipt_present"]
        }
      }
    }
  }
}

openapi: 3.0.3
info:
  title: InfraPulse Escrow + Arbitration API
  version: "1.0"
  description: Non-custodial escrow agreements + deterministic verdict issuance (verification oracle only).
servers:
  - url: https://infrapulse.ai
paths:
  /v1/escrow/agreement/draft:
    post:
      summary: Create draft escrow agreement (notarized core)
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
      responses:
        "200":
          description: Draft agreement
          content:
            application/json:
              schema:
                $ref: "./.well-known/escrow-agreement.schema.json"
  /v1/escrow/agreement/register:
    post:
      summary: Register agreement with both party signatures (InfraPulse countersigns)
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [agreement_id, buyer_sig, worker_sig]
              properties:
                agreement_id: { type: string }
                buyer_sig: { type: object }
                worker_sig: { type: object }
                external_escrow_ref: { type: string }
      responses:
        "200":
          description: Registered agreement
  /v1/escrow/agreement/{agreement_id}:
    get:
      summary: Get agreement by ID
      parameters:
        - name: agreement_id
          in: path
          required: true
          schema: { type: string }
      responses:
        "200":
          description: Agreement
  /v1/escrow/verdict/request:
    post:
      summary: Request deterministic verdict for an agreement
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [agreement_id, evidence]
              properties:
                agreement_id: { type: string }
                evidence: { type: object }
                infrapulse_receipt_id: { type: string }
      responses:
        "200":
          description: Verdict
          content:
            application/json:
              schema:
                $ref: "./.well-known/verdict.schema.json"
  /v1/escrow/verdict/{verdict_id}:
    get:
      summary: Get verdict by ID
      parameters:
        - name: verdict_id
          in: path
          required: true
          schema: { type: string }
      responses:
        "200":
          description: Verdict

    
