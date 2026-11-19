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

## Intento autorizado WSL — 2026-11-19

El operador autorizó expresamente **iniciar y usar Ubuntu-24.04 WSL en BMAX-PJ solo para pruebas sintéticas locales**, sin runners, secretos, CAS/leases ni hosted. Se ejecutó únicamente un comando de identificación de entorno `wsl.exe -d Ubuntu-24.04 -- bash -lc '...'`, que **no llegó a ejecutar bash**: fallo `Wsl/Service/CreateInstance/CreateVm/ERROR_TIMEOUT` tras ~90 s. `wsl --list --verbose` mostró `Ubuntu-24.04 Stopped 2` (consulta terminada con código 1). Los servicios `vmcompute` y `hns` aparecieron `Running`; `.wslconfig` existente define `memory=6GB`, `processors=2`, `swap=2GB`, NAT y `instanceIdleTimeout=-1`. No se cambió esa configuración ni se reinició servicio/host. **0 pruebas ejecutadas en Linux**; siguen vigentes los 518 passed / 15 failed de Windows. Siguiente fase: diagnóstico read-only del arranque WSL; cualquier reinicio, modificación de servicios o configuración requiere autorización separada.

## Diagnóstico de recuperación WSL autorizado — 2026-11-19 08:14+

Autorización expresa para recuperación controlada de WSL **sin reiniciar Windows ni tocar servicios de producción**. Preflight read-only: sesión administrativa elevada; `WslService`, `vmcompute`, `hns` en `Running`; VM `OpenClaw-Ubuntu` **Running** (~2.7 GiB asignados); `vmmemWSL` activo (~1.6–1.8 GiB working set); demás VM apagadas. Eventos `Microsoft-Windows-Hyper-V-Compute-Admin`: ID 10015, alta presión de memoria, y 10018, error `0x8007000E` al ampliar memoria de `OpenClaw-Ubuntu`. RAM host: **15.75 GiB total / 0.78 GiB libre** en el momento de la medición; `.wslconfig` limita WSL a 6 GB. Procesos Brave múltiples consumen RAM considerable, pero son sesiones interactivas y no se cerraron.

**Decisión segura:** no reiniciar `vmcompute`, `hns`, `WslService`, ni ejecutar `wsl --shutdown` mientras hay servicios activos y memoria insuficiente; no cerrar Brave ni reducir recursos de OpenClaw sin autorización explícita para el impacto. No se ha recuperado Ubuntu; **0 pruebas Linux**. Siguiente paso requiere liberar memoria de forma coordinada con el operador, o autorizar una ventana de mantenimiento con riesgos delimitados. Diagnóstico probable: presión de memoria, no prueba concluyente de causalidad única.

## Cierre de clasificación de workflows ambiguos — 2026-11-19

Inspección estática del checkout temporal GOS `4e92603` y PJ Quant `2b98dab` (sin ejecutar CI):

- `pj-promotion-constructor.yml`: `issue_comment.created`, job `ubuntu-latest`, condicionado a issue **#1041**, usuario `pjguti` y prefijo exacto `/construct-pj-promotion `; emplea GitHub App token de observación de runner. **Hosted, disparador por comando**, no push/PR general.
- `pj-promotion-evidence-capability-probe.yml`: mismo disparador condicionado a `/probe-pj-promotion-evidence `, `ubuntu-latest`, consulta de runners. **Hosted, disparador por comando**.
- `ref-lifecycle-delete-operator.yml`: mismo disparador condicionado a `/execute-ref-delete `, `ubuntu-latest`, validaciones de autorización/ref. **Hosted, operación destructiva protegida**; no modificar ni reubicar sin revisión de seguridad y autorización explícita.
- PJ Quant `superior-policy-guard.yml`: `push` a main y `pull_request`; job **reutilizable** `uses: pjguti/github-operating-system/.github/workflows/consumer-superior-policy-guard.yml@main`, sin `runs-on` local. Se verificó en GOS que el llamado declara `runs-on: [self-hosted, windows, x64]`. **No es GitHub-hosted directo ni indirecto según el YAML inspeccionado**; depende de disponibilidad de runner self-hosted y del ref mutable `@main`. No eliminar guard de gobernanza.

Conclusión: la prioridad de ahorro de hosted sigue siendo el grupo de 38 `document-broker-*-validation.yml` con triggers PR/push filtrados. Los tres `issue_comment` tienen riesgo operativo y de seguridad distinto. El workflow reusable PJ Quant no debe contabilizarse como hosted por ausencia de `runs-on` en el llamador.

## Controles de coste en 38 validaciones hosted Document Broker — 2026-11-19

Inspección estática de los 38 workflows `document-broker-*-validation.yml` con `runs-on: ubuntu-*` en GOS `4e92603`:

| Indicador | Resultado |
| --- | ---: |
| Workflows hosted | 38 |
| PR + push filtrado por rutas | 10 |
| Solo PR filtrado por rutas | 28 |
| Con `paths` | 38 |
| Con `actions/setup-python` | 38 |
| Con instalación pip explícita | 38 |
| Con `timeout-minutes` | 18 |
| Sin `timeout-minutes` | **20** |
| Con `concurrency` a nivel raíz | **0** |

**Prioridades para propuesta, sin aplicar:** (1) añadir límites de tiempo adecuados y explícitos a los 20 jobs sin timeout, previa evaluación de duración real; (2) evaluar `concurrency`/cancelación de PR obsoletos, sin cancelar tareas de seguridad críticas ni trabajos de `push` que aporten evidencia; (3) agrupar selectivamente instalaciones y ejecuciones redundantes sin perder aislamiento, rutas ni gates; (4) medir coste real por workflow antes de atribuir ahorros. Las 38 rutas filtradas no implican que cada PR dispare los 38. La mera adición de `concurrency` o modificación de YAML podría disparar CI hosted; **no aplicar sin autorización y plan de ventana de cuota**.

## Plan de implementación propuesto, aún NO autorizado — 2026-11-19

**Objetivo:** reducir consumo GitHub-hosted sin degradar los controles de seguridad de Document Broker, ni modificar las rutas de aprobación, los artefactos de evidencia o los workflows de comandos sensibles. Baseline estático: 38 validaciones hosted con filtros `paths`, 20 sin `timeout-minutes`, 0 con `concurrency` raíz; 28 solo PR y 10 PR+push. Baseline de tests local Windows: 518/533; 15 incompatibilidades/semánticas POSIX pendientes de validación Linux por presión RAM host.

**Fase 0 — solo análisis (completada parcialmente):** preservar inventario por nombre, trigger, runner, filtros de rutas, dependencias y test exacto; obtener después tiempos reales de ejecución/cola y minutos facturados, separando fallos por cuota de fallos de código. No estimar ahorro monetario sin métricas.

**Fase 1 — límites de tiempo:** propuesta de PR de alcance acotado para los 20 YAML sin `timeout-minutes`. Elegir límites por datos históricos y pruebas reales, no aplicar indiscriminadamente 5 minutos a pruebas e2e/SQLite. Conservar triggers, filtros, `permissions`, `uses`, secretos, comandos, rutas y nombres de checks. Validación local de parseo YAML, comparación de diffs semánticos y revisión de seguridad; la mera apertura de PR puede disparar CI hosted. **Requiere autorización específica antes de cambiar YAML o abrir PR.**

**Fase 2 — concurrencia:** evaluar por workflow un `concurrency.group` estable con identidad de workflow y PR/ref; `cancel-in-progress` solo para ejecuciones PR reemplazadas, nunca cancelar evidencia obligatoria en `push` ni tareas con efectos externos. Comprobar colisiones entre workflows, forks y reglas de required checks; aprobar por separado.

**Fase 3 — consolidación:** matriz de solapamientos entre test commands, requirements, paths y gates; proponer grupos pequeños conservando trazabilidad y required status checks. Nunca agrupar un gate sensible sin equivalencia demostrada; comparar resultado en Linux y Windows y revisar seguridad antes de migrar.

**Fase 4 — despliegue gradual:** un subconjunto piloto, métricas de coste, tiempo de feedback, falsos verdes y seguridad, rollback explícito y autorización de cuota. No iniciar runners ARM64/QNAP ni usar GitHub-hosted hasta aprobación expresa.

**Bloqueo operativo:** BMAX-PJ ~0.78 GiB RAM libre, VM OpenClaw-Ubuntu en producción y Brave interactivo protegido; no cerrar Brave ni reiniciar WSL/VM. El intento WSL previo falló `CreateVm/ERROR_TIMEOUT`. Continuar con análisis estático y documentos, sin elevar consumo de memoria.

## Implementación fase 1 preparada localmente — 2026-11-19

Autorización expresa del operador para **crear rama aislada y preparar cambios de timeout** en GOS, excluyendo push, PR, GitHub Actions, merge y activación de runners. En BMAX-PJ, checkout temporal `%TEMP%\\ci-hosted-audit-20261119\\github-operating-system` (base GOS `4e92603a5516948c60f387ec6ba37991522dbfef`), rama local `audit/ci-document-broker-timeouts-20261119`, commit **`51e2d8ed0f957cf8855c24ae5da29b54a3baf705`**.

Diff exacto: **20 workflows**, **20 inserciones**, **0 eliminaciones**; se añadió solo `timeout-minutes: 30` bajo el job `runs-on` de cada YAML hosted Document Broker que no tenía timeout. Comprobaciones locales: `git diff --check` correcto; PyYAML 6.0.2 pudo cargar todos; comparación semántica con `git show HEAD:<file>` antes del commit confirmó que, tras retirar el nuevo campo, todos los demás valores YAML eran idénticos a la versión original. Los archivos `__pycache__` no rastreados preexistentes quedaron fuera del commit.

**Advertencia:** 30 minutos es un **límite provisional conservador, no validado con métricas históricas**. Revisar especialmente e2e, recovery y SQLite antes de producción. Rama y commit **solo locales**, no existen aún en GitHub remoto; **0 ejecuciones hosted**. Próxima decisión: autorizar o no publicación de rama/PR, sabiendo que el push o la PR pueden disparar CI y que la cuota hosted está agotada. Preferible validar duración histórica y acordar estrategia de cuota antes de publicar.

## Publicación autorizada de rama y PR draft — 2026-11-19

El operador autorizó expresamente publicar la rama y abrir una PR en borrador **sin merge ni dispatch manual**. Se publicó en `pjguti/github-operating-system` la rama `audit/ci-document-broker-timeouts-20261119`, head `51e2d8ed0f957cf8855c24ae5da29b54a3baf705`, y se creó **Draft PR #1616**: https://github.com/pjguti/github-operating-system/pull/1616 (base `main` en `4e92603a5516948c60f387ec6ba37991522dbfef`). API confirma `draft=true`, `merged=false`, **20 archivos, 20 inserciones, 0 eliminaciones**. El indicador `mergeable=false` en respuesta inicial no demuestra causa (puede reflejar estado inicial o checks); revisar antes de actuar. No se inició manualmente ningún workflow ni se fusionó. Los triggers automáticos de GitHub pueden reaccionar a la PR y no se deben interpretar como dispatch manual.

## PR #1616: primer resultado automático y bloqueo de admisión — 2026-11-19

Consulta read-only de GitHub: para el head `51e2d8ed0f957cf8855c24ae5da29b54a3baf705`, la primera página de runs PR devuelve `Control Plane Validation` **#44486364112**, `completed/failure`. Job `Classify control-plane validation scope` **failure**; los dos jobs `Validate governance control plane` quedaron **skipped**. Log del job clasificador: `admission_error=admission state is not admitted`, `exit code 3`. El workflow `.github/workflows/control-plane-validation.yml` obtiene estado desde `system-control/control-plane-admission-state` y usa clasificador confiable de la base; no es error de parseo del timeout.

Consulta read-only del singleton `governance/control-plane-mutation-admission-state.json` en la rama de admisión: **`status: idle`**, `head_sha: null` en el momento de consulta. **No hay admisión activa para PR #1616.** No cambiar ese estado, ni forzar checks, ni reintentar jobs sin autorización específica y revisión del protocolo de admisión. La consulta de runs del conector solo devuelve la primera página y los eventos PR; **no prueba ausencia de otros runs/checks**. La PR sigue en borrador, sin merge.
