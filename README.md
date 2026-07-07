# 📆 ReservasEC

**ReservasEC** es una plataforma fullstack de gestión de reservas desarrollada con una arquitectura de microservicios. Permite a los usuarios registrarse, iniciar sesión, gestionar su perfil, crear y cancelar reservas, y recibir notificaciones. El sistema está dockerizado para facilitar el despliegue local.

## 🚀 Tecnologías principales

- **Frontend:** Next.js + Tailwind CSS
- **Backend (Microservicios):**
  - Auth Service (Node.js + Express)
  - Booking Service (Node.js + Express)
  - User Service (Node.js + Express)
  - Notification Service (Node.js + Express + Nodemailer)
- **Base de datos:** MongoDB
- **Autenticación:** JSON Web Tokens (JWT)
- **Contenedores:** Docker + Docker Compose

---

## 📁 Estructura de carpetas

```plaintext
/reservas-ec
├── frontend/             # Next.js App
├── auth-service/         # Servicio de autenticación
├── user-service/         # Servicio de usuarios
├── booking-service/      # Servicio de reservas
├── notification-service/ # Servicio de notificaciones por email
└── docker-compose.yml    # Orquestación de todos los servicios
```

---

## ⚙️ Configuración del entorno

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/reservas-ec.git
cd reservas-ec
```

### 2. Variables de entorno

🔐 Frontend (frontend/.env.production.local)

```bash
NEXT_PUBLIC_API_URL=/api/auth
NEXT_PUBLIC_BOOKING_URL=/api/bookings
NEXT_PUBLIC_USER_URL=/api/users
```

🔐 Backend .env (cada microservicio)
Ejemplo para auth-service:

```bash
PORT=4000
MONGO_URI=mongodb://mongo:27017/auth-db
JWT_SECRET=supersecretkey
```

Repite para los demás servicios cambiando PORT, MONGO_URI y usando el mismo JWT_SECRET.

### 3. 🐳 Uso con Docker

1. Construir los contenedores

```bash
docker-compose build
```

3. Levantar los servicios

```bash
docker-compose up
```

La app estará disponible en http://localhost:3000

## ✅ Funcionalidades principales

- Registro e inicio de sesión de usuarios

- Perfil editable

- Creación y cancelación de reservas

- Historial de reservas activas y canceladas

- Límite de 5 reservas canceladas visibles

- Notificaciones por email (reserva y cancelación)

- Gestión de microservicios independientes

---

## 🧪 Calidad de código con SonarQube

### Levantar SonarQube en local

El proyecto usa SonarQube Community Edition. La forma más simple de levantarlo es con Docker:

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  sonarqube:community
```

1. Abrir [http://localhost:9000](http://localhost:9000).
2. Iniciar sesión con el usuario por defecto (`admin` / `admin`) y cambiar la contraseña cuando lo pida.
3. Generar un token de análisis en **My Account → Security → Generate Tokens**. Este token se usa como `SONAR_TOKEN` (ver sección de secrets más abajo) y nunca debe subirse al repositorio.

### Quality Gate: StrictGate

Cada proyecto (uno por microservicio) usa el Quality Gate personalizado **StrictGate**, exportado en [`qualitygate.json`](qualitygate.json). Un análisis se considera exitoso solo si se cumplen todas estas condiciones:

| Métrica                       | Condición        | Umbral |
|--------------------------------|------------------|--------|
| Blocker Issues                 | is greater than  | 0      |
| Critical / High Issues         | is greater than  | 0      |
| Major / Medium Issues          | is greater than  | 5      |
| Security Hotspots Reviewed     | is less than     | 100%   |
| Coverage                       | is less than     | 80%    |
| Duplicated Lines (%)           | is greater than  | 3%     |
| Technical Debt Ratio           | is greater than  | 2.5%   |
| Cyclomatic Complexity (total)  | is greater than  | 50     |
| Cognitive Complexity (total)   | is greater than  | 30     |

> Nota: SonarQube renombró algunas severidades a su modelo "Software Quality" (Blocker/High/Medium/Low), por lo que en la UI pueden verse como *High Issues* en vez de *Critical Issues*, pero corresponden a las mismas condiciones definidas en `qualitygate.json`.

Para asignar StrictGate a un proyecto: **Quality Gates → StrictGate → Projects → agregar el proyecto**.

### Ejecutar el análisis manualmente

Cada microservicio tiene su propio `sonar-project.properties` en la raíz de su carpeta. Para analizar un servicio en particular:

```bash
cd auth-service   # o booking-service, user-service, notification-service, frontend
sonar-scanner \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=<TU_TOKEN_DE_SONARQUBE>
```

Si no tienes `sonar-scanner` instalado localmente, puedes usar la imagen oficial sin instalar nada:

```bash
docker run --rm \
  -e SONAR_HOST_URL=http://localhost:9000 \
  -e SONAR_TOKEN=<TU_TOKEN_DE_SONARQUBE> \
  -v "$(pwd):/usr/src" \
  sonarsource/sonar-scanner-cli
```

Este mismo análisis se ejecuta automáticamente en CI vía [`.github/workflows/sonarqube.yml`](.github/workflows/sonarqube.yml) en cada push/PR a `main` y `develop`, y falla el pipeline si `StrictGate` no se cumple (`sonar.qualitygate.wait=true`).

---

## 🤖 Bot de Telegram (notificaciones de commits)

Cada push a `main` o `develop` dispara [`.github/workflows/telegram-notify.yml`](.github/workflows/telegram-notify.yml), que envía al grupo del equipo: autor del commit, rama, archivos modificados y el link al commit en GitHub.

### Configuración (sin exponer tokens)

1. Crear el bot con **@BotFather** en Telegram (`/newbot`) y guardar el token que entrega — **nunca compartirlo en capturas, commits ni documentación**.
2. Crear un grupo de Telegram, añadir el bot y enviar un mensaje cualquiera.
3. Obtener el `chat_id` del grupo consultando `https://api.telegram.org/bot<TOKEN>/getUpdates`.
4. Guardar ambos valores como **secrets** del repositorio en GitHub (ver siguiente sección), nunca en el código:
   - `TELEGRAM_BOT_TOKEN`
   - `TELEGRAM_CHAT_ID`

Si el token llega a exponerse accidentalmente (por ejemplo en una captura de pantalla), debe revocarse de inmediato desde BotFather (`/mybots` → bot → **API Token** → **Revoke current token**) y reemplazarse en los secrets.

### Secrets necesarios en GitHub

En **Settings → Secrets and variables → Actions** del repositorio, configurar:

| Secret              | Descripción                                      |
|---------------------|---------------------------------------------------|
| `SONAR_TOKEN`        | Token generado en SonarQube (My Account → Security) |
| `SONAR_HOST_URL`     | URL donde corre SonarQube (ej. la del runner local) |
| `TELEGRAM_BOT_TOKEN` | Token del bot obtenido de BotFather                |
| `TELEGRAM_CHAT_ID`   | Chat ID del grupo de Telegram del equipo           |

---

## 👥 Roles del equipo

| Rol                | Responsable       | Responsabilidad                                                                 |
|--------------------|-------------------|----------------------------------------------------------------------------------|
| Líder de calidad   | Sebastián Parra   | Configurar SonarQube y definir el Quality Gate (`StrictGate`).                   |
| DevOps             | Sebastián Parra   | Mantener los pipelines de CI/CD y la integración con Telegram.                   |
| Desarrollador      | Sebastián Parra   | Corregir el código para cumplir los umbrales de calidad definidos.               |

> Proyecto individual: los tres roles son asumidos por el mismo integrante.
