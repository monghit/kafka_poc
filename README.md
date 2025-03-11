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

## Instalación y Configuración
### 1. Clonar el Repositorio
```sh
 git clone https://github.com/tu-repo/kafka-poc.git
 cd kafka-poc
```

### 2. Levantar el Entorno con Docker Compose
```sh
docker-compose up -d
```
Esto iniciará los siguientes servicios:
- **Zookeeper** en `localhost:2181`
- **Kafka** en `localhost:9092`
- **Kafka UI** en `http://localhost:8080`
- **Prometheus** en `http://localhost:9090`
- **Grafana** en `http://localhost:3000`

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

## Configuración de Kafka con Spring Boot
### 1. Dependencias en `pom.xml`
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
</dependency>
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-protobuf-serializer</artifactId>
    <version>7.0.1</version>
</dependency>
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
</dependency>
```

### 2. Configuración de Kafka en `application.yml`
```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: io.confluent.kafka.serializers.protobuf.KafkaProtobufSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: io.confluent.kafka.serializers.protobuf.KafkaProtobufDeserializer
      properties:
        specific.protobuf.value.type: com.tu.paquete.Employee
```

## Implementación de un Productor en Spring Boot
```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;

public class KafkaProtobufProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "io.confluent.kafka.serializers.protobuf.KafkaProtobufSerializer");

        KafkaProducer<String, EmployeeProto.Employee> producer = new KafkaProducer<>(props);
        
        EmployeeProto.Employee employee = EmployeeProto.Employee.newBuilder()
                .setId(1)
                .setName("José")
                .setEmail("jose@example.com")
                .addSkills("Java")
                .build();

        producer.send(new ProducerRecord<>("employee-topic", "key1", employee));
        producer.close();
    }
}
```

## Implementación de un Consumidor en Spring Boot
```java
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import java.util.Collections;
import java.util.Properties;

public class KafkaProtobufConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "employee-consumer-group");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "io.confluent.kafka.serializers.protobuf.KafkaProtobufDeserializer");
        props.put("specific.protobuf.value.type", EmployeeProto.Employee.class.getName());

        KafkaConsumer<String, EmployeeProto.Employee> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("employee-topic"));

        while (true) {
            ConsumerRecords<String, EmployeeProto.Employee> records = consumer.poll(100);
            records.forEach(record -> {
                System.out.println("Empleado recibido: " + record.value().getName());
            });
        }
    }
}
```

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
Este proyecto está bajo la licencia Apache 2.0

