
# Clase 8: Paginación, Filtrado y Búsqueda – Optimizando la API

**Objetivos de la clase:**

* Evitar la sobrecarga del servidor y del cliente limitando la cantidad de resultados devueltos (`PageNumberPagination`).


* Permitir a los clientes consultar subconjuntos de datos específicos integrando `django-filter`.


* Implementar búsquedas dinámicas por texto y ordenamiento de resultados.



---

## Paso 1: El problema de devolver todo (El "Por qué" de la Paginación)

**Explicación para la clase:**
Hasta ahora, cuando hacemos una petición `GET` a nuestro endpoint de, por ejemplo, `/articulos/`, el `ModelViewSet` ejecuta un `Articulo.objects.all()` y devuelve absolutamente todo. Si la base de datos tiene 10 registros, no hay problema. Pero, ¿qué pasa si el proyecto crece y tiene 50.000 artículos? El servidor consumirá muchísima memoria armando ese JSON, la transferencia de red será lentísima, y la aplicación frontend del cliente probablemente colapse al intentar renderizar 50.000 elementos de golpe.

La solución es la **Paginación**: entregar los datos en pequeños bloques o "páginas".

### Implementación con `PageNumberPagination`

La forma más sencilla de aplicarlo en DRF es a nivel global desde el archivo `settings.py`.

**Código de ejemplo (`settings.py`):**

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10  # Cantidad de elementos por página
}

```

*El por qué:* Al configurarlo aquí, todos los listados de la API automáticamente devolverán la data estructurada con metadatos útiles: el `count` total de elementos, y las URLs `next` y `previous` para navegar. El cliente solo debe agregar el parámetro `?page=2` a su petición.

## Paso 2: Encontrar la aguja en el pajar (El "Por qué" del Filtrado)

**Explicación:**
Incluso con los datos paginados, los clientes de nuestra API rara vez quieren navegar página por página si buscan algo específico. Necesitamos permitirles filtrar (por ejemplo, obtener solo los artículos de la categoría "Tecnología" o aquellos con estado "Publicado").

### Integración de `django-filter`

Esta es una librería externa recomendada oficialmente por DRF. Permite hacer coincidencias exactas.

**Código de ejemplo (`views.py`):**

```python
# Primero instalar: pip install django-filter
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import viewsets

class ArticuloViewSet(viewsets.ModelViewSet):
    queryset = Articulo.objects.all()
    serializer_class = ArticuloSerializer
    
    # 1. Habilitamos el backend de filtrado
    filter_backends = [DjangoFilterBackend]
    # 2. Definimos por qué campos exactos queremos permitir filtrar
    filterset_fields = ['categoria', 'estado']

```

*El por qué:* Con estas líneas, la API ahora entiende peticiones como `/articulos/?categoria=2&estado=Publicado`. Todo el trabajo pesado de parsear los parámetros y aplicar los `.filter()` en el ORM lo hace el `DjangoFilterBackend`.

## Paso 3: Búsquedas abiertas por texto



**Explicación:**
El `DjangoFilterBackend` exige que el valor coincida exactamente (ej: buscar el estado exacto). Pero, ¿qué pasa si queremos un buscador general, donde el usuario escriba "Python" y la API devuelva todos los artículos que contengan esa palabra en su título o contenido? Para eso usamos `SearchFilter`.

**Código de ejemplo (Añadiendo al `views.py`):**

```python
from rest_framework.filters import SearchFilter

class ArticuloViewSet(viewsets.ModelViewSet):
    queryset = Articulo.objects.all()
    serializer_class = ArticuloSerializer
    
    # Sumamos SearchFilter a los backends
    filter_backends = [DjangoFilterBackend, SearchFilter]
    filterset_fields = ['categoria', 'estado']
    
    # Campos donde buscará el texto (usando LIKE / ILIKE en la DB)
    search_fields = ['titulo', 'contenido']

```

*El por qué:* Ahora podemos consumir `/articulos/?search=Python`. DRF buscará la palabra de forma parcial y sin importar mayúsculas/minúsculas dentro de los campos definidos en `search_fields`.

## Paso 4: Dejando que el cliente decida el Ordenamiento



**Explicación:**
Finalmente, los usuarios querrán ver los artículos más recientes primero, o tal vez ordenados alfabéticamente. En lugar de forzar un `order_by` fijo en el modelo o en el queryset, le damos el control al frontend mediante `OrderingFilter`.

**Código de ejemplo (Vista final en `views.py`):**

```python
from rest_framework.filters import SearchFilter, OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend

class ArticuloViewSet(viewsets.ModelViewSet):
    queryset = Articulo.objects.all()
    serializer_class = ArticuloSerializer
    
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['categoria', 'estado']
    search_fields = ['titulo', 'contenido']
    
    # Campos por los que el cliente tiene permiso de ordenar
    ordering_fields = ['fecha_creacion', 'titulo']

```

*El por qué:* Esto habilita el parámetro `?ordering=`. El cliente puede pedir `/articulos/?ordering=fecha_creacion` (ascendente) o `/articulos/?ordering=-fecha_creacion` (descendente, agregando el signo menos).

---
