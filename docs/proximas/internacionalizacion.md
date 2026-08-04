

## Paso 1: Configurar el motor en `settings.py`

Primero, le decimos a Django qué idiomas vamos a soportar y dónde guardaremos las traducciones.

1. Abre `settings.py` y asegúrate de tener estas variables:

```python
from django.utils.translation import gettext_lazy as _
import os

# 1. Activar i18n y l10n
USE_I18N = True
USE_L10N = True

# 2. Idioma por defecto (si no hay prefijo en la URL o cookie)
LANGUAGE_CODE = 'en-us'

# 3. Idiomas disponibles en el sitio
LANGUAGES = [
    ('en', _('English')),
    ('es', _('Spanish')),
]

# 4. Carpeta donde vivirán las traducciones
LOCALE_PATHS = [
    os.path.join(BASE_DIR, 'locale'),
]

```

2. En el mismo archivo, añade el `LocaleMiddleware`. **Atención aquí:** el orden es estricto. Debe ir justo después de `SessionMiddleware`.

```python
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.locale.LocaleMiddleware', # <-- AÑADIR AQUÍ
    'django.middleware.common.CommonMiddleware',
    # ... otros middlewares
]

```

---

## Paso 2: Configurar las URLs por idioma

Para que el usuario pueda visitar `/es/contacto/` o `/en/contacto/`, debemos modificar el archivo `urls.py` principal del proyecto.

1. Abre tu `urls.py` principal y envuelve tus rutas con `i18n_patterns`:

```python
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.i18n import i18n_patterns

# Rutas que NO cambian de idioma (ej. APIs)
urlpatterns = [
    path('api/', include('api.urls')), 
]

# Rutas que SÍ tendrán el prefijo de idioma (/es/, /en/)
urlpatterns += i18n_patterns(
    path('admin/', admin.site.urls),
    path('', include('core.urls')), # Las URLs de tu app principal
)

```

---

## Paso 3: Marcar los textos en Python

Ahora vamos a decirle a Django qué cadenas de texto del código fuente deben ser traducidas. Usamos `gettext_lazy` en modelos o formularios para que la traducción ocurra cuando el usuario carga la página, no cuando arranca el servidor.

1. Abre un archivo `models.py` o `views.py`:

```python
from django.db import models
from django.utils.translation import gettext_lazy as _

class Product(models.Model):
    # Marcamos "Product Name" para traducción
    name = models.CharField(_('Product Name'), max_length=100)
    
    class Meta:
        verbose_name = _('Product')
        verbose_name_plural = _('Products')

```

---

## Paso 4: Marcar los textos en las plantillas HTML

En el frontend, el proceso es ligeramente distinto. Hay que cargar la librería de etiquetas y usar `translate`.

1. Abre un archivo `.html` de tu proyecto:

```django
{% load i18n %} 

<!DOCTYPE html>
<html>
<body>
    <h1>{% translate "Welcome to our store" %}</h1>

    <p>
        {% blocktranslate with item_name=product.name %}
            You are viewing the product: {{ item_name }}
        {% endblocktranslate %}
    </p>
</body>
</html>

```
> blocktranslate es por si tu mensaje tiene variables que necesiten ser traducidas.
---

## Paso 5: Crear la carpeta y extraer mensajes

Es hora de usar la terminal. Vamos a buscar todos los textos marcados en los pasos 3 y 4.

1. En la raíz de tu proyecto (donde está `manage.py`), crea una carpeta llamada `locale` (es la que definimos en el Paso 1):
```bash
mkdir locale

```


2. Ejecuta el comando para extraer los textos y generar el archivo para el idioma español (`es`):
```bash
python manage.py makemessages -l es

```


> **Nota:** Si tienes textos en HTML, a veces es útil asegurarte de que Django escanee todo incluyendo la extensión correcta: `python manage.py makemessages -l es -e html,py,txt`



---

## Paso 6: Hacer la traducción manual

El comando anterior creó un archivo en esta ruta: `locale/es/LC_MESSAGES/django.po`.

1. Abre el archivo `django.po` con tu editor de código. Verás bloques como este:

```po
#: core/models.py:6
msgid "Product Name"
msgstr ""

#: core/templates/index.html:8
msgid "Welcome to our store"
msgstr ""

```

2. Tu tarea (o la del traductor) es rellenar las comillas vacías de `msgstr` con la traducción al español:

```po
#: core/models.py:6
msgid "Product Name"
msgstr "Nombre del Producto"

#: core/templates/index.html:8
msgid "Welcome to our store"
msgstr "Bienvenido a nuestra tienda"

```

*Guarda el archivo cuando termines.*

---

## Paso 7: Compilar y probar

Django no lee los archivos `.po` directamente en producción porque son archivos de texto lentos de procesar. Necesita compilarlos a archivos binarios `.mo`.

1. En la terminal, ejecuta:
```bash
python manage.py compilemessages

```


2. Arranca tu servidor:
```bash
python manage.py runserver

```



**¡Listo!**
Ve a tu navegador e ingresa a `http://127.0.0.1:8000/es/`. Deberías ver la página completamente traducida al español. Si cambias la URL a `/en/`, volverá al idioma original (inglés).