# 1) Vista General - Alto Nivel

Sistema serverless en AWS con OCR, validación por reglas y cumplimiento normativo.

```mermaid
flowchart LR
  subgraph Usuario
    U[Usuario ciudadano]
    F[Funcionario]
  end

  U -->|HTTPs| CF[(Amazon CloudFront)]
  CF --- S3FE[(Amazon S3 - SPA)]
  CF -->|OIDC| COG[Cognito (ClaveÚnica OIDC)]

  CF -->|REST| WAF[WAF + Shield]
  WAF --> APIG[API Gateway]

  subgraph Orquestación & Cómputo
    APIG --> SFN[Step Functions]
    APIG --> L0[Lambda create-case]
    L0 --> RDS[(RDS PostgreSQL)]
    L0 --> SQS[[SQS Queue]]
    SQS --> L1[Lambda preprocess-image]
    L1 --> S3DOC[(S3 - Documentos)]
    L1 --> L2[Lambda run-textract]
    L2 --> TEX[Amazon Textract]
    L2 --> S3JSON[(S3 - JSON OCR)]
    L2 --> RDS
    L2 --> L3[Lambda normalize-parse]
    L3 --> RDS
    L3 --> L4[Lambda rules-engine]
    L4 --> RDS
    L4 --> SNS[Amazon SNS/SES]
  end

  subgraph Datos & Auditoría
    DDB[(DynamoDB Auditoría opcional)]
    GL[Glacier - Logs inmutables]
    S3DOC -->|Lifecycle| GL
  end

  subgraph Observabilidad & Seguridad
    CW[CloudWatch Logs/Métricas/Alarmas]
    XR[AWS X-Ray]
    CT[CloudTrail]
    KMS[KMS CMK]
    SM[Secrets Manager]
    CFG[AWS Config]
    BCK[AWS Backup]
  end

  APIG -->|Invocación| CW
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

  F -->|RBAC Cognito| CF
  CF --> APIG
``` 