# Diagramas de Arquitectura - Sistema VisApp

Este directorio contiene los diagramas de arquitectura para el sistema de visación de residencia propuesto, implementado como una solución serverless en AWS con OCR, validación por reglas y cumplimiento normativo.

## Diagramas Disponibles

### 1. [Vista General - Alto Nivel](01-vista-general-alto-nivel.md)
- **Propósito**: Vista general del sistema completo
- **Tipo**: Flowchart (flujo de componentes)
- **Enfoque**: Arquitectura general con todos los servicios AWS

### 2. [Flujo de Datos - Secuencia](02-flujo-datos-secuencia.md)
- **Propósito**: Flujo detallado del proceso de tramitación
- **Tipo**: Sequence Diagram (diagrama de secuencia)
- **Enfoque**: Interacciones entre componentes durante el procesamiento

### 3. [Modelo de Datos - ER](03-modelo-datos-er.md)
- **Propósito**: Modelo entidad-relación de la base de datos
- **Tipo**: ER Diagram (diagrama entidad-relación)
- **Enfoque**: Estructura de datos y relaciones entre entidades

### 4. [Despliegue Lógico por Capas](04-despliegue-logico-capas.md)
- **Propósito**: Vista de contenedores organizada por capas
- **Tipo**: Flowchart con subgraphs y estilos
- **Enfoque**: Organización lógica de componentes por responsabilidad

## Cómo Usar los Diagramas

### Visualización
Los diagramas están escritos en **Mermaid** y pueden ser visualizados en:
- GitHub (renderizado automático)
- GitLab
- Confluence
- Herramientas online como [Mermaid Live Editor](https://mermaid.live/)

### Exportación
Para exportar a PNG/SVG:
1. Copiar el código Mermaid
2. Pegar en [Mermaid Live Editor](https://mermaid.live/)
3. Descargar en formato PNG o SVG

### Integración en Documentación
Los diagramas pueden integrarse en:
- Documentación técnica
- Presentaciones
- Tesis académica
- Documentación de arquitectura

## Características del Sistema

### Arquitectura Serverless
- **Frontend**: React SPA en S3 + CloudFront
- **Backend**: API Gateway + Lambda Functions
- **Base de Datos**: RDS PostgreSQL Multi-AZ
- **OCR**: Amazon Textract
- **Cola de Procesamiento**: SQS + Step Functions

### Seguridad y Cumplimiento
- **Autenticación**: Cognito con ClaveÚnica OIDC
- **Autorización**: RBAC por roles
- **Cifrado**: KMS CMK para datos en reposo
- **Auditoría**: CloudTrail + CloudWatch
- **Cumplimiento**: ARCO, retención de datos

### Observabilidad
- **Monitoreo**: CloudWatch Logs/Métricas/Alarmas
- **Tracing**: AWS X-Ray
- **Backup**: AWS Backup
- **Configuración**: AWS Config

## Notas Técnicas

- Los diagramas están optimizados para legibilidad
- Se incluyen comentarios explicativos en cada diagrama
- Los estilos de colores ayudan a distinguir las diferentes capas
- La documentación incluye consideraciones de costos y escalabilidad

## Próximos Pasos

1. Revisar y validar los diagramas con el equipo técnico
2. Generar versiones PNG/SVG para documentación
3. Integrar en la documentación de la tesis
4. Actualizar según feedback y cambios en requisitos 