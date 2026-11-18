# Política de consumo mínimo GitHub-hosted Actions (2026-11-18)

**Hecho comunicado por el operador:** la cuota mensual de GitHub-hosted es muy reducida y se agotó el día 4. Hasta nueva evidencia de reposición, considerar agotado el presupuesto de minutos hosted. No interpretar automáticamente los errores de Actions como impagos ni insistir en Billing.

## Reglas transversales
1. No lanzar ni reintentar workflows con runners GitHub-hosted por defecto. La autorización para consultar GitHub NO implica autorización para gastar cuota.
2. Priorizar pruebas offline en BMAX/Windows y laboratorios aislados; pytest, PowerShell, análisis estático y fixtures sintéticas. GitHub API para inspección de código/PR/issues sin dispatch.
3. Self-hosted solo tras aprobación de activación y revisión de aislamiento, confianza de código, secretos y permisos. Esta política NO autoriza activar ARM64/QNAP.
4. Si existe cuota y se necesita excepción: estimar minutos, minimizar matrices, usar filtros de rutas, concurrencia/cancelación y timeouts, y obtener autorización expresa para ese gasto.
5. Si falla antes de runner por límite de gasto, registrar bloqueo de capacidad y NO reintentar. Una prueba local no equivale a atestación autenticada.
6. Mantener restricciones DENY_ONLY, CAS, permisos y segregación de cuentas ChatGPT; no habilitar merges, despliegues o escrituras de estado por esta política.
7. Propagar la política a la documentación de gobierno/continuidad de cada proyecto accesible; los repositorios no heredan automáticamente este documento.

**Referencias de seguimiento:** GOS #1613, IA Core #351, Coordinación transversal #92, PJ_QuantStrategist #1990.
