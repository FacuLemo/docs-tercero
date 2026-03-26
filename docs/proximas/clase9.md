
# Clase 9: Formularios para crear datos

Ya conocimos el panel de admin que da acceso a un CRUD para los superusuarios. Ahora vamos a crear un crud como nosotros querramos, directamente desde las vistas a través de endpoints personalizados.

Para eso, crearemos los Formularios y los implementaremos en nuestras vistas y templates.

### 1. Crear el Formulario (`forms.py`)

*Crearemos el formulario en: `holamundo/forms.py*`
Django no crea este archivo por defecto. Tenemos que crearlo en la carpeta de la app `holamundo`.

```python
from django import forms
from .models import Tarea

class TareaForm(forms.ModelForm):
    class Meta:
        model = Tarea
        fields = ['titulo', 'completada'] # Los campos que queremos que el usuario llene

```

> TareaForm, al heredar de `ModelForm`, podemos espeficicarle un modelo y Django ya va a saber qué tipo de dato será cada uno de los campos que le digamos. En nuestro caso, sabrá que `titulo` es un Charfield y `completada` un bool.

---

### 2. La Vista de Creación (`views.py`)

Ahora la lógica de nuestras vistas se vuelve un poco más compleja. Tenemos que manejar dos situaciones en una misma función:

1. **GET:** Cuando el usuario entra a la página (mostrar formulario vacío, a ser rellenado).
2. **POST:** Cuando el usuario le da a "Enviar" (recibir datos y guardar en la db).

*Archivo: `holamundo/views.py*`

```python
from django.shortcuts import render, redirect # <--- 1. Importar redirect
from .forms import TareaForm # <--- 2. Importar el formulario

def crear(request):
    if request.method == 'POST':
        # Si enviaron datos, llenamos el formulario con ellos
        formulario = TareaForm(request.POST)
        if formulario.is_valid():
            formulario.save() # ¡Guardar en BD!
            return redirect('saludo') # Volver al inicio
    else:
        # Si es GET (primera vez), formulario vacío
        formulario = TareaForm()

    return render(request, 'holamundo/crear.html', {'form': formulario}) #Pasamos el form como contexto

```

---

### 3. El Template del Formulario (`crear.html`)

Creamos un nuevo archivo HTML para mostrar el formulario.

*Archivo: `holamundo/templates/holamundo/crear.html*`

```html
{% extends 'holamundo/base.html' %}

{% block titulo %} Crear Tarea {% endblock %}

{% block contenido %}
    <h2>Nueva Tarea</h2>
    
    <form method="post">
        {% csrf_token %} {{ form.as_p }} 
        
        <button type="submit">Guardar Tarea</button>
    </form>
{% endblock %}

```

> **Puntos Críticos:**
> 1. `method="post" en la etiqueta form de html`: Sin esto, los datos viajan por la URL (inseguro y malformado para guardar).
> 2. `{% csrf_token %}`: Es el guardia de seguridad de Django. Si no está, Django rechazará el formulario para evitar ataques de hackers (Cross Site Request Forgery).
> 3. `{{ form.as_p }}`: "renderizame el formulario como párrafos (<p>)".
> 

---

### 4. La URL y el Enlace

Finalmente, conectamos todo.

**4.1. `urls.py` de la app**

```python
# ... imports ...
urlpatterns = [
    path('/', views.saludo, name='saludo'),
    path('acerca/', views.acerca, name='acerca'),
    path('crear/', views.crear, name='crear'), # <--- Nueva ruta
]

```

**4.2. Agregar botón en `index.html**`
Para que sea fácil llegar, ponemos un enlace en la página principal.

```html
<a href="{% url 'crear' %}">➕ Agregar Nueva Tarea</a>
<hr>

```

---

### 5. Prueba Final de funcionamiento

1. Ir al Home (`localhost:8000/`).
2. Clic en "Agregar Nueva Tarea".
3. Llenar el título (ej: "Prueba Forms") y marcar el checkbox.
4. Clic en Guardar.
5. **Resultado esperado:** Django debe redirigir al Home automáticamente y la nueva tarea debe aparecer en la lista.

---

### Errores Comunes (Troubleshooting)

1. **Forbidden (403) - CSRF verification failed:**
* *Causa:* Se olvidaron de poner la etiqueta `{% csrf_token %}` dentro del `<form>`.


2. **Los datos no se guardan (vuelve a mostrar el form vacío):**
* *Causa:* En la vista, olvidaron poner `formulario.save()`.
* *Causa 2:* El `if request.method == 'POST':` está mal escrito (ej: minúsculas 'post').


3. **No redirige después de guardar:**
* *Causa:* Falta el `return redirect('saludo')` o no importaron `redirect` arriba.


4. **Error "TareaForm is not defined":**
* *Causa:* No importaron la clase en `views.py` (`from .forms import TareaForm`).



---