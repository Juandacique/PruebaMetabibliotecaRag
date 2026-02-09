# 📄 Sistema RAG para Conversación con Documentos PDF

## 1. Descripción general

Un sistema de información basado en la arquitectura **RAG (Retrieval-Augmented Generation)** que permite **ingerir documentos PDF desde URLs públicas** y **conversar con su contenido** mediante un asistente inteligente.

El sistema está diseñado para trabajar con documentos académicos, tesis, artículos científicos y documentos técnicos disponibles en Internet, garantizando que las respuestas generadas se basen exclusivamente en el contenido del documento consultado.

## Enlaces sistemas publicados:
# N8N: https://n8n-prueba4.metadatos.org/
# Front: https://ragmetabiblioteca.lovable.app/


---

## 2. Funcionalidades principales

- 📥 Ingesta de documentos PDF desde enlaces públicos  
- 🧾 Extracción de texto y metadatos del documento  
- 🧩 División del contenido en fragmentos semánticos (chunking)  
- 🧠 Vectorización del contenido mediante embeddings  
- 📦 Almacenamiento en un vector store (Qdrant)  
- 💬 Chat contextual sobre documentos específicos  
- 🔄 Soporte para re-ingesta y actualización de documentos  

---

## 3. Arquitectura del sistema

El sistema está compuesto por los siguientes elementos:

### Componentes

- **Frontend (Lovable)**
  - Interfaz web para:
    - Enviar URLs de documentos PDF
    - Interactuar con el chatbot

- **n8n**
  - Orquestador de flujos de trabajo
  - Procesamiento documental
  - Vectorización y consultas RAG

- **Qdrant**
  - Almacenamiento vectorial
  - Indexación de embeddings y metadata

- **Modelo LLM**
  - Generación de respuestas basadas en contexto recuperado

- **PostgreSQL**
  - Base de datos interna para n8n

> El sistema sigue un enfoque desacoplado y modular, facilitando su escalabilidad y mantenimiento.

---

## 4. Interfaz de usuario

La interfaz web permite dos acciones principales:

### 📥 Enviar PDF
El usuario puede pegar una URL pública de un documento PDF.  
El sistema procesa automáticamente el documento y lo deja disponible para consultas.

### 💬 Chat con documentos
El usuario puede realizar preguntas sobre un documento previamente procesado.  
Las respuestas se generan utilizando únicamente el contenido del documento asociado.

*(Las capturas de pantalla del frontend se encuentran en la carpeta `/frontend`)*

---

## 5. Flujo 1 – Ingesta y vectorización de documentos

### Objetivo
Procesar documentos PDF desde URLs públicas y almacenarlos en el vector store junto con su metadata asociada.

### Descripción del flujo

1. Recepción de la URL del documento mediante Webhook  
2. Normalización de la URL  
3. Generación de un identificador único (`doc_id`)  
4. Descarga del archivo PDF  
5. Extracción de:
   - Texto completo
   - Metadatos disponibles del documento  
6. División del texto en fragmentos (chunks)  
7. Generación de embeddings  
8. Inserción en Qdrant con metadata asociada  

### Metadata almacenada

Cada fragmento se almacena con la siguiente metadata:

```json
{
  "doc_id": "hash_documento",
  "source_url": "https://...",
  "file_name": "documento.pdf",
  "author": "Autor (si existe)",
  "creation_date": "Fecha del documento",
}
```
## 6. Flujo 2 – Chat RAG sobre documentos

### Objetivo

Permitir la consulta inteligente de documentos previamente indexados mediante un enfoque RAG (Retrieval-Augmented Generation).

### Funcionamiento

1. El usuario envía una pregunta junto con la URL del documento  
2. Se genera el mismo `doc_id` utilizado durante la ingesta  
3. Se consulta el vector store filtrando por `doc_id`  
4. Se recuperan los fragmentos más relevantes del documento  
5. El modelo LLM genera la respuesta utilizando únicamente el contexto recuperado  

### Beneficios

- ✔️ Respuestas acotadas al documento consultado  
- ✔️ No hay contaminación de contexto entre documentos  
- ✔️ Escalable a múltiples documentos sin crear colecciones separadas  

---

## 7. Diseño del vector store (Qdrant)

### Estrategia de colección

- Se utiliza **una única colección**
- Los documentos se diferencian mediante el campo `doc_id` dentro de la metadata

### Configuración vectorial

- **Dimensión del embedding:** 1536  
- **Métrica de similitud:** Cosine  

### Campos indexados

- `doc_id`  
- `source_url`  
- `author`  
- `creation_date`  

### Este diseño permite:

- Borrado por documento  
- Reindexación controlada  
- Filtros precisos durante la búsqueda  

---

## 8. Manejo de re-ingesta y actualización

El sistema soporta re-ingesta de documentos de forma segura y controlada:

- Se genera siempre el mismo `doc_id` para una misma URL  
- Antes de insertar nuevos datos, se eliminan los puntos asociados a ese `doc_id`  
- Se evita la duplicación de información en el vector store  

Este enfoque garantiza consistencia, trazabilidad y control del estado de los documentos indexados.

---

## 9. Infraestructura y despliegue

La infraestructura se despliega mediante **Docker Compose** e incluye los siguientes servicios:

- n8n  
- PostgreSQL  
- Qdrant  
- Servicios auxiliares necesarios para el flujo  

## 10. Decisiones técnicas relevantes

- Uso de una sola colección en Qdrant para mejorar escalabilidad

- Identificación determinística de documentos mediante doc_id

- Chunking configurable para balance entre precisión y costo computacional

- Separación clara entre los flujos de ingesta y consulta

- Orquestación completa del sistema mediante flujos en n8n

## 11. Contenido del repositorio
/infra      – Infraestructura y despliegue
/workflows  – Flujos exportados de n8n
/frontend   – Evidencia de la interfaz
/docs       – Documentación técnica adicional
