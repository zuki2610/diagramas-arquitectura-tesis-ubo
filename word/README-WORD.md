# Diagramas Optimizados para Word

Esta carpeta contiene versiones de los diagramas de arquitectura optimizadas específicamente para su uso en documentos de Microsoft Word.

## Características de las Versiones Word

### 🎨 Optimizaciones Visuales
- **Fondo blanco**: Todos los diagramas tienen fondo blanco para mejor integración
- **Bordes más gruesos**: Líneas de 2px para mejor visibilidad en impresión
- **Colores suaves**: Paleta de colores optimizada para documentos académicos
- **Alto contraste**: Texto y elementos bien definidos

### 📊 Diagramas Disponibles

1. **01-vista-general-word.md** - Vista general del sistema
2. **02-flujo-datos-word.md** - Diagrama de secuencia del proceso
3. **03-modelo-datos-word.md** - Modelo entidad-relación
4. **04-despliegue-word.md** - Despliegue por capas

## Cómo Usar en Word

### Método 1: Exportación Directa
1. Copiar el código Mermaid de cada archivo
2. Pegar en [Mermaid Live Editor](https://mermaid.live/)
3. Exportar como PNG con fondo blanco
4. Insertar en Word

### Método 2: Captura de Pantalla
1. Abrir el diagrama en GitHub/GitLab
2. Hacer captura de pantalla
3. Pegar directamente en Word

### Método 3: Plugin Mermaid
- Instalar plugin Mermaid para Word (si está disponible)
- Pegar código directamente en el documento

## Paleta de Colores

- **Frontend**: Azul claro (#e3f2fd)
- **API**: Verde claro (#e8f5e8)
- **Cómputo**: Naranja claro (#fff3e0)
- **Datos**: Púrpura claro (#f3e5f5)
- **Observabilidad**: Verde azulado (#e0f2f1)
- **Seguridad**: Rojo claro (#ffebee)
- **Gobierno**: Gris claro (#fafafa)

## Recomendaciones para Word

1. **Tamaño**: Exportar en alta resolución (300 DPI mínimo)
2. **Formato**: PNG para mejor calidad
3. **Alineación**: Centrar los diagramas en el documento
4. **Referencias**: Incluir numeración de figuras
5. **Leyendas**: Agregar títulos descriptivos

## Ejemplo de Referencia en Tesis

```
Figura 1.1: Arquitectura general del sistema VisApp
La figura muestra la arquitectura serverless propuesta para el sistema de visación de residencia, 
implementada en AWS con servicios de OCR, validación automática y cumplimiento normativo.
``` 