# 🛠️ kafka-infra

Infraestructura dockerizada para **Apache Kafka** en entornos de desarrollo y base para producción.

## 📦 Descripción

**kafka-infra** define y gestiona un broker de Apache Kafka usando Docker y Docker Compose.  
Incluye una configuración lista para desarrollo local con un único broker que actúa también como controlador (KRaft), exponiendo los puertos necesarios para clientes y control plane.

Sirve como punto de partida para escalar a staging/producción con múltiples brokers y servicios complementarios.

---

## 🔧 Requisitos previos

- **Docker:** [Instalación](https://docs.docker.com/get-docker/)
- **Docker Compose:** Incluido en Docker Desktop (o `docker compose` plugin).
- **WSL2 en Windows:** [Guía oficial](https://learn.microsoft.com/windows/wsl/install)
- **Puertos libres:** 
  - **9092:** Tráfico desde otros contenedores.
  - **9093:** Listener del controlador.
  - **9094:** Tráfico local desde el host local.

> 💡 **Sugerencia:** verifica que no haya procesos ocupando 9092/9093/9094 antes de levantar la infraestructura.

---

## ▶️ Levantar el entorno de desarrollo

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/Miguel-Alejandro-Castillo/kafka-infra.git
   cd kafka-infra
   ```

2. **Arrancar servicios:**
   ```bash
   docker compose -f docker-compose.dev.yaml up -d
   ```

3. **Verificar el contenedor `kafka-broker`:**
   ```bash
   docker ps --filter "name=kafka-broker"
   ```
   Deberías ver el contenedor `kafka-broker` en estado **Up**.

---

## 🔌 Probar conectividad y ejecutar comandos de Kafka

- **Bootstrap server:** `localhost:9092`

1. **Abrir una shell dentro del contenedor en los binarios de Kafka:**
   ```bash
   docker exec --workdir /opt/kafka/bin/ -it kafka-broker sh
   ```

2. **Ejecutar scripts:**

   - 📌 **Crear un tópico**
     ```bash
     ./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <nombre-topic>
     ```

   - 📌 **Enviar mensajes a un tópico**
     ```bash
     ./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <nombre-topic>
     ```
     Luego escribe mensajes en consola y presiona Enter.  
     Usa `Ctrl+C` para salir del productor.

   - 📌 **Eliminar un tópico**
     ```bash
     ./kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic <nombre-topic>
     ```
     ⚠️ Esto eliminará el tópico y todos sus mensajes. No se puede deshacer.

---

## 🧃 Comandos útiles

- **Ver estado del contenedor**
  ```bash
  docker ps --filter "name=kafka-broker"
  ```

- **Detener el contenedor**
  ```bash
  docker stop kafka-broker
  ```

- **Reiniciar el contenedor**
  ```bash
  docker restart kafka-broker
  ```

- **Ver logs en tiempo real**
  ```bash
  docker logs -f kafka-broker
  ```
  Usa `Ctrl+C` para salir de la vista de logs.

- **Detener y limpiar servicios del compose**
  ```bash
  docker compose -f docker-compose.dev.yaml down
  ```

---

## ⚙️ Configuración incluida

El `docker-compose.dev.yaml` utiliza la imagen `apache/kafka:3.9.1` y define un **broker único con KRaft**:

- **Listeners:**
  - `PLAINTEXT://0.0.0.0:9092` (clientes contenedores)
  - `CONTROLLER://0.0.0.0:9093` (controlador)
  - `PLAINTEXT_LOCALHOST://0.0.0.0:9094` (clientes local)
- **Advertised listeners:**
  - `PLAINTEXT://kafka-broker:9092` (para acceso desde contenedores)
  - `PLAINTEXT_LOCALHOST://localhost:9094` (para acceso desde host)

- **Roles y KRaft:**
  ```yaml
  KAFKA_PROCESS_ROLES=broker,controller
  KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
  KAFKA_CONTROLLER_QUORUM_VOTERS="1@kafka-broker:9093"
  ```

- **Parámetros de desarrollo:**
  ```yaml
  KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
  KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1
  KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1
  KAFKA_MIN_INSYNC_REPLICAS=1
  KAFKA_AUTO_CREATE_TOPICS_ENABLE=false
  KAFKA_NUM_PARTITIONS=1
  ```

> ℹ️ La auto-creación de topics está **deshabilitada** para que la creación sea explícita desde herramientas o pipelines.

---

## 📁 Estructura del proyecto

```plaintext
kafka-infra/
├── .dockerignore
├── docker-compose.dev.yaml
├── LICENSE
├── README.Docker.md
└── README.md
```

A medida que el proyecto crezca, se pueden añadir:

- `docker-compose.prod.yaml`
- Archivos de configuración adicionales
- Directorios para scripts y utilidades

---

## 🧱 Próximos pasos

- **Múltiples brokers:** Definir 3 nodos con KRaft y `KAFKA_CONTROLLER_QUORUM_VOTERS` ajustado.  
- **Herramientas de observabilidad:** Agregar **Kafdrop / Kafka UI** para inspeccionar topics y consumer groups.  
- **Schema Registry:** Integrar **Confluent Schema Registry** si usas Avro/Protobuf/JSON Schema.  
- **Seguridad:** Habilitar **SASL/SSL** para autenticación y cifrado en entornos no locales.  
- **Producción:** Crear `docker-compose.prod.yaml` con:
  - Volúmenes persistentes mapeados a discos dedicados.
  - Recursos limitados.
  - Redes separadas para clientes y control plane.
  - Healthchecks y políticas de reinicio.

---

## ❓ Preguntas frecuentes

**❓ No puedo conectarme a `localhost:9092`.**  
✔️ Verifica que el contenedor esté **Up**.  
✔️ En **WSL2**, accede desde Windows a `localhost:9092`.  
✔️ Si el cliente corre en otra red/host, actualiza `KAFKA_ADVERTISED_LISTENERS` a una IP reachable por el cliente.

**❓ Quiero crear o eliminar topics manualmente.**  
✔️ Usa los comandos `./kafka-topics.sh` dentro del contenedor como se indica en la sección de pruebas.

---

## 📜 Licencia

MIT. Puedes usar, modificar y distribuir este proyecto libremente.

---
## 🚀 ¡Gracias por usar este proyecto!
Esperamos que esta configuración te sea útil para tu desarrollo
