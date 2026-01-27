
# Segunda clase: Creando un proyecto en Django

### 1. Creación del entorno virtual
Ahora que ya sabemos qué es Django, podemos empezar por instalarlo y crear un proyecto. Al ser Django un framework de python, el mismo debe ser instalado a través de un gestor de paquetes del lenguaje, en nuestro caso, pip. Para hacerlo, lo instalaremos dentro de un entorno virtual para que los paquetes estén aislados y no se mezclen con otros proyectos locales.

```bash
python3 -m venv venv
source venv/bin/activate
```

!!! note "Si estás en windows..."
    Si estás trabajando desde windows, deberás correr los comandos de la siguiente manera:
    ```bash
    python -m venv venv
    .\venv\Scripts\activate
    ```

> Debería aparecer `(venv)` al inicio de la línea de comandos.

> Podremos ver una lista de pip vacía luego del pip list si todo salió bien.

---

### 2. Instalación de Django

Ahora tenemos que instalar los paquetes de Django. Lo hacemos escribiendo en la consola:

```bash
pip install django

```

---

### 3. Creación del Proyecto

Tenemos ahora que crear un proyecto. Este es el contenedor principal de toda nuestra aplicación web.
Lo haremos con el siguiente comando:

```bash
django-admin startproject primer-django .

```

> El comando es django-admin startproyect «Nombre del proyecto» «Carpeta contenedora»

> Al colocar el punto al final no estamos creamos otra carpeta dentro de nuestro proyecto, sino usando la carpeta en donde estamos, manteniendo la estructura más limpia y sin tantas anidaciones.

---

### 4. Estructura de Archivos (Breve repaso)

Abre tu editor de código y muestra rápidamente:

* **`manage.py`**: El control remoto del proyecto. No se toca mucho, pero se usa para ejecutar comandos.
* **`settings.py`**: El centro de control (base de datos, apps instaladas, idioma).
* **`urls.py`**: La "recepción" del sitio. Decide a dónde va cada petición web.

---

### 5. Primera Ejecución (Sanity Check)

Antes de seguir, probamos que el cohete despegue; que la aplicación inicie.

```bash
python3 manage.py runserver

```
!!! note "¿manage.py?"
    Si bien creamos el proyecto con django-admin, de ahora en más para ejecutar comandos los usaremos a través de ejecutar el script de manage.py.


* Abrir navegador en: `http://127.0.0.1:8000/`
* Si ven el cohete de Django, es porque la instalación fue exitosa.
* **Para detener:** `Ctrl + C`.
* **Para salir del venv:** escriben por consola: `deactivate`.

---

### 6. Creación de una Aplicación (App)

Tenemos creado nuestro proyecto. Para empezar a desarrollar funciones, crearemos lo que Django llama una aplicación.
En aplicación será una carpeta que contiene distintos módulos orientados para funciones específicas.

* **Proyecto:** El sitio entero (ej. "Sistema Escolar").
* **App:** Una funcionalidad específica (ej. "Alumnos", "Pagos", "Blog").

Creamos la aplicación con el siguiente comando:

```bash
python manage.py startapp nombre_app

```


---

### 7. Conectar la App al Proyecto 

Si bien creamos la app, Django no la registrará hasta que no se añada dentro de la configuración.

1. Ir al archivo `settings.py`.
2. Buscar la lista `INSTALLED_APPS`.
3. Agregar el nombre de la app al final:

```py title="primer-django/settings.py"
INSTALLED_APPS = [
    'django.contrib.admin',
    # ... otras apps ...
    'nombre_app',  # <--- ¡Importante no olvidar la coma!
]

```

---

### Resumen de Comandos Rápidos

| Acción | Comando |
| --- | --- |
| Crear entorno | `python -m venv venv` |
| Activar entorno (Win) | `.\venv\Scripts\activate` |
| Instalar Django | `pip install django` |
| Nuevo Proyecto | `django-admin startproject nombre .` |
| Nueva App | `python manage.py startapp nombre` |
| Correr Servidor | `python manage.py runserver` |

---
