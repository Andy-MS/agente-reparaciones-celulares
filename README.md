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

## 📋 Estado del proyecto

- [x] Infraestructura desplegada en OCI (Compute, VM.Standard.E2.1.Micro, Ubuntu)
- [x] n8n instalado vía Docker y accesible públicamente
- [x] Flujo base del agente (Chat → AI Agent → Groq + Memoria + Vector Store)
- [ ] Carga de documentos a la base de conocimiento
- [ ] Pruebas de respuesta del agente
- [ ] Video/captura de evidencia funcionando en la nube
