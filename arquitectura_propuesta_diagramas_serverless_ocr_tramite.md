# Arquitectura Propuesta — Diagramas

> Sistema serverless en AWS con OCR, validación por reglas y cumplimiento normativo.

---

## 1) Vista general (alto nivel)

```mermaid
flowchart LR
  subgraph Usuario
    U[Usuario ciudadano]
    F[Funcionario]
  end

  U -->|HTTPS| CF[(Amazon CloudFront)]
  CF --- S3FE[(Amazon S3 - SPA)]
  CF -->|OIDC| COG[Cognito ClaveÚnica]

  CF -->|REST| WAF[WAF + Shield]
  WAF --> APIG[API Gateway]

  subgraph Orquestación
    APIG --> SFN[Step Functions]
    APIG --> L0[Lambda create-case]
    L0 --> RDS[(RDS PostgreSQL)]
    L0 --> SQS[SQS Queue]
    SQS --> L1[Lambda preprocess]
    L1 --> S3DOC[(S3 Documentos)]
    L1 --> L2[Lambda textract]
    L2 --> TEX[Amazon Textract]
    L2 --> S3JSON[(S3 JSON OCR)]
    L2 --> RDS
    L2 --> L3[Lambda normalize]
    L3 --> RDS
    L3 --> L4[Lambda rules]
    L4 --> RDS
    L4 --> SNS[SNS/SES]
  end

  subgraph Datos
    DDB[(DynamoDB Audit)]
    GL[Glacier Logs]
    S3DOC -->|Lifecycle| GL
  end

  subgraph Observabilidad
    CW[CloudWatch]
    XR[X-Ray]
    CT[CloudTrail]
    KMS[KMS CMK]
    SM[Secrets Manager]
    CFG[AWS Config]
    BCK[AWS Backup]
  end

  APIG --> CW
  L0 --> CW
  L1 --> CW
  L2 --> CW
  L3 --> CW
  L4 --> CW
  CW --- XR
  APIG -.auditoría.-> CT
  RDS -.backups.-> BCK
  S3DOC -.KMS.-> KMS
  RDS -.KMS.-> KMS
  DDB -.KMS.-> KMS
  TEX -.KMS.-> KMS
  APIG -.credenciales.-> SM

  F -->|RBAC| CF
  CF --> APIG
```

---

## 2) Flujo de datos (secuencia 7.2)

```mermaid
sequenceDiagram
  autonumber
  participant User as Usuario SPA
  participant CF as CloudFront/S3
  participant COG as Cognito OIDC
  participant APIG as API Gateway
  participant L0 as Lambda create-case
  participant S3 as S3 docs
  participant SQS as SQS
  participant L1 as Lambda preprocess
  participant L2 as Lambda textract
  participant TEX as Amazon Textract
  participant L3 as Lambda normalize
  participant L4 as Lambda rules
  participant RDS as RDS PostgreSQL
  participant SNS as SNS/SES

  User->>CF: (1) Acceso SPA
  CF->>COG: OIDC Login ClaveÚnica
  COG-->>User: Token OIDC
  User->>S3: (2) Upload directo presigned URL
  User->>APIG: (3) create-case token
  APIG->>L0: Invoke
  L0->>RDS: Insert expediente
  L0->>SQS: Enqueue mensaje
  SQS-->>L1: (4) Trigger preprocess
  L1->>S3: Guarda procesado
  L1->>L2: Invoca OCR
  L2->>TEX: (5) Start/Analyze
  TEX-->>L2: Resultados OCR
  L2->>S3: Guarda JSON OCR
  L2->>RDS: Metadatos extracción
  L2->>L3: (6) Normalización
  L3->>RDS: Upsert campos normalizados
  L3->>L4: (7) Reglas negocio
  L4->>RDS: Update estado/validación
  L4->>SNS: (8) Notificación aprob/rechazo
  SNS-->>User: Email/SMS
  User->>APIG: (9) Panel funcionario si aplica
  APIG->>RDS: Consultas
  APIG-->>User: Datos expediente
  Note over User,APIG: (10) Cierre: acta PDF en S3 y hash en RDS
```

---

## 3) Modelo de datos mínimo viable (ER)

```mermaid
erDiagram
  CASE ||--o{ DOCUMENT : contiene
  DOCUMENT ||--o{ EXTRACTION : produce
  EXTRACTION ||--o{ FIELDS : tiene
  CASE ||--o{ VALIDATION : evalua

  AUDIT {
    uuid id
    string actor
    string accion
    string recurso
    json before
    json after
    string ip
    timestamp ts
  }

  CASE {
    uuid id
    uuid user_id
    string tipo_tramite
    string estado
    timestamp created_at
    timestamp updated_at
  }

  DOCUMENT {
    uuid id
    uuid case_id
    string tipo
    string s3_key_original
    string s3_key_processed
    string hash_sha256
    int pages
    timestamp uploaded_at
  }

  EXTRACTION {
    uuid id
    uuid document_id
    string ocr_provider
    string s3_key_json
    float quality_score
    timestamp extracted_at
  }

  FIELDS {
    uuid id
    uuid extraction_id
    string nombre_campo
    string valor
    float conf
    int pagina
    json bbox
  }

  VALIDATION {
    uuid id
    uuid case_id
    string regla
    string resultado
    string detalle
    timestamp evaluated_at
  }
```

> Nota: Auditoría de alta frecuencia puede ir en DynamoDB (append-only) con TTL y export a S3 para archivado.

---

## 4) Despliegue lógico por capas (vista de contenedores)

```mermaid
flowchart TB
  subgraph Frontend
    SPA[React SPA en S3]:::fe
    CF[CloudFront]:::fe
    COG[Cognito OIDC]:::sec
  end
  subgraph Perimetro
    WAF[WAF + Shield]:::sec
    APIG[API Gateway REST]:::api
  end
  subgraph Computo
    SFN[Step Functions]:::comp
    LAMBDAS[Lambdas]:::comp
    SQS[SQS + DLQ]:::comp
  end
  subgraph Datos
    S3DOC[S3 Buckets]:::data
    RDS[RDS PostgreSQL]:::data
    DDB[DynamoDB Audit]:::data
  end
  subgraph Observabilidad
    CW[CloudWatch/X-Ray]:::obs
    CT[CloudTrail]:::obs
    CFG[AWS Config]:::gov
    BCK[AWS Backup]:::gov
    KMS[KMS CMK]:::sec
    SM[Secrets Manager]:::sec
  end

  SPA-->CF-->WAF-->APIG
  CF-->COG
  APIG-->SFN
  APIG-->LAMBDAS
  LAMBDAS-->SQS
  LAMBDAS-->S3DOC
  LAMBDAS-->RDS
  LAMBDAS-->DDB
  LAMBDAS-->CW
  APIG-->CT
  S3DOC-->KMS
  RDS-->KMS
  DDB-->KMS
  RDS---BCK
  CFG---APIG

  classDef fe fill:#f4f9ff,stroke:#6aa3ff,stroke-width:1.2
  classDef api fill:#eefaf1,stroke:#58c26a,stroke-width:1.2
  classDef comp fill:#fff8e6,stroke:#ffb200,stroke-width:1.2
  classDef data fill:#fcf0ff,stroke:#b26bff,stroke-width:1.2
  classDef obs fill:#f0f7f7,stroke:#4fb0ae,stroke-width:1.2
  classDef sec fill:#fff0f0,stroke:#ff6b6b,stroke-width:1.2
  classDef gov fill:#f7f7f7,stroke:#9e9e9e,stroke-width:1.2
```

---

## 5) Seguridad y cumplimiento (resumen operable)

- **Cifrado**: TLS 1.2+, S3/RDS/DDB/Textract con CMK administrada; SSE-KMS; políticas de bucket sin acceso público.
- **IAM**: Principio de mínimo privilegio por función Lambda; roles separados para lectura/escritura; condiciones por VPC/SourceArn en APIG.
- **PII**: Minimización, clasificación, etiquetas en S3; retención 18–24 meses; processings con Comprehend (opcional) para redacción.
- **Logs inmutables**: Export CloudTrail/CloudWatch a S3 con Object Lock + Glacier Vault Lock.
- **ARCO**: Endpoints dedicados (consulta/rectificación/oposición/cancelación) auditados; flujo de borrado lógico y físico por retención.
- **DR/Backups**: RPO ≤ 24h / RTO ≤ 8h; pruebas de restore trimestrales; runbooks en Systems Manager Documents.

---

## 6) Calidad y observabilidad

- **KPIs**: tasa extracción, % FAIL por regla, TTP (tiempo de trámite), retrabajos.
- **Alarmas**: `quality_score < umbral`, errores > N/min en una Lambda, DLQ con >0 mensajes, latencia APIG p95.
- **A/B**: variantes de preprocesamiento (deskew/denoise) sobre golden dataset para optimizar precisión OCR.

---

## 7) Costeo referencial (indicativo)

- **Textract**: 0.01–0.015 USD/página (según API).
- **S3**: almacenamiento originales, procesados, JSON OCR + lifecycle a Glacier.
- **Lambda/SQS/SFN**: pago por uso; revisar cargas pico y tamaños de payload.

---

### Notas

- Diagramas en Mermaid: pueden exportarse a PNG/SVG aquí o integrarse en Confluence/Markdown.
- Si se requiere, puedo generar una variante en formato C4 (Context/Container/Component) o PlantUML con íconos oficiales de AWS.

