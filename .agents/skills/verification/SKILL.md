---
name: verification
description: Define y registra verificaciones reproducibles para los objetivos de AutoMatch.
---

# Verificación

- Verifica todos los cambios con comprobaciones reproducibles y apropiadas al objetivo autorizado.
- Revisa el diff final y confirma que no contiene cambios ajenos al objetivo ni secretos.
- Registra los comandos ejecutados, sus resultados reales y errores relevantes. Distingue lo comprobado de lo pendiente.
- No declares un objetivo completado si sus criterios de aceptación no se verificaron.
- No crees pruebas backend salvo que el usuario las solicite. No añadas pruebas o ejecutes suites no relacionadas con el cambio.
- Si una verificación falla o no puede ejecutarse, informa el motivo y el impacto en el resultado.
