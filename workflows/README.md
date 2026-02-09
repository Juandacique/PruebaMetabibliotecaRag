# Workflows 

Esta carpeta contiene los flujos de trabajo exportados desde **n8n**, los cuales implementan la lógica principal del sistema **DocFlow** para la ingesta documental y la consulta RAG.

Cada flujo está diseñado para cumplir una función específica dentro del sistema y puede ejecutarse de forma independiente.

---

## Flujo 1 – Ingesta y vectorización de documentos PDF

### Archivo
flujo_rag.json
### Objetivo
Recibir una URL pública de un documento PDF, procesar su contenido y almacenar los fragmentos vectorizados junto con su metadata en el vector store (Qdrant).

---

### Descripción general del flujo

Este flujo se encarga de transformar un documento PDF externo en información estructurada y vectorizada, lista para ser consultada posteriormente por el chatbot.

---

### Pasos principales

1. **Recepción de la URL**
   - El flujo se activa mediante un **Webhook**
   - Recibe en el body la URL del documento PDF

2. **Procesamiento inicial (Code Node)**
   - Normaliza la URL
   - Extrae el nombre del archivo
   - Genera un `doc_id` determinístico usando `crypto`

3. **Descarga del documento**
   - Descarga el archivo PDF desde la URL proporcionada

4. **Extracción de contenido**
   - Extrae el texto completo del PDF
   - Extrae metadata disponible (autor, fecha, etc.)

5. **Eliminación previa de datos**
   - Se eliminan los puntos existentes en Qdrant asociados al `doc_id`
   - Evita duplicación y permite re-ingesta

6. **Chunking del contenido**
   - Fragmentación del texto en bloques semánticos
   - Configuración utilizada:
     - Chunk size: 1000
     - Overlap: 200

7. **Vectorización**
   - Generación de embeddings
   - Embedding batch size: 100

8. **Almacenamiento en Qdrant**
   - Inserción de los embeddings junto con su metadata
   - Uso de una única colección con diferenciación por `doc_id`

---

### Metadata almacenada

Cada fragmento se almacena con metadata asociada, por ejemplo:

```json
{
  "doc_id": "hash_documento",
  "source_url": "https://...",
  "file_name": "documento.pdf",
  "author": "Autor (si existe)",
  "creation_date": "Fecha del documento"
}
```
## Flujo 2 – Chat RAG sobre documentos
### Archivo
chatbot.json

### Objetivo

Permitir la consulta inteligente de documentos previamente procesados y almacenados en el vector store.

---

### Descripción general del flujo

Este flujo implementa la lógica de recuperación de contexto y generación de respuestas utilizando un modelo de lenguaje, asegurando que las respuestas estén acotadas al documento consultado.

---

### Pasos principales

#### 1. Recepción de la consulta

- El flujo se activa mediante un **Webhook**
- Recibe:
  - Pregunta del usuario
  - URL del documento

---

#### 2. Generación del `doc_id`

- Se genera el mismo identificador utilizado durante la ingesta
- Permite filtrar correctamente el vector store y asegurar coherencia entre flujos

---

#### 3. Consulta al vector store

- Se consulta Qdrant filtrando por `doc_id`
- Configuración de recuperación utilizada:
  - **Límite de fragmentos:** 20
  - **Content payload key:** `text`

---

#### 4. Recuperación de contexto

- Se obtienen los fragmentos más relevantes del documento
- El contexto recuperado se prepara para su envío al modelo de lenguaje

---

#### 5. Generación de respuesta

- Se envía el contexto al modelo LLM
- El modelo genera la respuesta basada únicamente en el contenido recuperado

---

### Modelo de lenguaje utilizado

- **Proveedor:** OpenRouter  
- **Modelo:** OpenAI GPT-4.1 Mini  

La selección del modelo se basa en su equilibrio entre capacidad de razonamiento, estabilidad y costo computacional.

---

### Consideraciones generales sobre los workflows

- Los flujos están desacoplados por responsabilidad
- Permiten ejecución independiente
- Facilitan la depuración y el mantenimiento
- Son fácilmente extensibles a nuevos tipos de documentos o modelos

---

### Notas adicionales

- Los flujos se exportan desde n8n en formato `.json`
- Pueden importarse directamente en otra instancia de n8n
- Las variables sensibles se gestionan mediante variables de entorno
