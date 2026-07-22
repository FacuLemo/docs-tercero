#### Proteger Rutas en el Backend (Decoradores)

Para asegurar que un visitante no pueda crear, editar o eliminar un videojuego del catálogo sin estar autenticado o sin tener el rol adecuado, usamos decoradores.

**En `views.py` (de la app de videojuegos):**

```python
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required, permission_required
from .models import Videojuego
from .forms import VideojuegoForm

# Solo usuarios logueados pueden ver esta vista
@login_required
def listar_videojuegos_privados(request):
    juegos = Videojuego.objects.all()
    return render(request, 'catalogo/lista_privada.html', {'juegos': juegos})

# Solo logueados Y que además tengan el permiso específico de añadir
@login_required

@permission_required('catalogo.add_videojuego', raise_exception=True) #app.action_model (todo minúscula)

def crear_videojuego(request):
    if request.method == 'POST':
        form = VideojuegoForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('lista_juegos')
    else:
        form = VideojuegoForm()
    return render(request, 'catalogo/formulario_juego.html', {'form': form})

```

*Nota didáctica: El permiso se estructura como `nombre_app.accion_modelo`. Django los crea automáticamente al hacer migraciones (add, change, delete, view).*

---

#### Ocultamiento Condicional en el Frontend (Templates)

Si un usuario no tiene permisos para crear un juego, no deberíamos siquiera mostrarle el botón de "Crear Videojuego". Usaremos la variable global `{{ perms }}` de los templates.

**En tu archivo HTML (ej. `lista_juegos.html`):**

```html
<h2>Catálogo de Videojuegos</h2>

<!-- Mostrar el saludo y el avatar si está logueado -->
{% if user.is_authenticated %}
    <p>Bienvenido, {{ user.username }}</p>
    {% if user.avatar %}
        <img src="{{ user.avatar.url }}" alt="Avatar" width="50">
    {% endif %}
{% endif %}

<!-- Ocultamiento condicional basado en permisos -->
{% if perms.catalogo.add_videojuego %}
    <a href="{% url 'crear_videojuego' %}" class="btn btn-success">
        + Añadir Nuevo Videojuego
    </a>
{% else %}
    <p class="text-muted">No tienes permisos para añadir juegos al catálogo.</p>
{% endif %}

<ul>
    {% for juego in juegos %}
        <li>
            {{ juego.titulo }}
            
            <!-- Botón de borrar solo visible si tiene permiso de borrado -->
            {% if perms.catalogo.delete_videojuego %}
                <a href="{% url 'borrar_videojuego' juego.id %}" style="color: red;">Eliminar</a>
            {% endif %}
        </li>
    {% endfor %}
</ul>

```
