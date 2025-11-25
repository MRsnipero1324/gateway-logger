# 🚀 Gateway Logger – Microservicio de Registro y Redirección  

Microservicio desarrollado con **FastAPI**, **Docker** y **PostgreSQL**, encargado de recibir TODAS las peticiones del proyecto, registrarlas y redirigirlas al microservicio correspondiente.

Este servicio actúa como un **API Gateway** y como un **sistema de auditoría**, ya que almacena cada acción en la base de datos.

---

# 📌 Funcionalidades principales

✔ Recibe todas las peticiones HTTP del sistema  
✔ Registra cada petición en PostgreSQL  
✔ Guarda: fecha, acción y resultado (Completado/Fallido)  
✔ Redirige a los microservicios internos  
✔ Espera la respuesta y actualiza el log  
✔ Corre completamente en Docker  

---

# 📂 Estructura del proyecto

gateway-logger/
│── app/
│ ├── main.py
│ ├── router.py
│ ├── database.py
│ ├── models.py
│ └── init.py
│── Dockerfile
│── docker-compose.yml
│── requirements.txt
└── README.md

yaml
Copiar código

---

# 🗄 Base de datos – Tabla de logs

El microservicio guarda registros en la tabla `logs` con esta estructura:

| Campo       | Descripción                                 |
|-------------|---------------------------------------------|
| **id**      | Identificador único autoincremental          |
| **fecha**   | Fecha y hora de la petición                  |
| **accion**  | Método + ruta (ej: `GET personas/listar`)    |
| **resultado** | `"Completado"` o `"Fallido"`                |

---

# 🔗 Flujo del gateway

Cliente →
Gateway Logger →
Guarda log →
Llama al microservicio correspondiente →
Recibe respuesta →
Actualiza log →
Responde al cliente

yaml
Copiar código

---

# 🟦 Integración con los microservicios del equipo

Para que los microservicios del resto del equipo funcionen con este gateway, deben cumplirse estas reglas:

---

## ✔ 1️⃣ Nombres obligatorios de contenedores

El gateway redirige automáticamente basándose en el nombre del contenedor:

| Microservicio | Contenedor |
|---------------|------------|
| Personas      | `ms-personas` |
| Consulta      | `ms-consulta` |
| RAG           | `ms-rag` |

Ejemplo dentro del docker-compose:

```yaml
container_name: ms-personas
✔ 2️⃣ Puertos obligatorios
Cada microservicio debe escuchar en su puerto asignado:

Microservicio	Puerto
ms-personas	8001
ms-consulta	8002
ms-rag	8003

Ejemplo:

yaml
Copiar código
ports:
  - "8001:8001"
✔ 3️⃣ Rutas que deben coincidir
El gateway redirige según el primer segmento de la URL:

Ruta recibida	Microservicio destino
/personas/...	ms-personas
/consulta/...	ms-consulta
/rag/...	ms-rag

Por lo tanto, cada microservicio debe iniciar sus rutas así:

Ejemplos:
ms-personas

python
Copiar código
@app.get("/personas/listar")
@app.post("/personas/crear")
ms-consulta

python
Copiar código
@app.get("/consulta/datos")
ms-rag

python
Copiar código
@app.post("/rag/preguntar")
✔ 4️⃣ Dockerfile obligatorio en cada microservicio
Ejemplo de Dockerfile para un microservicio FastAPI:

dockerfile
Copiar código
FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install fastapi uvicorn
CMD ["uvicorn", "main:app", "--host","0.0.0.0","--port","8001"]
✔ 5️⃣ Reemplazar el comando sleep infinity
En el docker-compose se usa esto temporalmente:

yaml
Copiar código
command: ["sleep", "infinity"]
Cada compañero debe reemplazarlo por el comando de su servicio:

yaml
Copiar código
command: ["uvicorn","main:app","--host","0.0.0.0","--port","8001"]
(ajustando el puerto según su microservicio)

▶️ Cómo ejecutar el proyecto
Clonar este repositorio

Abrir Docker Desktop

En la carpeta del proyecto, ejecutar:

css
Copiar código
docker-compose up --build
Esto levanta automáticamente:

PostgreSQL

gateway-logger

ms-personas

ms-consulta

ms-rag

Cuando los microservicios estén implementados, el gateway empezará a redirigir todas las peticiones correctamente.

📊 Cómo ver los logs
Ejecutar:

bash
Copiar código
docker exec -it logs-db psql -U postgres -d logs
Dentro de PostgreSQL:

sql
Copiar código
SELECT * FROM logs;
