# 🚀 Guía de Despliegue en Koyeb - TODO App

Despliega tu aplicación TODO completa (Frontend + Backend + Database) en **Koyeb** **GRATIS** y **SIN tarjeta de crédito**.

---

## ✅ Requisitos Previos

- Cuenta en [GitHub](https://github.com)
- Cuenta en [Koyeb](https://www.koyeb.com) (GRATIS, sin tarjeta)
- Tu código subido a un repositorio de GitHub

---

## 📋 Lo que Koyeb ofrece GRATIS

| Servicio | Recursos | Limitación |
|----------|----------|------------|
| **Backend** (Spring Boot) | 512MB RAM, 0.1 vCPU, 2GB SSD | Siempre activo, gratis para siempre |
| **Base de Datos** (PostgreSQL) | 1GB RAM, 0.25 CPU, 1GB storage | 50 horas activas/mes (se duerme cuando no se usa) |
| **Frontend** (Static) | Hosting de archivos estáticos | Gratis |

---

## 📦 Paso 1: Subir código a GitHub

```bash
cd "C:\Users\guerr\Desktop\Capy Tareas"

# Inicializar git (si no está inicializado)
git init

# Agregar todos los archivos
git add .

# Crear commit
git commit -m "Preparar proyecto para despliegue en Koyeb"

# Conectar con tu repositorio de GitHub
git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git

# Subir código
git push -u origin main
```

---

## 🔐 Paso 2: Crear cuenta en Koyeb

1. Ve a [koyeb.com/auth/signup](https://www.koyeb.com/auth/signup)
2. **Regístrate con GitHub** (opción recomendada)
3. Autoriza a Koyeb para acceder a tus repositorios
4. **NO necesitas agregar tarjeta de crédito** (en la mayoría de ubicaciones)

---

## 🗄️ Paso 3: Crear Base de Datos PostgreSQL

### Opción A: Base de datos en Koyeb (Recomendado)

1. En el dashboard de Koyeb, haz clic en **"Create Database"**
2. Selecciona **PostgreSQL**
3. Configuración:
   - **Name:** `todoapp-db`
   - **Region:** Frankfurt o Washington D.C.
   - **Plan:** **Free (Hobby)**
4. Haz clic en **"Create Database"**
5. Espera 1-2 minutos mientras se crea
6. **Guarda las credenciales:**
   - Host
   - Port
   - Database
   - Username
   - Password
   - Connection String (la necesitarás)

### Opción B: Base de datos externa gratuita

Si Koyeb te limita, usa [Neon.tech](https://neon.tech) o [Supabase](https://supabase.com) (ambos gratis, sin tarjeta):

1. Crea cuenta en Neon.tech
2. Crea un proyecto PostgreSQL
3. Copia el **Connection String**

---

## 🔧 Paso 4: Desplegar Backend (Spring Boot)

1. En Koyeb, haz clic en **"Create Web Service"**
2. Selecciona **"GitHub"** como fuente
3. Conecta tu repositorio `Capy Tareas`
4. Configuración del servicio:

### Builder
- **Builder:** Docker
- **Dockerfile path:** `back/ToDo API/Dockerfile`
- **Docker build context:** `back/ToDo API`

### Instance
- **Region:** Frankfurt o Washington D.C.
- **Instance type:** **Free (Hobby)** - 512MB RAM

### Environment Variables (Variables de Entorno)

Agrega estas variables:

| Variable | Valor |
|----------|-------|
| `SPRING_PROFILES_ACTIVE` | `prod` |
| `DATABASE_URL` | `jdbc:postgresql://[HOST]:[PORT]/[DATABASE]` |
| `DB_USERNAME` | Tu usuario de PostgreSQL |
| `DB_PASSWORD` | Tu contraseña de PostgreSQL |
| `PORT` | `8080` |

**Ejemplo de DATABASE_URL:**
```
jdbc:postgresql://todoapp-db-xxx.koyeb.app:5432/koyebdb
```

### Health Check
- **Path:** `/api/v1/tasks`
- **Port:** `8080`

5. Haz clic en **"Deploy"**
6. Espera 5-10 minutos mientras Koyeb:
   - Construye la imagen Docker
   - Descarga dependencias de Maven
   - Compila el proyecto
   - Despliega el contenedor

7. **Guarda la URL del backend**, será algo como:
   ```
   https://todoapp-backend-[tu-usuario].koyeb.app
   ```

---

## 🌐 Paso 5: Desplegar Frontend

### Opción A: GitHub Pages (Recomendado - 100% Gratis)

1. **Actualizar URL del backend en el código:**

Edita: [`front/assets/script/data/path.js`](front/assets/script/data/path.js)

```javascript
// Cambia esto:
export const path = "http://127.0.0.1:8080/api/v1/tasks/";

// Por esto:
export const path = "https://todoapp-backend-TU-USUARIO.koyeb.app/api/v1/tasks/";
```

2. **Commit y push:**
```bash
git add front/assets/script/data/path.js
git commit -m "Actualizar URL del backend para producción"
git push
```

3. **Habilitar GitHub Pages:**
   - Ve a tu repositorio en GitHub
   - Settings → Pages
   - Source: **Deploy from a branch**
   - Branch: **main** → Folder: **/ (root)**
   - **Subdirectory:** Cambia a `/front`
   - Click **Save**

4. **Espera 1-2 minutos** y tu frontend estará en:
   ```
   https://TU-USUARIO.github.io/TU-REPOSITORIO/
   ```

### Opción B: Netlify (Alternativa)

1. Ve a [netlify.com](https://netlify.com)
2. Regístrate con GitHub
3. **New site from Git** → Conecta tu repo
4. Build settings:
   - **Base directory:** `front`
   - **Build command:** (dejar vacío)
   - **Publish directory:** `front`
5. Click **Deploy**

---

## 🔄 Paso 6: Actualizar CORS (si es necesario)

Si tu URL del frontend es diferente, actualiza:

[`back/ToDo API/src/main/java/ar/com/utnfrsr/todoapp/config/CorsConfig.java`](back/ToDo API/src/main/java/ar/com/utnfrsr/todoapp/config/CorsConfig.java:12-16)

Ya está configurado para aceptar:
- `https://*.koyeb.app` ✅
- `https://*.github.io` (si usas GitHub Pages, agrégalo)

Si usas GitHub Pages, agrega:
```java
.allowedOriginPatterns(
    "http://localhost:5500",
    "http://127.0.0.1:5500",
    "https://*.koyeb.app",
    "https://*.github.io",  // Agregar esta línea
    "https://*.onrender.com"
)
```

Haz commit y push para que Koyeb reconstruya.

---

## ✅ Paso 7: Verificar Despliegue

### Backend
- **URL:** `https://todoapp-backend-[usuario].koyeb.app`
- **Swagger UI:** `https://todoapp-backend-[usuario].koyeb.app/swagger-ui.html`
- **API Tasks:** `https://todoapp-backend-[usuario].koyeb.app/api/v1/tasks`

### Frontend
- **GitHub Pages:** `https://[usuario].github.io/[repositorio]/`
- **Netlify:** `https://[nombre].netlify.app`

### Base de Datos
- Verifica en el dashboard de Koyeb que esté "Running"
- Las tablas se crean automáticamente (Hibernate DDL auto-update)

---

## 🐛 Solución de Problemas

### ❌ Backend no inicia

**1. Revisa los logs:**
   - Dashboard de Koyeb → Tu servicio → **Logs**
   - Busca errores de:
     - Conexión a base de datos
     - Versión de Java incorrecta
     - Puerto incorrecto

**2. Problemas comunes:**
   - ❌ `DATABASE_URL` mal configurada → Verifica el formato `jdbc:postgresql://...`
   - ❌ Credenciales incorrectas → Verifica `DB_USERNAME` y `DB_PASSWORD`
   - ❌ Puerto incorrecto → Debe ser `8080`
   - ❌ Perfil incorrecto → Debe ser `SPRING_PROFILES_ACTIVE=prod`

### ❌ Frontend no se conecta al Backend

**Verifica:**
1. ✅ URL del backend actualizada en `path.js`
2. ✅ CORS configurado correctamente
3. ✅ Backend está activo (status "Running" en Koyeb)

**Prueba manualmente:**
```bash
curl https://todoapp-backend-[tu-usuario].koyeb.app/api/v1/tasks
```

Deberías recibir: `[]` (array vacío) o tus tareas.

### ❌ Error de CORS

**Síntoma:** Consola del navegador muestra:
```
Access to fetch at 'https://...' has been blocked by CORS policy
```

**Solución:**
1. Verifica que `CorsConfig.java` incluya tu dominio
2. Si usas GitHub Pages, agrega `https://*.github.io`
3. Commit, push y espera a que Koyeb reconstruya (5-10 min)

### ❌ Base de datos se duerme

**Koyeb Free Tier:** La base de datos se duerme después de inactividad.

**Solución:**
- Primera petición después de dormir tardará 30-60 segundos
- Esto es normal en el plan gratuito
- Considera usar [Neon.tech](https://neon.tech) si necesitas más horas activas

### ❌ Error: "Payment Required"

Si Koyeb te pide tarjeta:
1. Usa **Neon.tech** o **Supabase** para la base de datos (gratis, sin tarjeta)
2. Despliega el backend en **Railway.app** (puede no requerir tarjeta)
3. Usa **GitHub Pages** para el frontend

---

## 🔄 Actualizar la Aplicación

Para desplegar nuevos cambios:

```bash
# Hacer cambios en el código
git add .
git commit -m "Descripción de cambios"
git push
```

**Koyeb automáticamente:**
- Detecta el push
- Reconstruye la imagen Docker
- Despliega la nueva versión
- Tiempo: 5-10 minutos

---

## 📊 Monitoreo

### Koyeb Dashboard
- **Status:** Running / Building / Sleeping
- **Logs:** Ver logs en tiempo real
- **Metrics:** CPU, RAM, requests

### Health Check
Koyeb verifica automáticamente `/api/v1/tasks` cada 30 segundos.

---

## 💰 Costos

| Servicio | Plan | Costo |
|----------|------|-------|
| Backend (Koyeb) | Free | **$0** |
| Database (Koyeb) | Free | **$0** |
| Frontend (GitHub Pages) | Free | **$0** |
| **TOTAL** | | **$0** |

**Sin tarjeta de crédito requerida** ✅

---

## 📚 Recursos Útiles

- [Documentación Koyeb](https://www.koyeb.com/docs)
- [Guía Spring Boot en Koyeb](https://www.koyeb.com/tutorials/using-spring-authorization-server-as-an-auth-solution-on-koyeb)
- [GitHub Pages Docs](https://docs.github.com/pages)
- [Neon PostgreSQL](https://neon.tech/docs/introduction)

---

## 🎉 ¡Listo!

Tu aplicación TODO está desplegada y funcionando:

✅ **Backend:** Spring Boot en Koyeb
✅ **Database:** PostgreSQL en Koyeb
✅ **Frontend:** GitHub Pages
✅ **Costo:** $0
✅ **Tarjeta:** No requerida

---

## 📝 Checklist Final

- [ ] Código subido a GitHub
- [ ] Cuenta creada en Koyeb
- [ ] Base de datos PostgreSQL creada
- [ ] Backend desplegado con variables de entorno
- [ ] Frontend desplegado (GitHub Pages o Netlify)
- [ ] URL del backend actualizada en `path.js`
- [ ] CORS configurado correctamente
- [ ] Aplicación funcionando correctamente

---

¿Problemas? Revisa los logs en Koyeb o abre un issue en tu repositorio de GitHub.
