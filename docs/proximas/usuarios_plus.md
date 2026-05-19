Aquí tienes la guía paso a paso y los ejemplos de código diseñados para que los alumnos puedan implementar la autenticación avanzada y los permisos en sus proyectos, manteniendo el enfoque en las Vistas Basadas en Funciones (FBVs) y utilizando el catálogo de videojuegos como referencia práctica.

### Clase 4: Autenticación y Autorización Avanzada (Guía Práctica)

Esta guía detalla cómo extender el sistema de usuarios nativo de Django y cómo proteger las rutas y la interfaz gráfica de forma eficiente.

---

#### Paso 1: Configurar el Custom User Model

Es una excelente práctica en Django definir un modelo de usuario personalizado al inicio del proyecto. Esto nos permitirá agregar campos extra, como un avatar o un número de teléfono.

**1.1 Crear el modelo (en `models.py` de tu aplicación de usuarios):**

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class UsuarioPersonalizado(AbstractUser):
    # Heredamos todos los campos nativos (username, email, password, etc.)
    telefono = models.CharField(max_length=15, blank=True, null=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)

    def __str__(self):
        return self.username

```

**1.2 Actualizar las configuraciones (en `settings.py`):**
Le indicamos a Django que este será el nuevo modelo de usuario por defecto.

```python
# Le decimos a Django: "Usa este modelo en lugar del tuyo"
# Formato: 'nombre_de_la_app.NombreDelModelo'
AUTH_USER_MODEL = 'usuarios.UsuarioPersonalizado'

```

---

#### Paso 2: Adaptar los Formularios de Registro

El formulario por defecto `UserCreationForm` apunta al usuario clásico de Django. Debemos crear uno propio que apunte a nuestro `UsuarioPersonalizado`.

**En `forms.py`:**

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from .models import UsuarioPersonalizado

class RegistroUsuarioForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = UsuarioPersonalizado
        # Añadimos nuestros campos personalizados a los que ya trae por defecto
        fields = UserCreationForm.Meta.fields + ('email', 'telefono', 'avatar',)

```

---

#### Paso 3: Crear la Vista de Registro (FBV)

Aprovechando que la clase domina las funciones, gestionaremos el flujo GET/POST del nuevo formulario.

**En `views.py`:**

```python
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import RegistroUsuarioForm

def registrar_usuario(request):
    if request.method == 'POST':
        form = RegistroUsuarioForm(request.POST, request.FILES) # request.FILES es vital para el avatar
        if form.is_valid():
            form.save()
            messages.success(request, '¡Cuenta creada con éxito! Ya puedes iniciar sesión.')
            return redirect('login')
    else:
        form = RegistroUsuarioForm()
    
    return render(request, 'registration/registro.html', {'form': form})

```

---

#### Paso 4: Proteger Rutas en el Backend (Decoradores)

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

#### Paso 5: Ocultamiento Condicional en el Frontend (Templates)

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

### Cómo solucionarlo en `todolist/models.py`

**1. Elimina la importación directa del modelo User.**
Seguramente en ese archivo tienes algo como esto, que debes **borrar**:

```python
# ELIMINAR ESTA LÍNEA
from django.contrib.auth.models import User

```

**2. Importa las configuraciones (settings) de Django.**
En lugar de importar el modelo, importamos las configuraciones globales:

```python
# AGREGAR ESTA LÍNEA
from django.conf import settings

```

**El código corregido debe quedar así:**

```python
class Tarea(models.Model):
    # ... otros campos ...
    responsable = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)

```