# 🧪 CodeLab — Plugin de Evaluación de Código para Moodle

<p align="center">
  <img src="https://img.shields.io/badge/Moodle-4.0%2B-orange?style=for-the-badge&logo=moodle" alt="Moodle 4.0+">
  <img src="https://img.shields.io/badge/PHP-8.0%2B-777BB4?style=for-the-badge&logo=php&logoColor=white" alt="PHP 8.0+">
  <img src="https://img.shields.io/badge/Judge0-API-0078D4?style=for-the-badge" alt="Judge0">
  <img src="https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Licencia-GPL%20v3-blue?style=for-the-badge" alt="GPL v3">
</p>

**CodeLab** es un plugin de actividad para Moodle que permite a los profesores crear tareas de programación con **calificación automática**. Los estudiantes escriben, ejecutan y entregan código directamente en la plataforma, y el sistema evalúa sus soluciones contra casos de prueba definidos por el profesor.

---

## 📋 Tabla de contenidos

- [¿Qué problema resuelve?](#-qué-problema-resuelve)
- [Características principales](#-características-principales)
- [Arquitectura del sistema](#-arquitectura-del-sistema)
- [Requisitos previos](#-requisitos-previos)
- [Instalación y configuración](#-instalación-y-configuración)
- [Uso del plugin](#-uso-del-plugin)
- [Lenguajes soportados](#-lenguajes-de-programación-soportados)
- [Comandos útiles](#-comandos-útiles)
- [Solución de problemas](#-solución-de-problemas)
- [Licencia](#-licencia)

---

## 🎯 ¿Qué problema resuelve?

En cursos de programación, los profesores necesitan evaluar el código de decenas o cientos de estudiantes. El proceso manual es lento y propenso a errores. **CodeLab** automatiza este flujo de trabajo:

1. **El profesor** crea una tarea y define los casos de prueba (entrada → salida esperada).
2. **El estudiante** escribe su solución en un editor de código integrado en Moodle.
3. **El sistema** ejecuta el código del estudiante contra los casos de prueba en un entorno aislado (sandbox) y calcula la calificación automáticamente.

---

## ✨ Características principales

| Característica | Descripción |
|---|---|
| **Editor de código integrado** | Editor con resaltado de sintaxis directamente en Moodle, sin herramientas externas |
| **Ejecución segura en sandbox** | El código se ejecuta en contenedores aislados mediante Judge0, protegiendo el servidor |
| **Calificación automática** | Se compara la salida del estudiante contra los casos de prueba y se calcula la nota |
| **Casos de prueba visibles y ocultos** | Los profesores pueden definir casos visibles (para que el estudiante practique) y ocultos (para la evaluación final) |
| **Modo simplificado (runner)** | Permite que el estudiante escriba solo una función, mientras el sistema se encarga de la entrada/salida |
| **Múltiples lenguajes** | Soporte para Python, Java, C, C++, JavaScript, y más |
| **Retroalimentación instantánea** | El estudiante ve inmediatamente qué casos pasó y cuáles falló |
| **Panel de entregas para el profesor** | Vista con todas las entregas, notas y la posibilidad de ajustar calificaciones manualmente |
| **Estadísticas de la actividad** | Métricas como promedio de calificaciones, tasa de aprobados y casos más fallados |
| **Múltiples intentos** | Configurable: intentos ilimitados o un número máximo definido por el profesor |

---

## 🏗 Arquitectura del sistema

```
  Navegador del usuario
        │
        ▼  http://localhost:8080
  ┌─────────────────────────┐
  │        MOODLE            │  ← Plugin CodeLab incluido
  │     (PHP + Apache)       │
  └─────────────────────────┘
        │                 │
        ▼                 ▼
  ┌───────────┐    ┌──────────────────────────┐
  │  MariaDB  │    │    Judge0 Server          │
  │ (base de  │    │    :2358                  │
  │  datos)   │    │                           │
  └───────────┘    │  Ejecuta código en        │
                   │  sandbox seguro (Docker)  │
                   └──────────────────────────┘
                        │            │
                   ┌────┘       ┌────┘
                   ▼            ▼
              PostgreSQL      Redis
             (judge0_db)   (judge0_redis)
```

El proyecto utiliza **Docker Compose** para orquestar todos los servicios. El plugin de Moodle se comunica con Judge0 vía API REST para ejecutar el código de los estudiantes de forma aislada y segura.

---

## 📦 Requisitos previos

| Requisito | Notas |
|---|---|
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) | Incluye Docker Engine y Docker Compose. Es lo único que necesitas instalar. |
| ~4 GB de RAM disponibles | Para correr Moodle + Judge0 simultáneamente |
| ~5 GB de espacio en disco | Para las imágenes de Docker y datos |

---

## 🚀 Instalación y configuración

### 1. Clonar el repositorio

```bash
git clone https://github.com/<tu-usuario>/ProyectoEditorMoodle.git
cd ProyectoEditorMoodle
```

### 2. Abrir Docker Desktop

Asegúrate de que Docker Desktop esté corriendo antes de continuar (el ícono de la ballena 🐳 en la barra de tareas debe estar activo).

### 3. Construir y levantar los servicios

```bash
docker compose up -d --build
```

> ⏱️ **La primera vez tarda entre 10 y 20 minutos** porque descarga las imágenes de Docker (~2 GB), instala Moodle 4.5 y configura la base de datos.
>
> Las **siguientes veces** arranca en menos de 1 minuto con:
> ```bash
> docker compose up -d
> ```

### 4. Verificar que todo está corriendo

```bash
docker compose ps
```

Todos los servicios deben mostrar estado `running` o `Up`:

```
NAME              STATUS
mariadb           Up (healthy)
moodle            Up
judge0_db         Up (healthy)
judge0_redis      Up
judge0_server     Up
judge0_worker     Up
```

### 5. Ver el progreso de la instalación de Moodle

```bash
docker compose logs -f moodle
```

Cuando la instalación termine, verás un mensaje de confirmación con la URL y las credenciales de acceso. Presiona `Ctrl + C` para salir de los logs.

### 6. Acceder a Moodle

Abre tu navegador en:

```
http://localhost:8080
```

Inicia sesión con las credenciales de administrador que aparecieron en los logs del paso anterior.

### 7. Instalar el plugin CodeLab en Moodle

1. Inicia sesión como administrador.
2. Ve a **Administración del sitio → Notificaciones**.
3. Moodle detectará el plugin automáticamente — haz clic en **Actualizar base de datos de Moodle**.
4. Sigue los pasos en pantalla hasta completar la instalación.

### 8. Configurar el motor de ejecución (Judge0)

1. Ve a **Administración del sitio → Plugins → Módulos de actividad → CodeLab**.
2. Configura los campos:

| Campo | Valor |
|---|---|
| URL de la API Judge0 | `http://judge0_server:2358` |
| Instancia auto-alojada | ✅ Activado |
| Clave API | *(dejar vacío)* |

3. Guarda los cambios.

---

## 📝 Uso del plugin

### Como profesor — Crear una tarea

1. Entra a un curso en Moodle y activa el **modo de edición**.
2. Haz clic en **"+ Añadir actividad o recurso" → CodeLab**.
3. Configura la actividad:
   - **Nombre** y **descripción** de la tarea.
   - **Lenguaje por defecto** (ej. Python, Java, C++).
   - **Código inicial** que verán los estudiantes al abrir la actividad.
   - **Calificación máxima** y **número de intentos** permitidos.
4. Define los **casos de prueba** con entrada, salida esperada, puntos y visibilidad:

| Nombre del caso | Entrada | Salida esperada | Puntos | Oculto |
|---|---|---|---|---|
| Caso básico | `2 4` | `6` | 30 | No |
| Caso con negativo | `-1 5` | `4` | 30 | No |
| Caso de evaluación | `100 200` | `300` | 40 | Sí |

> 💡 **Modo simplificado (runner):** Puedes definir un *código envolvente* que maneje la entrada/salida, de modo que el estudiante solo necesite escribir la función solución. Si prefieres el modo clásico (stdin/stdout manual), deja el runner vacío.

5. Guarda la actividad.

### Como estudiante — Resolver y entregar

1. Accede al curso y abre la actividad CodeLab.
2. Escribe tu solución en el **editor de código integrado**.
3. Haz clic en **"Ejecutar y probar"** para ver los resultados:
   - ✅ **Verde** — caso pasado.
   - ❌ **Rojo** — caso fallado (muestra salida esperada vs. obtenida).
4. Ajusta tu código hasta que pases los casos visibles.
5. Haz clic en **"Entregar"** para registrar tu solución final.
6. El sistema calcula tu calificación automáticamente (incluyendo los casos ocultos).

### Como profesor — Revisar entregas

1. Abre la actividad y accede a la pestaña **"Entregas"**.
2. Revisa la tabla con: estudiante, fecha de entrega, lenguaje usado, tests pasados y calificación.
3. Haz clic en **"Ver"** para revisar el código de un estudiante y, si lo deseas, ajustar la nota manualmente.
4. Consulta la pestaña **"Estadísticas"** para ver métricas generales de la actividad.

---

## 🌐 Lenguajes de programación soportados

Python · JavaScript (Node.js) · Java · C · C++ · PHP · C# · Ruby · Go · Kotlin

---

## 🔧 Comandos útiles

```bash
# Iniciar el proyecto
docker compose up -d

# Detener el proyecto (sin borrar datos)
docker compose down

# Ver qué servicios están corriendo
docker compose ps

# Ver logs de Moodle en tiempo real
docker compose logs -f moodle

# Ver logs de Judge0 (ejecución de código)
docker compose logs -f judge0_server

# Reconstruir la imagen de Moodle (solo si modificas el Dockerfile)
docker compose up -d --build moodle

# Borrar TODO y empezar desde cero
docker compose down -v
docker compose up -d --build
```

---

## 🛠 Solución de problemas

| Problema | Solución |
|---|---|
| Moodle muestra "sitio en mantenimiento" o pantalla en blanco | Ejecuta `docker compose logs moodle --tail=30` y espera a que termine la instalación. |
| Error "No se pudo conectar a la base de datos" | La base de datos tarda en arrancar. El sistema reintenta automáticamente. Si persiste, ejecuta `docker compose restart mariadb`. |
| Judge0 no ejecuta código | Verifica que Judge0 responde: abre `http://localhost:2358/languages` en tu navegador. |
| El puerto 8080 ya está en uso | Edita `docker-compose.yml`, cambia `"8080:80"` por otro puerto (ej. `"8090:80"`) y accede a `http://localhost:<nuevo_puerto>`. |

---

## 📁 Estructura del proyecto

```
ProyectoEditorMoodle/
│
├── docker-compose.yml        # Orquesta todos los servicios
├── Dockerfile                # Construye la imagen de Moodle
├── docker-entrypoint.sh      # Instala Moodle automáticamente al arrancar
├── config_moodle.php         # Configuración de Moodle
│
├── judge0/                   # Motor de ejecución de código
│   └── judge0.conf           # Configuración de Judge0
│
└── mod/
    └── codelab/              # Plugin CodeLab
        ├── version.php       # Versión y metadatos del plugin
        ├── lib.php           # Funciones principales de la API de Moodle
        ├── view.php          # Vista del editor de código
        ├── execute.php       # Endpoint de ejecución de código
        ├── submission.php    # Gestión de entregas
        ├── mod_form.php      # Formulario de configuración de la actividad
        ├── settings.php      # Configuración del plugin en admin
        ├── styles.css        # Estilos del editor y la interfaz
        ├── db/               # Esquema de base de datos y upgrades
        ├── lang/             # Archivos de idioma
        ├── templates/        # Plantillas Mustache
        ├── amd/              # Módulos JavaScript (AMD)
        ├── js/               # Scripts JavaScript adicionales
        ├── classes/          # Clases PHP del plugin
        ├── backup/           # Soporte de backup/restore
        └── pix/              # Iconos del plugin
```

---

## 📄 Licencia

Este proyecto está licenciado bajo la **GNU General Public License v3.0**. Consulta el archivo [LICENSE](LICENSE) para más detalles.
