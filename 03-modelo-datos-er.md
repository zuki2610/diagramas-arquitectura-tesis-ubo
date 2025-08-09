# 3) Modelo de Datos - ER Mínimo Viable

Modelo entidad-relación para el sistema de visación de residencia.

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

> **Nota**: Auditoría de alta frecuencia puede ir en DynamoDB (append-only) con TTL y export a S3 para archivado. 