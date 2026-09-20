# Registro de defectos y resultados

Curso: Testing y Validacion de Software  
Proyecto: Pruebas de Carga y Rendimiento  
Fecha: 2026-09-17

## Criterios

- p95 de latencia: <= 300 ms
- p99 de latencia: <= 800 ms
- Errores de negocio: < 1%

## Resultado PERF-01: estado contaminado entre corridas

- **Escenario:** baseline repetido sin reiniciar el servicio.
- **Resultado esperado:** respuestas `VALID` para IDs nuevos.
- **Resultado obtenido:** respuestas `DUPLICATED` para IDs ya registrados.
- **Causa confirmada:** la base H2 permanece viva mientras el proceso continua ejecutandose.
- **Impacto:** la prueba reporta errores de negocio aunque la API funcione correctamente.
- **Correccion:** reiniciar el servicio entre corridas o desplazar el rango de IDs con `ID_BASE`.
- **Estado:** Resuelto.

## Resultado PERF-02: carga post-correccion

- **Escenario:** 50 VUs durante 30 segundos.
- **Peticiones:** 47.033.
- **p95:** 73,96 ms.
- **p99:** inferior a 800 ms.
- **Errores HTTP:** 0.
- **Errores de negocio:** 0.
- **Resultado:** cumple el SLO de latencia y el criterio de negocio.
- **Estado:** Validado.

## Evidencia

Resumen generado en `perf/results/summary-load-after-pool-valid.json`.

La ejecucion se realizo con HikariCP activo. El log del servicio confirma el arranque de `HikariPool-1` y `HikariPool-2`, y el endpoint `/actuator/health` respondio con estado `UP` antes de iniciar la carga.

## Resultado PERF-03: estrés no sostenible bajo 600 VUs

- **Escenario:** estrés progresivo de 200 a 600 VUs.
- **Resultado obtenido:** el cliente registró `connection refused` y `request timeout` durante la rampa.
- **Interpretación:** la ejecución no cumple el criterio de errores menor al 1% y no debe usarse como medición de latencia válida.
- **Causa investigada:** el servicio pierde disponibilidad bajo la presión máxima; el pool Hikari fue ampliado de 20 a 80 conexiones como corrección inicial.
- **Estado:** Reproducido, corrección en validación.

La prueba pequeña posterior al cambio del pool continuó respondiendo correctamente. Se requieren mediciones adicionales con observación de CPU, hilos y conexiones para aislar el límite de saturación antes de declarar resuelto PERF-03.
