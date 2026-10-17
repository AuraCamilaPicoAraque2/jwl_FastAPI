# Campus API

API REST construida con FastAPI para practicar registro de usuarios y autenticación mediante tokens JWT. Incluye rutas públicas y rutas protegidas para consultar un perfil y una lista de cursos.

## Requisitos

- Python 3.10 o posterior
- pip

## Instalación

Desde la carpeta del proyecto, crea y activa un entorno virtual:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Instala las dependencias:

```powershell
pip install -r requirements.txt
```

## Configuración

La aplicación necesita la variable `JWT_SECRET_KEY` para firmar los tokens. Crea un archivo `.env` en la raíz del proyecto:

```env
JWT_SECRET_KEY=pon_aqui_una_clave_larga_y_aleatoria
```

Genera una clave segura para cada entorno y no la publiques ni la guardes en el control de versiones. Si no se configura esta variable, la aplicación no inicia.

## Ejecución

Inicia el servidor de desarrollo con Uvicorn:

```powershell
uvicorn main:app --reload
```

La API estará disponible en `http://127.0.0.1:8000`. La documentación interactiva de Swagger UI está en `http://127.0.0.1:8000/docs` y OpenAPI en `http://127.0.0.1:8000/openapi.json`.

## Endpoints

| Método | Ruta | Acceso | Descripción |
| --- | --- | --- | --- |
| `GET` | `/publico` | Público | Mensaje de bienvenida. |
| `POST` | `/usuarios/registro` | Público | Registra un usuario. |
| `POST` | `/token` | Público | Valida las credenciales y entrega un token JWT. |
| `GET` | `/perfil` | Requiere token | Devuelve el nombre del usuario autenticado. |
| `GET` | `/cursos` | Requiere token | Devuelve una lista de cursos para el usuario autenticado. |

### Flujo de autenticación

1. Registra un usuario enviando `username` y `password` como JSON a `/usuarios/registro`. El nombre debe tener entre 3 y 50 caracteres y la contraseña al menos 8.
2. Envía el nombre y la contraseña como formulario (`application/x-www-form-urlencoded`) a `/token`. Los campos son `username` y `password`.
3. Usa el `access_token` recibido en las rutas protegidas con el encabezado `Authorization: Bearer <token>`.

Los tokens se firman con HS256 y expiran después de 60 minutos. Las contraseñas se procesan con `pwdlib` usando su configuración recomendadas.

## Ejemplos

Registro:

```bash
curl -X POST http://127.0.0.1:8000/usuarios/registro \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"camila\",\"password\":\"una-clave-segura\"}"
```

Inicio de sesión:

```bash
curl -X POST http://127.0.0.1:8000/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=camila&password=una-clave-segura"
```

Consulta protegida (reemplaza `<token>` por el token obtenido):

```bash
curl http://127.0.0.1:8000/perfil \
  -H "Authorization: Bearer <token>"
```

## Limitaciones

Los usuarios se almacenan en un diccionario en memoria: se pierden al reiniciar el proceso y no se comparten entre varias instancias. Este proyecto es una base de aprendizaje, no una solución lista para producción; antes de desplegarlo se requiere persistencia, configuración segura de secretos y una revisión de seguridad.
