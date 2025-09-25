# 🐳 README.docker.md — Guía Docker para Desarrollo Local

## 📦 Requisitos

- Docker instalado → [Instalar Docker](https://docs.docker.com/get-docker/)
- Docker Compose v2+
- Acceso a terminal (CMD, PowerShell, Git Bash, etc.)

---

## ⚙️ Comandos estándar para desarrollo

### 🔨 Crear infraestructura y levantar contenedores

```bash
docker compose -f docker-compose.dev.yaml up --build
```

Construye las imágenes desde cero y levanta los servicios definidos en `docker-compose.dev.yaml`. El flag `--build` permite ver todo el proceso en consola, ideal para desarrollo.

---

### 🛑 Parar y eliminar contenedores

```bash
docker compose -f docker-compose.dev.yaml down
```

Detiene los servicios y elimina contenedores, redes y volúmenes definidos.

---

### 🛑 Parar los servicios (sin eliminar contenedores)

```bash
docker compose -f docker-compose.dev.yaml stop
```

Detiene los contenedores pero conserva su estado. Útil para pausas temporales sin perder datos.

---

### 🚀 Reanudar servicios detenidos

```bash
docker compose -f docker-compose.dev.yaml start
```

Reactiva los contenedores previamente detenidos con `stop`.

---

### 🔁 Reiniciar servicios

```bash
docker compose -f docker-compose.dev.yaml restart
```

Reinicia los contenedores sin reconstruir las imágenes.

---

### 📜 Ver logs en tiempo real

```bash
docker compose -f docker-compose.dev.yaml logs -f
```

Muestra los logs de todos los servicios en tiempo real.

---

### 📥 Acceder a un contenedor

```bash
docker exec -it <nombre-del-contenedor> sh
```

Ejemplo: `docker exec -it kafka-broker sh`  
Usamos `sh` por compatibilidad con imágenes livianas como Alpine.

---

### 📋 Listar contenedores activos

```bash
docker ps
```

---

### 🧼 Limpiar contenedores, volúmenes y redes no utilizados

```bash
docker system prune -a
```

⚠️ Elimina todo lo que no esté en uso. Usar con precaución.

---

## 📁 Estructura recomendada del proyecto

```
.
├── docker-compose.dev.yaml
├── Dockerfile.dev
├── README.Docker.md
└── .dockerignore
```

---

## 🧠 Buenas prácticas

- Usá `.env` para variables sensibles o configurables.
- Nombrá los servicios claramente en `docker-compose.dev.yaml`.
- Documentá los puertos expuestos y redes utilizadas.
- Mantené este README actualizado con comandos específicos si el proyecto lo requiere.
