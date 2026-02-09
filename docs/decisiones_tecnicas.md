## Decisiones de configuración y diseño técnico

Esta sección describe las principales decisiones técnicas relacionadas con la **configuración operativa** del sistema, tomadas a partir de pruebas prácticas y del tipo de documentos procesados.

---

### Uso de `crypto` para la generación de `doc_id`

Se utilizó el módulo `crypto` para generar un identificador único (`doc_id`) determinístico a partir de la URL del documento.

**Justificación:**

- Garantiza que una misma URL genere siempre el mismo `doc_id`
- Permite detectar re-ingestas del mismo documento
- Facilita la eliminación, actualización y trazabilidad de los datos vectorizados
- Evita colisiones y duplicados en el vector store

Esta estrategia asegura consistencia entre los flujos de ingesta y consulta.

---

### Uso de un nodo de código para extracción de URL y nombre del archivo

Se implementó un **nodo de código** para:

- Normalizar la URL recibida
- Extraer el nombre del archivo PDF desde la URL
- Centralizar la lógica de generación de metadata

**Justificación:**

- Mayor control sobre el formato de los datos
- Flexibilidad frente a variaciones en la estructura de la URL
- Evita depender de configuraciones implícitas de otros nodos
- Facilita mantenimiento y depuración del flujo

---

### Configuración de chunking y embeddings

Durante las pruebas se evaluaron distintas configuraciones de fragmentación y batch de embeddings.  
La configuración final seleccionada fue:

- **Chunk size:** 1000  
- **Chunk overlap:** 200  
- **Embedding batch size:** 100  

**Justificación:**

- Se adaptó mejor a documentos largos de tipo académico y técnico
- Reduce pérdida de contexto entre fragmentos
- Mantiene un balance adecuado entre precisión semántica y costo computacional
- Ofrece mejores resultados en la recuperación de fragmentos relevantes durante la consulta

Esta configuración fue seleccionada tras pruebas empíricas con diferentes documentos PDF.

---

### Eliminación previa de puntos (points) en el vector store

Antes de insertar nuevos embeddings de un documento, se ejecuta un proceso de **eliminación de los puntos existentes asociados al `doc_id`**.

**Justificación:**

- Evita duplicación de fragmentos en re-ingestas
- Permite actualizar documentos ya procesados
- Mantiene coherencia y limpieza en la colección
- Facilita control del ciclo de vida del documento

Esta decisión es clave para garantizar un vector store consistente y mantenible a largo plazo.



- Precisión en la recuperación
- Trazabilidad de los documentos
- Control sobre re-ingestas
- Simplicidad operativa


## Configuración del chatbot y proceso de recuperación

Esta sección describe las decisiones técnicas asociadas a la configuración del chatbot y al proceso de recuperación de información desde el vector store.

---

### Configuración del modelo de recuperación (retrieval)

Para el proceso de recuperación de contexto desde el vector store se configuraron los siguientes parámetros:

- **Limit:** 20  
- **Content payload key:** `text`  

**Justificación:**

- Un límite de 20 fragmentos permite recuperar suficiente contexto sin introducir ruido excesivo
- Mejora la cobertura de información en documentos extensos
- Ofrece un buen balance entre precisión de respuesta y rendimiento
- El uso de `text` como payload key estandariza el acceso al contenido almacenado en Qdrant

Esta configuración fue validada mediante pruebas prácticas con distintos tipos de preguntas.

---

### Selección del modelo de lenguaje (LLM)

El chatbot utiliza un modelo de lenguaje accesible mediante **OpenRouter**, configurado con el modelo:

- **Modelo:** `OpenAI GPT-4.1 Mini`

**Justificación:**

- Ofrece buena capacidad de razonamiento para tareas de preguntas y respuestas
- Presenta un costo computacional moderado
- Responde de forma consistente cuando se le suministra contexto acotado
- Se adapta correctamente a flujos RAG basados en recuperación previa

La integración mediante OpenRouter permite flexibilidad para cambiar de modelo en el futuro sin modificar la lógica del flujo.
