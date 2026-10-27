# Política global — propuesta de adopción de canal canónico de merge

**Fecha:** 2026-10-27. **Estado:** PROPOSAL_ONLY / DRAFT / NO_NORMATIVE_EFFECT.
**Owner de coordinación:** SYSTEM CONTROL. **Fuente técnica:** `pjguti/github-operating-system`.
**Decisión:** opción A autorizada **como objetivo de diseño**, pendiente de aprobación normativa e implementación.
**Antecedentes:** [GOS Draft #1294](https://github.com/pjguti/github-operating-system/pull/1294), GOS #1216; propuesta técnica GOS en `docs/proposals/system-control-canonical-merge-option-a-20261027.md`.

## 1. Cláusula global a considerar en promoción posterior (no vigente)

Para cualquier PR que cambie política, control plane o infraestructura compartida de `pjguti`, la fusión solo podrá realizarse mediante el ejecutor de merge gobernado aprobado y disponible para el repositorio, con autorización expresamente vinculada a repo, PR, HEAD, base, método y gates. El ejecutor debe releer política desde fuente confiable, validar cobertura y pruebas aplicables y volver a comprobar base, HEAD y estado inmediatamente antes de una única acción de merge con SHA condicionado. La operación debe producir receipt post-merge.

La mera disponibilidad de GitHub UI, REST, plugin, `gh`, Desktop Commander o consola **no autoriza su uso como vía alternativa de escritura** cuando el procedimiento canónico falle, esté ausente o carezca de permisos. Conector GitHub y API siguen permitidos para lectura, diagnóstico y preparación de propuestas dentro de sus ámbitos autorizados.

`BLOCKED` es un resultado válido: sin identidad, permiso, canal, runner/capacidad, autorización, validación o postcondición verificable, detenerse. No saltar gates, no credenciales ad hoc, no reintentos sobre resultado ambiguo, no segundo writer, ni coste recurrente adicional.

## 2. Scope por repositorio y autoridad

- GOS es owner del código/contratos del broker y de las decisiones sobre control plane compartido.
- El manifest y gates de GOS **no pueden aplicarse sin adaptación a `pjguti/.github`**. La política de cada repositorio debe derivarse de un origen confiable y ser validada expresamente. No permitir ejecución de merge en `.github` hasta probar su integración real en el mismo mecanismo canónico o en un procedimiento delegado formalmente equivalente autorizado por SYSTEM CONTROL.
- Los consumidores no adquieren capacidad de fusionar cambios compartidos; requests y grants son independientes de acceso técnico.
- La serialización CAS de GOS permanece vigente para mutaciones estructurales; esta propuesta no crea un segundo lock o reconciliador.

## 3. Compatibilidad con REMOTE_OPERATIONS_POLICY.md

Este documento **no modifica** la política vigente de observación controlada de Desktop Commander ni sus protecciones para documentos privados QNAP. El permitir observación remota read-only no concede merge, administración ni lectura de contenido privado.

Como deuda editorial separada, `REMOTE_OPERATIONS_POLICY.md` en main todavía contiene redacción prospectiva («eventual aprobación», «sustituiría», «hasta entonces»), pese a su merge autorizado de 2026-10-26. Preparar corrección limitada a tiempo verbal/estado y comprobar que no cambie ningún permiso, grant ni umbral. Esa corrección requiere una PR y gates propios; no se efectúa aquí.

## 4. Gates de una futura promoción normativa

1. Decisión formal del owner sobre texto global y cobertura de repositorios.
2. Implementación de broker único y pruebas de negativas/TOCTOU en GOS, con admisión estructural y validación exact-HEAD; ningún PR técnico se fusiona por esta propuesta.
3. Superficie de ejecución real aprobada y verificada; si no existe, no promover una regla que vuelva imposible una operación necesaria sin procedimiento de bloqueo explícito.
4. Validar que `.github` tiene manifest/política específica y criterios de check aplicables; no exigir gates inexistentes ni omitir gates reales.
5. Abrir PR normativa independiente contra `REMOTE_OPERATIONS_POLICY.md` o política superior adecuada, con review, SHA exacto, gates vigentes, coordinación GOS y autorización de merge separada.
6. Mantener trazabilidad de #1291 y `.github#5` como merges históricos por conector; no fingir retroactividad.

**No autorización:** no merge, no cambios efectivos de policy, no cambios de seguridad/ownership, infraestructura, CAS, secretos, NAS ni documentos privados.
