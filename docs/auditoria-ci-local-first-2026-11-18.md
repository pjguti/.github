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

## Ampliación de auditoría (solo lectura, 2026-11-18)

La inspección de los árboles confirma 83 workflows en GOS y 70 en PJ_QuantStrategist; **no se han clasificado aún todos**. Muestreo con lectura directa del YAML:

| Repo | Workflow | Runner declarado | Disparador observado |
|---|---|---|---|
| GOS | control-plane-hosted-emergency.yml | ubuntu-24.04-arm y windows-2025 | workflow_dispatch |
| GOS | control-plane-validation.yml | 3 jobs self-hosted Windows | pull_request |
| GOS | consumer-superior-policy-guard.yml | self-hosted Windows | workflow_call |
| GOS | build-qnap-remote-mcp-image.yml | ubuntu-latest | workflow_dispatch |
| GOS | 0668-qnap-materialization.yml | self-hosted Windows | push |
| GOS | 0668-qnap-overlay.yml | self-hosted Windows | push |
| GOS | bootstrap-validation-review.yml | self-hosted Windows | workflow_dispatch |
| GOS | cloudflare-runner-recovery-queue-consumer-bootstrap.yml | self-hosted Windows | workflow_dispatch |
| GOS | cloudflare-runner-recovery-runtime-activate.yml | self-hosted Windows | workflow_dispatch |
| GOS | cloudflare-runner-recovery-storage-bootstrap.yml | self-hosted Windows | workflow_dispatch |
| GOS | cloudflare-runner-recovery-worker-bootstrap.yml | self-hosted Windows | workflow_dispatch |
| PJ_QuantStrategist | a1-advanced-strategy-real-data.yml | self-hosted Windows pjq-main | pull_request, workflow_dispatch |
| PJ_QuantStrategist | a10-blind-holdout-evaluation.yml | self-hosted Windows pjq-main | pull_request, workflow_dispatch |

**Hallazgos:** los jobs GitHub-hosted identificados en GOS son manuales, pero siguen consumiendo cuota si se lanzan. Existen controles de PR sobre self-hosted, sin que esto autorice su activación ni pruebe su seguridad operacional. Los eventos YAML observados no equivalen a una evaluación completa de triggers ni a un cálculo de minutos.

**Prioridad siguiente:** clasificar el resto de los 153 workflows por runner y trigger, comprobar paths/concurrency, y documentar alternativas de ejecución offline. Evitar reintentos de API masivos que fallen por timeout. No despachar Actions.

## Inventario completo estático — 2026-11-19

Se han clonado en BMAX-PJ **copias temporales parciales** (solo `.github/workflows`) de las ramas por defecto. Commits auditados: GOS `4e92603` (83 YAML) y PJ_QuantStrategist `2b98dab` (70 YAML). No se han lanzado Actions ni modificado los repositorios de origen.

| Repositorio | YAML | Archivos con hosted | Archivos con self-hosted | Jobs hosted declarados | Jobs self-hosted declarados |
|---|---:|---:|---:|---:|---:|
| GOS | 83 | 47 | 36 | 48 | 49 |
| PJ_QuantStrategist | 70 | 5 | 64 | 7 | 66 |

Clasificación mediante búsqueda textual de `runs-on:` en líneas YAML: los números son un inventario estático y **no** estiman minutos facturados ni equivalen a validación semántica completa. En QuantStrategist hay un archivo sin `runs-on` simple. Un archivo puede contener varios jobs.

### Hallazgo de prioridad alta: GOS Document Broker

De los 47 workflows GOS que declaran hosted, **38 tienen triggers automáticos**: 28 `pull_request` y 10 `pull_request` + `push`; todos los 38 son `document-broker-*-validation.yml` y declaran filtros `paths:`. Ninguno de estos 38 declara `cancel-in-progress: true` (inspección textual). Otros 5 hosted declaran `workflow_dispatch`, 1 `workflow_call` y 3 no coincidieron con los triggers sencillos inspeccionados (requieren lectura semántica adicional).

En PJ_QuantStrategist, los **5 workflows hosted** detectados declaran `workflow_dispatch`; no se identificaron hosted automáticos en esta revisión textual.

### Plan seguro sin alterar controles

1. Obtener estadísticas históricas **solo lectura** de minutos por workflow/job cuando estén disponibles; no inferir el consumo de los conteos de archivos.
2. Diseñar cambios en Draft PRs independientes para Document Broker: `concurrency` por PR/ref y `cancel-in-progress` cuando sea seguro; evaluar `paths` existentes y evitar doble PR/push sin perder validaciones obligatorias.
3. Clasificar cada validación según si es reproducible offline en BMAX o requiere Linux, servicios, secretos o atestación. No migrar automáticamente un gate de confianza a un runner propio.
4. Evaluar primero efectos sobre branch protection, required checks, PRs de forks, permisos y cancelación de jobs con efectos secundarios.
5. No cambiar ni ejecutar workflows de producción, CAS, lease, ARM64 o GitHub-hosted sin autorización específica.

## Ensayo offline BMAX — 2026-11-19

Checkout temporal parcial GOS `4e92603`, Python 3.12.10 / pytest 8.4.2 en Windows. Se añadieron a sparse-checkout únicamente los directorios de tests, herramientas, esquemas, registry, catálogo y políticas necesarios; no se modificó el repositorio original ni se lanzó GitHub Actions.

1. Cinco archivos seleccionados por los workflows `document-broker-credential-provider-validation.yml` y `document-broker-activation-v2-validation.yml`: **40 passed**, pytest 2.58s, tras incluir esquemas JSON requeridos.
2. Ampliación a 63 archivos `tests/test_document_broker_*.py`: los intentos iniciales fallaron por fixtures ausentes del checkout parcial (`registry/document-broker`, `catalog/document-broker`, `policies/data`); se añadieron sin cambiar código.
3. Con esas fixtures disponibles: **158 passed, 1 failed**, pytest 15.13s, `--maxfail=1`. Fallo en `test_document_broker_durable_document_e2e.py::test_real_synthetic_document_execution_flows_into_durable_delivery_and_ack`: `validate_source_root_binding` exige `mount_root` absoluto POSIX (`startswith('/')`), pero `tmp_path` en Windows produce `C:/...`. No se ha demostrado defecto de seguridad ni equivalencia Linux. Las pruebas restantes no se ejecutaron.

**Decisión:** no relajar `source_boundary.py` para adaptar la prueba. Clasificar los tests por portabilidad y ejecutar las integraciones POSIX en Linux aislado cuando esté disponible y autorizado, sin GitHub-hosted. La cobertura local y la atestación remota son evidencias distintas.

## Matriz de portabilidad offline — ampliación 2026-11-19

En BMAX-PJ Windows, checkout temporal GOS `4e92603`, se ejecutaron **todos los tests de los 63 archivos** `tests/test_document_broker_*.py` con pytest sin `--maxfail` (533 casos reportados). La primera ronda, con checkout parcial incompleto, produjo **457 passed / 76 failed** en 50.30s; el análisis de errores identificó fixtures faltantes bajo `handoffs/`, `reviews/`, `promotions/`, `architecture/` y `tools/windows/`.

Tras ampliar **solo** el sparse-checkout con esos directorios, segunda ronda: **496 passed / 37 failed** en 39.66s. Los fallos pendientes incluyen pruebas POSIX de `mount_root`, permisos SQLite de directorios, enlaces simbólicos/listados en Windows, y algunos tests de inventario/publicación/promoción que requieren revisión de fixtures y de su entorno. No se ha modificado el código fuente ni se ha ejecutado GitHub Actions.

**Interpretación:** 496 tests demuestran portabilidad y ejecución local parcial; 37 resultados fallidos **no son automáticamente defectos del producto** ni deben marcarse como aprobados. Separar (a) dependencias incompletas del checkout, (b) semántica Windows/POSIX, (c) posibles fallos reales antes de proponer migración. No debilitar comprobaciones de rutas, ACL, SQLite o enlaces simbólicos. Validación Linux aislada pendiente de autorización de infraestructura si no hay entorno ya disponible y permitido.

## Cierre de diagnóstico Windows y punto de autorización — 2026-11-19

Tercera ejecución de los 63 archivos `tests/test_document_broker_*.py` en checkout parcial temporal GOS `4e92603` tras incorporar `publications/document-broker` y `templates/runners/organizational-document-broker`: **518 passed / 15 failed / 533 total**, pytest 45.57 s. Sin cambios en el código ni en el repositorio remoto.

Los 15 fallos se concentran en:
- 5 casos de ejecución/documentos durables y límites de solicitud (`mount_root` POSIX absoluto).
- 4 casos de `source_boundary` (rutas y symlinks bajo Windows).
- 4 casos de `sqlite_storage_boundary` (semántica de permisos/ACL de directorios).
- 2 casos de `listing_regression` (filtrado de symlinks bajo Windows).

La enumeración **solo lectura** `wsl.exe --list --quiet` en BMAX-PJ confirmó una distribución **Ubuntu-24.04** registrada. No se ha iniciado ni activado esa distribución. `docker` no aparece disponible como comando. La siguiente validación apropiada es ejecutar el mismo commit y los 533 casos en Ubuntu WSL existente, previa autorización explícita para iniciar/usar la distribución y comprobación de aislamiento/efectos secundarios. No ejecutar Actions, no activar runners ni alterar CAS/leases. El 97,2 % es tasa de tests superados, no cobertura de código.
