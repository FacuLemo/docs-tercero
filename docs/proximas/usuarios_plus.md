
### Autenticación y Autorización Avanzada 

Hoy aprendemos cómo extender el sistema de usuarios nativo de Django para tener disponibles campos propios sin perder las herramientas que nos da Django por defecto.

---

#### Paso 1: Configurar el Custom User Model

Es una excelente práctica en Django definir un modelo de usuario personalizado al inicio del proyecto. Esto nos permitirá agregar campos extra, como un avatar o un número de teléfono.

Como nosotros ya tenemos el proyecto inicializado, debemos borrar la base de datos y las migraciones existentes para poder empezar de "cero".

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
            return redirect('login')
    else:
        form = RegistroUsuarioForm()
    
    return render(request, 'registration/registro.html', {'form': form})

```

---

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