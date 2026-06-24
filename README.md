# BDII — Sistema de Ticketing Mundial 2026

Sistema integral de **ticketing** para la compra, transferencia y validación de entradas de los partidos del Mundial 2026.

Este repositorio es el **proyecto raíz** y agrupa las dos partes del sistema, cada una en su propio repositorio:

| Parte                       | Repositorio                                                            | Tecnología                     |
|-----------------------------|-----------------------------------------------------------------------|--------------------------------|
| **Backend**                 | https://github.com/agustingarciaa/BDII_Obligatorio_Ticketing_Backend  | NestJS + MySQL (Docker)        |
| **Frontend** (web + mobile) | https://github.com/agustingarciaa/BDII_Obligatorio_Ticketing_Frontend | Next.js (web) · Expo (mobile)  |

> **Cloná los dos repositorios por separado** con los links de arriba (sección 2). **No** uses `--recurse-submodules` sobre este repo: los submódulos pueden apuntar a una versión anterior. La última versión está siempre en la rama `main` de cada repositorio.

---
## Aclaración
La **web** (`frontend_ticketing`) sirve a **cliente y administrador** (redirige según el rol al iniciar sesión). El **funcionario NO entra por la web** — la web lo rechaza con el mensaje *"Los funcionarios de validación ingresan desde la app móvil"*. El funcionario opera **solo desde la app móvil** (`mobile_ticketing`).

---

## Requisitos previos

- **Docker Desktop** (incluye Docker Compose v2)
- **Node.js 20+** y **npm** (para la app móvil / Expo) — probado con Node 22
- **Git**

---

## 2. Clonar los dos repositorios

```bash
git clone https://github.com/agustingarciaa/BDII_Obligatorio_Ticketing_Backend.git
git clone https://github.com/agustingarciaa/BDII_Obligatorio_Ticketing_Frontend.git
```

---

## 3. Configurar el backend (`.env`)

Dentro de **`BDII_Obligatorio_Ticketing_Backend/`** creá un archivo **`.env`** con este contenido (credenciales de desarrollo, no sensibles):

```env
# App
PORT=3000
ALLOWED_ORIGINS=http://localhost:5173,http://localhost:3001
JWT_SECRET=cambia_esto_en_produccion

# Database
DB_HOST=db
DB_PORT=3306
DB_NAME=ticketing_db
MYSQL_ROOT_PASSWORD=root_pass

# Conexión usada por el backend y por docker-compose
DB_USER=ticketing_sistema
DB_PASSWORD=sistema_pass

# Usuarios de base de datos (deben coincidir con los de init.sql)
DB_SISTEMA_PASSWORD=sistema_pass
DB_ADMIN_PASSWORD=admin_pass
DB_USUARIO_PASSWORD=usuario_pass
DB_FUNCIONARIO_PASSWORD=funcionario_pass
```

---

## 4. Levantar el backend (Docker)

```bash
cd BDII_Obligatorio_Ticketing_Backend
docker compose up --build -d
```

Levanta tres servicios:

| Servicio  | Qué hace                                                          |
|-----------|-------------------------------------------------------------------|
| `db`      | MySQL 8.4. Crea el esquema y los usuarios a partir de `init.sql`  |
| `backend` | La API NestJS en **http://localhost:3000**                        |
| `seeder`  | **Carga datos de prueba automáticamente la primera vez**          |


Verificar:
```bash
docker compose logs seeder        # debería terminar con "Hecho en X s"
docker compose ps                 # db y backend en estado "running"
```

---

## 5. Levantar el frontend web (Docker) — Cliente + Administrador

En **otra terminal**:

```bash
cd BDII_Obligatorio_Ticketing_Frontend
docker compose up --build -d
```

La web queda en **http://localhost:3001** y apunta por defecto a la API en `http://localhost:3000`.
Es la **misma app** para cliente y administrador: al loguearte, te redirige a tu panel según el rol.

---

## 6. Usuarios de prueba

El seeder carga datos **con usuarios listos para usar**.

**Contraseña de todos los usuarios de prueba: `Password123`**

| Rol                            | Email de ejemplo       | Entra por      | Notas                                       |
|--------------------------------|------------------------|----------------|---------------------------------------------|
| **Administrador por sede**     | `bulk_admin0@test.com` | **Web**        | Jurisdicción **Estados Unidos**             |
|                                | `bulk_admin1@test.com` | **Web**        | Jurisdicción **Canadá**                     |
|                                | `bulk_admin2@test.com` | **Web**        | Jurisdicción **México**                     |
| **Cliente (usuario general)**  | `bulk_cli0@test.com`   | **Web**        | 300 clientes: `bulk_cli0` … `bulk_cli299`   |
| **Funcionario de validación**  | `bulk_fun0@test.com`   | **App móvil**  | 16 funcionarios: `bulk_fun0` … `bulk_fun15` |

> También podés **registrar tu propio cliente** desde la pantalla de registro de la web.

---

## 8. App móvil — Funcionario de validación (emulador de Android Studio)

El escaneo del **QR en puerta** se hace desde la **app móvil** (Expo), que vive en `mobile_ticketing/`. Para la corrección se levanta en el **emulador de Android** de Android Studio (no hace falta un celular físico).

> 🔑 **Detalle clave de red:** dentro del emulador de Android, `localhost` apunta al **propio emulador**, no a tu computadora. Para llegar al backend (que corre en Docker en tu compu) se usa la IP especial **`10.0.2.2`**, que el emulador mapea al `localhost` del host.

### Requisitos
- El **backend levantado** (sección 4).
- **Node.js 20+**.
- **Android Studio** instalado, con un **emulador (AVD)** creado.
  - En Android Studio: *More Actions → Virtual Device Manager → Create Device* (cualquier teléfono, ej. Pixel, con una imagen de sistema reciente).

### 8.1. Configurá la URL de la API

Dentro de **`mobile_ticketing/`** creá un archivo **`.env`** con la IP del host vista desde el emulador:

```env
EXPO_PUBLIC_API_URL=http://10.0.2.2:3000
```

### 8.2. Instalá las dependencias

```bash
cd BDII_Obligatorio_Ticketing_Frontend/mobile_ticketing
npm install
```

### 8.3. Arrancá el emulador

Abrí **Android Studio → Virtual Device Manager** y dale ▶️ a tu emulador (o desde la terminal, `emulator -avd <nombre_del_avd>`). Esperá a que cargue el escritorio de Android.

### 8.4. Levantá la app en el emulador

Con el emulador ya abierto:

```bash
npx expo start -c
```
Cuando arranque, presioná **`a`** en la terminal (o corré directamente `npm run android`). Expo instala/abre **Expo Go** en el emulador y carga la app.

> El `-c` limpia la caché. Es **necesario** cada vez que cambies el `.env`, porque las variables `EXPO_PUBLIC_*` se fijan al arrancar.

### 8.5. Usar la app
Logueate como **funcionario** (`bulk_fun0@test.com` / `Password123`):
- **Mis sectores:** sectores-partido a los que está asignado.
- **Escanear QR:** validar el QR del cliente en puerta.

## 🔌 Puertos

| Servicio              | URL / Puerto          |
|-----------------------|-----------------------|
| Frontend web          | http://localhost:3001 |
| Backend (API)         | http://localhost:3000 |
| MySQL (desde el host) | `localhost:3307`      |
| App móvil (Expo)      | se abre en Expo Go    |

### Acceso directo a la base (opcional, ej. DataGrip)
- **Host:** `localhost` · **Puerto:** `3307`
- **Usuario:** `ticketing_sistema` · **Contraseña:** `sistema_pass` (acceso completo) — o `root` / `root_pass`

---

## 🧪 Scripts de prueba (opcionales)

Desde **`BDII_Obligatorio_Ticketing_Backend/backend_ticketing/`**, con el backend levantado:

```bash
node scripts/test-validacion.mjs   # test de validación de entrada (caso feliz + casos de error)
node scripts/seed-masivo.mjs       # regenera el set de datos de prueba
```

---

## 🔄 Reiniciar / regenerar datos

```bash
# (en BDII_Obligatorio_Ticketing_Backend)
# Re-seedear sin borrar la base
docker compose run --rm -e SEED_SKIP_IF_EXISTS=0 seeder

# Reset total (borra el volumen y reinicializa + seedea de cero)
docker compose down -v
docker compose up --build -d
```
