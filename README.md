# TresR — Arquitectura de Microservicios para Gestión de Reciclaje

## Descripción del proyecto

**TresR** es un sistema distribuido para la gestión integral del servicio de retiro y reciclaje de
materiales para empresas clientes. Permite administrar empresas, contratos de servicio, conductores,
vehículos, rutas de recolección, incidencias durante el retiro, materiales recolectados, reportes y
estadísticas del negocio.

La solución está compuesta por **10 microservicios independientes** desarrollados en Spring Boot,
comunicados entre sí mediante REST (WebClient), y centralizados a través de un **API Gateway** (Spring
Cloud Gateway).

> ⚠️ Completar antes de la entrega:
> **Integrantes del equipo:** Nombre Apellido — Nombre Apellido — Nombre Apellido

## Microservicios implementados

| # | Microservicio | Puerto | Dominio |
|---|---|---|---|
| 1 | service-empresas | 8081 | Empresas clientes y su tipo de industria |
| 2 | service-contratos | 8082 | Contratos de servicio entre TresR y la empresa cliente |
| 3 | service-recolecciones | 8083 | Registro de retiros/recolecciones realizadas |
| 4 | service-materiales | 8084 | Catálogo de materiales reciclables gestionados |
| 5 | service-reportes | 8085 | Reportes consolidados de operación |
| 6 | service-conductores | 8086 | Conductores asignados a las rutas |
| 7 | service-vehiculos | 8087 | Vehículos de la flota |
| 8 | service-rutas | 8088 | Rutas de recolección |
| 9 | service-incidencias | 8089 | Incidencias ocurridas durante el retiro |
| 10 | service-estadisticas | 8090 | Estadísticas y métricas del negocio |
| — | gateway | 9091 | API Gateway — enrutamiento centralizado |

## Rutas principales del Gateway

Todas las peticiones se realizan a través del Gateway en `http://localhost:9091`, que redirige
internamente a cada microservicio:

```
/empresas/**       → service-empresas      (8081)
/contratos/**      → service-contratos     (8082)
/recolecciones/**  → service-recolecciones (8083)
/materiales/**     → service-materiales    (8084)
/reportes/**       → service-reportes      (8085)
/conductores/**    → service-conductores   (8086)
/vehiculos/**      → service-vehiculos     (8087)
/rutas/**          → service-rutas         (8088)
/incidencias/**    → service-incidencias   (8089)
/estadisticas/**   → service-estadisticas  (8090)
```

## Documentación Swagger / OpenAPI

Cada microservicio expone su propia documentación interactiva en:

```
http://localhost:<puerto>/swagger-ui/index.html
```

Por ejemplo:
- Empresas: http://localhost:8081/swagger-ui/index.html
- Contratos: http://localhost:8082/swagger-ui/index.html
- Conductores: http://localhost:8086/swagger-ui/index.html

> Completar con el enlace remoto una vez desplegado (Railway/Render).

## Arquitectura y patrón de diseño

Cada microservicio sigue el patrón **CSR (Controller–Service–Repository/Model)**:

- `controller/` — expone los endpoints REST, solo orquesta llamadas al service.
- `service/` — concentra la lógica de negocio, validaciones y reglas del dominio.
- `repository/` — interfaces `JpaRepository` que acceden a la base de datos.
- `model/` — entidades JPA.
- `dto/` — objetos de transferencia usados en los endpoints (no se expone la entidad directamente).
- `exception/` — excepciones de dominio y manejo centralizado de errores (`GlobalExceptionHandler`).
- `config/` — configuración de Swagger (`OpenApiConfig`) y, donde aplica, del cliente HTTP
  (`WebClientConfig`) usado para comunicarse con otros microservicios.

Los servicios **estadísticas**, **recolecciones**, **reportes** y **contratos** consumen información de
otros microservicios mediante `WebClient`, resolviendo así la comunicación distribuida.

## Configuración con YAML

Cada microservicio usa `application.yml` con dos perfiles:

- **dev** (activo por defecto): apunta a MySQL local (`localhost:3306`), `ddl-auto: update`,
  `show-sql: true`. Pensado para desarrollo y ejecución desde el IDE.
- **prod**: usa variables de entorno (`DB_URL`, `DB_USERNAME`, `DB_PASSWORD`) para apuntar a la base de
  datos del entorno de despliegue (Docker/Railway/Render), `ddl-auto: validate`, `show-sql: false`.

## Pruebas unitarias

Las pruebas se ubican en `src/test/java`, usan JUnit 5 + Mockito, siguen la estructura
**Given–When–Then** y mockean el `repository` (y el `WebClient` en los servicios que consumen otros
microservicios) para no depender de una base de datos real durante la ejecución de los tests.

```bash
# Ejecutar tests de un microservicio puntual
mvn test
```

## Instrucciones de ejecución

### Ejecución local (perfil dev)

1. Tener MySQL corriendo en `localhost:3306` (las bases de datos se crean automáticamente gracias a
   `createDatabaseIfNotExist=true`).
2. Ejecutar cada microservicio desde el IDE, o:
   ```bash
   cd service-empresa/service-empresa
   mvn spring-boot:run
   ```
3. Levantar el Gateway al final:
   ```bash
   cd gateway/gateway
   mvn spring-boot:run
   ```
4. Probar a través del Gateway: `http://localhost:9091/empresas`

### Ejecución con Docker (perfil prod)

Cada microservicio incluye su propio `Dockerfile` (build multi-etapa: compila con Maven y genera una
imagen final liviana con el `.jar`).

```bash
# Ejemplo: construir y correr service-empresas
cd service-empresa/service-empresa
docker build -t tresr-empresas .
docker run -p 8081:8081 \
  -e DB_URL=jdbc:mysql://<host>:3306/db_tresr_empresas \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=<password> \
  tresr-empresas
```

### Ejecución remota (Railway / Render)

> Completar con las instrucciones específicas y los enlaces una vez desplegado cada microservicio.

## Herramientas de gestión

- **GitHub**: control de versiones, commits distribuidos por integrante.
- **Trello**: tablero de tareas y seguimiento de avance del equipo.
  > Completar con el enlace al tablero de Trello del equipo.
