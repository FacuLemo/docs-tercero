
# Clase 7: Modelos y  Base de Datos

**Objetivo:** Reemplazar la lista hardcodeada de Python por una tabla real en la base de datos (SQLite) y gestionarla desde el Panel de Administración.

### 1. El Concepto (ORM)

No vamos a escribir comandos de SQL; vamos a escribir clases de Python, y Django se encargará de traducir eso a SQL por nosotros. A esto se le llama ORM.

De esta manera vamos a reemplazar comandos como `SELECT * FROM tarea WHERE id=4` por `Tarea.objects.get(id=id)`

---

### 2. Crear el Modelo (`models.py`)

Vamos a definir cómo es una "Tarea".
*Archivo: `holamundo/models.py*`

```python
from django.db import models

class Tarea(models.Model):
    titulo = models.CharField(max_length=100)
    completada = models.BooleanField(default=False)
    timestamp = models.DateTimeField(auto_add_now=True)
    
    def __str__(self):
        return self.titulo #O alternativamente: return f"Tarea {self.titulo}"

```

---

### 3. Las Migraciones (El paso de 2 tiempos)

Django sabe que el código cambió, pero la base de datos no. Hay que sincronizarlos.

**Paso 3.1: Preparar la migración (`makemigrations`)**
Genera el archivo con las instrucciones de cambio.
corremos en consola:

```bash
python manage.py makemigrations

```

*Deberían ver:* `Create model Tarea`.

**Paso 3.2: Aplicar la migración (`migrate`)**
Ejecuta esas instrucciones en la base de datos real.

```bash
python manage.py migrate

```

*Deberían ver:* Una lista de `OK`, incluyendo la de nuestra app.

**Paso 3.3: Crear un registro desde la shell (`shell`)**
Accederemos a un entorno de ejecución para crear un registro usando el ORM de Django.

```bash
python manage.py migrate

```

*Deberían ver:* Datos de la ejecución de la consola interna, prompt para escribir `>>`

Una vez dentro instanciamos nuestro modelo y lo sincronizamos con la base de datos:

```python
from holamundo.models import Tarea
Tarea.objects.create(titulo="Pasear al perro")
exit()
```
Podremos ver que el registro se creó correctamente si accedemos a la tabla `holamundo_tarea` desde SQLite

---


### 4. Conectar Base de Datos con la Vista

El último paso es dejar de usar la lista falsa en `views.py` y traer las tareas que acaban de crear en el Admin.

*Archivo: `holamundo/views.py*`

```python
from django.shortcuts import render
from .models import Tarea  # <--- IMPORTANTE: Importar el modelo

def saludo(request):
    # ANTES: lista_tareas = ['Aprender Python', ...]
    
    # AHORA: Consultamos a la base de datos (SELECT * FROM Tarea)
    tareas_desde_db = Tarea.objects.all()
    
    datos = {
        'nombre_usuario': 'Profe Django',
        'tareas': tareas_desde_db, # Pasamos los objetos reales
        # 'tiene_tareas' ya no es necesario hardcodearlo, 
        # podemos usar {% if tareas %} en el template directamente.
    }

    return render(request, 'holamundo/index.html', datos)

```

> **Nota:** No hace falta tocar el HTML (`index.html`) porque el bucle `{% for tarea in tareas %}` funciona igual, ya sea iterando una lista de textos o una lista de objetos (gracias al método `__str__` que definimos).

---

### Errores Comunes

1. **"no such table: holamundo_tarea" (OperationalError):**
* *Causa:* Hicieron el `makemigrations` pero se olvidaron del `migrate`. La tabla no existe en la BD.


3. **NameError: name 'Tarea' is not defined (en views.py):**
* *Causa:* Falta la línea `from .models import Tarea`.


4. **En el Admin sale "Tarea object (1)" en lugar del nombre:**
* *Causa:* Olvidaron agregar el método `def __str__(self):` en el modelo. Funciona, pero es feo.


