# Frontend – Interfaz de usuario

## Descripción general

El frontend del proyecto fue desarrollado utilizando **Lovable**, con el objetivo de proporcionar una interfaz simple, clara y funcional para interactuar con el sistema RAG construido sobre n8n y Qdrant.

La interfaz permite:

- Enviar enlaces de documentos PDF para su procesamiento
- Realizar consultas inteligentes sobre documentos previamente indexados
- Visualizar respuestas generadas por el modelo de lenguaje en tiempo real

---

## Tecnologías utilizadas

Lovable genera una aplicación frontend moderna basada en las siguientes tecnologías:

- **Lenguaje:** TypeScript
- **Framework:** React
- **Bundler:** Vite
- **Estilos:** Tailwind CSS
- **Configuración adicional:** ESLint, PostCSS

Estas tecnologías permiten un desarrollo rápido, tipado fuerte, buen rendimiento y facilidad de mantenimiento.

---

## Estructura del proyecto

La estructura generada por Lovable es la siguiente:

```text
/public
/src
.gitignore
components.json
eslint.config.js
index.html
package.json
postcss.config.js
README.md
tailwind.config.ts
tsconfig.app.json
tsconfig.json
tsconfig.node.json
vite.config.ts
vitest.config.ts
```
## Descripción de carpetas principales

### `/src`
Contiene el código fuente de la aplicación, incluyendo componentes, lógica de negocio y vistas de la interfaz.

### `/public`
Almacena los archivos estáticos públicos utilizados por la aplicación.

### `index.html`
Punto de entrada principal de la aplicación frontend.

---

## Interacción con el backend

El frontend se comunica con el backend (**n8n**) mediante **Webhooks HTTP**, cumpliendo dos funciones principales:

### 1. Envío de documentos

- El usuario ingresa la URL de un documento PDF
- El frontend envía esta URL mediante un Webhook al flujo de ingesta en n8n
- El backend procesa el documento, lo indexa y lo almacena en el vector store

### 2. Chat sobre documentos

- El usuario realiza preguntas sobre un documento previamente procesado
- El frontend envía:
  - La pregunta del usuario
  - La URL del documento
- La comunicación se realiza hacia el Webhook del flujo de chat
- El backend consulta Qdrant, recupera el contexto relevante y devuelve la respuesta generada por el modelo LLM

---

## Objetivos del diseño de la interfaz

- Facilitar la interacción con el sistema RAG sin exponer complejidad técnica
- Separar claramente los procesos de:
  - Ingesta de documentos
  - Consulta mediante chat
- Proveer una experiencia fluida y comprensible para el usuario final
- Servir como capa de demostración funcional del sistema completo

