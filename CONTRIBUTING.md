# Contribuir a los repositorios de pjguti

## Principio general

Todo cambio debe ser pequeño, revisable, reversible y auditable. El uso desde móvil o mediante ChatGPT no reduce los controles exigidos.

## Pull requests

Antes de fusionar un PR:

1. Identificar el SHA exacto del head.
2. Comprobar los checks obligatorios definidos por el repositorio.
3. No aceptar estados de un SHA anterior.
4. Tratar una lista vacía de estados como ausencia de validación, nunca como éxito.
5. Inspeccionar los workflows de GitHub Actions asociados al mismo SHA.
6. No declarar superados workflows `queued` o `in_progress`.
7. Releer el PR inmediatamente antes del merge.
8. Confirmar que sigue abierto, no es draft, es fusionable y conserva el head validado.
9. Usar el método de merge autorizado y `expected_head_sha` cuando esté disponible.
10. Registrar el SHA resultante y el criterio de validación aplicado.

## Contingencia CircleCI

La contingencia CircleCI solo puede usarse cuando el repositorio haya completado un alta explícita y tenga registrados:

- sus contextos obligatorios reales;
- la rama base;
- el método de merge;
- la política de GitHub Actions;
- la autorización operativa aplicable.

No debe copiarse automáticamente el contrato de otro repositorio.

## Seguridad

No se deben modificar producción, secretos, protecciones, rulesets, proveedores externos, pagos ni datos reales sin autorización específica y comprobable.
