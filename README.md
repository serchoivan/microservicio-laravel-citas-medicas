# **MICROSERVICIO DE GESTIÓN DE USUARIOS – SISTEMA DE CITAS MÉDICAS**  
## *Informe Técnico Final – Laravel 12 + Sanctum + MySQL*  
**Integrantes**   
- Carlos Fernández
- Carlos Cantuña
- Jonathan Hernández
- Marco Chacón
- Sandy Mariño 
- Sergio Condo 

---

### **1. Objetivos del Proyecto**  
El desarrollo de este microservicio tuvo como propósito principal implementar una **API RESTful segura, escalable y profesional** para la gestión de usuarios en un sistema de citas médicas. Se logró la creación, autenticación, consulta, actualización y eliminación de usuarios con **validaciones exhaustivas**, **autenticación basada en tokens** y **arquitectura stateless** que permite múltiples instancias sin compartir estado local. El sistema cumple con estándares de seguridad (hash de contraseñas, tokens Bearer) y está completamente documentado en español para facilitar su integración y mantenimiento.

---

### **2. Tecnologías y Conceptos Clave**  
- **Laravel 12**: Framework PHP que proporciona estructura MVC, migraciones, Eloquent ORM y herramientas de validación.  
- **Laravel Sanctum**: Sistema de autenticación por **tokens personales** (Personal Access Tokens), ideal para APIs sin sesiones.  
- **Stateless Architecture**: El microservicio **no almacena estado** entre peticiones; toda la información persiste en **MySQL**.  
- **Hash Bcrypt**: Contraseñas encriptadas con `Hash::make()` → imposibles de recuperar en texto plano.  
- **Validaciones**: Uso de `Validator::make()` con reglas como `required`, `email`, `min:6`, `confirmed`, `in:`, `sometimes`.  
- **Escalabilidad Horizontal**: Múltiples instancias (`php artisan serve --port=8000`, `--port=8001`) comparten una única base de datos.

---

### **3. Seguridad Implementada**  
- **Hash de contraseñas**: `bcrypt` → nunca se guarda texto plano.  
- **Tokens Bearer**: Generados por Sanctum, requeridos en rutas protegidas vía middleware `auth:sanctum`.  
- **Validación estricta**: Todos los puntos de entrada verifican formato, longitud y obligatoriedad.  
- **Errores en español**: Respuestas `422` con clave `errores` para claridad.  
- **Ocultación de datos sensibles**: Campo `password` excluido en respuestas JSON mediante `$hidden` en el modelo.

---

### **4. Rutas de la API (Prefijo `/api`)**  
| **Método** | **Ruta** | **Descripción** | **Autenticación** |  
|-----------|---------|------------------|-------------------|  
| `POST` | `/registrar-usuario` | Registro completo con token | No |  
| `POST` | `/iniciar-sesion` | Login → nuevo token | No |  
| `GET` | `/listar-usuarios` | Listado completo | Sí |  
| `GET` | `/usuario/{id}` | Detalle de usuario | Sí |  
| `PUT` | `/actualizar-usuario/{id}` | Actualización parcial (`sometimes`) | Sí |  
| `DELETE` | `/eliminar-usuario/{id}` | Eliminación | Sí |  

---

### **5. Estructura de la Base de Datos**  
La base de datos **`sistema_citas_medicas`** se creó en **phpMyAdmin** con codificación `utf8mb4_unicode_ci`. La tabla `users` incluye:  
- `id`, `nombre`, `email` (único), `password` (hash), `fecha_nacimiento`  
- `sexo` (`enum`: Masculino, Femenino, Otro)  
- `numero_seguro` (**obligatorio**)  
- `historial_medico` (**obligatorio**)  
- `contacto_emergencia` (máx. 20 caracteres)  
- `created_at`, `updated_at`  

**Migración ejecutada con:** `php artisan migrate`

---

### **6. Análisis de Archivos Clave**  
- **`app/Models/User.php`**: Modelo Eloquent con `HasApiTokens`, `$fillable` y `$hidden = ['password']`.  
- **`app/Http/Controllers/UserController.php`**: Lógica CRUD con validaciones, hash y respuestas en español. Usa `sometimes` en `update` para actualizaciones parciales.  
- **`routes/api.php`**: Rutas públicas y protegidas agrupadas con `auth:sanctum`.  
- **`.env.example`**: Plantilla de configuración segura (sin credenciales reales).

---

### **7. Escalabilidad y Múltiples Instancias**  
El sistema es **100% stateless**. Se probó con dos instancias simultáneas:  
```bash
php artisan serve --port=8000
php artisan serve --port=8001
