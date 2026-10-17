# Students App

Aplicación de ejemplo construida con Flask y PostgreSQL, pensada para enseñar
el flujo completo: primero clonar y preparar el entorno local, después
Dockerizar la aplicación.

## Cómo se conecta a la base de datos

La app se conecta a PostgreSQL con `psycopg2`, indicando host, usuario,
contraseña y nombre de la base de datos (ver [`app.py`](app.py)):

```python
psycopg2.connect(
    host=os.getenv("DB_HOST"),
    port=os.getenv("DB_PORT"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD"),
    dbname=os.getenv("DB_NAME"),
)
```

Ninguno de esos valores está escrito directamente en el código
(`hardcodeado`). Cada uno se lee con `os.getenv(...)` desde una variable de
entorno, y esas variables se definen en un archivo `.env` que cada quien crea
localmente a partir de [`.env.example`](.env.example) y que **nunca se sube al
repositorio** (está en [`.gitignore`](.gitignore)). Así, las credenciales
pueden cambiar entre tu máquina, la de un compañero o producción, sin tocar
una sola línea de código.

## 1. Entorno local (sin Docker)

Este paso no requiere tener una base de datos levantada. La app arranca igual
y, si no encuentra la base de datos, la página de alumnos se muestra vacía en
vez de dar un error.

### Crear y activar el entorno virtual

```bash
python3 -m venv .venv
source .venv/bin/activate      # En Windows: .venv\Scripts\activate
```

### Instalar las dependencias

```bash
pip install -r requirements.txt
```

### Configurar las variables de entorno

```bash
cp .env.example .env
```

Puedes revisar y ajustar los valores en `.env` si lo necesitas.

### Ejecutar la aplicación

```bash
python3 app.py
```

Abre [http://localhost:5000](http://localhost:5000). Verás "No hay alumnos
registrados." porque todavía no hay ninguna base de datos disponible — y no
un error 500, que es justo el comportamiento esperado en este punto.

## 2. Con Docker (base de datos incluida)

Una vez entendido el paso anterior, se levanta todo el stack (app + PostgreSQL)
con Docker Compose:

```bash
docker compose up --build
```

Esto crea el contenedor de PostgreSQL, carga los datos iniciales de
[`database_students.sql`](database_students.sql) y arranca la app conectada a
esa base de datos. Fíjate que en `docker-compose.yml` las mismas variables
(`DB_HOST`, `DB_USER`, etc.) se definen directamente ahí en vez de en un
`.env` — es Docker Compose quien se las entrega al contenedor. Ahora en
[http://localhost:5000](http://localhost:5000) sí se verá el listado de
alumnos.

Para detener y eliminar los contenedores:

```bash
docker compose down
```

## Estructura del proyecto

```
app.py                    # Aplicación Flask (rutas + conexión a la BD)
templates/                # Vistas Jinja2 (Bootstrap)
database_students.sql     # Datos iniciales para PostgreSQL
.env.example               # Plantilla de variables de entorno
```

## Evidencias

![Captura de la página](evidencias/Captura%20de%20la%20pagina.png)
![Captura de la terminal](evidencias/Captura%20de%20la%20terminal%20Visual.png)
![Captura del contenedor](evidencias/Captura%20del%20contenedor.png)


## Docker Compose con PostgreSQL

En esta parte se agregó un archivo `docker-compose.yml` que levanta dos servicios juntos:

- **postgres**: contenedor con la imagen `postgres:16-alpine`. Al crearse por primera vez, carga automáticamente el archivo `database_students.sql`, que crea la tabla `students` e inserta tres alumnos. Incluye un healthcheck para saber cuándo la base de datos está lista.
- **pythonapp**: contenedor construido con el `Dockerfile` del proyecto. Espera a que Postgres esté saludable y se conecta a él con las variables de entorno `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD` y `DB_NAME`. Se publica en el puerto 5001.

### Comandos utilizados

Primera vez (construye las imágenes):

```bash
docker-compose up --build
```

Siguientes veces:

```bash
docker-compose up
```

Para detener los servicios:

```bash
docker-compose stop
```

La aplicación se consulta en `http://127.0.0.1:5001/`.

### Evidencias de Docker Compose

![Terminal con docker-compose](evidencias/Captura%20del%20docker-build%20arrancando.png)
![Página con los alumnos](evidencias/Captura%20del%20sql%20en%20la%20pagina.png)
![Docker Desktop con los contenedores](evidencias/captura%20de%20los%20dos%20contenedores%20.png)