# Config Server

Servidor de configuración centralizada basado en **Spring Cloud Config Server**. Forma parte del ecosistema de microservicios **Fintech UTN**, actuando como fuente única de verdad para las propiedades de todos los servicios de la plataforma.

## Rol en el ecosistema

El ecosistema Fintech está compuesto por múltiples microservicios (tarjetas, cuentas, pagos, usuarios, etc.) que necesitan compartir y gestionar configuración de forma consistente entre entornos (local, dev, staging, producción). Este config server centraliza esa configuración, evitando duplicación y permitiendo cambios sin necesidad de redesplegar cada servicio.

```
                        ┌─────────────────┐
                        │  Config Server  │  :8888
                        │  (este servicio)│
                        └────────┬────────┘
                                 │ sirve configuración
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
       tarjetas-api         cuentas-api         pagos-api
```

## Tecnologías

- Java 21
- Spring Boot 4.1
- Spring Cloud Config Server 2025.x

## Configuración

El servidor lee las propiedades desde un repositorio Git. Cada microservicio tiene su propio archivo de configuración nombrado según su `spring.application.name`:

```
config-repo/
├── tarjetas-api.yml
├── cuentas-api.yml
└── pagos-api.yml
```

### `application.yaml` del config server

```yaml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: <URL del repositorio de configuraciones>
          default-label: main

server:
  port: 8888
```

## Cómo levantar

```bash
./mvnw spring-boot:run
```

El servidor queda disponible en `http://localhost:8888`.

Para verificar que está sirviendo la config de un servicio:

```
GET http://localhost:8888/tarjetas-api/default
```

## Integración con los microservicios

Cada microservicio cliente debe:

1. Agregar la dependencia `spring-cloud-starter-config` en su `pom.xml`.
2. Declarar en su `application.yml`:

```yaml
spring:
  config:
    import: optional:configserver:http://localhost:8888
```

> **Importante:** el config server debe estar levantado antes que los microservicios que dependen de él.
