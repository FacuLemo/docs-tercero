# Módulo 2: Vistas, Seguridad y Evaluación


## Clase 4: Vistas Basadas en Clases (CBVs) y Enrutamiento


---

### 🎯 Objetivos Pedagógicos de la Clase

1. **Comprender la transición:** Analizar las ventajas de la Programación Orientada a Objetos (POO) sobre las Vistas Basadas en Funciones (`@api_view`).
2. **Dominar `APIView`:** Implementar control granular para peticiones HTTP (`GET`, `POST`, `PUT`, `DELETE`) definiendo métodos explícitos.
3. **Reutilizar código con `GenericAPIView` y Mixins:** Comprender cómo los 5 mixins principales encapsulan las operaciones CRUD habituales.
4. **Simplificar desarrollo con Vistas Genéricas Concretas:** Utilizar `ListCreateAPIView` y `RetrieveUpdateDestroyAPIView` para crear APIs limpias y mantenibles.
5. **Mapear URLs con `.as_view()`:** Registrar Vistas Basadas en Clases en el archivo de enrutamiento de Django.

---

### 0. Contexto y Evolución: ¿Por qué pasamos a Vistas Basadas en Clases?

En el Módulo 1 construimos vistas funcionales decoradas con `@api_view`. Aunque este enfoque es claro al inicio, a medida que un proyecto crece surgen inconvenientes:

* **Repetición de código (incumplimiento de DRY):** Repetimos la lógica para buscar objetos (`get_object_or_404`), validar serializadores y estructurar respuestas de error.
* **Estructuras if/elif extensas:** Un solo endpoint condensa la lógica de múltiples métodos HTTP dentro de bloques condicionales (`if request.method == 'GET': ...`).
* **Poca extensibilidad:** Es complejo reutilizar o extender el comportamiento mediante herencia.

Django REST Framework resuelve esto ofreciendo tres niveles progresivos de abstracción basados en clases.

---

### Paso 1: Control Granular con `APIView`

`APIView` es la piedra angular de las CBVs en DRF. Extiende la clase `View` nativa de Django, pero agrega:

* Transformación de `HttpRequest` a `Request` de DRF.
* Manejo automático del objeto `Response` y negociación de formatos (JSON / Browsable API).
* Mapeo directo de los verbos HTTP a métodos con su mismo nombre (`get()`, `post()`, `put()`, `delete()`).

#### Código de Ejemplo: CRUD Manual con `APIView`

Supongamos que trabajamos sobre un modelo `Producto`:

```python
# apps/productos/views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404
from .models import Producto
from .serializers import ProductoSerializer

class ProductoListCreateAPIView(APIView):
    """
    Vista para listar la colección de productos o crear uno nuevo.
    """
    def get(self, request):
        productos = Producto.objects.all()
        serializer = ProductoSerializer(productos, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)

    def post(self, request):
        serializer = ProductoSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class ProductoDetailAPIView(APIView):
    """
    Vista para obtener, actualizar o eliminar un producto por su ID (PK).
    """
    def get_object(self, pk):
        return get_object_or_404(Producto, pk=pk)

    def get(self, request, pk):
        producto = self.get_object(pk)
        serializer = ProductoSerializer(producto)
        return Response(serializer.data, status=status.HTTP_200_OK)

    def put(self, request, pk):
        producto = self.get_object(pk)
        serializer = ProductoSerializer(producto, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_200_OK)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        producto = self.get_object(pk)
        producto.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

---

### Paso 2: Abstracción Reutilizable con `GenericAPIView` y Mixins



Aunque `APIView` organiza el código en métodos, seguimos escribiendo las mismas líneas de consulta a la base de datos y llamado a serializadores. DRF nos ofrece `GenericAPIView` junto con **Mixins** para reutilizar esta lógica.

#### Atributos de `GenericAPIView`:

* `queryset`: Especifica el conjunto de objetos base.
* `serializer_class`: Define el serializador a utilizar.
* `get_queryset()`: Permite filtrar o modificar las consultas de forma dinámica.
* `get_object()`: Recupera la instancia por clave primaria (`lookup_field`).

#### Los 5 Mixins Fundamentales:

| Mixin | Método de Clase | Operación CRUD | Verbo HTTP |
| --- | --- | --- | --- |
| `ListModelMixin` | `list()` | Listar colección | `GET` |
| `CreateModelMixin` | `create()` | Crear registro | `POST` |
| `RetrieveModelMixin` | `retrieve()` | Obtener detalle | `GET` (instancia) |
| `UpdateModelMixin` | `update()` | Actualizar registro | `PUT` / `PATCH` |
| `DestroyModelMixin` | `destroy()` | Eliminar registro | `DELETE` |

#### Implementación con Mixins:

```python
from rest_framework import generics, mixins
from .models import Producto
from .serializers import ProductoSerializer

class ProductoListCreateMixinView(mixins.ListModelMixin,
                                  mixins.CreateModelMixin,
                                  generics.GenericAPIView):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

```

---

### Paso 3: Vistas Genéricas Concretas (Concrete Generic Views)



Para evitar incluso asociar manualmente `get()` con `self.list()`, DRF combina `GenericAPIView` con los Mixins en **clases genéricas predefinidas**.

#### Principales Vistas Genéricas Concretas:

* `ListAPIView`: Solo lectura de colecciones (`GET`).
* `CreateAPIView`: Solo creación de instancias (`POST`).
* `ListCreateAPIView`: Lectura y creación de colecciones (`GET`, `POST`).


* `RetrieveAPIView`: Solo lectura de un objeto individual (`GET`).
* `RetrieveUpdateAPIView`: Lectura y edición de un objeto (`GET`, `PUT`, `PATCH`).
* `RetrieveUpdateDestroyAPIView`: Lectura, actualización y eliminación completa de un objeto (`GET`, `PUT`, `PATCH`, `DELETE`).



#### Código Definitivo (Versión Producción):

```python
from rest_framework import generics
from .models import Producto
from .serializers import ProductoSerializer

class ProductoListCreateView(generics.ListCreateAPIView):
    """
    Lista todos los productos (GET) o crea uno nuevo (POST).
    """
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer


class ProductoDetailView(generics.RetrieveUpdateDestroyAPIView):
    """
    Obtiene (GET), actualiza (PUT/PATCH) o elimina (DELETE) un producto.
    """
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer

```

---

### Paso 4: Enrutamiento y Mapeo en `urls.py`

Dado que Django espera funciones como manejadores de vistas en las URLs, las clases deben registrarse llamando al método estático `.as_view()`.

```python
# apps/productos/urls.py
from django.urls import path
from .views import ProductoListCreateView, ProductoDetailView

urlpatterns = [
    # Endpoint de Colección (Listar y Crear)
    path('productos/', ProductoListCreateView.as_view(), name='producto-list-create'),
    
    # Endpoint de Instancia (Detalle, Actualizar, Eliminar)
    path('productos/<int:pk>/', ProductoDetailView.as_view(), name='producto-detail'),
]

```

---

### Paso 5: Cuadro Comparativo de Enfoques

| Criterio | `APIView` | `GenericAPIView` + Mixins | Vistas Genéricas Concretas |
| --- | --- | --- | --- |
| **Nivel de Abstracción** | Bajo (Control total) | Medio (Modular) | Alto (Declarativo) |
| **Líneas de Código** | Alto | Moderado | Mínimo |
| **Flexibilidad** | Máxima (Lógica customizada) | Alta (Personalización de negocio) | Alta para CRUDs tradicionales |
| **Caso de Uso Ideal** | Endpoints de procesamiento, APIs externas o lógica compleja | Comportamientos CRUD no estándar | APIs REST convencionales asociadas a modelos |

---

### Paso 6: Ejercicio Práctico Guiado y Desafío para la Clase

#### Consigna:

Dado un modelo `Categoria` con los campos `nombre` (CharField) y `descripcion` (TextField):

1. Crea un serializador `CategoriaSerializer` basado en `ModelSerializer`.
2. Implementa `CategoriaListCreateView` heredando de la vista genérica concreta adecuada.
3. Sobrescribe el método `get_queryset()` para permitir filtrar las categorías cuyo nombre contenga un término enviado por parámetro URL (ej. `/api/categorias/?search=electronica`).
4. Configura el archivo `urls.py` correspondientemente.

#### Solución sugerida para mostrar al finalizar:

```python
# views.py
from rest_framework import generics
from .models import Categoria
from .serializers import CategoriaSerializer

class CategoriaListCreateView(generics.ListCreateAPIView):
    serializer_class = CategoriaSerializer

    def get_queryset(self):
        queryset = Categoria.objects.all()
        search_query = self.request.query_params.get('search', None)
        if search_query:
            queryset = queryset.filter(nombre__icontains=search_query)
        return queryset

```