---
name: n8n-workflow
description: Analiza, modifica, verifica e importa workflows n8n de AutoMatch dentro del objetivo autorizado.
---

# Workflows de n8n

- Usa las herramientas disponibles del MCP oficial de n8n autenticado por OAuth como vía prioritaria y obligatoria para interactuar con la instancia.
- Antes de cada operación, comprueba las herramientas, capacidades, permisos OAuth, instancia y recurso objetivo que existan realmente. No supongas que una operación está disponible.
- Limita las consultas y cambios a los recursos de AutoMatch autorizados. No consultes ni modifiques workflows ajenos.
- No uses automatización de interfaz gráfica ni accedas o modifiques directamente la base interna de n8n. Si una operación necesaria no es posible mediante MCP, informa la limitación y solicita autorización explícita antes de considerar una alternativa.
- Distingue validaciones previas de cambios persistidos y ejecuciones reales. Registra evidencia verificable de cada operación.
- No exportes ni publiques credenciales, tokens, IDs sensibles o valores secretos. Configura credenciales en el entorno autorizado, nunca en el repositorio.
- Antes de modificar un flujo compartido, una regla de negocio o la arquitectura existente, detente e informa el impacto para que el usuario decida.
- No actives workflows ni despliegues cambios compartidos sin autorización explícita.
