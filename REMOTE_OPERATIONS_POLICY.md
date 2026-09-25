# Política global de operación remota y Desktop Commander

Vigencia: 2026-09-25  
Ámbito: repositorios y proyectos de la cuenta GitHub `pjguti`.

## Regla principal

Desktop Commander no forma parte del camino operativo ordinario.

El orden obligatorio para trabajo remoto es:

1. GitHub connector / GitHub API.
2. GitHub Actions sobre self-hosted runners ya autorizados.
3. Workflows acotados, reproducibles y auditables disparados desde GitHub.
4. APIs o conectores específicos del servicio.
5. Acción manual local mínima cuando una operación física no pueda automatizarse de forma segura.
6. Desktop Commander únicamente como contingencia de último recurso.

## Presupuesto de Desktop Commander

Desktop Commander se trata como un recurso escaso.

- consumo < 80%: no autoriza uso rutinario; sigue siendo contingencia;
- consumo >= 80%: evitar cualquier uso salvo bootstrap o diagnóstico sin alternativa;
- consumo >= 90%: **hard freeze**;
- durante hard freeze, no se permite ninguna llamada salvo emergencia crítica, ausencia demostrada de alternativa y autorización explícita del operador en ese momento.

El hecho de que Desktop Commander esté disponible técnicamente no constituye autorización de uso.

## Prohibiciones durante hard freeze

No usar Desktop Commander para:

- polling de procesos o servicios;
- leer repetidamente stdout/stderr;
- comprobar estados que GitHub Actions/API pueda mostrar;
- explorar archivos o procesos por conveniencia;
- ejecutar validaciones que puedan correr en self-hosted runners;
- reiniciar runners para acelerar una cola;
- mantener sesiones abiertas;
- sustituir logs/evidencia de GitHub;
- automatización periódica o recurrente.

## Uso excepcional

Un uso excepcional debe cumplir simultáneamente:

- objetivo crítico y concreto;
- no existe alternativa GitHub/API/self-hosted/conector;
- llamada mínima y acotada;
- sin exposición de secretos;
- autorización explícita del operador en el turno actual;
- evidencia posterior trasladada al sistema autoritativo del proyecto.

## Arquitectura preferida

Los proyectos que necesiten capacidad local o física deben migrar hacia:

- self-hosted runner dedicado con labels específicas;
- servicio persistente del runner;
- workflows con vocabulario/acciones allow-listed;
- SHA exacto;
- `persist-credentials: false` cuando aplique;
- dependencias fijadas;
- evidencia en GitHub Actions;
- sin shell remoto arbitrario;
- sin reutilizar runners de otro proyecto si viola aislamiento de cuentas/proyectos.

## Aislamiento

Cada cuenta de ChatGPT trabaja únicamente con los proyectos autorizados en esa cuenta.

No se usa la disponibilidad técnica de otro repositorio, runner, cuenta o máquina como autorización para cruzar límites de proyecto.

## Criterio de cierre

Un proyecto deja de depender de Desktop Commander cuando sus operaciones rutinarias necesarias pueden:

- iniciarse mediante GitHub/API/conector;
- ejecutarse en infraestructura autorizada;
- producir evidencia auditable sin Desktop Commander;
- recuperarse o pedir una acción manual mínima cuando falta capacidad física.

Hasta entonces, cualquier dependencia de Desktop Commander debe registrarse como deuda operativa, no como arquitectura permanente.
