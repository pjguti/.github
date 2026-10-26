# Política global de operación remota y Desktop Commander

**Vigencia:** desde su eventual aprobación y merge coordinados; el contenido de una PR Draft no modifica la política efectiva.

Ámbito: proyectos y repositorios de la cuenta `pjguti` sujetos a esta política global.

## Principio de autoridad

Desktop Commander es un canal de acceso técnico, no un propietario operativo, reconciliador ni fuente de verdad. La fuente de verdad de infraestructura compartida es `pjguti/github-operating-system` y las decisiones transversales se coordinan desde SYSTEM CONTROL.

El acceso efectivo a una máquina no concede permisos sobre todas sus cuentas, carpetas, proyectos, servicios o datos. Cualquier acción debe corresponder a una necesidad autorizada, un host identificado y un alcance mínimo.

## Modalidad ordinaria: observación controlada

Se permite Desktop Commander para diagnósticos rutinarios **de solo lectura** sobre recursos expresamente autorizados: estado de servicios y procesos, salud de runners, conectividad, inventario acotado y lectura mínima de logs pertinentes. Las comprobaciones deben minimizar volumen, repetición y exposición de datos.

GitHub Connector/API y los conectores específicos siguen siendo preferentes cuando ofrecen la misma evidencia completa de forma más directa; Desktop Commander puede aportar observación independiente del propio runner o de una máquina local cuando GitHub no puede hacerlo.

La cuota ilimitada comunicada por el operador el 26 de octubre de 2026 sustituye al presupuesto finito histórico. No se aplica el umbral de bloqueo del 90 %, ni límites porcentuales diarios o por número de llamadas, mientras esa condición permanezca vigente. No se interpreta como garantía contractual del proveedor ni como autorización ilimitada de operaciones.

## Mutaciones: autorización separada

Lectura no autoriza escritura. Reiniciar servicios, ejecutar instalaciones, modificar archivos, permisos, red, credenciales o configuración, registrar runners, borrar datos o desplegar componentes requiere autorización explícita para la operación concreta y todos los gates del control plane.

La ejecución remota de una mutación debe utilizar **el canal aprobado para esa intervención**. La disponibilidad de Desktop Commander no habilita por sí misma un shell genérico ni sustituye el operador o ejecutor prescrito por el procedimiento vigente.

Las operaciones sobre documentos privados exigen, además, el grant específico por proyecto, usuario, ruta, finalidad y modalidad (lectura, creación, sobrescritura, borrado). Se aplican límites de divulgación, integridad, auditoría y retención. Ninguna regla general de observación concede acceso implícito a QNAP o a documentos privados.

## Arquitectura y preferencia operativa

1. Usar API o conector específico cuando sea adecuado.
2. Usar runner y workflows acotados, reproducibles y con SHA exacto para cambios recurrentes autorizados.
3. Usar Desktop Commander para observación independiente y diagnósticos locales controlados.
4. Solicitar autorización y canal específico para intervenciones de escritura o recuperación.

No añadir watchdogs, schedulers, reconciliadores, reintentos automáticos o writers paralelos. Los proyectos consumidores no administran infraestructura compartida. Si una acción falla, clasificarla antes de repetirla.

## Seguridad y trazabilidad

- No recoger ni publicar tokens, contraseñas, contenido documental, rutas privadas completas o datos personales innecesarios.
- No cruzar límites de cuentas, repositorios, servicios, máquinas ni proyectos por el mero hecho de disponer de acceso técnico.
- Registrar host, momento, herramienta, alcance, resultado y evidencia redactada cuando una observación fundamente una decisión material.
- Registrar las mutaciones relevantes en el sistema autoritativo, con comprobación previa, posterior y rollback acotado cuando corresponda.
- No modificar cuotas, planes, compras ni suscripciones sin autorización financiera separada.
- Si cambian la cuota o los términos del servicio, revisar esta política expresamente; no reactivar de forma implícita umbrales históricos.

## Documentos privados y QNAP: alcance conservado

La modalidad ordinaria de observación controlada **no concede** capacidad operativa de lectura de contenido privado, creación, sobrescritura, borrado ni limpieza documental. La suscripción ilimitada comunicada por el operador no equivale a un grant de datos ni a autorización de escritura.

Para cada operación documental privada en QNAP se requiere una autorización vigente que identifique proyecto, usuario/identidad, ruta o conjunto acotado, finalidad, modalidad exacta y canal aprobado. Las lecturas se limitan al contenido necesario; las creaciones deben evitar sobrescrituras por defecto y verificar integridad. Las sobrescrituras, borrados y limpieza de temporales también son mutaciones y requieren autorización específica, evidencia y reversibilidad cuando corresponda.

No se concede administración general del NAS, modificación de ACL o credenciales, despliegues, uso de Qsync como sistema de gobierno, sincronización permanente fuera de QNAP, acceso interproyecto ni autoridad para nuevos reconciliadores. Los documentos permanentes siguen en su almacenamiento autorizado y se respeta clasificación, retención y minimización de datos.

Esta sección preserva los controles de las propuestas históricas `pjguti/.github#4` y `pjguti/github-operating-system#1263`, pero **no activa** sus excepciones de escritura ni reproduce el régimen obsoleto de hard freeze. Cada mutación seguirá requiriendo una decisión operativa separada.

## Relación con propuestas anteriores

Esta política sustituiría el régimen global de *hard freeze* cuando se apruebe. La excepción documental QNAP propuesta en `pjguti/.github#4` y su propuesta correlativa en GOS deberán reconciliarse expresamente antes de cualquier merge para evitar dos reglas contradictorias. Nada de este texto aprueba esas PR, concede grants documentales ni activa servicios.

## Vigencia y gobierno

La entrada en vigor requiere revisión transversal, validación de las pruebas y manifiestos de adopción del control plane, y merge autorizado. Hasta entonces sigue aplicándose la política actualmente fusionada en `main`.
