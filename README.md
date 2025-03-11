# Prueba de Concepto: Kafka con Microservicios en Spring Boot 3

## Descripción
Este proyecto es una prueba de concepto para evaluar las capacidades de **Kafka** en una arquitectura de microservicios desarrollada en **Spring Boot 3**. La mensajería utiliza **Protocol Buffers (Protobuf)** como esquema de serialización y se han configurado herramientas de monitoreo con **Grafana** y **Prometheus**.

## Tecnologías Utilizadas
- **Kafka** (Mensajería)
- **Zookeeper** (Gestión de brokers de Kafka)
- **Spring Boot 3** (Microservicios)
- **Protobuf** (Serialización de mensajes)
- **Kafka UI** (Interfaz gráfica para administrar Kafka)
- **Kafka Exporter** (Métricas para Prometheus)
- **Prometheus** (Monitoreo y métricas)
- **Grafana** (Alertas y visualización de métricas)

## Arquitectura del Proyecto
![Arquitectura](https://your-architecture-diagram-link.com) *(Reemplazar con la URL de la imagen si es necesario)*

## Estructura del Proyecto
```
kafka-poc/
│── infrastructure/                # Infraestructura general
│   ├── docker-compose.yml         # Orquestación de servicios
│   ├── kafka/                     # Configuración específica de Kafka
│   │   ├── Dockerfile
│   │   ├── kafka-config.yml
│   │   ├── zookeeper-config.yml
│   ├── monitoring/                # Configuración de monitoreo
│   │   ├── prometheus.yml
│   │   ├── grafana/
│   │   │   ├── dashboards/
│   │   │   ├── datasources/
│   │   ├── kafka-exporter/
│   ├── protobuf/                   # Definiciones de Protobuf
│   │   ├── employee.proto
│   │   ├── another-message.proto
│
│── services/                       # Código de los microservicios
│   ├── producer-service/           # Productor Kafka
│   │   ├── src/main/java/...       # Código fuente
│   │   ├── src/main/proto/         # Definiciones de Protobuf (si aplica)
│   │   ├── pom.xml                 # Dependencias
│   │   ├── Dockerfile              # Imagen Docker del servicio
│   │   ├── application.yml
│   ├── consumer-service/           # Consumidor Kafka
│   │   ├── src/main/java/...       # Código fuente
│   │   ├── pom.xml                 # Dependencias
│   │   ├── Dockerfile              # Imagen Docker del servicio
│   │   ├── application.yml
│
│── api-gateway/                    # API Gateway (si aplica)
│   ├── src/main/java/...
│   ├── pom.xml
│   ├── Dockerfile
│
│── docs/                            # Documentación
│   ├── architecture.md              # Explicación de la arquitectura
│   ├── api-specs/                   # Especificaciones de API
│   ├── diagrams/                    # Diagramas de arquitectura
│
│── scripts/                         # Scripts útiles
│   ├── create-topics.sh             # Script para creación de tópicos en Kafka
│   ├── clean-containers.sh          # Limpieza de contenedores Docker
│
│── .gitignore
│── README.md
│── LICENSE
```

## Instalación y Configuración
### 1. Clonar el Repositorio
```sh
 git clone https://github.com/monghit/kafka_poc.git
 cd kafka-poc
```

### 2. Levantar el Entorno con Docker Compose
```sh
docker-compose up -d
```
Esto iniciará los siguientes servicios:
- **Zookeeper** en `localhost:10204`
- **Kafka** en `localhost:10200`
- **Kafka UI** en `http://localhost:10201`
- **Prometheus** en `http://localhost:10202`
- **Grafana** en `http://localhost:10203`

## Uso de Protobuf
### 1. Definición del Esquema (`src/main/proto/employee.proto`)
```proto
syntax = "proto3";

package mypackage;

message Employee {
  int32 id = 1;
  string name = 2;
  string email = 3;
  repeated string skills = 4;
}
```

### 2. Generación de Clases Java desde Protobuf
Ejecutar:
```sh
mvn clean compile
```
Esto generará las clases en `target/generated-sources/protobuf/`.

## Configuración de Alertas en Grafana
### 1. Configuración de Prometheus (`prometheus.yml`)
```yaml
scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka-exporter:9308']
```

### 2. Creación de Alertas en Grafana
1. Abre Grafana (`http://localhost:3000`).
2. Configura Prometheus como fuente de datos.
3. Crea un panel con la siguiente consulta:
   ```
   kafka_topic_partitions{topic="employee-topic"}
   ```
4. Define una alerta si el número de particiones cae a 0.

## Contribución
Si deseas contribuir, abre un issue o un pull request en el repositorio.

## Licencia
Este proyecto está bajo la licencia Apache 2.0.

