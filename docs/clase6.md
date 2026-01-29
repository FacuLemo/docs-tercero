# Clase 6: Herencia de Plantillas y Navegación

**Objetivo:** Crear un archivo maestro (`base.html`) y hacer que todas las páginas hereden su estructura (menú, pie de página, CSS), evitando repetir código.

### 1. Crear la Plantilla Base (`base.html`)

Este archivo tendrá el diseño común de todo el sitio.

*Archivo: `holamundo/templates/holamundo/base.html*`

```html
{% load static %} <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>{% block titulo %}Mi Sitio{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'holamundo/estilos.css' %}">
</head>
<body>

    <nav>
        <a href="{% url 'saludo' %}">Inicio</a> |
        <a href="{% url 'acerca' %}">Acerca de</a>
    </nav>

    <hr>

    <div class="contenido">
        {% block contenido %}
        {% endblock %}
    </div>

    <hr>
    <footer>
        <p>Copyright © 2024 - Clase de Django</p>
    </footer>

</body>
</html>

```

> **Concepto Clave (Bloques):**
> Explica que `{% block nombre %}` ... `{% endblock %}` son "ventanas" o "huecos" que las plantillas hijas podrán rellenar. Todo lo que esté FUERA de los bloques es fijo e inamovible.

---

### 2. Modificar el `index.html` (Heredar)

Vamos a "limpiar" el archivo que hicimos la clase pasada. Borramos todo el HTML estructural y dejamos solo lo específico.

*Archivo: `holamundo/templates/holamundo/index.html*`

```html
{% extends 'holamundo/base.html' %}

{% block titulo %} Inicio - Mi Web {% endblock %}

{% block contenido %}
    <h1>Bienvenido, {{ nombre_usuario }}</h1>

    {% if tiene_tareas %}
        <p>Tienes tareas pendientes:</p>
        <ul>
            {% for tarea in tareas %}
                <li>{{ tarea }}</li>
            {% endfor %}
        </ul>
    {% else %}
        <p>Estás libre.</p>
    {% endif %}
{% endblock %}

```

> **Regla de Oro:** En una plantilla hija (que tiene `extends`), **NO** puede haber código HTML fuera de los `{% block %}`. Si escriben algo fuera, Django lo ignorará o dará error.

---

### 3. Crear una Segunda Página ("Acerca de")

Para probar que la navegación funciona, necesitamos otro destino.

**3.1. La Vista (`views.py`)**

```python
def acerca(request):
    return render(request, 'holamundo/acerca.html')

```

**3.2. La URL (`urls.py` de la app)**

```python
urlpatterns = [
    path('hola/', views.saludo, name='saludo'), # Ya existía
    path('acerca/', views.acerca, name='acerca'), # Nueva
]

```

> **Tip Docente:** Recálcales la importancia del `name='acerca'`. Es el nombre interno que usamos en el template (`{% url 'acerca' %}`). Si cambian la ruta `'acerca/'` por `'sobre-mi/'` en el futuro, el enlace seguirá funcionando automáticamente.

**3.3. El Template (`acerca.html`)**
*Archivo: `holamundo/templates/holamundo/acerca.html*`

```html
{% extends 'holamundo/base.html' %}

{% block titulo %} Acerca de {% endblock %}

{% block contenido %}
    <h1>Acerca de esta clase</h1>
    <p>Estamos aprendiendo a no repetir código usando Herencia.</p>
    <p>Nota como el menú y el footer aparecen solos.</p>
{% endblock %}

```

---

### 4. Práctica en Vivo

1. Pídeles que naveguen a `/hola/` y luego hagan clic en "Acerca de".
2. Deberían ver cómo el contenido central cambia, pero el menú y el footer se mantienen idénticos sin haberlos copiado.

---

### 🚨 Errores Comunes (Troubleshooting)

1. **TemplateSyntaxError: Invalid block tag... 'endblock'**
* *Causa:* Olvidaron poner `{% endblock %}` al final de su contenido en `index.html`.


2. **No se ve el CSS en la página nueva:**
* *Causa:* Probablemente definieron el link del CSS en `index.html` (el viejo) y no en `base.html`. Al borrar el head del index, perdieron el estilo. Deben asegurarse de que el `<link>` esté en el `base.html`.


3. **ReverseMatch at /...**
* *Causa:* En el menú pusieron `{% url 'contacto' %}` pero no definieron ningún `path(..., name='contacto')` en `urls.py`. Django explota si intentas linkear a algo que no existe.



---

### ¿Próximo paso?

Con esto tienen la estructura frontend lista. El proyecto ya parece un sitio real.
Para la siguiente clase (la 4ª interacción), ahora sí recomendaría **Modelos y Base de Datos**.

El flujo sería:

1. "Nos cansamos de escribir las tareas en una lista de Python en `views.py`".
2. Crear el Modelo `Tarea`.
3. Migraciones.
4. Usar el **Admin** para crear tareas reales.
5. Mostrar esas tareas de la BD en el template.