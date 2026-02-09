# Infraestructura 

Esta carpeta contiene los archivos necesarios para el despliegue de la infraestructura del sistema **DocFlow**, utilizada para la ingesta, vectorización y consulta de documentos PDF mediante un enfoque RAG.

La infraestructura se despliega utilizando **Docker Compose**, permitiendo una instalación reproducible y desacoplada del entorno.

---

## Servicios incluidos

El archivo `docker-compose.yml` define los siguientes servicios:

### 🧠 n8n
- Orquestador principal de los flujos de trabajo
- Gestiona:
  - Ingesta de documentos PDF
  - Extracción de texto y metadatos
  - Vectorización
  - Consultas RAG
- Expuesto mediante HTTPS usando Traefik

### 🐘 PostgreSQL
- Base de datos utilizada internamente por n8n
- Almacena:
  - Configuración
  - Estados de ejecución
  - Historial de workflows

### 📦 Qdrant
- Vector store utilizado para almacenar:
  - Embeddings de los fragmentos de documentos
  - Metadata asociada (`doc_id`, URL, autor, fecha, etc.)
- Permite búsquedas semánticas y filtrado por metadata

### 🌐 Servicios auxiliares
- Traefik (reverse proxy)
- Manejo de certificados TLS
- Enrutamiento HTTPS

---

## Variables de entorno

El archivo `.env.example` contiene un ejemplo de las variables necesarias para el despliegue del sistema.
Ejemplo de variables definidas:

```env
# n8n
N8N_HOST=
N8N_PORT=5678
N8N_PROTOCOL=https

# Base de datos
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=

# Qdrant
QDRANT_API_KEY=
```

