# Instrucciones del proyecto AutoMatch

## Autoridad y alcance

- Este repositorio contiene el harness inicial del proyecto AutoMatch de inventario. Los requisitos funcionales y la arquitectura se definirán por objetivos; no los infieras de este scaffold.
- Trabaja un objetivo autorizado a la vez. No comiences el siguiente hasta que el usuario diga «listo».
- Antes de implementar, revisa el flujo existente, las dependencias, las convenciones y los riesgos. Para cambios no evidentes, presenta primero un análisis o plan mínimo.
- No amplíes el objetivo, inventes reglas de negocio, endpoints, estados ni arquitectura, ni modifiques sistemas o repositorios ajenos.
- Si el cambio requiere alterar un flujo compartido o una regla de negocio existente, detente y reporta el impacto antes de implementarlo.
- No copies secretos, tokens, credenciales ni datos reales al repositorio. No solicites secretos por chat.

## Ciclo ODD

- Sigue `.agents/skills/odd-cycle/SKILL.md` en cada objetivo y trabaja mediante cambios pequeños y revisables.
- Completa implementar → verificar → commit para el objetivo autorizado. Registra evidencia real y distingue lo ejecutado de lo pendiente.
- La verificación es obligatoria. Sigue `.agents/skills/verification/SKILL.md` y ejecuta comprobaciones apropiadas al cambio.
- Aplica `.agents/skills/n8n-workflow/SKILL.md` cuando un objetivo afecte workflows de n8n.
- No crees workflows ni lógica de inventario como parte del objetivo de inicialización.

## Git, ramas y aprobación humana

- `main` es la rama estable y `dev` es la rama de integración.
- Sigue [docs/git-flow.md](docs/git-flow.md). Usa una rama por objetivo cuando corresponda; no crees ramas fuera del objetivo autorizado.
- Los cambios hacia `main` requieren Pull Request y aprobación humana explícita. Codex no aprueba ni hace merge.
- No hagas force-push, borres ramas remotas, cambies reglas de protección ni alteres permisos sin autorización explícita.
- No hagas commit ni push salvo que el objetivo autorizado los solicite. Nunca simules commits ni historial.
- El repositorio es público para clonación. El acceso de escritura se administra en GitHub; no supongas que una URL pública concede permiso para modificar el repositorio.

## Seguridad y veracidad

- No expongas credenciales ni datos sensibles en logs, ejemplos, documentación o commits.
- No actives ni despliegues recursos compartidos sin autorización explícita.
- Informa con precisión sobre limitaciones, fallos y verificaciones pendientes.
