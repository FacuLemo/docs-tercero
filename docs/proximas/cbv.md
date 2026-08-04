
## Plan de Clase: Migración de CRUD a Class-Based Views (CBV)

### 1. Introducción: ¿Por qué migrar a CBV?

**Objetivo de esta sección:** Mostrar a la clase que las CBV no son "magia", sino una forma de no repetir el mismo código de validación y renderizado una y otra vez.

* **Menos Boilerplate:** Las funciones CRUD suelen repetir el patrón `if request.method == 'POST'`. Las CBV encapsulan esto.
* **Reusabilidad:** Al ser clases, podemos usar herencia y mixins (como control de acceso) más fácilmente.
* **Código más limpio:** Pasamos de escribir 15 líneas lógicas a declarar 4 o 5 atributos.

### 2. El Escenario Base

Para que la clase no pierda el hilo, partimos de un modelo genérico estándar y su formulario.

```python
# models.py
from django.db import models

class Articulo(models.Model):
    titulo = models.CharField(max_length=100)
    contenido = models.TextField()

    def __str__(self):
        return self.titulo

```

```python
# forms.py
from django import forms
from .models import Articulo

class ArticuloForm(forms.ModelForm):
    class Meta:
        model = Articulo
        fields = ['titulo', 'contenido']

```

---

### 3. Paso a Paso: Refactorizando el CRUD

#### A. CREATE (De Función a `CreateView`)

**Dinámica sugerida:** Muestra la función primero y luego cómo la clase la reemplaza casi por completo definiendo solo el "qué" en lugar del "cómo".

**El código antiguo (Función):**

```python
def crear_articulo(request):
    if request.method == 'POST':
        form = ArticuloForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('lista_articulos')
    else:
        form = ArticuloForm()
    return render(request, 'app/articulo_form.html', {'form': form})

```

**El nuevo código (CBV):**

```python
# views.py
from django.urls import reverse_lazy
from django.views.generic import CreateView
from .models import Articulo
from .forms import ArticuloForm

class ArticuloCreateView(CreateView):
    model = Articulo
    form_class = ArticuloForm
    template_name = 'app/articulo_form.html'
    success_url = reverse_lazy('lista_articulos') 
    # reverse_lazy asegura que la URL se cargue solo cuando sea necesario

```

#### B. READ (De Función a `ListView` y `DetailView`)

Las vistas de lectura son las que más se benefician de la abstracción.

**El nuevo código (CBV):**

```python
from django.views.generic import ListView, DetailView

# Lista todos los artículos
class ArticuloListView(ListView):
    model = Articulo
    template_name = 'app/articulo_list.html'
    context_object_name = 'articulos' # Por defecto Django usaría 'object_list'

# Detalle de un artículo específico
class ArticuloDetailView(DetailView):
    model = Articulo
    template_name = 'app/articulo_detail.html'
    context_object_name = 'articulo' # Por defecto Django usaría 'object'

```

#### C. UPDATE (De Función a `UpdateView`)

**Tip para la clase:** Explica que `UpdateView` es prácticamente idéntica a `CreateView`, con la única diferencia de que Django automáticamente busca el objeto en la base de datos usando el `pk` (Primary Key) que le llega por la URL, y pre-rellena el formulario.

**El nuevo código (CBV):**

```python
from django.views.generic import UpdateView

class ArticuloUpdateView(UpdateView):
    model = Articulo
    form_class = ArticuloForm
    template_name = 'app/articulo_form.html' # ¡Reutilizamos el mismo template de Create!
    success_url = reverse_lazy('lista_articulos')

```

#### D. DELETE (De Función a `DeleteView`)

La vista de eliminación requiere una confirmación por seguridad (método POST).

**El nuevo código (CBV):**

```python
from django.views.generic import DeleteView

class ArticuloDeleteView(DeleteView):
    model = Articulo
    template_name = 'app/articulo_confirm_delete.html'
    success_url = reverse_lazy('lista_articulos')

```

---

### 4. La Conexión: Actualizando `urls.py`

Este es el punto donde los alumnos suelen tropezar. Las URLs en Django esperan *funciones* llamables, no clases. Aquí es donde entra el método `.as_view()`.

```python
# urls.py
from django.urls import path
from .views import (
    ArticuloListView, 
    ArticuloDetailView, 
    ArticuloCreateView, 
    ArticuloUpdateView, 
    ArticuloDeleteView
)

urlpatterns = [
    path('', ArticuloListView.as_view(), name='lista_articulos'),
    path('articulo/<int:pk>/', ArticuloDetailView.as_view(), name='detalle_articulo'),
    path('articulo/nuevo/', ArticuloCreateView.as_view(), name='crear_articulo'),
    path('articulo/<int:pk>/editar/', ArticuloUpdateView.as_view(), name='editar_articulo'),
    path('articulo/<int:pk>/eliminar/', ArticuloDeleteView.as_view(), name='eliminar_articulo'),
]

```

> `<int:pk>` refiere a la Primary Key, en otras palabras, al id.


---

### Método 1: La forma nativa de las CBV (Uso de Mixins)

**Concepto clave para transmitir:** Un *Mixin* es simplemente una clase pequeña que contiene un comportamiento específico (como verificar un login) y se "mezcla" con nuestra vista principal mediante herencia.

Django ya trae construidos los equivalentes a `@login_required` y `@permission_required` en formato clase.

**El código (Ejemplo con UpdateView):**

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.views.generic import UpdateView
from .models import Articulo
from .forms import ArticuloForm

class ArticuloUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):
    model = Articulo
    form_class = ArticuloForm
    template_name = 'app/articulo_form.html'
    
    # Configuraciones del LoginRequiredMixin
    login_url = '/login/' # Opcional si ya está definido LOGIN_URL en settings.py
    
    # Configuraciones del PermissionRequiredMixin
    permission_required = 'app.change_articulo' # 'app_label.accion_model'

```

> **⚠️ Alerta de error común:** > El orden de herencia en Python (MRO - Method Resolution Order) importa muchísimo. **Los Mixins de seguridad siempre deben ir a la izquierda** de la vista genérica (`UpdateView`). Si los invierten y ponen `(UpdateView, LoginRequiredMixin)`, la vista se ejecutará antes de que el Mixin pueda bloquear el acceso, anulando la seguridad.

