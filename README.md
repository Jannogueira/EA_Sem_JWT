# Ejercicio Seminario JWT - Jan Nogueira

**Estudiante:** Jan Nogueira  
**Nota:** Ejercicio desarrollado íntegramente sin uso de IA.

---

¡Bienvenido al **Seminario 7 de Entornos de Aplicaciones (EA)**! Este proyecto demuestra una implementación avanzada y segura de autenticación mediante **JWT (JSON Web Tokens)** utilizando una arquitectura de **Access y Refresh Tokens**, ahora refactorizada siguiendo principios de diseño profesional (Separación de responsabilidades y Configuración Centralizada).

> **IMPORTANTE: Web Sockets**
> El código de **Web Sockets** se encuentra en un repositorio independiente para mantener la limpieza arquitectónica: [EA_Sem7_Socket](https://github.com/JairoLopezCarbo/EA_Sem7_Socket)

---

## Novedades de la Arquitectura (Refactorizada)

Tras la última revisión, el proyecto ha evolucionado para cumplir con estándares de escalabilidad:

1.  **Configuración Centralizada (`config/config.ts`)**: El acceso a variables de entorno (`process.env`) está restringido a un solo archivo.
2.  **Capa de Servicios (`services/`)**: Se ha extraído la lógica de negocio de los controladores. Los servicios gestionan la base de datos y la validación.
3.  **Protección de Recursos**: Implementación de lógica de **Autorización**. Gracias a `req.user`, el servidor valida que un usuario solo pueda modificar o eliminar su propia información (403 Forbidden).
4.  **Endpoint de Identidad (`/auth/me`)**: Nuevo punto de acceso que extrae la identidad del usuario directamente del token.

---

## Estructura del Proyecto

```
.
├── src/                   # Backend source
│   ├── config/            # Configuración de entorno y constantes
│   ├── controllers/       # Gestión de peticiones Express
│   ├── library/           # Utilidades compartidas (Logging)
│   ├── middleware/        # Auth Guards (inyectan datos en req.user)
│   ├── models/            # Esquemas de Mongoose y Tipos
│   ├── routes/            # Definición de endpoints
│   ├── services/          # Lógica de negocio (Auth, Servicios)
│   ├── utils/             # Helpers de JWT y validación
│   ├── swagger.ts         # Configuración Swagger/OpenAPI
│   └── server.ts          # Punto de entrada y conexión DB
├── frontend/              # Cliente Angular principal
└── .env                   # Variables de entorno y secretos
```

---

## Seguridad: Access + Refresh Token

### 1. ¿Dónde se guarda cada token?
- **Access Token (15 min):** Se envía en el cuerpo de la respuesta. El frontend lo guarda en memoria y lo usa en la cabecera `Authorization: Bearer <token>`.
- **Refresh Token (7 días):** Se envía en una **Cookie `httpOnly`**. Es invisible para JavaScript (protección XSS) y se envía automáticamente al renovar sesión.

### 2. El flujo de validación segura
1. **Identificación**: El middleware de auth valida el token y guarda los datos en `req.user`.
2. **Autorización**: En rutas sensibles, el controlador compara `req.user.id` con `req.params.id`.
3. **Control**: Si no coinciden, se devuelve un `403 Forbidden`, impidiendo la suplantación.

---

## Endpoints Principales (API)

| Método | URL | Privado | Descripción |
|---|---|---|---|
| `POST`   | `/auth/login`    | No     | Genera Access Token y Cookie de Refresh. |
| `GET`    | `/auth/me`       | **Sí** | Devuelve el perfil del usuario logueado. |
| `POST`   | `/auth/refresh`  | No     | Renueva el Access Token usando la Cookie. |
| `POST`   | `/auth/logout`   | No     | Limpia la cookie de sesión. |
| `DELETE` | `/usuarios/:id`  | **Sí** | Borrado seguro y verificado.             |

---

## Instalación y Uso

Asegúrate de tener un archivo `.env` configurado:
```env
MONGO_URI="mongodb://localhost:27017/tu_bd"
SERVER_PORT=1337
JWT_ACCESS_SECRET="clave_secreta"
JWT_REFRESH_SECRET="otra_clave_secreta"
```

1. **Backend**: `npm install` y `npm start`.
2. **Swagger**: Documentación interactiva en `http://localhost:1337/api`.
3. **Frontend (Angular)**: `cd frontend`, `npm install` y `npm start`.

---
¡Explora el código y fíjate en cómo la separación de servicios y controladores hace que todo sea más fácil de testear y mantener! 