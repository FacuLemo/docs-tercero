
# Clase 3: Nuestro Hola Mundo con vistas y urls

###  El Flujo de Django
Ahora empezaremos a crear nuestras propias pantallas en Django. Deberemos crear una vista con su respectiva lógica deseada y registrar esa vista en un endpoint en `urls.py`.

El flujo ocn el que funciona django, es el siguiente:
`Usuario (Navegador)` -> `urls.py` (interpreta la url) -> `views.py` (Ejecula la lógica y retorna) -> `Respuesta`

---

### Paso 1: La Vista 

Vamos al archivo `views.py` de tu aplicación (ej. `holamundo/views.py`).

**Objetivo:** Crear una función que reciba una petición y devuelva texto.

```python
from django.shortcuts import render
from django.http import HttpResponse # <--- 1. Importar esto

# 2. Crear la función
def saludo(request):
    return HttpResponse("¡Hola Mundo desde Django!")

```

>  `request` es obligatorio como primer parámetro, aunque no se use todavía. Son datos que envía el navegador al acceder a la vista.

---

### Paso 2: URLs de la App 

Django no crea un `urls.py` dentro de la app por defecto. Hay que crearlo.

1. Crear archivo `urls.py` dentro de la carpeta `holamundo`.
2. Escribir el siguiente código:

```python
from django.urls import path
from . import views  # Importamos las vistas de ESTA carpeta (.)

urlpatterns = [
    # path(ruta_url, funcion_vista, nombre_interno)
    path('hola/', views.saludo, name='saludo'),
]

```

> La ruta `'hola/'` en este caso será lo que se escriba después del dominio (ej: `...:8000/hola/`). Si dejas las comillas vacías `''`, será la página de inicio de esa app.

---

### Paso 3: Conectar al Proyecto

Ahora hay que decirle al proyecto principal que incluya las rutas de nuestra app.
Ir a `urls.py` **de la carpeta del proyecto** (donde está `settings.py`).

```python
from django.contrib import admin
from django.urls import path, include # <--- 1. Importar include

urlpatterns = [
    path('admin/', admin.site.urls),
    # 2. Conectar la app
    path('', include('holamundo.urls')), 
]

```

> Si en el `path` pones `path('tienda/', include(...))`, todas las URLs de la app tendrán el prefijo "tienda" (ej: `...:8000/tienda/hola/`). Al no incluir nada, todas las urls de la app `holamundo` estarán sobre la raíz del proyecto entero.

---

### Paso 4: Prueba de Fuego

1. Asegurar que el servidor esté corriendo (`python manage.py runserver`).
2. Ir al navegador.
3. Entrar a: `http://127.0.0.1:8000/hola/`

**Resultado esperado:** Una página blanca simple que dice "¡Hola Mundo desde Django!".

---

### Errores Comunes

1. **Error:** `ModuleNotFoundError: No module named 'nombre_app.urls'`
* **Causa:** Se olvidaron de crear el archivo `urls.py` dentro de la carpeta de la app.


2. **Error:** `Page not found (404)`
* **Causa:** Están entrando a `127.0.0.1:8000` (la raíz) pero definimos la ruta como `'hola/'`. Deben agregar `/hola/` al final de la URL.


3. **Error:** `AttributeError: module '...views' has no attribute 'saludo'`
* **Causa:** No guardaron el archivo `views.py` o escribieron mal el nombre de la función.

