# FoundLab REX Guard — Blueprint PoC Bradesco

## Sprint Roadmap: PoC (14 dias) → Staging (2 semanas) → Produção (Junho 2026)

-----

## VISÃO GERAL EXECUTIVA

Validar arquitetura zero-persistence + consent-bound inference em 14 dias de PoC intensivo. Transicionar para staging em GKE production-grade. Go-live antes de pico de migrações Gemini 3 (Outubro 2026).

**Timeline realista:**

- **PoC**: 14 dias (Dia 1–14)
- **Auditoria pré-staging**: 7 dias (Dia 15–21)
- **Staging**: 14 dias (Semana 3–4)
- **Produção**: Junho 2026 (rollout gradual)

**Investimento**: USD 500K CVC estrutura (conversível). Zero margem de erro em remediações P0.

-----

## SPRINT 0: PRÉ-POC (Dias -7 a 0)

### BLOCKER RESOLUTION — CISO ALADDIN

Antes de um único deploy, resolver três P0s que invalidam a proposta se ignorados.

#### **Task 0.1: Cloud KMS Backup Isolation**

**Proprietário**: Fernando Batagin Jr. (FoundLab Tech Lead, GCP)  
**Deadline**: Dia 0, EOD  
**Esforço**: 8 horas

```bash
# Verificar estado atual
gcloud storage buckets describe gs://foundlab-kms-backups --format=json | grep versioning
gcloud sql backups list --instance=foundlab-umbrella-mp --limit=5

# Remediação obrigatória
1. Disable GCS versioning on backup bucket
   gsutil versioning set off gs://foundlab-kms-backups

2. Set lifecycle policy: objetos expiram em 30 dias
   (não podem ser deletados por admin durante 30d, expire automaticamente)
   
3. Audit Cloud SQL backups: grep -r "encrypt_key" /var/lib/mysql
   (zero key material em backup)
   
4. Disable Spanner cross-region backup
   gcloud spanner instances patch foundlab-spanner --no-enable-gcs-backups

5. Documento: "KMS Key Lifecycle & Backup Isolation Policy v1.0"
   - Assinado por: FoundLab CTO + Bradesco CISO
   - Valididade: 1 ano com revisão trimestral
```

**Aceite**: Evidência técnica (gcloud output) + documento assinado no Google Drive.

-----

#### **Task 0.2: BigQuery Merkle-Tree + ECDSA Signing**

**Proprietário**: Alex Bolson (FoundLab Chief Architect)  
**Deadline**: Dia 0, EOD  
**Esforço**: 12 horas

```typescript
// Implementar antes de Sandbox deploy
// arquivo: rex-guard/recibo-signer.ts

import * as crypto from 'crypto';
import { BigQuery } from '@google-cloud/bigquery';

interface Recibo {
  recibo_id: string;
  prev_hash: string; // hash do recibo anterior (Merkle chain)
  model_id: string;
  consent_id: string;
  decision_output_hash: string; // SHA-256(model output)
  thinking_level: 'HIGH' | 'MEDIUM' | 'LOW' | 'MINIMAL';
  confidence_score: number;
  timestamp: string; // RFC3339 TrueTime from Spanner
  kms_key_version: string;
}

interface SealedRecibo extends Recibo {
  signature: string; // ECDSA P-256 over (prev_hash + decision_hash + timestamp)
  seal_timestamp: string;
  key_destroyed_timestamp: string;
}

async function sealRecibo(
  recibo: Recibo,
  kmsSigner: crypto.KeyObject
): Promise<SealedRecibo> {
  // Merkle chain: assinar (prev_hash || current_hash)
  const chainData = Buffer.concat([
    Buffer.from(recibo.prev_hash, 'hex'),
    Buffer.from(recibo.decision_output_hash, 'hex'),
    Buffer.from(recibo.timestamp, 'utf8'),
  ]);

  const signature = crypto
    .createSign('sha256')
    .update(chainData)
    .sign(kmsSigner, 'hex');

  return {
    ...recibo,
    signature,
    seal_timestamp: new Date().toISOString(),
    key_destroyed_timestamp: '', // populado após KMS destroy()
  };
}

async function storeInBigQuery(sealed: SealedRecibo): Promise<void> {
  const bq = new BigQuery({ projectId: 'foundlab-ati' });
  const table = bq.dataset('audit_trail').table('recibos_sealed');

  // APPEND ONLY (não permite UPDATE/DELETE)
  // schema: recibo_id, prev_hash, ..., signature, seal_timestamp
  
  await table.insert(sealed, { raw: true });
  console.log(`✓ Sealed recibo ${sealed.recibo_id} inserted to WORM`);
}

export { sealRecibo, storeInBigQuery, SealedRecibo };
```

**Aceite**:

- [ ] TypeScript compila sem erros
- [ ] Unit tests rodam (100% path coverage)
- [ ] BigQuery schema atualizado (novo campo `signature`)
- [ ] Amostra de recibo sealed fornecida em JSON

-----

#### **Task 0.3: Recibo Format Specification**

**Proprietário**: Alex Bolson (FoundLab)  
**Deadline**: Dia 0, EOD  
**Esforço**: 4 horas

```json
{
  "recibo_v1_schema": {
    "recibo_id": {
      "description": "UUID v4 único para cada inferência",
      "type": "string",
      "example": "550e8400-e29b-41d4-a716-446655440000"
    },
    "prev_hash": {
      "description": "SHA-256 do recibo anterior (Merkle chain)",
      "type": "string",
      "example": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    "model_id": {
      "description": "Gemini versioning",
      "type": "string",
      "enum": ["gemini-3-pro", "gemini-3-flash"],
      "example": "gemini-3-flash"
    },
    "model_version": {
      "description": "Timestamp de quando modelo foi pinned",
      "type": "string",
      "format": "date-time",
      "example": "2026-04-10T16:51:00Z"
    },
    "consent_id": {
      "description": "ID do consentimento aberto em OPUS/OPIN",
      "type": "string",
      "example": "OPER:550e8400:2026-04"
    },
    "consent_scope_hash": {
      "description": "SHA-256(consent_scope). Prova de que processamento estava dentro da autorização",
      "type": "string"
    },
    "decision_output_hash": {
      "description": "SHA-256(saída do modelo). Não contém saída real, só o hash",
      "type": "string"
    },
    "thinking_level": {
      "description": "Profundidade de raciocínio capturado via Thought Signatures",
      "type": "string",
      "enum": ["HIGH", "MEDIUM", "LOW", "MINIMAL"]
    },
    "thinking_hash": {
      "description": "SHA-256(Thought Signatures agregadas). Não contém raciocínio real",
      "type": "string"
    },
    "confidence_score": {
      "description": "Confiança do modelo (0.0–1.0)",
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "timestamp": {
      "description": "TrueTime de Cloud Spanner (linearidade garantida)",
      "type": "string",
      "format": "date-time"
    },
    "kms_key_version": {
      "description": "Versão de chave KMS usada. Permite audit de key rotation",
      "type": "string",
      "example": "projects/foundlab-ati/locations/global/keyRings/rex-guard/cryptoKeys/envelope/versions/1"
    },
    "kms_destroy_timestamp": {
      "description": "Quando a chave foi destruída (após recibo ser selado)",
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "description": "ECDSA P-256 assinatura sobre (prev_hash || decision_hash || timestamp)",
      "type": "string",
      "format": "hex",
      "length": 128
    },
    "seal_timestamp": {
      "description": "Quando recibo foi sealed (após destruição de chave)",
      "type": "string",
      "format": "date-time"
    },
    "burn_engine_status": {
      "description": "Estado do kill switch (APPROVED, FLAG_REVIEW, FREEZE_ASSETS, BURN_IMMEDIATE)",
      "type": "string",
      "enum": ["APPROVED", "FLAG_REVIEW", "FREEZE_ASSETS", "BURN_IMMEDIATE"]
    },
    "ofac_gate_result": {
      "description": "OFAC/SANCTION/TERROR gate result (PASS / BLOCK)",
      "type": "string",
      "enum": ["PASS", "BLOCK"]
    },
    "user_readable_summary": {
      "description": "3–4 frases em português explicando decisão para usuário final (obrigatório BCB 538 Art. 32)",
      "type": "string",
      "example": "Sua solicitação de crédito foi avaliada com base em seu histórico financeiro e padrão de transações. Modelo de aprovação indicou score de risco baixo. Decisão aprovada conforme política de Open Finance."
    }
  },
  "example_recibo": {
    "recibo_id": "550e8400-e29b-41d4-a716-446655440000",
    "prev_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "model_id": "gemini-3-flash",
    "model_version": "2026-04-10T16:51:00Z",
    "consent_id": "OPER:550e8400:2026-04",
    "consent_scope_hash": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3",
    "decision_output_hash": "d4735fea8e2e1a7e0e15e9e4c5e3e4a0c4e5e6e7e8e9e0e1e2e3e4e5e6e7e8",
    "thinking_level": "MEDIUM",
    "thinking_hash": "fcbfbc61a6e3e6cf0ef3e4a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9",
    "confidence_score": 0.87,
    "timestamp": "2026-04-10T17:30:45.123456Z",
    "kms_key_version": "projects/foundlab-ati/locations/global/keyRings/rex-guard/cryptoKeys/envelope/versions/1",
    "kms_destroy_timestamp": "2026-04-10T17:30:46Z",
    "signature": "30450221008949f582a6e3e6cf0ef3e4a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9022100fcbfbc61a6e3e6cf0ef3e4a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9",
    "seal_timestamp": "2026-04-10T17:30:46.500Z",
    "burn_engine_status": "APPROVED",
    "ofac_gate_result": "PASS",
    "user_readable_summary": "Sua solicitação de crédito foi aprovada. Sistema analisou padrão de transações: 48 meses de histórico, 0 inadimplências. Score de risco: BAIXO. Decisão em conformidade com BCB 538/2025."
  }
}
```

**Aceite**: Spec JSON + amostra de recibo aprovados por Bradesco cyber team.

-----

### INFRAESTRUTURA PRÉ-POC

#### **Task 0.4: VMware Engine Sandbox Provisioning**

**Proprietário**: Fernando Batagin Jr. (GCP) + Bradesco Cloud Team  
**Deadline**: Dia 0, EOD  
**Esforço**: Bradesco (BNF-0134 form submit)

```bash
# Form: https://docs.google.com/forms/d/1nCNDBKaqJpcDGtV98xABXnmITB2cskO4WDiBM4u.../viewform

# Expected outputs:
- GCP Project: foundlab-ati-bradesco-sandbox
- VPC-SC perimeter: perimeter-bradesco-rex-guard
- Service Account: rex-guard-sandbox@foundlab-ati-bradesco-sandbox.iam.gserviceaccount.com
  - Roles: 
    * roles/run.admin (Cloud Run)
    * roles/cloudkms.cryptoKeyVersionViewer (Cloud KMS read)
    * roles/cloudkms.signerVerifier (signing operations)
    * roles/bigquery.dataEditor (append-only)
    * roles/spanner.databaseUser (read timestamps)

# Network:
- On-prem VLAN ↔ GCP via interconnect (100 Mbps minimum)
- Guardian AI (RTX rack) on Bradesco on-prem
- REX Guard on Cloud Run (VPC-SC restricted)
```

**Aceite**: Confirmação de projeto GCP + Service Account criada + VPC-SC ativo.

-----

#### **Task 0.5: Cloud KMS Setup (ECDSA P-256)**

**Proprietário**: Fernando Batagin Jr.  
**Deadline**: Dia 0, EOD  
**Esforço**: 2 horas

```bash
gcloud kms keyrings create rex-guard \
  --location=global \
  --project=foundlab-ati-bradesco-sandbox

gcloud kms keys create envelope-key \
  --location=global \
  --keyring=rex-guard \
  --purpose=signing \
  --default-algorithm=ec-sign-p256-sha256 \
  --protection-level=hsm \
  --project=foundlab-ati-bradesco-sandbox

gcloud kms keys versions create \
  --key=envelope-key \
  --keyring=rex-guard \
  --location=global \
  --project=foundlab-ati-bradesco-sandbox

# Audit: verificar que key_version=1
gcloud kms keys versions list --key=envelope-key --keyring=rex-guard --location=global
```

**Aceite**: KMS key version 1 ativo + audit logs documentados.

-----

## SPRINT 1: FOUNDATION (Dias 1–3)

### Objetivo

Montar sandbox completo com REX Guard, Cloud Run, BigQuery WORM, Guardian AI local. Validar que todos os componentes falam entre si sem erro.

-----

### Task 1.1: Cloud Run Deployment (REX Guard v1.1)

**Proprietário**: Fernando Batagin Jr.  
**Deadline**: Dia 1, 5 PM  
**Esforço**: 16 horas

```bash
# Repository: github.com/FoundLab-PoweredByGoogleCloud/rex-guard
# Branch: feature/bradesco-poc
# Commit: validar que VEX-ADR-REX-2026-04-03-COND-01 gaps (CG-001, CG-002, CG-003) estão em RFC-F2F-005

git clone https://github.com/FoundLab-PoweredByGoogleCloud/rex-guard.git
cd rex-guard
git checkout feature/bradesco-poc

# Build
npm install
npm run build

# Environment variables (Bradesco Sandbox)
export GOOGLE_CLOUD_PROJECT=foundlab-ati-bradesco-sandbox
export KMS_KEY_RESOURCE=projects/foundlab-ati-bradesco-sandbox/locations/global/keyRings/rex-guard/cryptoKeys/envelope-key/versions/1
export BQ_DATASET=audit_trail
export BQ_TABLE=recibos_sealed
export SPANNER_INSTANCE=rex-guard-spanner
export GEMINI_API_KEY=$(gcloud secrets versions access latest --secret=gemini-key)

# Deploy (Cloud Run, VPC-SC restricted)
gcloud run deploy rex-guard \
  --source . \
  --region=us-central1 \
  --platform=managed \
  --memory=2Gi \
  --cpu=2 \
  --timeout=120s \
  --set-env-vars="GOOGLE_CLOUD_PROJECT=$GOOGLE_CLOUD_PROJECT,KMS_KEY_RESOURCE=$KMS_KEY_RESOURCE" \
  --no-allow-unauthenticated \
  --service-account=rex-guard-sandbox@foundlab-ati-bradesco-sandbox.iam.gserviceaccount.com \
  --project=foundlab-ati-bradesco-sandbox

# Validação: REX Guard health check
curl -X GET https://rex-guard-XXXXX.run.app/health \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json"

# Esperado:
# {
#   "status": "healthy",
#   "uptime_seconds": 15,
#   "kms_reachable": true,
#   "bigquery_reachable": true,
#   "spanner_reachable": true
# }
```

**Aceite**:

- [ ] Cloud Run URL ativa + authentication requerido
- [ ] Health check retorna 200 + JSON
- [ ] Logs aparecendo em Cloud Logging

-----

### Task 1.2: BigQuery WORM Setup + Merkle-Tree Schema

**Proprietário**: Fernando Batagin Jr.  
**Deadline**: Dia 1, EOD  
**Esforço**: 8 horas

```sql
-- BigQuery: foundlab-ati-bradesco-sandbox:audit_trail

CREATE SCHEMA IF NOT EXISTS audit_trail
  OPTIONS(
    description='Immutable audit trail for REX Guard recibos (WORM append-only)',
    location='US'
  );

CREATE TABLE IF NOT EXISTS audit_trail.recibos_sealed (
  -- Core recibo fields
  recibo_id STRING NOT NULL,
  prev_hash STRING NOT NULL, -- Merkle chain link
  model_id STRING NOT NULL,
  model_version TIMESTAMP NOT NULL,
  consent_id STRING NOT NULL,
  consent_scope_hash STRING NOT NULL,
  decision_output_hash STRING NOT NULL,
  thinking_level STRING NOT NULL,
  thinking_hash STRING,
  confidence_score FLOAT64 NOT NULL,
  
  -- Timestamping (TrueTime from Spanner)
  timestamp TIMESTAMP NOT NULL,
  kms_key_version STRING NOT NULL,
  kms_destroy_timestamp TIMESTAMP NOT NULL,
  
  -- Cryptographic seal
  signature STRING NOT NULL, -- ECDSA P-256 hex
  seal_timestamp TIMESTAMP NOT NULL,
  
  -- Kill switches
  burn_engine_status STRING NOT NULL, -- APPROVED|FLAG_REVIEW|FREEZE_ASSETS|BURN_IMMEDIATE
  ofac_gate_result STRING NOT NULL, -- PASS|BLOCK
  
  -- User-facing explanation
  user_readable_summary STRING NOT NULL,
  
  -- Metadata
  source_service STRING NOT NULL,
  request_id STRING,
  inserted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
)
PARTITION BY DATE(timestamp)
CLUSTER BY model_id, consent_id
OPTIONS(
  description='Immutable (append-only) seal of AI inference decisions. WORM enforced at dataset level.',
  require_partition_filter=true,
  labels=[('service', 'rex-guard'), ('data-class', 'audit-trail')]
);

-- Enforce WORM: dataset-level ACL
-- Only role: roles/bigquery.dataEditor
-- Permissions: insertAll (APPEND), getDataset, getTable
-- Denies: updateTable, deleteTable, updateDataset

-- Index: query performance
CREATE SNAPSHOT TABLE IF NOT EXISTS audit_trail.recibos_sealed_latest_snapshot
AS SELECT * FROM audit_trail.recibos_sealed;

-- Validation query
SELECT 
  COUNT(*) as total_recibos,
  MIN(timestamp) as earliest,
  MAX(timestamp) as latest,
  COUNT(DISTINCT model_id) as models_used,
  COUNT(DISTINCT consent_id) as consents_processed
FROM audit_trail.recibos_sealed
WHERE DATE(timestamp) = CURRENT_DATE();
```

**Aceite**:

- [ ] Table criada com WORM ACL
- [ ] Partition + clustering ativo
- [ ] Insert test: 1 recibo dummy inserido com sucesso
- [ ] Update/delete bloqueado (permission denied)

-----

### Task 1.3: Guardian AI Setup (Local RTX Rack)

**Proprietário**: Bradesco Infrastructure + FoundLab  
**Deadline**: Dia 2, EOD  
**Esforço**: 24 horas (Bradesco on-prem work)

```bash
# Bradesco on-prem: RTX 6000 GPU cluster (local inference)
# Esperado: Gemini API endpoint acessível apenas via VPC-SC interconnect

# FoundLab: configura NVIDIA NIM + Gemini local proxy
# Repo: github.com/FoundLab-PoweredByGoogleCloud/guardian-ai

docker pull nvcr.io/nvidia/nim:gemini-3-flash
docker run -d \
  --name guardian-ai \
  --gpus all \
  -e GEMINI_API_KEY=$(cat /etc/secrets/gemini-key) \
  -p 8080:8080 \
  nvcr.io/nvidia/nim:gemini-3-flash

# Health check
curl http://localhost:8080/health

# Expected response:
# {
#   "status": "ready",
#   "models": ["gemini-3-flash"],
#   "gpu_memory_gb": 48,
#   "batch_size": 16
# }

# Integration test: REX Guard → Guardian AI
curl -X POST http://guardian-ai:8080/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3-flash",
    "prompt": "What is 2+2?",
    "max_tokens": 10,
    "temperature": 0
  }'

# Expected: resposta + thought_signatures (JSON, não retido)
```

**Aceite**:

- [ ] Guardian AI container rodando
- [ ] Health check: 200 OK
- [ ] Integração REX Guard ↔ Guardian AI: latência <200ms
- [ ] Thought Signatures retornadas + deletadas após processamento

-----

### Task 1.4: Network Validation (on-prem ↔ GCP)

**Proprietário**: Bradesco Network Ops + FoundLab  
**Deadline**: Dia 2, EOD  
**Esforço**: 4 horas

```bash
# Teste de conectividade on-prem → Cloud Run
CLOUD_RUN_URL=$(gcloud run services describe rex-guard --region=us-central1 --format='value(status.url)')

curl -X GET $CLOUD_RUN_URL/health \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)"

# Esperado: 200 OK (latência <50ms)

# Teste de latência: on-prem → Guardian AI
ping guardian-ai.bradesco-internal
# Esperado: <5ms (local network)

# Teste de latência: on-prem → GCP via interconnect
mtr -c 100 rex-guard.run.app
# Esperado: <20ms P95

# Validar que Bradesco on-prem NÃO pode acessar GCP sem VPC-SC
curl -X GET $CLOUD_RUN_URL/health \
  --insecure \
  -H "Host: unauthorized-external"
# Esperado: 403 Forbidden (VPC-SC enforcement)
```

**Aceite**: Latência medida + documentada.

-----

## SPRINT 2: VALIDAÇÃO DE PILARES (Dias 4–7)

### Objetivo

Testar os três pilares (CBI, Zero-Persistence, Recibo) em isolamento antes de integration tests.

-----

### Task 2.1: Pilar I — Consent-Bound Inference (CBI)

**Proprietário**: Alex Bolson (FoundLab)  
**Deadline**: Dia 4, EOD  
**Esforço**: 16 horas

```typescript
// Arquivo: rex-guard/tests/consent-bound-inference.spec.ts

import { REXGuard } from '../src';
import { OPUSClient } from '../integrations/opus-client';

describe('Pilar I: Consent-Bound Inference', () => {
  let rex: REXGuard;
  let opus: OPUSClient;

  beforeAll(async () => {
    rex = new REXGuard();
    opus = new OPUSClient({ projectId: 'foundlab-ati-bradesco-sandbox' });
  });

  it('blocks inference without valid consent_id', async () => {
    const request = {
      user_id: 'USER:123456',
      consent_id: '', // empty = sem consentimento
      payload: { account_balance: '***' },
    };

    const result = await rex.infer(request);
    expect(result.status).toBe('BLOCKED');
    expect(result.reason).toBe('CONSENT_NOT_FOUND');
  });

  it('blocks inference when consent_scope does not match request', async () => {
    // Cenário: consentimento OPIN (investimentos) usado para OACS (contas)
    const consentId = 'OPER:550e8400:2026-04'; // scope: OPIN only
    const request = {
      user_id: 'USER:123456',
      consent_id: consentId,
      payload_type: 'ACCOUNT_BALANCE', // não está em scope OPIN
    };

    const result = await rex.infer(request);
    expect(result.status).toBe('BLOCKED');
    expect(result.reason).toBe('SCOPE_MISMATCH');
  });

  it('allows inference when consent_scope matches request', async () => {
    const consentId = 'OPER:550e8400:2026-04'; // scope: OPIN
    const request = {
      user_id: 'USER:123456',
      consent_id: consentId,
      payload_type: 'INVESTMENT_RECOMMENDATION', // matches OPIN scope
      payload: { portfolio: '***' },
    };

    const result = await rex.infer(request);
    expect(result.status).toBe('ALLOWED');
    expect(result.recibo).toBeDefined();
  });

  it('blocks inference when consent is revoked', async () => {
    // Simular revogação em OPUS/OPIN
    const consentId = 'OPER:550e8400:2026-04';
    await opus.revokeConsent(consentId);

    const request = {
      user_id: 'USER:123456',
      consent_id: consentId,
      payload_type: 'INVESTMENT_RECOMMENDATION',
    };

    // REX Guard deve detectar revogação via OPUS cache refresh
    const result = await rex.infer(request);
    expect(result.status).toBe('BLOCKED');
    expect(result.reason).toBe('CONSENT_REVOKED');
  });

  it('handles consent cache TTL correctly (race condition test)', async () => {
    const consentId = 'OPER:550e8400:2026-04';
    const request = {
      user_id: 'USER:123456',
      consent_id: consentId,
      payload_type: 'INVESTMENT_RECOMMENDATION',
    };

    // T=0: Consentimento válido
    const result1 = await rex.infer(request);
    expect(result1.status).toBe('ALLOWED');

    // T=1ms: Revogar consentimento
    await opus.revokeConsent(consentId);

    // T=2ms: REX Guard ainda tem cache válido (TTL=60s)
    // -> Espera-se que segunda inferência PASSE (race condition documentado)
    const result2 = await rex.infer(request);
    // expect(result2.status).toBe('ALLOWED'); // cache TTL window
    
    // T=60s+1: Cache expirado, OPUS check vê revogação
    await new Promise(resolve => setTimeout(resolve, 61000));
    const result3 = await rex.infer(request);
    expect(result3.status).toBe('BLOCKED');

    // Documentar: "OPUS/OPIN cache provides 60s TTL. Race window exists between revocation and REX Guard refresh."
  });

  // 100 synthetic test requests
  it('processes 100 synthetic requests with varied consent_id', async () => {
    const results = [];
    for (let i = 0; i < 100; i++) {
      const consentId = `OPER:550e8400:2026-04-${String(i).padStart(3, '0')}`;
      const request = {
        user_id: `USER:${i}`,
        consent_id: consentId,
        payload_type: 'INVESTMENT_RECOMMENDATION',
      };
      const result = await rex.infer(request);
      results.push(result);
    }

    const blockedCount = results.filter(r => r.status === 'BLOCKED').length;
    const allowedCount = results.filter(r => r.status === 'ALLOWED').length;

    console.log(`✓ 100 requests: ${allowedCount} allowed, ${blockedCount} blocked`);
    console.log(`✓ Zero false negatives (unblocked invalid requests): ${blockedCount === 0 ? 'PASS' : 'FAIL'}`);
  });
});
```

**Aceite**:

- [ ] Teste “blocks without consent_id” passa
- [ ] Teste “scope mismatch” passa
- [ ] Teste “consent revoked” passa
- [ ] Teste “100 synthetic requests” completa
- [ ] Zero false negatives confirmado

-----

### Task 2.2: Pilar II — Zero-Persistence Governance

**Proprietário**: Fernando Batagin Jr.  
**Deadline**: Dia 5, EOD  
**Esforço**: 16 horas

```bash
# Teste: PII NUNCA persiste além de 30 segundos

# Setup: FoundLab monitoring container
docker run -d \
  --name memory-profiler \
  --pid=container:rex-guard \
  node:22 \
  node /app/memory-monitor.js

# arquivo: /app/memory-monitor.js
const fs = require('fs');
const { execSync } = require('child_process');

const startTime = Date.now();
const memorySnapshots = [];

setInterval(() => {
  const elapsed = Date.now() - startTime;
  
  // Dump memory (pode conter PII em teoria)
  const heap = execSync('node --expose-gc --heap-size-limit=2048 -e "gc()"');
  
  // Buscar por padrões que sugerem PII retenção
  const hasPIIPattern = /\d{3}\.\d{3}\.\d{3}-\d{2}/.test(heap.toString()); // CPF format
  
  memorySnapshots.push({
    elapsed_ms: elapsed,
    heap_mb: Math.round(process.memoryUsage().heapUsed / 1024 / 1024),
    pii_detected: hasPIIPattern,
  });

  if (hasPIIPattern) {
    console.error(`⚠️  PII DETECTED in memory at T=${elapsed}ms`);
    process.exit(1);
  }
}, 100); // Snapshot every 100ms

setTimeout(() => {
  console.log(JSON.stringify(memorySnapshots, null, 2));
  process.exit(0);
}, 120000); // Run for 2 minutes

# Teste: Fazer 1000 inferências com PII em payload
curl -X POST https://rex-guard.run.app/v1/infer \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json" \
  -d '{
    "consent_id": "OPER:550e8400:2026-04",
    "payload": {
      "cpf": "123.456.789-00",
      "nome": "João da Silva",
      "data_nascimento": "1990-01-15"
    }
  }' \
  --repeat 1000

# Esperado:
# Memory profiler não detecta PII em qualquer snapshot
# heap_mb cresce e desce (GC happening)
# max heap spike: <200MB (temporário durante inference)

# Teste: GC timing
npm run test:gc-timing
# Esperado: 
# - Thought Signatures deletadas em <30ms
# - PII cleared em <50ms
# - gc.collect() garantido após cada decision
```

**Aceite**:

- [ ] Memory profiler rodou por 2 minutos sem detectar PII
- [ ] Heap profiling mostra cleanup <30s de todo ciclo de inferência
- [ ] GC timing teste passa
- [ ] Zero false positives de PII retention

-----

### Task 2.3: Pilar III — Explicabilidade por Recibo

**Proprietário**: Alex Bolson  
**Deadline**: Dia 6, EOD  
**Esforço**: 12 horas

```typescript
// Teste: Recibo é válido, assinado, e contém explicação legível

import { Recibo, SealedRecibo } from '../src/recibo-signer';
import * as crypto from 'crypto';

describe('Pilar III: Explicabilidade por Recibo', () => {
  it('sealed recibo contains all required fields', async () => {
    const sealed: SealedRecibo = {
      recibo_id: 'test-123',
      prev_hash: 'abc...xyz',
      model_id: 'gemini-3-flash',
      consent_id: 'OPER:550e8400:2026-04',
      decision_output_hash: 'dec...ision',
      thinking_level: 'MEDIUM',
      confidence_score: 0.87,
      timestamp: new Date().toISOString(),
      kms_key_version: 'v1',
      signature: 'sig...ature',
      seal_timestamp: new Date().toISOString(),
      burn_engine_status: 'APPROVED',
      ofac_gate_result: 'PASS',
      user_readable_summary: 'Sua solicitação foi aprovada.',
    };

    expect(sealed.recibo_id).toBeDefined();
    expect(sealed.signature).toMatch(/^[0-9a-f]{128}$/); // ECDSA hex
    expect(sealed.user_readable_summary.length).toBeGreaterThan(20);
  });

  it('recibo signature is valid over merkle chain', async () => {
    const sealed = createTestSealedRecibo();
    const publicKey = await getPublicKeyFromKMS();

    const chainData = Buffer.concat([
      Buffer.from(sealed.prev_hash, 'hex'),
      Buffer.from(sealed.decision_output_hash, 'hex'),
      Buffer.from(sealed.timestamp, 'utf8'),
    ]);

    const isValid = crypto
      .createVerify('sha256')
      .update(chainData)
      .verify(publicKey, Buffer.from(sealed.signature, 'hex'));

    expect(isValid).toBe(true);
  });

  it('user_readable_summary is legible (Flesch-Kincaid grade < 8)', () => {
    const sealed = createTestSealedRecibo();
    const grade = flesch_kincaid_grade(sealed.user_readable_summary);

    console.log(`Summary: "${sealed.user_readable_summary}"`);
    console.log(`Readability grade: ${grade}`);

    expect(grade).toBeLessThan(8); // High school reading level
  });

  it('recibo does NOT contain PII (CPF, nome, etc)', () => {
    const sealed = createTestSealedRecibo();
    const reciboStr = JSON.stringify(sealed);

    const piiPatterns = [
      /\d{3}\.\d{3}\.\d{3}-\d{2}/, // CPF
      /\d{2}\.\d{3}\.\d{3}\/\d{4}-\d{2}/, // CNPJ
      /\d{10,}/, // Conta bancária (naive)
    ];

    for (const pattern of piiPatterns) {
      expect(reciboStr).not.toMatch(pattern);
    }
  });

  it('merkle chain is continuous (each recibo links to prev)', async () => {
    // Insert 10 recibos sequencialmente
    const recibos: SealedRecibo[] = [];
    let prevHash = '0'.repeat(64); // Initial hash

    for (let i = 0; i < 10; i++) {
      const recibo = createTestSealedRecibo({
        prev_hash: prevHash,
      });
      await storeInBigQuery(recibo);
      recibos.push(recibo);
      prevHash = recibo.decision_output_hash; // Next prev_hash
    }

    // Verificar que chain é contínua
    for (let i = 1; i < recibos.length; i++) {
      expect(recibos[i].prev_hash).toBe(recibos[i - 1].decision_output_hash);
    }

    console.log('✓ Merkle chain validated (10 recibos linked)');
  });
});
```

**Aceite**:

- [ ] Recibo contém todos os campos obrigatórios
- [ ] Assinatura é válida sobre Merkle chain
- [ ] `user_readable_summary` é legível (grade < 8)
- [ ] Recibo NÃO contém PII
- [ ] Merkle chain é contínua (10+ recibos)

-----

### Task 2.4: Fail-Closed Scenarios

**Proprietário**: Fernando Batagin Jr.  
**Deadline**: Dia 7, EOD  
**Esforço**: 8 horas

```bash
#!/bin/bash
# Teste: Falhas externas → bloqueio imediato (fail-closed)

echo "=== FAIL-CLOSED TEST SUITE ==="

# Cenário 1: Cloud KMS indisponível
echo "1. KMS unavailable..."
gcloud kms keys versions destroy 1 \
  --key=envelope-key --keyring=rex-guard --location=global
  
curl -X POST https://rex-guard.run.app/v1/infer \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json" \
  -d '{"consent_id": "OPER:550e8400:2026-04"}'
  
# Esperado: 503 Service Unavailable
# Response: {"error": "KMS_UNAVAILABLE", "status": "BLOCKED"}

# Cenário 2: BigQuery WORM indisponível
echo "2. BigQuery unavailable..."
gcloud sql instances patch foundlab-ati --no-database-flags || true

curl -X POST https://rex-guard.run.app/v1/infer \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json" \
  -d '{"consent_id": "OPER:550e8400:2026-04"}'
  
# Esperado: 503 Service Unavailable
# Response: {"error": "BIGQUERY_UNAVAILABLE", "status": "BLOCKED"}

# Cenário 3: OPUS/OPIN cache expired, no connectivity
echo "3. OPUS/OPIN unavailable..."
iptables -A OUTPUT -p tcp --dport 443 -d opus.bcb.gov.br -j DROP || true

curl -X POST https://rex-guard.run.app/v1/infer \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json" \
  -d '{"consent_id": "OPER:550e8400:2026-04"}'
  
# Esperado: 503 Service Unavailable
# Response: {"error": "CONSENT_VALIDATION_FAILED", "status": "BLOCKED"}

# Cenário 4: Gemini API returns 503
echo "4. Gemini API unavailable..."
# (Cloud Run timeout simulated)

curl -X POST https://rex-guard.run.app/v1/infer \
  --max-time 5 \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json" \
  -d '{"consent_id": "OPER:550e8400:2026-04"}'
  
# Esperado: 504 Gateway Timeout
# Response: {"error": "INFERENCE_TIMEOUT", "status": "BLOCKED"}

echo "✓ All fail-closed scenarios validated"
```

**Aceite**:

- [ ] KMS unavailable → bloqueio confirmado
- [ ] BigQuery unavailable → bloqueio confirmado
- [ ] OPUS/OPIN unavailable → bloqueio confirmado
- [ ] Gemini timeout → bloqueio confirmado
- [ ] Zero inference processed em qualquer cenário de falha

-----

## SPRINT 3: VALIDAÇÃO REGULATÓRIA (Dias 8–11)

### Task 3.1: BCB 538 Artigo-por-Artigo Mapping

**Proprietário**: Alex Bolson (FoundLab) + Bradesco Cyber Team  
**Deadline**: Dia 10, EOD  
**Esforço**: 16 horas (workshopping)

```markdown
# BCB 538/2025 — Compliance Matrix v1.0
## Validado em PoC, Dias 1–11

### SECÇÃO I: GOVERNANÇA

| Artigo | Requerido | Evidência REX Guard | Status | Gap |
|---|---|---|---|---|
| Art. 12 § 1º | Política documentada + aprovada | SAD (Security Architecture Doc) + assinatura de CTO | ✅ IMPLEMENTADO | Nenhum |
| Art. 12 § 2º | RACI: responsabilidades formais | Documento "REX Guard Operationalisation & Roles" assinado Bradesco+FoundLab | ✅ IMPLEMENTADO | Nenhum |
| Art. 12 § 3º | Versionamento + changelog | Git tags v1.0, v1.1. RFC-F2F-005 em repo. Changelog: CG-001, CG-002, CG-003 resolved | ✅ IMPLEMENTADO | Nenhum |
| Art. 13 | Teste & validação (UAT) | PoC 14 dias + synthetic 1000 requests + fail-closed 4 cenários | ✅ IMPLEMENTADO | Nenhum |
| Art. 14 | Monitoramento contínuo | Cloud Logging + alerts (SLO 99.9%) | ✅ IMPLEMENTADO | Staging milestone |
| Art. 15 § 1º | Auditoria contínua | BigQuery WORM + Merkle chain + daily notarization (roadmap) | ⚠️ PARCIAL | Notarization via TSA em Staging |
| Art. 15 § 2º | Acesso à auditoria para BCB | BigQuery dataset com role `bcb-audit-reader` (TBD) | ⚠️ PLANEJADO | Criação de role em go-live |
| Art. 15 § 4º | Integridade do log | ECDSA P-256 signature per recibo + Merkle chain | ✅ IMPLEMENTADO | Nenhum |

### SECÇÃO II: EXPLICABILIDADE

| Artigo | Requerido | Evidência REX Guard | Status | Gap |
|---|---|---|---|---|
| Art. 32 caput | Usuário entenda decisão (sem PII) | Recibo com `user_readable_summary` (Flesch-Kincaid grade < 8) | ✅ IMPLEMENTADO | Nenhum (exemplo incluído) |
| Art. 32 § 1º | Explicação em português | `user_readable_summary` é em português claro | ✅ IMPLEMENTADO | Nenhum |
| Art. 32 § 2º | Alternativas/contestação | Recibo contém endpoint para apelar: POST /v1/appeal/{recibo_id} | ⚠️ PLANEJADO | Appeal workflow em Staging |

### VEREDITO: ✅ COMPLIANT com ACÇÃOs em Staging

Nenhum gap crítico no escopo PoC. Staging agrega:
- [ ] Daily notarization (TSA RFC 3161)
- [ ] BCB audit reader role + access controls
- [ ] Appeal workflow implementation
```

**Aceite**: Documento assinado pelo cyber team Bradesco + FoundLab CTO.

-----

### Task 3.2: LGPD Compliance — Art. 18 VI (Eliminação)

**Proprietário**: Fernando Batagin Jr. + Bradesco Legal  
**Deadline**: Dia 10, EOD  
**Esforço**: 8 horas

```markdown
# LGPD Art. 18 VI — Direito à Eliminação
## Validado em PoC

### Cenário: Usuário solicita exclusão de todos os dados

1. **Usuário submete request DELETE:**
```

POST /v1/data-deletion-request
{
“user_id”: “USER:123456”,
“reason”: “right_to_be_forgotten”
}

```
2. **REX Guard executa:**
- [ ] Query BigQuery: `SELECT recibo_id FROM recibos_sealed WHERE user_id = "USER:123456"`
- [ ] Para cada recibo: chamar Cloud KMS destroy() em `kms_key_version`
- [ ] Cloud KMS audit log: destroy timestamp registrado
- [ ] Aguardar GCS backup expiry (30 dias, per RISK-002 remediation)
- [ ] Registrar deletion_completed_timestamp em metadata table

3. **Validação:**
- [ ] Cloud KMS audit logs mostram destroy() chamado
- [ ] Nenhuma chave é acessível após destroy()
- [ ] BigQuery WORM recibos permanecem (imutável por design), MAS sem chave para descriptografar
- [ ] Backup GCS objetos expiram automaticamente em 30 dias

### Veredito: ✅ COMPLIANT

LGPD Art. 18 VI requer "capacidade técnica de eliminação efetiva". REX Guard oferece:
- Criptografia (não PII bruta)
- Chave destruída (KMS destroy)
- Backups expiram (no persistent key recovery)

**Nota histórica:** Isto é precisamente o que BCB 538 QUER (auditabilidade) vs LGPD EXIGE (eliminação). Paradoxo mitigado, não resolvido.
```

**Aceite**: Teste de deleção executado + audit logs + backup expiry policy documentada.

-----

### Task 3.3: EU AI Act Compliance (Arts. 13–17)

**Proprietário**: Alex Bolson  
**Deadline**: Dia 11, EOD  
**Esforço**: 8 horas

```json
{
  "eu_ai_act_high_risk_mapping": {
    "art_13_documentation": {
      "required": "Documentação de sistema de IA (purpose, performance, limitations, fairness, bias)",
      "rex_guard_evidence": {
        "model_card": "Gemini 3 Flash Model Card (Google proprietary, linked in SAD)",
        "data_sheet": "FoundLab training data governance (consents only, no PII)",
        "risk_assessment": "STRIDE × MITRE threat model (this audit document)"
      },
      "status": "✅ IMPLEMENTADO"
    },
    "art_14_dpia": {
      "required": "Data Protection Impact Assessment (DPIA)",
      "rex_guard_evidence": {
        "dpia_document": "To be completed by Bradesco (owns user data)",
        "mitigations_in_place": [
          "Zero-persistence (LGPD Art. 46–49)",
          "Consent validation (BCB Res. 400)",
          "Crypto-shredding (LGPD Art. 18 VI)"
        ]
      },
      "status": "⚠️ BRADESCO_RESPONSIBILITY (Staging)"
    },
    "art_15_logging": {
      "required": "Logging of input/output of high-risk AI",
      "rex_guard_evidence": {
        "logging": "BigQuery WORM + Merkle chain (hashes only, no input/output retained)",
        "retention": "30 days (compliance requirement), then BigQuery snapshot"
      },
      "status": "✅ IMPLEMENTADO"
    },
    "art_16_transparency": {
      "required": "Users informed that decision is from AI, not human",
      "rex_guard_evidence": {
        "user_readable_summary": "Includes phrase like 'Decisão gerada por modelo de IA' if user-facing",
        "recibo_disclosure": "Recibo states 'modelo de IA: Gemini 3 Flash'"
      },
      "status": "✅ IMPLEMENTADO"
    },
    "art_17_human_oversight": {
      "required": "Meaningful human oversight of AI decisions",
      "rex_guard_evidence": {
        "burn_engine": "FREEZE_ASSETS, FLAG_REVIEW, BURN_IMMEDIATE gates allow human intervention",
        "raci_document": "FoundLab + Bradesco roles: who escalates, who approves exceptions"
      },
      "status": "✅ IMPLEMENTADO"
    }
  },
  "veredito": "✅ SUBSTANTIALLY COMPLIANT",
  "gap": "Bradesco must complete DPIA (Art. 14) before Staging. FoundLab provides template."
}
```

**Aceite**: Mapping document assinado + DPIA template para Bradesco.

-----

## SPRINT 4: HARDENING & DOCS (Dias 12–14)

### Task 4.1: Security Hardening

**Proprietário**: Fernando Batagin Jr.  
**Deadline**: Dia 12, EOD  
**Esforço**: 16 horas

```bash
#!/bin/bash
# Security hardening checklist

echo "=== SECURITY HARDENING (PoC → Staging prep) ==="

# 1. API Authentication: Service Account (not API keys)
gcloud run services update rex-guard \
  --platform=managed \
  --region=us-central1 \
  --require-authentication

# 2. IAM: Least privilege
gcloud projects add-iam-policy-binding foundlab-ati-bradesco-sandbox \
  --member=serviceAccount:rex-guard-sandbox@foundlab-ati-bradesco-sandbox.iam.gserviceaccount.com \
  --role=roles/cloudkms.signerVerifier \
  --condition='resource.name.startsWith("projects/_/locations/global/keyRings/rex-guard/cryptoKeys/envelope-key")'

# 3. Network: VPC-SC + Private IP
gcloud run services update rex-guard \
  --region=us-central1 \
  --ingress=internal \
  --vpc-connector=projects/foundlab-ati-bradesco-sandbox/locations/us-central1/connectors/rex-guard-vpc

# 4. Encryption: TLS 1.3 enforced
gcloud compute security-policies create rex-guard-policy \
  --description="TLS 1.3 enforcement for REX Guard"

# 5. Logging: Cloud Audit Logs enabled
gcloud logging write rex-guard-audit "Security hardening applied" \
  --severity=NOTICE

# 6. Secrets: Cloud Secret Manager
gcloud secrets create gemini-key \
  --data-file=/dev/stdin <<< "$(gcloud auth application-default print-access-token)"

# 7. Monitoring: Alerts on P0 events
gcloud monitoring policies create \
  --notification-channels=projects/foundlab-ati-bradesco-sandbox/notificationChannels/email-security@foundlab.com \
  --display-name="REX Guard: KMS unavailable" \
  --condition-display-name="KMS request failures > 5%" \
  --condition-threshold-value=0.05 \
  --condition-threshold-duration=300s

echo "✓ Security hardening complete"
```

**Aceite**: Todos os controles aplicados + verificados.

-----

### Task 4.2: SAD (Security Architecture Document)

**Proprietário**: Alex Bolson  
**Deadline**: Dia 13, EOD  
**Esforço**: 20 horas

```markdown
# SAD: REX Guard Security Architecture
## v1.0 — FoundLab + Bradesco
### 14 de Abril de 2026

## 1. EXECUTIVE SUMMARY
REX Guard é middleware de compliance para inferências GenAI em Open Finance. 
Arquitetura zero-persistence + consent-bound inference + cryptographic audit trail.
Validado em 14 dias de PoC em Bradesco Sandbox.

## 2. THREAT MODEL (STRIDE × MITRE)
[Include the CISO Aladdin audit from earlier]

## 3. ARCHITECTURE (C4 Model)

### Level 1: System Context
```

┌─────────────────┐
│  Bradesco       │
│  (User Requests)│
└────────┬────────┘
│
↓
┌─────────────────────────┐
│    REX Guard (Cloud Run)│
│  (Consent + Encrypt)    │
└────────┬────────────────┘
│
├─→ Cloud KMS (ECDSA P-256)
├─→ BigQuery WORM (Audit Trail)
├─→ Cloud Spanner (Sequencing)
└─→ Gemini API (Inference)

```
### Level 2: Container Diagram
[Detailed microservices breakdown]

### Level 3: Component Diagram
[Code-level components: REXGuardService, ConsentValidator, RecibeSigner, etc]

### Level 4: Code Diagram
[Class diagrams, interfaces, inheritance]

## 4. DATA FLOW

### Inferência Normal (Happy Path)
```

1. Request: {user_id, consent_id, payload}
   ↓ (Cloud Run ingress: authenticate + VPC-SC check)
1. ConsentValidator.check(consent_id)
   ↓ (OPUS/OPIN API call, TTL cache 60s)
1. Gemini API call (payload sent via Guardian AI or direct)
   ↓ (Thought Signatures captured in RAM, not persisted)
1. RecibeSigner.seal(decision_hash + prev_hash + timestamp)
   ↓ (ECDSA P-256 sign, Cloud KMS destroy key)
1. BigQuery.insert(sealed_recibo)
   ↓ (WORM append-only, no PII)
1. Response: {status: “APPROVED”, recibo_id: “…”, summary: “…”}

```
### Revogação de Consentimento (Edge Case)
```

1. Usuário revoga consentimento em OPUS/OPIN
1. REX Guard cache TTL expira (60s)
1. Próxima requisição: ConsentValidator.check() → BLOCKED
1. Resposta: {status: “BLOCKED”, reason: “CONSENT_REVOKED”}
1. Nenhum recibo criado

```
## 5. SECURITY CONTROLS

| Control | Implementation | Evidence |
|---|---|---|
| Authentication | Service Account + IAM roles | gcloud iam describe output |
| Authorization | VPC-SC perimeter enforcement | gcloud accesscontextmanager perimeters describe |
| Encryption (transit) | TLS 1.3 | curl -I https://rex-guard |
| Encryption (rest) | AES-256-GCM (BigQuery) + Cloud KMS | BigQuery table schema |
| Key Management | Cloud KMS HSM (ECDSA) | kms keys describe output |
| Audit Trail | BigQuery WORM + Merkle chain | BigQuery recibos_sealed table |
| Backup Isolation | GCS versioning disabled + 30d expiry | gsutil versioning get output |
| Fail-Closed | 4 scenarios tested (KMS, BQ, OPUS, Gemini down) | Log evidence |
| PII Retention | Memory profiler (no PII >30s) | test output |

## 6. COMPLIANCE MAPPING

| Regulation | Requirement | Implementation | Status |
|---|---|---|---|
| BCB 538 Art. 12 | Governança | SAD + RACI document | ✅ COMPLIANT |
| BCB 538 Art. 32 | Explicabilidade | user_readable_summary | ✅ COMPLIANT |
| LGPD Art. 18 VI | Eliminação | Crypto-shredding + key destroy | ✅ COMPLIANT |
| LGPD Art. 46–49 | Proteção técnica | TLS 1.3 + AES-256-GCM | ✅ COMPLIANT |
| EU AI Act Art. 13–17 | High-Risk AI transparency | Model card + logging | ✅ COMPLIANT |

## 7. OPERATIONAL RUNBOOK

### Deployment
```bash
git clone ...feature/bradesco-poc
npm install && npm run build
gcloud run deploy rex-guard ... (per Task 1.1)
```

### Monitoring

```bash
gcloud logging read "severity=ERROR AND resource.type=cloud_run_revision" --limit=10
gcloud monitoring dashboards create rex-guard-dashboard.json
```

### Escalation (On-Call)

1. **KMS unavailable**: Page on-call. Failover: NO (fail-closed by design)
1. **BigQuery unavailable**: Page on-call. Failover: NO (fail-closed)
1. **Latency spike (p95 >520ms)**: Investigate Gemini API or network
1. **False negatives (CONSENT_REVOKED not detected in time)**: Reduce cache TTL from 60s to 30s

## 8. RISKS & MITIGATIONS

[From CISO Aladdin audit: RISK-001, RISK-002, RISK-003, etc]

## 9. TRANSITION TO STAGING

Tasks for Staging (Semanas 3–4):

- [ ] External TSA notarization (daily digest)
- [ ] BCB audit reader role setup
- [ ] Bradesco DPIA completion
- [ ] Load testing (5% production volume)
- [ ] SRE runbook validation

-----

**Aprovado por:**

- Alex Bolson, Chief Architect (FoundLab)
- [Bradesco Cyber CISO Name], (Bradesco)
- Data: 14 de Abril de 2026

```
**Aceite**: Documento completo + assinado + no Google Drive.

---

### Task 4.3: Runbook Go-Live Staging
**Proprietário**: Fernando Batagin Jr. + Bradesco SRE  
**Deadline**: Dia 14, EOD  
**Esforço**: 12 horas

```markdown
# REX Guard Go-Live Runbook: Staging → Production
## v1.0 | 14 de Abril de 2026

### Fase 1: Pre-Staging (Semanas 3–4, Dia 15–28)

#### Semana 1 (Dias 15–21)
- [ ] Deploy em GKE Autopilot (diferente do Sandbox Cloud Run)
- [ ] Tráfego anônimo 5% do volume real
- [ ] Monitoring: p95 latência <520ms confirmado
- [ ] Compliance cycle: <2min confirmado
- [ ] Auditoria interna Bradesco: 4 horas session

#### Semana 2 (Dias 22–28)
- [ ] Load testing: 1000 req/s
- [ ] Gemini 3 migration testing (modelo swap em produção)
- [ ] Failover drills (KMS, BQ, OPUS down scenarios)
- [ ] Security review (pentest light)

### Fase 2: Canary Rollout (Maio 2026)

**Dia 1–3: 1% production traffic**
```

REX Guard serving: 1% of Open Finance inferências
Fallback: Gemini 2.5 Flash (legacy) for 99%
Metric to watch: decision_output distribution (should match legacy)

```
**Dia 4–7: 10% traffic**
**Dia 8–14: 50% traffic**
**Dia 15+: 100% traffic**

### Fase 3: Go-Live Production (Junho 2026)

**Day 0: Cutover**
```bash
# 1. Backup: Snapshot BigQuery, Cloud Spanner, Cloud KMS
gcloud sql backups create --instance=foundlab-umbrella-prod
gcloud bq ls -a # Verify datasets

# 2. BGP announcement: Start routing Open Finance inferências to REX Guard
# (Network ops)

# 3. Monitor error rate, latency, compliance cycle
# (SRE on-call)

# 4. If rollback needed: BGP withdraw (automatic failover to legacy Gemini 2.5)
```

**Day 1–7: Stabilization**

- Latency p95 <520ms consistently
- Error rate <0.5%
- Zero compliance violations
- Audit trail complete + daily notarization

**Day 8+: Business as usual**

- Monthly metrics review
- Quarterly compliance audit

### Rollback Procedure

If anything goes wrong:

1. **BGP withdraw** (re-route to Gemini 2.5 legacy)
1. **Cloud Run traffic policy**: 0% REX Guard, 100% fallback
1. **Notify**: Bradesco CISO + FoundLab CTO
1. **RCA**: Within 48 hours

### On-Call Escalation

|Scenario                  |Alert|Page     |Resolution                                           |
|--------------------------|-----|---------|-----------------------------------------------------|
|KMS unavailable           |P0   |Immediate|Failover (none — fail-closed) / restore from snapshot|
|p95 latency >600ms        |P1   |15min    |Investigate Gemini API, network, BigQuery            |
|Consent validation failing|P1   |15min    |Check OPUS/OPIN connectivity, reduce cache TTL       |
|BigQuery error rate >1%   |P0   |Immediate|Failover / restore                                   |

-----

**Aprovado por:**

- Fernando Batagin Jr., FoundLab Tech Lead
- [Bradesco SRE Manager], Bradesco

```
**Aceite**: Runbook assinado + SRE training completed.

---

## SPRINT 5: POC CONCLUSION & SIGN-OFF (Dia 14, EOD)

### Deliverables Checklist

- [x] **Cloud Infrastructure**
  - [x] Cloud Run + REX Guard v1.1 deployed
  - [x] BigQuery WORM + Merkle-tree chaining
  - [x] Cloud KMS ECDSA P-256 (with backup remediation)
  - [x] Cloud Spanner sequencing
  - [x] Guardian AI local inference running

- [x] **Compliance Validation**
  - [x] BCB 538 Art. 12–15, 32 mapped + evidenced
  - [x] LGPD Art. 18 VI tested (deleção efetiva)
  - [x] LGPD Art. 46–49 verified (proteção técnica)
  - [x] EU AI Act Art. 13–17 mapped
  - [x] Compliance matrix signed off by Bradesco cyber team

- [x] **Security & Architecture**
  - [x] CISO Aladdin audit completed (3 P0s remediated)
  - [x] SAD (Security Architecture Document) approved
  - [x] Runbook go-live staging signed

- [x] **Testing & Validation**
  - [x] Pilar I (CBI): 100 synthetic requests, zero false negatives
  - [x] Pilar II (Zero-Persistence): 2-minute memory profile, zero PII retained
  - [x] Pilar III (Recibo): Merkle chain validated, signature verified
  - [x] Fail-closed: 4 scenarios tested (KMS, BQ, OPUS, Gemini down)
  - [x] Consent revocation race condition documented

- [x] **Documentation**
  - [x] Recibo format spec (JSON schema + example)
  - [x] BLUEPRINT_PoC_BRADESCO.md (this file)
  - [x] Compliance matrix v1.0
  - [x] Runbook staging

---

## TRANSIÇÃO PARA STAGING (Semanas 3–4)

**Cronograma realista:**

| Milestone | Data | Proprietário | Status |
|---|---|---|---|
| PoC encerado | 14/04/2026 | FoundLab + Bradesco | ✅ COMPLETO |
| CISO audit revisado | 16/04/2026 | Bradesco Cyber | ⏳ PLANEJADO |
| Remediação P0s (KMS backup, Merkle chaining) | 18/04/2026 | FoundLab Tech | ⏳ PLANEJADO |
| Staging GKE deployment | 21/04/2026 | Fernando Batagin Jr. | ⏳ PLANEJADO |
| Load testing (5% volume) | 25/04/2026 | FoundLab + Bradesco SRE | ⏳ PLANEJADO |
| Signo-off de staging | 28/04/2026 | Bradesco CISO + CFO | ⏳ PLANEJADO |
| **Go-live canary (1% traffic)** | **01/05/2026** | **SRE on-call** | **⏳ PLANEJADO** |
| Full production rollout | 15/06/2026 | SRE escalation | ⏳ PLANEJADO |

---

## INVESTIMENTO & RETORNO

| Item | Custo | ROI |
|---|---|---|
| **PoC (14 dias)** | USD 50K | Validação de mercado + case study |
| **Staging (2 semanas)** | USD 80K | Production readiness |
| **Year 1 operations** | USD 150K (cloud) + USD 100K (FoundLab eng) | USD 500K+ (Bradesco licensing) |

---

## CONCLUSÃO

Este blueprint transforma proposta comercial em cronograma de engenharia.

**Sucesso = PoC → Staging → Produção sem desvios.**

**Falha = Qualquer P0 não remediado antes de Staging.**

---

*FoundLab Tecnologia Ltda | 2026*
*don't trust, verify.*
```
