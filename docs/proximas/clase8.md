# Clase 8: Personalizando el Admin

Una vez que ya sabemos registrar el modelo de forma básica, podemos usar una forma más profesional que nos permite personalizar la interfaz.

En lugar de usar `admin.site.register(Tarea)`, usaremos el decorador `@admin.register` y una clase `ModelAdmin`.

Modificaremos el archivo `holamundo/admin.py`:

```python
from django.contrib import admin
from .models import Tarea

@admin.register(Tarea)
class TareaAdmin(admin.ModelAdmin):
    # 1. LIST DISPLAY: Qué columnas se ven en la lista general
    list_display = ('id', 'nombre', 'estado', 'fecha_creacion') 
    
    # 2. LIST FILTER: Crea un panel lateral para filtrar resultados
    # Útil para fechas, estados (booleanos) o categorías
    list_filter = ('estado', 'fecha_creacion')
    
    # 3. SEARCH FIELDS: Agrega una barra de búsqueda en la parte superior
    # Busca coincidencias en los campos que indiques aquí
    search_fields = ('nombre', 'descripcion')
    
    # 4. READONLY FIELDS: Campos que se ven pero no se pueden editar
    # Ideal para fechas de creación automáticas o IDs
    readonly_fields = ('fecha_creacion', 'fecha_actualizacion')

```


### ¿Qué logramos con esto?

* **list_filter:** Nos permite navegar rápidamente entre "Tareas completadas" y "pendientes" sin programar nada.
* **search_fields:** Es vital cuando tienes cientos de registros y necesitas encontrar uno específico por su texto.
* **readonly_fields:** Protege la integridad de los datos (evita que alguien modifique "a mano" cuándo se creó un registro).

---

## 6. Para seguir aprendiendo (Documentación)
Estos fueron tan sólo algunos de los atributos de personalización para nuestro panel de administración. Si quieres consultar todo lo que puedes personalizar, consulta la documentación:

* 🔗 **Documentación Oficial de Django Admin (ModelAdmin options):**
[https://docs.djangoproject.com/en/stable/ref/contrib/admin/#modeladmin-options](https://docs.djangoproject.com/en/stable/ref/contrib/admin/%23modeladmin-options)

---

