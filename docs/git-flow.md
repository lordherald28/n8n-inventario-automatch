# Git Flow de AutoMatch

## Ramas base

- `main`: versión estable y punto de referencia del repositorio.
- `dev`: integración de objetivos verificados antes de proponer su publicación estable.

## Trabajo por objetivo

1. Atiende un objetivo ODD a la vez y documenta su alcance y criterios observables.
2. Para cambios de desarrollo, crea una rama `feature/<objetivo>` desde `dev` cuando el objetivo lo autorice. Mantén la rama acotada al objetivo.
3. Verifica los cambios y crea un commit Conventional Commit después de superar los criterios del objetivo.
4. Integra el trabajo en `dev` mediante Pull Request cuando corresponda. Espera aprobación humana antes de cualquier merge.
5. Tras la validación de integración, propone `dev` hacia `main` mediante Pull Request. El usuario aprueba el merge.

## Reglas

- No se permiten cambios directos a `main` para el trabajo ordinario.
- No hagas merge, force-push, borres ramas ni cambies rulesets sin autorización explícita.
- No crees ramas fuera del objetivo autorizado. La inicialización contempla únicamente `main` y `dev`.
- Las protecciones técnicas de GitHub (rulesets, permisos y revisores) solo se configuran con autorización y verificación explícitas; este documento no afirma que estén activadas.
- El repositorio público permite clonar y consultar el contenido. GitHub puede permitir a terceros crear forks o proponer Pull Requests; eso no les da permiso para escribir directamente en el repositorio original. Gestiona el acceso de escritura desde la configuración de GitHub.
