# Guía de Estudio Completa para Parcial de CI/CD

> Basada en los procesos aplicados en este repositorio (`Laboratorio-CI-CD`), su flujo de ramas, sus pipelines de GitHub Actions, cobertura, SonarCloud y calidad de código.

---

## 1) Fundamentos: ¿Qué es CI/CD?

## CI (Continuous Integration)
**Definición:** práctica donde cada cambio pequeño de código se integra frecuentemente al repositorio y se valida automáticamente (lint + tests + build).  
**Objetivo:** detectar errores temprano.

### Tipos de integración en CI
- **Por Pull Request (PR-based CI):** se ejecuta al abrir/actualizar un PR.  
  - *Ejemplo del repo:* `ci-develop.yml` y `ci-master.yml`.
- **Por push directo:** se ejecuta al hacer push a una rama.
- **Programada (schedule):** ejecuciones periódicas (ej. nightly).

## CD (Continuous Delivery / Continuous Deployment)
**Definición general:** extensión de CI para preparar o publicar software de forma continua.

### Tipos de CD
- **Continuous Delivery (Entrega Continua):** el sistema queda siempre listo para desplegar, pero la publicación final puede ser manual.
- **Continuous Deployment (Despliegue Continuo):** todo cambio que pasa el pipeline se despliega automáticamente.

### Diferencia clave CI vs CD
- **CI valida calidad técnica del cambio.**
- **CD mueve cambios hacia ambientes de entrega/producción.**

---

## 2) Objetivos reales de CI/CD en equipos

- Reducir fallos en producción.
- Detectar errores de forma temprana y barata.
- Tener trazabilidad de cada cambio.
- Estandarizar calidad (mismas reglas para todos).
- Acelerar entregas sin perder control.

### Beneficios medibles
- Menor tiempo para detectar fallos.
- Menor tasa de regresiones.
- Menor “deuda invisible” (errores de estilo/calidad).
- Mayor confianza para liberar versiones.

---

## 3) Flujo de trabajo con Git y PRs (Git Flow del laboratorio)

## Ramas usadas
- **`develop`**: integración diaria y validación rápida.
- **`master` / `main`**: integración estricta con reglas de cobertura y calidad.
- **`feature/*`**: trabajo aislado por funcionalidad.

### ¿Por qué separar `develop` de `master`?
- Permite iterar rápido en `develop`.
- Exige estándares más duros antes de entrar a `master`.
- Disminuye riesgo de romper la rama estable.

### Flujo esperado
1. Crear rama `feature/*` desde `develop`.
2. Hacer cambios y abrir PR a `develop` (pipeline rápido).
3. Integrar en `develop`.
4. Abrir PR `develop -> master` (pipeline estricto).

---

## 4) GitHub Actions: motor de automatización del repo

## Conceptos principales
- **Workflow:** archivo YAML con automatización.
- **Evento (`on`):** disparador (ej. `pull_request`).
- **Job:** bloque de trabajo (corre en runner).
- **Step:** paso individual dentro de un job.
- **Runner:** máquina donde se ejecuta el pipeline.
- **Artifact:** archivo generado por un job para usar en otro.
- **`needs`:** dependencias entre jobs.

### Tipos de workflows en este repo

## A) `ci-develop.yml` (rápido)
Se ejecuta para PR con destino `develop` y valida:
- Lint backend con Ruff.
- Lint frontend con ESLint.
- Tests backend con pytest (sin gate duro de cobertura).
- Build frontend con Vite.

**Propósito:** feedback rápido para desarrollo diario.

## B) `ci-master.yml` (estricto)
Se ejecuta para PR con destino `master/main` y valida:
- Lint completo backend + frontend.
- Tests backend con cobertura mínima (>=80%).
- Tests frontend con cobertura.
- SonarCloud Quality Gate.
- Build de imágenes Docker.

**Propósito:** bloquear integración de cambios que no cumplan calidad global.

---

## 5) Análisis estático (Linters)

## ¿Qué es?
Revisión automática de código sin ejecutar la aplicación.

### Objetivos
- Detectar errores frecuentes.
- Uniformar estilo.
- Mejorar mantenibilidad.

### Tipos de análisis estático
- **Estilo y sintaxis:** formato, convenciones.
- **Bugs probables:** patrones peligrosos.
- **Buenas prácticas:** importaciones, complejidad, etc.

### Herramientas del laboratorio
- **Ruff (Python):** backend.
- **ESLint (TypeScript/React):** frontend.

### Riesgos comunes
- Ignorar advertencias “pequeñas”.
- Desactivar reglas sin justificación.
- Tratar lint como opcional (debe ser condición de merge).

---

## 6) Pruebas automatizadas

## ¿Qué son?
Validaciones automáticas del comportamiento del sistema.

### Tipos de pruebas (pirámide)
- **Unitarias:** prueban funciones/componentes aislados.
- **Integración:** prueban interacción entre módulos.
- **End-to-end (E2E):** prueban flujo completo de usuario.

### Herramientas del laboratorio
- **Backend:** `pytest`.
- **Frontend:** `Vitest` + Testing Library.

### Buenas prácticas
- Pruebas pequeñas, claras y repetibles.
- Un caso por escenario crítico.
- Cubrir caminos felices y de error.
- Ejecutarlas en cada PR.

### Errores frecuentes
- Tests frágiles acoplados a implementación interna.
- Falsa sensación de seguridad por tener “muchos” tests malos.
- No revisar tests cuando cambia lógica de negocio.

---

## 7) Cobertura de código

## ¿Qué mide?
Porcentaje de líneas/ramas ejecutadas por los tests.

### Tipos comunes de cobertura
- **Cobertura de líneas.**
- **Cobertura de ramas/condiciones.**
- **Cobertura de funciones.**

### Herramientas del repo
- **Backend:** `pytest-cov` genera `coverage.xml`.
- **Frontend:** Vitest genera reportes (ej. `lcov`).

### Gate de cobertura
- En este laboratorio se usa **umbral de 80%** como criterio de calidad para pasar a `master`.

### Importante para examen
- Cobertura alta **no garantiza** tests buenos.
- Cobertura baja suele indicar zonas sin validar.
- El valor correcto depende del contexto, pero debe haber un umbral explícito.

---

## 8) SonarCloud y Quality Gate

## ¿Qué hace SonarCloud?
Plataforma de análisis continuo de calidad y seguridad del código.

### Qué evalúa
- Bugs potenciales.
- Vulnerabilidades y *security hotspots*.
- Código duplicado.
- Métricas de mantenibilidad.
- Cobertura reportada por herramientas de test.

### Quality Gate
Conjunto de condiciones para aprobar/rechazar calidad.

### Tipos de condiciones de un Quality Gate
- Cobertura mínima.
- Límites de duplicación.
- Ausencia de issues críticos/bloqueantes.
- Reglas de “nuevo código” (new code period).

### En el flujo del repo
- Se descargan artifacts de cobertura.
- Sonar analiza backend y frontend.
- Si el gate falla, el PR a `master` debe bloquearse.

---

## 9) Artefactos en pipelines

## ¿Qué son?
Archivos producidos por jobs para preservar evidencia o compartir resultados.

### Tipos de artifacts usados
- `coverage-backend` (`backend/coverage.xml`)
- `coverage-frontend` (`frontend/coverage/`)

### ¿Para qué sirven aquí?
- Transferir resultados de cobertura entre jobs.
- Permitir que SonarCloud lea reportes generados antes.
- Tener evidencia auditable del pipeline.

---

## 10) Branch Protection

## ¿Qué es?
Regla de repositorio para proteger ramas críticas (`master/main`, `develop`).

### Tipos de protecciones comunes
- Requerir PR antes de merge.
- Requerir checks de CI en verde.
- Requerir revisiones de código.
- Bloquear force push.
- Restringir quién puede hacer push directo.

### Valor en CI/CD
- Convierte buenas prácticas en reglas técnicas.
- Evita saltarse validaciones por error o apuro.

---

## 11) Docker en CI/CD

## Rol en este laboratorio
- Validar que backend y frontend pueden construirse en contenedores.
- Detectar fallos de empaquetado antes de despliegue real.

### Tipos de uso de Docker en pipelines
- **Build-only:** solo construir imagen (como en este repo).
- **Build + push:** publicar imagen en registry.
- **Build + deploy:** desplegar automáticamente.

### Beneficios
- Entornos consistentes.
- Menos “en mi máquina funciona”.
- Base para CD real a staging/producción.

---

## 12) Recorrido por etapas del laboratorio (clave para parcial)

## Etapa 1: setup inicial
- Proyecto base funcionando.
- Pipelines activos.
- Validación de calidad inicial.

## Etapa 2: CRUD sin tests extra
- Sube funcionalidad.
- Cobertura puede quedar por debajo del umbral.
- Resultado esperado: PR a `master` falla gate de calidad.

## Etapa 3: agregar tests faltantes
- Se cubren casos no probados.
- Cobertura supera el umbral.
- Resultado esperado: pipeline estricto vuelve a pasar.

### Aprendizaje central de las etapas
No basta con “que funcione”; también debe cumplir calidad automatizada para integrarse a rama estable.

---

## 13) Fallas típicas y cómo diagnosticarlas

## A) Fallo en lint
- Revisar logs del job.
- Corregir reglas Ruff/ESLint.
- Re-ejecutar localmente antes de pushear.

## B) Fallo en tests
- Identificar prueba exacta rota.
- Revisar cambios recientes y datos de prueba.
- Reparar comportamiento o ajustar test correctamente.

## C) Fallo en cobertura
- Detectar archivos no cubiertos.
- Agregar pruebas a casos críticos faltantes.
- Evitar “tests vacíos” solo para subir porcentaje.

## D) Fallo en SonarCloud
- Revisar issues bloqueantes/críticos.
- Corregir hotspots relevantes.
- Verificar rutas correctas de reportes de cobertura.

## E) Fallo en build Docker
- Revisar Dockerfile/dependencias.
- Verificar contexto y rutas copiadas.
- Probar `docker compose build` local.

---

## 14) Comandos de estudio/práctica más importantes

```bash
# Backend
cd backend
ruff check app
pytest -q
pytest --cov=app --cov-report=xml --cov-report=term --cov-fail-under=80

# Frontend
cd frontend
npm install --no-audit --no-fund
npm run lint
npm run test
npm run test:coverage
npm run build

# Docker
docker compose build
```

---

## 15) Glosario rápido para examen

- **Pipeline:** cadena automática de validaciones.
- **Runner:** entorno donde corre CI.
- **Check:** resultado de validación en PR.
- **Gate:** condición obligatoria para aprobar.
- **Artifact:** archivo generado por CI para consumo posterior.
- **Static analysis:** análisis sin ejecutar el programa.
- **Coverage:** proporción de código ejercitado por tests.
- **Quality Gate:** reglas mínimas de calidad para aceptar cambios.
- **Shift Left:** mover calidad y pruebas a etapas tempranas.

---

## 16) Preguntas de autoevaluación (tipo parcial)

1. ¿Cuál es la diferencia entre Continuous Delivery y Continuous Deployment?  
2. ¿Por qué un PR puede pasar en `develop` y fallar en `master`?  
3. ¿Qué problemas detecta un linter que no detecta un test?  
4. ¿Por qué 100% de cobertura no garantiza ausencia de bugs?  
5. ¿Qué rol cumplen los artifacts en el job de SonarCloud?  
6. ¿Qué protecciones mínimas aplicarías a `master` en un equipo real?  
7. ¿Qué significa `needs` en GitHub Actions y cuándo usarlo?  
8. ¿Qué evidencia usarías para justificar que un cambio está listo para merge?  

---

## 17) Checklist final antes del parcial

- [ ] Puedo explicar CI y CD con ejemplos del laboratorio.  
- [ ] Entiendo el flujo `feature -> develop -> master`.  
- [ ] Sé leer un workflow YAML (on, jobs, steps, needs).  
- [ ] Sé diferenciar lint, test, cobertura y análisis Sonar.  
- [ ] Entiendo por qué existe un umbral de cobertura (80%).  
- [ ] Sé interpretar por qué falla una etapa del pipeline.  
- [ ] Puedo argumentar por qué Branch Protection es necesaria.  
- [ ] Puedo describir qué aprendí en Etapa 1, 2 y 3.  

---

## 18) Resumen final (idea central del parcial)

CI/CD no es solo “automatizar comandos”; es un **sistema de control de calidad continuo**.  
En este laboratorio, la combinación de **Git Flow + GitHub Actions + linters + tests + cobertura + SonarCloud + Branch Protection** demuestra cómo un equipo puede integrar cambios rápido en `develop`, pero exigir calidad estricta antes de consolidar en `master`.

