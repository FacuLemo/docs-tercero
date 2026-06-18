Los **Context Processors** (procesadores de contexto) son una de esas herramientas en Django que marcan el salto de un principiante a un desarrollador intermedio.

En términos sencillos: son funciones de Python que se ejecutan antes de renderizar un template y sirven para **inyectar variables globales** en todos tus archivos HTML.

Imagina que tienes una barra de navegación que muestra el nombre del usuario, el año actual en el pie de página, o las categorías de tu tienda online. En lugar de tener que pasar esas variables desde *todas* y cada una de tus vistas (views), un context processor lo hace automáticamente de fondo.

Aquí tienes la clase paso a paso para implementar uno desde cero. Vamos a crear un procesador que inyecte el "Año Actual" y un "Mensaje de Bienvenida" en todas las páginas.

> **Nota importante sobre rendimiento:** Debido a que esta función se ejecuta en *cada* petición (cada vez que alguien carga una página), debes evitar hacer cálculos pesados o consultas complejas a la base de datos dentro de un context processor, ya que podría ralentizar todo tu sitio web.



1. **Crear el archivo del procesador:** Mantén tu código organizado.
Por convención, los procesadores de contexto se guardan en un archivo llamado `context_processors.py` dentro del directorio de tu aplicación (donde están `views.py` y `models.py`).

Si tu aplicación se llama `core`, crea el archivo allí: `core/context_processors.py`.


2. **Escribir la función de Python:** El corazón del context processor.
Abre el archivo `context_processors.py` que acabas de crear.

La regla de oro de un context processor es muy simple: **debe recibir el objeto `request` como argumento y debe retornar siempre un diccionario.**

```python
    from datetime import datetime

    def datos_globales(request):
        # 1. Calculamos el año actual
        ano_actual = datetime.now().year
        
        # 2. Creamos un mensaje dinámico (opcionalmente usando el request)
        if request.user.is_authenticated:
            mensaje = f"¡Hola de nuevo, {request.user.username}!"
        else:
            mensaje = "¡Bienvenido, visitante!"
            
        # 3. Retornamos el diccionario con las variables a inyectar
        return {
            'current_year': ano_actual,
            'welcome_message': mensaje
        }
```
###  Registrar el procesador en settings.py
    Django no sabrá que esta función existe hasta que se lo indiques. Abre tu archivo `settings.py` y busca la lista `TEMPLATES`. Dentro de ella, en la sección `OPTIONS`, verás una lista llamada `context_processors`.

    Añade la ruta completa a tu nueva función al final de esa lista:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                # ACA el context processor:
                'core.context_processors.datos_globales', 
            ],
        },
    },
]
```
 ### Usar las variables en tus templates
    ¡Eso es todo! Ahora puedes usar las variables `current_year` y `welcome_message` en **cualquier** template de tu proyecto, sin importar qué vista lo esté renderizando.

    Por ejemplo, en tu `base.html`:

    

```html
{{ welcome_message }}
    <main>
        <!-- El contenido de la página -->
        {% block content %}
        {% endblock %}
    </main>

    <footer>
        <p>&copy; {{ current_year }} Mi Proyecto Django. Todos los derechos reservados.</p>
    </footer>
</body>
```

---------------------

## 1. Casos de Uso del Mundo Real

Después de enseñar el "Año Actual", es ideal mostrar ejemplos que los alumnos verán en su día a día. Puedes agregar una sección de código mostrando cómo se verían estos procesadores:

* **Carrito de compras:** Un procesador que consulte cuántos ítems tiene el usuario activo en su carrito de compras para mostrar un indicador numérico (`badge`) en la barra de navegación.

* **Menú de categorías dinámico:** Un e-commerce o blog necesita que el menú superior muestre siempre las categorías de productos o artículos. En lugar de pasarlo en cada vista, un procesador consulta `Etiqueta.objects.all()` y lo inyecta globalmente.

* **Notificaciones no leídas:** Un contador de alertas o mensajes privados que se muestra junto al avatar del usuario.

## 2. El peligro del rendimiento (y cómo solucionarlo)

Este es un tema crítico. Si agregas un procesador que hace una consulta a la base de datos (por ejemplo, obtener todas las categorías del menú) y tu sitio recibe 1,000 visitas por minuto, estarás haciendo 1,000 consultas por minuto a la base de datos **solo para renderizar el menú**.

Para alargar la clase, enseña cómo usar **Caché** dentro de un context processor:

```python
from django.core.cache import cache
from .models import Etiqueta

def etiquetas_globales(request):
    # Intentamos obtener las categorías de la memoria caché
    etiquetas = cache.get('menu_etiquetas')
    
    # Si no están en caché, consultamos la base de datos y guardamos el resultado
    if etiquetas is None:
        etiquetas = Etiqueta.objects.all()
        # Guardamos en caché por 1 hora (3600 segundos)
        cache.set('menu_etiquetas', etiquetas, 3600)
        
    return {'etiquetas_menu': etiquetas}

```


## 4. Los Procesadores Nativos (No reinventar la rueda)

Muchos alumnos crean procesadores de contexto para cosas que Django ya hace. Puedes dedicar una sección a explorar el archivo `settings.py` e inspeccionar los procesadores que vienen por defecto.

* `django.contrib.auth.context_processors.auth`: Es el que inyecta la variable `{{ user }}` y `{{ perms }}` en todo el sitio.
* `django.template.context_processors.request`: Permite usar la variable `{{ request }}` en cualquier HTML (útil para saber la URL actual con `{{ request.path }}`).
* `django.contrib.messages.context_processors.messages`: Es el que inyecta la lista `{{ messages }}` para mostrar alertas verdes o rojas (flashes) al usuario tras guardar un formulario.