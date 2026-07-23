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
