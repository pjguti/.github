# Auditoría inicial de capacidad CI — 2026-11-18 (solo lectura)

## Alcance y evidencia
Inspección de árboles de workflow en las ramas por defecto de `pjguti/github-operating-system`, `pjguti/ia-agentica-core`, `pjguti/coordinacion-transversal` y `pjguti/PJ_QuantStrategist`. Conteo de archivos YAML bajo `.github/workflows/`: 83, 5, 0 y 70, respectivamente (158 archivos). Estos conteos **no** representan ejecuciones, consumo ni necesariamente workflows activos.

## Hallazgo confirmado — IA Core
- `.github/workflows/ci.yml`: eventos `pull_request`, `push`, `workflow_dispatch`; `concurrency.cancel-in-progress: true`.
- Job `static`: `[self-hosted, Windows, X64, ia-agentica-bmax]`, timeout 20 minutos.
- Job `lease`: `ubuntu-latest`, timeout 25 minutos, condicionado a PR y éxito de `static`. Comprueba contratos y solicita un token GitHub App para intentar adquirir lease. **Este job usa GitHub-hosted y es parte de la cadena de control; no eliminar ni trasladar sin revisión de límites de confianza.**
- Job `test`: `[self-hosted, Linux, ARM64, ia-agentica-core]`, timeout 40 minutos, condicionado al éxito de `static` y `lease`. No se autoriza su activación por esta auditoría.
- `emergency-hosted-ci.yml`: activación manual, dos jobs `ubuntu-24.04-arm` con timeout 30 minutos cada uno. No lanzar mientras la cuota hosted esté agotada.
- `bmax-control.yml`, `dependency-lock.yml`, `gos-capacity-admission-smoke.yml`: runners self-hosted, según sus definiciones. Su ejecución sigue sujeta a seguridad y autorización aplicables.

## Decisiones y siguientes pasos
1. **No despachar** GitHub-hosted para diagnóstico o CI ordinario mientras no exista presupuesto disponible y autorización expresa.
2. Mantener pruebas sintéticas/offline en BMAX y revisión de PR mediante GitHub API. Las pruebas locales no sustituyen atestación autenticada.
3. Auditar en detalle los 83 workflows GOS y 70 de PJ_QuantStrategist para clasificar por eventos, runner, duración máxima, riesgo y alternativas; el conteo actual no permite estimar coste.
4. Diseñar una futura separación entre validación local y gate remoto, sin relajar las fronteras de confianza de `lease` ni habilitar ARM64/CAS.
5. No cambiar triggers, secrets, permisos, runners, CI requerido, branch protection o producción sin evaluación específica.

Política transversal: [GitHub-hosted local-first](../docs/politica-github-hosted-local-first.md). Issues: GOS #1613, IA Core #351, PJ QuantStrategist #1990.
