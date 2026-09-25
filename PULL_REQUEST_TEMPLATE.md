## Objetivo

Describe qué cambia y por qué.

## Alcance

- [ ] El cambio es pequeño, revisable y reversible.
- [ ] No modifica producción, secretos, pagos ni servicios externos sin autorización específica.
- [ ] No amplía permisos ni automatizaciones fuera del alcance declarado.

## Validación

- Head SHA exacto:
- Checks obligatorios del repositorio:
- GitHub Actions asociados al mismo SHA:
- ¿Se aplica contingencia CircleCI?: sí / no
- Si se aplica, contextos exactos utilizados:

## Operación remota

- [ ] Se aplicó `REMOTE_OPERATIONS_POLICY.md`.
- [ ] Desktop Commander no se usó para polling, logs, validación, exploración, recuperación rutinaria ni automatización recurrente.
- [ ] Si Desktop Commander estaba en hard freeze (>=90% de cuota), no se usó salvo excepción crítica con ausencia de alternativa y autorización explícita del operador en ese turno.
- [ ] Las operaciones remotas ordinarias usaron GitHub/API, self-hosted runners, workflows acotados, conectores específicos o la acción manual local mínima.

## Auditoría previa al merge

- [ ] PR abierto.
- [ ] `draft=false`.
- [ ] `mergeable=true`.
- [ ] Head sin cambios desde la validación.
- [ ] Método de merge autorizado.
- [ ] `expected_head_sha` utilizado cuando está disponible.

## Resultado

- SHA resultante del merge:
- Limitaciones o trabajos posteriores:
