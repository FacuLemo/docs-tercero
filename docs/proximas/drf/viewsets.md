#  ViewSets y Routers 

**Objetivos de la clase:**

* Comprender la necesidad de reducir el código de las vistas genéricas.
* Implementar `ModelViewSet` y `ReadOnlyModelViewSet`.


* Automatizar la creación de URLs mediante la abstracción de rutas con `DefaultRouter`.


* Crear endpoints específicos (fuera del CRUD estándar) utilizando acciones personalizadas (`@action`).



---

## Paso 1: El Motivo

Hasta la clase anterior, veníamos trabajando con `APIView` y Vistas Genéricas. Si queríamos hacer un CRUD completo para un modelo (por ejemplo, `Articulo`), teníamos que crear al menos dos vistas:

1. Una `ListCreateAPIView` (para el GET de la lista y el POST).
2. Una `RetrieveUpdateDestroyAPIView` (para el GET, PUT/PATCH y DELETE de un objeto específico).

Luego, teníamos que ir a `urls.py` y conectar ambas vistas manualmente. Funciona, pero... ¿no es código muy repetitivo? Si tenemos 10 modelos en nuestro proyecto Django, tendremos que escribir 20 vistas y 20 rutas que hacen exactamente lo mismo. Aquí es donde entra la abstracción de DRF.

## Paso 2: Introducción a los ViewSets

Un `ViewSet` es una clase que combina la lógica de múltiples vistas relacionadas en una sola. En lugar de definir métodos HTTP como `.get()` o `.post()`, los ViewSets definen acciones como `.list()`, `.retrieve()`, `.create()`, `.update()`, y `.destroy()`.

Vamos a ver los dos más utilizados:

### A. `ModelViewSet`

Este ViewSet nos regala el CRUD completo con solo dos líneas de código. Hereda todos los mixins necesarios.

**Código de ejemplo (`views.py`):**

```python
from rest_framework import viewsets
from .models import Articulo
from .serializers import ArticuloSerializer

class ArticuloViewSet(viewsets.ModelViewSet):
    queryset = Articulo.objects.all()
    serializer_class = ArticuloSerializer

```

*Ya está:* Con estas tres líneas tenemos listado, creación, lectura de un solo elemento, actualización y eliminación. Django REST Framework se encarga de conectar la base de datos (con el ORM) y el Serializador automáticamente.

### B. `ReadOnlyModelViewSet`

¿Qué pasa si queremos exponer datos (como una lista de Categorías) pero no queremos que los usuarios puedan crear, editar o borrar desde la API?

**Código de ejemplo (`views.py`):**

```python
class CategoriaViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Categoria.objects.all()
    serializer_class = CategoriaSerializer

```

*El por qué:* Brinda seguridad por diseño. Solo habilita las acciones `.list()` y `.retrieve()`. Cualquier intento de POST, PUT o DELETE devolverá un error "405 Method Not Allowed" automáticamente.

## Paso 3: Abstracción de rutas con `DefaultRouter`

Si los ViewSets no tienen métodos `.get()` o `.post()`, ¿cómo los conectamos en `urls.py`? En lugar de hacer mapeos manuales y complicados, usamos un Router. El `DefaultRouter` inspecciona el ViewSet y genera todas las URLs necesarias automáticamente (incluyendo las terminaciones con el ID del recurso).

**Código de ejemplo (`urls.py`):**

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ArticuloViewSet, CategoriaViewSet

# 1. Instanciamos el router
router = DefaultRouter()

# 2. Registramos nuestros ViewSets
router.register(r'articulos', ArticuloViewSet, basename='articulo')
router.register(r'categorias', CategoriaViewSet, basename='categoria')

# 3. Incluimos las URLs generadas en los urlpatterns
urlpatterns = [
    path('', include(router.urls)),
]

```

*El por qué:* Esto nos ahorra tener que escribir expresiones regulares o conversores de rutas (`<int:pk>`). Además, `DefaultRouter` incluye una vista raíz (Root API view) en la URL base, que actúa como un índice navegable para los desarrolladores.

## Paso 4: Acciones personalizadas con `@action`

El CRUD estándar es fantástico, pero las aplicaciones reales necesitan más. ¿Qué pasa si queremos un endpoint para publicar un artículo específico que cambie su estado en la base de datos? No tiene sentido crear un ViewSet nuevo solo para eso. Utilizamos el decorador `@action`.

**Código de ejemplo (`views.py`):**

```python
from rest_framework.decorators import action
from rest_framework.response import Response

class ArticuloViewSet(viewsets.ModelViewSet):
    queryset = Articulo.objects.all()
    serializer_class = ArticuloSerializer

    # detail=True significa que aplica a un objeto específico (necesita ID en la URL)
    # Ejemplo: /articulos/1/publicar/
    @action(detail=True, methods=['post'])
    def publicar(self, request, pk=None):
        articulo = self.get_object()
        articulo.estado = 'Publicado'
        articulo.save()
        return Response({'status': 'Artículo publicado exitosamente'})
        
    # detail=False significa que aplica a la colección entera
    # Ejemplo: /articulos/recientes/
    @action(detail=False, methods=['get'])
    def recientes(self, request):
        recientes = Articulo.objects.order_by('-fecha_creacion')[:5]
        serializer = self.get_serializer(recientes, many=True)
        return Response(serializer.data)

```

*El por qué:* `@action` permite inyectar endpoints adicionales directamente dentro de la URL base del ViewSet. El `DefaultRouter` detecta estos decoradores y arma las rutas (`/articulos/1/publicar/` y `/articulos/recientes/`) por nosotros.

---
