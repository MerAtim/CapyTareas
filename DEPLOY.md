# Guía de Despliegue - TODO App en Render

Esta guía te ayudará a desplegar tu aplicación TODO completa (Frontend + Backend + Database) en **Render** de forma gratuita.

## Requisitos Previos

- Cuenta en [GitHub](https://github.com)
- Cuenta en [Render](https://render.com) (gratis)
- Tu código subido a un repositorio de GitHub

---

## Arquitectura del Despliegue

```
┌─────────────────────────────────────────┐
│           Render Platform               │
│                                         │
│  ┌──────────────┐   ┌────────────────┐ │
│  │   Frontend   │   │    Backend     │ │
│  │   (Static)   │──▶│ (Spring Boot)  │ │
│  │              │   │   (Docker)     │ │
│  └──────────────┘   └───────┬────────┘ │
│                             │          │
│                     ┌───────▼────────┐ │
│                     │   PostgreSQL   │ │
│                     │   (Database)   │ │
│                     └────────────────┘ │
└─────────────────────────────────────────┘
```

---

## Paso 1: Preparar el Repositorio

### 1.1 Subir el código a GitHub

```bash
cd "C:\Users\guerr\Desktop\Capy Tareas"

# Inicializar git (si no está inicializado)
git init

# Agregar todos los archivos
git add .

# Crear commit
git commit -m "Preparar para despliegue en Render con Docker"

# Conectar con tu repositorio de GitHub
git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git

# Subir código
git push -u origin main
```

### 1.2 Verificar archivos necesarios

Asegúrate de que estos archivos existan en tu repositorio:
- ✅ `render.yaml` (raíz del proyecto)
- ✅ `back/ToDo API/Dockerfile` (nuevo - para contenedor Docker)
- ✅ `back/ToDo API/.dockerignore` (nuevo - optimización)
- ✅ `back/ToDo API/pom.xml` (con dependencia de PostgreSQL)
- ✅ `back/ToDo API/src/main/resources/application-prod.yml`
- ✅ `back/ToDo API/src/main/java/ar/com/utnfrsr/todoapp/config/CorsConfig.java` (actualizado)
- ✅ `front/index.html`
- ✅ `front/assets/css/styles.css`

---

## Paso 2: Desplegar en Render

### 2.1 Crear cuenta en Render

1. Ve a [render.com](https://render.com)
2. Regístrate con tu cuenta de GitHub
3. Autoriza a Render para acceder a tus repositorios

### 2.2 Desplegar usando Blueprint (Recomendado)

**Render Blueprint despliega automáticamente:**
- ✅ Backend (Spring Boot en Docker)
- ✅ Base de datos (PostgreSQL)
- ✅ Frontend (Static Site)

**Pasos:**

1. En el dashboard de Render, haz clic en **"New +"** → **"Blueprint"**
2. Conecta tu repositorio de GitHub
3. Render detectará automáticamente el archivo `render.yaml`
4. Revisa la configuración y haz clic en **"Apply"**
5. Espera 10-15 minutos mientras Render:
   - Construye la imagen Docker del backend
   - Crea la base de datos PostgreSQL
   - Despliega el frontend estático

---

## Paso 3: Actualizar URL del Backend en el Frontend

Una vez desplegado el backend, obtendrás una URL como:
```
https://todoapp-backend.onrender.com
```

**Actualiza el archivo:** `front/assets/script/data/path.js`

```javascript
// Cambia esto:
export const path = "http://127.0.0.1:8080/api/v1/tasks/";

// Por esto:
export const path = "https://todoapp-backend.onrender.com/api/v1/tasks/";
```

**Haz commit y push:**
```bash
git add front/assets/script/data/path.js
git commit -m "Actualizar URL del backend para producción"
git push
```

Render automáticamente re-desplegará el frontend.

---

## Paso 4: Actualizar CORS (si es necesario)

Si la URL de tu frontend en Render es diferente a `https://todoapp-frontend.onrender.com`, actualiza el archivo:

**`back/ToDo API/src/main/java/ar/com/utnfrsr/todoapp/config/CorsConfig.java`**

```java
.allowedOrigins(
    "http://localhost:5500",
    "http://127.0.0.1:5500",
    "https://TU-FRONTEND.onrender.com"  // Actualiza con tu URL real
)
```

Haz commit y push para que Render reconstruya el backend.

---

## Paso 5: Verificar el Despliegue

### Backend
- URL: `https://todoapp-backend.onrender.com`
- Swagger: `https://todoapp-backend.onrender.com/swagger-ui.html`
- API: `https://todoapp-backend.onrender.com/api/v1/tasks`

### Frontend
- URL: `https://todoapp-frontend.onrender.com`

### Base de Datos
- Verifica en el dashboard de Render que PostgreSQL esté activo
- Las tablas se crearán automáticamente (Hibernate DDL auto-update)

---

## Solución de Problemas

### Backend no inicia

**Revisa los logs en Render:**
1. Ve a tu servicio backend
2. Click en **"Logs"**
3. Busca errores de conexión a la base de datos o Docker

**Problemas comunes:**
- ❌ Variable `DATABASE_URL` no configurada → Verifica que la base de datos esté creada
- ❌ Error de build Docker → Revisa que el Dockerfile esté correcto
- ❌ Puerto incorrecto → Usa `${PORT:8080}` en application-prod.yml

### Frontend no se conecta al Backend

**Verifica:**
- ✅ CORS configurado correctamente en CorsConfig.java
- ✅ URL del backend actualizada en `path.js`
- ✅ Backend está activo (revisa status en Render)

**Prueba el endpoint manualmente:**
```bash
curl https://todoapp-backend.onrender.com/api/v1/tasks
```

### Error de CORS

Si ves errores de CORS en la consola del navegador:

1. Actualiza `CorsConfig.java` con la URL correcta del frontend
2. Haz commit y push
3. Espera que Render re-despliegue el backend (5-10 min)

### Error de Base de Datos

Si ves errores de conexión a PostgreSQL:

1. Verifica que la variable `DATABASE_URL` esté configurada correctamente
2. Verifica que `application-prod.yml` use el driver de PostgreSQL
3. Revisa los logs de la base de datos en Render

---

## Costos

**Todo es GRATIS con el plan Free de Render:**

| Servicio | Plan | Costo |
|----------|------|-------|
| Backend (Docker) | Free | $0 |
| Frontend (Static) | Free | $0 |
| PostgreSQL | Free | $0 |

**Limitaciones del plan gratuito:**
- El backend se "duerme" después de 15 minutos sin uso
- Primera petición después de dormir puede tardar 30-60 segundos
- 750 horas/mes de runtime
- 100GB de ancho de banda

---

## Actualizar la Aplicación

Para desplegar nuevos cambios:

```bash
# Hacer cambios en el código
git add .
git commit -m "Descripción de cambios"
git push
```

Render automáticamente detectará los cambios y re-desplegará.

---

## Configuración de Docker

Este proyecto usa Docker para desplegar el backend. El `Dockerfile` tiene dos etapas:

1. **Build Stage**: Compila el proyecto con Maven
2. **Runtime Stage**: Ejecuta el JAR en un contenedor ligero

**Ventajas:**
- ✅ Build reproducible y consistente
- ✅ Imagen ligera (solo JRE, no JDK completo)
- ✅ Mayor compatibilidad con Render
- ✅ Cacheo de dependencias para builds más rápidos

---

## URLs Importantes

- [Documentación Render](https://render.com/docs)
- [Render Blueprint Spec](https://render.com/docs/blueprint-spec)
- [Docker en Render](https://render.com/docs/docker)

---

## Resumen

1. ✅ Sube código a GitHub
2. ✅ Crea cuenta en Render
3. ✅ Despliega usando Blueprint
4. ✅ Actualiza URL del backend en el frontend
5. ✅ Verifica que todo funcione
6. ✅ Disfruta de tu app en producción

**Tu app estará disponible en:**
- Frontend: `https://todoapp-frontend.onrender.com`
- Backend: `https://todoapp-backend.onrender.com`
- API Docs: `https://todoapp-backend.onrender.com/swagger-ui.html`

---

¿Problemas? Revisa los logs en Render o consulta la documentación oficial.
