---
name: odd-cycle
description: Gestiona objetivos pequeños de AutoMatch con criterios observables, verificación y aprobación humana.
---

# Ciclo ODD

- Trabaja solo en el objetivo autorizado actual. No avances al siguiente hasta que el usuario diga «listo».
- Antes de implementar, confirma el alcance, inspecciona arquitectura y dependencias, identifica riesgos y presenta un plan mínimo si el cambio no es evidente.
- Define criterios de aceptación observables sin inventar reglas de negocio.
- Realiza únicamente cambios necesarios y revisables. Reutiliza los flujos existentes cuando sea posible.
- Verifica el cambio con comprobaciones apropiadas y registra comandos, resultados reales, errores y límites.
- Si la verificación falla, informa el bloqueo y no presentes el objetivo como completado.
- Crea commits solo cuando estén solicitados por el objetivo y las verificaciones hayan sido satisfactorias. Usa mensajes Conventional Commits.
- Solicita aprobación humana para integrar cambios a `main`; no apruebes ni hagas merge en nombre del usuario.
