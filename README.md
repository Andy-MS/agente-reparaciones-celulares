# Analyze Tech — Agente Inteligente de Reparación de Celulares

Agente conversacional basado en RAG (Retrieval-Augmented Generation) que responde preguntas en lenguaje natural sobre los servicios de un negocio de reparación de celulares: tipos de reparación, precios, tiempos de entrega, garantías y políticas de servicio.

Proyecto desarrollado como parte del **Challenge Alura Agente — ONE AI FOR TECH** (Oracle Next Education).

## 🎯 Objetivo

Construir un agente de IA que:
- Responda preguntas de clientes usando una base de conocimiento real (documentos del negocio).
- Esté desplegado y accesible públicamente en la nube.
- Cuente con documentación clara de arquitectura y uso.

## 🏗️ Arquitectura

El sistema se compone de las siguientes capas:

| Componente | Tecnología | Rol |
|---|---|---|
| Infraestructura | Oracle Cloud Infrastructure (OCI) — Compute, modelo **IaaS** | Servidor virtual (Ubuntu) donde corre toda la aplicación |
| Orquestación | Docker | Contenedor que aísla y ejecuta n8n |
| Motor de automatización / agente | n8n (self-hosted) | Construcción visual del flujo del agente (chat, RAG, memoria) |
| Modelo de lenguaje (LLM) | Groq (Llama 3.3 70B) | Genera las respuestas del agente |
| Embeddings | Cohere (Embed Multilingual v3.0) | Convierte los documentos y preguntas a vectores semánticos en español |
| Base de conocimiento | Vector Store en memoria (n8n) | Almacena los documentos indexados para búsqueda semántica |
| Interfaz | Chat público de n8n | Punto de acceso para que cualquier usuario hable con el agente |


**¿Por qué esta arquitectura?**

Se optó por **OCI Compute (IaaS)** en lugar de un servicio administrado, para tener control total sobre la configuración del entorno y cumplir con el requisito de despliegue en OCI del Challenge. Se eligió **n8n** como motor del agente por su enfoque visual, que permite construir flujos RAG complejos (documentos → embeddings → búsqueda vectorial → LLM) sin escribir código, ideal para iterar rápido bajo tiempo limitado. **Groq** y **Cohere** se seleccionaron por ofrecer capas gratuitas suficientes para el alcance del proyecto y buen soporte multilingüe.


## 🧠 Cómo funciona el RAG (Retrieval-Augmented Generation)

El agente no responde de memoria ni improvisa: cada respuesta se basa en documentos reales del negocio, indexados y recuperados por significado semántico. El proceso ocurre en dos flujos separados dentro de n8n:

### Flujo 1 — Carga de documentos (`Carga-documentos.json`)
Se ejecuta manualmente para construir la base de conocimiento:

1. **Extracción**: dos nodos HTTP Request obtienen el contenido en crudo de los documentos alojados en este mismo repositorio (FAQ, precios, términos, política de datos).
2. **Unificación**: un nodo Merge combina todos los documentos en un único flujo de datos.
3. **Fragmentación**: un Text Splitter divide los documentos en fragmentos de ~1000 caracteres (con 200 de traslape), para que cada fragmento contenga una idea completa y sea fácil de recuperar con precisión.
4. **Vectorización**: cada fragmento se convierte en un embedding (vector numérico de 1024 dimensiones) mediante **Cohere Embed-Multilingual-v3.0**, que representa el significado del texto en español, no solo las palabras.
5. **Almacenamiento**: los vectores resultantes se guardan en un Vector Store en memoria, listo para búsquedas por similitud semántica.

### Flujo 2 — Agente conversacional (`agente-chat.json`)
Se activa cada vez que un usuario envía un mensaje:

1. El mensaje del usuario llega mediante el nodo de chat (interfaz pública).
2. El **AI Agent** decide, según la pregunta, si necesita consultar la base de conocimiento (herramienta RAG) antes de responder.
3. Si es necesario, convierte la pregunta a un embedding con el mismo modelo de Cohere, y busca en el Vector Store los fragmentos con significado más cercano.
4. Esos fragmentos recuperados se entregan como contexto al modelo de lenguaje (**Groq — Llama 3.3 70B**), que genera la respuesta final basada en información real, no inventada.
5. Una memoria de sesión (Simple Memory) permite que el agente mantenga contexto durante la conversación.

**Nota de diseño:** el Vector Store usado (Simple Vector Store) almacena los datos en memoria del contenedor, no en disco. Esto significa que si el contenedor de n8n se reinicia, es necesario volver a ejecutar el Flujo 1 (Carga de documentos) para reconstruir la base de conocimiento. Se optó por esta solución por su simplicidad de configuración, adecuada al alcance y tiempo del Challenge; para producción real se recomendaría una base vectorial persistente (ej. Postgres + PGVector).


## 📋 Estado del proyecto

- [x] Infraestructura desplegada en OCI (Compute, VM.Standard.E2.1.Micro, Ubuntu)
- [x] n8n instalado vía Docker y accesible públicamente
- [x] Flujo base del agente (Chat → AI Agent → Groq + Memoria + Vector Store)
- [ ] Carga de documentos a la base de conocimiento
- [ ] Pruebas de respuesta del agente
- [ ] Video/captura de evidencia funcionando en la nube
