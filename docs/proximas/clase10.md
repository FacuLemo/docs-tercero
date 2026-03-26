# Clase 10: Editar y Eliminar desde las vistas (CRUD Completo)

La clase anterior creamos la vista que nos permitia crear un registro, en esta clase crearemos las que nos permiten modificar tareas existentes o borrarlas.

### 1. URLs Dinámicas (`urls.py`)

Necesitamos rutas que acepten un número variable (el ID de la tarea).

*Archivo: `holamundo/urls.py*`

```python
urlpatterns = [
    # ... rutas anteriores ...
    path('editar/<int:id>/', views.editar, name='editar'),
    path('eliminar/<int:id>/', views.eliminar, name='eliminar'),
]

```

> Al definir el path de esa manera, haremos que Django interprete una variable llama `id` de tipo `int` y se la pasa a la vista.
> Ej. `localhost:8000/eliminar/4/` Interpreta correctamente el 4 como variable `id`.

---

### 2. Editar Tarea (Update)

Es muy parecido a "Crear", pero con una diferencia clave: el formulario no empieza vacío, sino lleno con los datos de la tarea.

**2.1. La Vista (`views.py`)**
Necesitamos importar `get_object_or_404` para manejar el caso de que inventen un ID que no existe.

```python
from django.shortcuts import render, redirect, get_object_or_404 # <--- Nuevo import

def editar(request, id):
    # 1. Buscar la tarea (o dar error 404 si no existe)
    tarea = get_object_or_404(Tarea, id=id)

    if request.method == 'POST':
        # 2. Clave: Pasamos 'instance=tarea' para que actualice ESA tarea
        formulario = TareaForm(request.POST, instance=tarea)
        if formulario.is_valid():
            formulario.save()
            return redirect('saludo')
    else:
        # 3. GET: Llenamos el form con los datos actuales
        formulario = TareaForm(instance=tarea)

    return render(request, 'holamundo/editar.html', {'form': formulario})

```

**2.2. El Template (`editar.html`)**
Es casi idéntico al de crear. Puedes copiar y pegar `crear.html`, solo cambiando el título.

```html
{% extends 'holamundo/base.html' %}
{% block contenido %}
    <h2>Editar Tarea</h2>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Actualizar Cambios</button>
    </form>
{% endblock %}

```

---

### 3. Eliminar Tarea (Delete)

 Por seguridad y para evitar borrados accidentales ejecutaremos la lógica de borrado en un POST.

**3.1. La Vista (`views.py`)**

```python
def eliminar(request, id):
    tarea = get_object_or_404(Tarea, id=id)

    if request.method == 'POST':
        # Si confirman (aprietan el botón rojo)
        tarea.delete() #Se llama al metodo delete() propio del objeto obtenido (del registro)
        return redirect('saludo')
    
    # Si es GET, mostramos la página de "¿Estás seguro?"
    return render(request, 'holamundo/eliminar.html', {'tarea': tarea}) #Como contexto enbiamos el objeto tarea.

```

**3.2. El Template de Confirmación (`eliminar.html`)**

```html
{% extends 'holamundo/base.html' %}
{% block contenido %}
    <h2>¿Eliminar Tarea?</h2>
    <p>Estás a punto de borrar: <strong>{{ tarea.titulo }}</strong></p>
    
    <form method="post">
        {% csrf_token %}
        <button type="submit" style="color: red;">Sí, eliminar</button>
        
        <a href="{% url 'saludo' %}">Cancelar</a>
    </form>
{% endblock %}

```

---

### 4. Actualizar el Listado Principal (`index.html`)

Ahora debemos poner los botones de acción al lado de cada tarea.

No sirve hacerlo en la barra de navegación ya que necesitamos trabajar con un id específico.

*Archivo: `holamundo/templates/holamundo/index.html*`

```html
{% for tarea in tareas %}
    <li>
        {{ tarea.titulo }} 
        {% if tarea.completada %}(✅){% endif %}
        
        <small>
            <a href="{% url 'editar' tarea.id %}">Editar</a> | 
            <a href="{% url 'eliminar' tarea.id %}">Eliminar</a>
        </small>
    </li>
{% endfor %}

```

---


### Errores Comunes (Troubleshooting)

1. **NoReverseMatch:** `Reverse for 'editar' with arguments '('',)' not found.`
* **Causa:** En el template `index.html` pusieron `{% url 'editar' %}` sin pasarle el ID. Debe ser `{% url 'editar' tarea.id %}`.


2. **El formulario crea una tarea NUEVA en vez de editar la vieja:**
* **Causa:** En la vista `editar`, al guardar el form (`TareaForm(request.POST)`), se olvidaron de poner `instance=tarea`. Sin `instance`, Django cree que es una creación nueva.


3. **Error 404 al intentar editar:**
* **Causa:** Están usando un ID en la URL que ya borraron de la base de datos.


4. **TypeError: editar() missing 1 required positional argument: 'id'**
* **Causa:** Definieron la vista como `def editar(request):` olvidando el parámetro `id`.

---
