
#  Introducción a Django REST Framework (DRF)

## Objetivos de la clase

1. Comprender la diferencia entre el desarrollo web con Django tradicional (MVT) y la arquitectura cliente-servidor mediante APIs REST.
2. Configurar un entorno de desarrollo profesional con Django y Django REST Framework (DRF).
3. Entender el flujo de una petición/respuesta HTTP en una API.
4. Crear los primeros endpoints utilizando el decorador `@api_view` y la interfaz navegable (*Browsable API*) de DRF.

---

## Paso 1: Introducción Teórica y Arquitectura REST

### 1.1. Django Tradicional vs. Django REST Framework

* **Django Tradicional (MVT):** El servidor procesa la lógica de negocio, consulta la base de datos y **renderiza código HTML** mediante plantillas (*templates*). El cliente (navegador) recibe la página web completa procesada.
* **Django REST Framework (DRF):** El servidor actúa únicamente como un **proveedor de datos**. La API procesa las solicitudes y responde en un formato ligero y estándar (usualmente **JSON**). El cliente (React, Angular, Vue, aplicaciones móviles, etc.) consume esos datos y se encarga de renderizar la interfaz.

### 1.2. Principios clave de REST

* **Recursos:** Todo elemento accesible mediante la API (ejemplo: un producto, un usuario).
* **Endpoints (URLs):** Rutas que identifican el recurso (ejemplo: `/api/productos/`).
* **Verbos HTTP:** Definen la acción a realizar sobre el recurso:
* `GET`: Recuperar información.
* `POST`: Crear un nuevo recurso.
* `PUT` / `PATCH`: Actualizar un recurso existente (completo o parcial).
* `DELETE`: Eliminar un recurso.


* **Códigos de estado HTTP (Status Codes):** Indican el resultado de la operación:
* `200 OK` / `201 Created`: Éxito.
* `400 Bad Request`: Error de validación o sintaxis enviada por el cliente.
* `404 Not Found`: Recurso no encontrado.
* `500 Internal Server Error`: Error interno del servidor.



---

## Paso 2: Configuración del Entorno de Desarrollo

Vamos a implementar DRF a nuestro proyecto en el cual venimos trabajando.

### 2.1. Crear el entorno virtual e instalar dependencias

```bash
# Instalar Django y Django REST Framework
pip install django djangorestframework

```

### 2.3. Registrar la app y DRF en `settings.py`

Abre `settings.py` e incluye `'rest_framework'` dentro de la lista `INSTALLED_APPS`:

```python

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    #...
    'rest_framework',
]

```

---

## Paso 3: Primeras respuestas con `@api_view`

DRF ofrece el decorador `@api_view` para transformar funciones comunes de Python en **vistas basadas en funciones (FBVs)** capaces de manejar peticiones HTTP y devolver respuestas formateadas mediante la clase `Response`.

### 3.1. Definir la primera vista en `tarea/views.py`

Abre `tarea/views.py` y agrega los siguientes ejemplos:

```python
# tarea/views.py

from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

# 1. Endpoint simple tipo GET
@api_view(['GET'])
def estado_api(request):
    """
    Devuelve un estado general de la API para verificar conectividad.
    """
    data = {
        "mensaje": "¡Bienvenido a la API con Django REST Framework!",
        "estado": "Online",
        "version": 1.0
    }
    return Response(data, status=status.HTTP_200_OK)


# 2. Endpoint que procesa datos recibidos por POST
@api_view(['GET', 'POST'])
def demo_saludo(request):
    """
    Endpoint que responde a GET con una bienvenida y a POST procesando un JSON.
    """
    if request.method == 'GET':
        return Response({"mensaje": "Enviá una petición POST con tu nombre para ser saludado."})

    elif request.method == 'POST':
        # request.data lee automáticamente el cuerpo JSON de la petición
        nombre = request.data.get("nombre", "Anónimo")
        
        if not nombre or nombre == "Anónimo":
            return Response(
                {"error": "Debes proporcionar un 'nombre' en el cuerpo JSON."},
                status=status.HTTP_400_BAD_REQUEST
            )

        respuesta = {
            "mensaje": f"¡Hola, {nombre}! Tu petición POST fue procesada correctamente."
        }
        return Response(respuesta, status=status.HTTP_201_CREATED)

```

---

## Paso 4: Configuración de Rutas (`urls.py`)

Ahora debemos vincular nuestras vistas a URLs para poder invocarlas.

### 4.1. Crear `productos/urls.py`

Crea un archivo llamado `urls.py` dentro de la carpeta `productos/`:

```python
# productos/urls.py

from django.urls import path
from .views import estado_api, demo_saludo

urlpatterns = [
    path('estado/', estado_api, name='estado_api'),
    path('saludo/', demo_saludo, name='demo_saludo'),
]

```

### 4.2. Incluir las URLs de la app en `config/urls.py`

Edita el archivo principal `config/urls.py`:

```python
# config/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('productos.urls')), # Prefijo /api/ para nuestras rutas
]

```

---

## Paso 5: Probar la API (Browsable API)

1. Inicia el servidor de desarrollo:
```bash
python manage.py runserver

```


2. Abre el navegador e ingresa a:
* `[http://127.0.0.1:8000/api/estado/](http://127.0.0.1:8000/api/estado/)`
* `[http://127.0.0.1:8000/api/saludo/](http://127.0.0.1:8000/api/saludo/)`



### ¿Qué notarás en la pantalla?

* **Browsability:** DRF no muestra un JSON crudo plano, sino una interfaz web interactiva que permite ver encabezados HTTP, modificar el contenido enviado y probar solicitudes `POST` mediante un formulario interactivo al pie de página.
* **Formato de Respuesta:** Si consultas esa URL desde una herramienta externa como **Postman**, **Thunder Client** o mediante `curl`, DRF responderá automáticamente con un JSON puro gracias a la negociación de contenido (*Content Negotiation*).

---

## Paso 6: Desafío Práctico para los Alumnos

**Consigna:**

1. Crear una función vista `@api_view(['GET', 'POST'])` llamada `calculadora`.
2. Si recibe una petición `GET`, debe devolver las instrucciones: `"Envía 'num1', 'num2' y 'operacion' (suma, resta, multiplicacion) por POST"`.
3. Si recibe una petición `POST`:
* Debe extraer `num1`, `num2` y `operacion` desde `request.data`.
* Realizar la operación aritmética solicitada y retornar el resultado en un JSON con status `200 OK`.
* Si la operación no es válida o faltan datos, retornar un mensaje de error claro con status `400 BAD_REQUEST`.