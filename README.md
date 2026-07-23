# Políticas y plantillas globales de pjguti

Este repositorio público proporciona archivos comunitarios predeterminados para los repositorios de la cuenta que no definan sus propias versiones.

## Desarrollo móvil y contingencia CircleCI

El protocolo técnico central se mantiene en el repositorio privado `pjguti/github-operating-system`.

Principios públicos aplicables:

- validar siempre el SHA exacto del head del pull request;
- no considerar una lista vacía de estados como éxito;
- no considerar workflows `queued` o `in_progress` como superados;
- revalidar el PR inmediatamente antes del merge;
- usar `expected_head_sha` cuando el repositorio lo permita;
- registrar claramente cuándo se aplica contingencia CircleCI;
- exigir alta y autorización específica por repositorio.

Estas plantillas no activan CI, protecciones, permisos ni automatizaciones por sí mismas.
