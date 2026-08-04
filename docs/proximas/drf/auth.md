

# Módulo 2: Vistas, Seguridad y Evaluación



## Clase 5: Autenticación y Autorización en DRF



---

### 🎯 Objetivos Pedagógicos de la Clase

1. **Diferenciar conceptos clave:** Clarificar la distinción entre **Autenticación** (*¿quién eres?*) y **Autorización** (*¿qué tienes permitido hacer?*).
2. **Comparar esquemas de autenticación:** Evaluar las diferencias entre Session, Token nativo y JWT (JSON Web Tokens).
3. **Implementar JWT con `SimpleJWT`:** Configurar un esquema de tokens sin estado (*stateless*) con tokens de acceso y refresco.
4. **Utilizar permisos nativos:** Aplicar permisos predefinidos de DRF (`IsAuthenticated`, `IsAdminUser`, `IsAuthenticatedOrReadOnly`).
5. **Crear permisos personalizados:** Heredar de `BasePermission` para implementar reglas de negocio a nivel de vista (`has_permission`) y a nivel de objeto (`has_object_permission`).



---

### 0. Contexto: Autenticación vs. Autorización

Antes de escribir código, es crucial distinguir ambos conceptos en arquitectura REST:

* **Autenticación (Authentication):** Es el proceso de verificar la identidad de un cliente o usuario. DRF examina las credenciales entrantes (encabezados, cookies, tokens) y adjunta el usuario validado a `request.user` (o `AnonymousUser` si falla o no se proveen).
* **Autorización (Permissions):** Se ejecuta **después** de la autenticación. Evalúa si el usuario identificado (`request.user`) tiene permiso para realizar la acción solicitada sobre la vista o el recurso específico.

---

### Paso 1: Esquemas de Autenticación en DRF



Django REST Framework soporta múltiples esquemas de autenticación out-of-the-box o mediante paquetes de terceros:

1. **`SessionAuthentication`:** Utiliza las sesiones nativas de Django basadas en cookies. Excelente para interfaces renderizadas en el servidor o para interactuar con la *Browsable API* de DRF en desarrollo.
2. **`TokenAuthentication`:** Esquema basado en tokens persistidos en la base de datos de Django (`rest_framework.authtoken`). Cada usuario tiene un token estático.
3. **JWT (`SimpleJWT`):** Esquema moderno basado en tokens firmados criptográficamente. **No requiere consultas a la base de datos** para validar cada petición (*stateless*), lo que lo convierte en el estándar de la industria para aplicaciones separadas (React, Vue, Flutter, Mobile).

---

### Paso 2: Configuración de JWT con `djangorestframework-simplejwt`

#### 1. Instalación

```bash
pip install djangorestframework-simplejwt

```

#### 2. Configuración global en `settings.py`

Definimos JWT como el mecanismo de autenticación por defecto de nuestra API:

```python
# config/settings.py

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
        'rest_framework.authentication.SessionAuthentication',  # Opcional: para Browsable API
    )
}

# Configuración opcional de duraciones
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'ROTATE_REFRESH_TOKENS': True,
    'AUTH_HEADER_TYPES': ('Bearer',),
}

```

#### 3. Configuración de endpoints en `urls.py`

Invocamos las vistas provistas por `SimpleJWT` para obtener y renovar tokens:

```python
# config/urls.py (o apps/usuarios/urls.py)
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    # Endpoint para iniciar sesión (devuelve access y refresh token)
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    
    # Endpoint para renovar el access token caducado
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]

```

#### 4. ¿Cómo consume el cliente la API protegida?

El cliente envía el token en cada petición HTTP dentro del encabezado `Authorization`:

```http
Authorization: Bearer <tu_access_token_aqui>

```

---

### Paso 3: Permisos Nativos de DRF



Los permisos se pueden aplicar de forma **global** en `settings.py` o **específica** en cada clase de vista mediante la propiedad `permission_classes`.

#### Principales Permisos Integrados:

* `AllowAny`: Otorga acceso total sin importar si está autenticado o no.
* `IsAuthenticated`: Restringe el acceso únicamente a usuarios autenticados.
* `IsAdminUser`: Exige que `user.is_staff` sea `True`.
* `IsAuthenticatedOrReadOnly`: Permite lectura libre (`GET`, `HEAD`, `OPTIONS`) a cualquiera, pero exige estar autenticado para operaciones de escritura (`POST`, `PUT`, `PATCH`, `DELETE`).

#### Aplicación por vista:

```python
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly
from .models import Producto
from .serializers import ProductoSerializer

class ProductoListCreateView(generics.ListCreateAPIView):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    # Solo usuarios autenticados pueden crear; lectura pública
    permission_classes = [IsAuthenticatedOrReadOnly]

```

---

### Paso 4: Creación de Permisos Personalizados (`BasePermission`)



Para implementar reglas de negocio específicas (por ejemplo: *"Solo el creador de una publicación puede editarla o borrarla"*), creamos clases que heredan de `rest_framework.permissions.BasePermission`.

Una clase de permiso puede evaluar dos métodos:

1. `has_permission(self, request, view)`: Evalúa el acceso general a la vista (antes de consultar la BD).
2. `has_object_permission(self, request, view, obj)`: Evalúa el acceso a un objeto o instancia en particular.

#### Ejemplo Práctico: Permiso `EsPropietarioOLectura` (`IsOwnerOrReadOnly`)

```python
# apps/productos/permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Permiso personalizado:
    - Cualquier usuario puede leer (GET, HEAD, OPTIONS).
    - Solo el usuario propietario del recurso puede editarlo o eliminarlo.
    """

    def has_object_permission(self, request, view, obj):
        # 1. Permitir métodos de lectura seguros (SAFE_METHODS = GET, HEAD, OPTIONS)
        if request.method in permissions.SAFE_METHODS:
            return True

        # 2. Verificar si el usuario autenticado es el creador/dueño del objeto
        return obj.creador == request.user

```

---

### Paso 5: Integración Completa en una Vista Genérica

Unimos Autenticación, Vistas Genéricas y Permisos Personalizados en una vista de detalle:

```python
# apps/productos/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Producto
from .serializers import ProductoSerializer
from .permissions import IsOwnerOrReadOnly

class ProductoDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    
    # Combinación de permisos: Debe estar autenticado Y ser el dueño para editar
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]

    def perform_create(self, serializer):
        # Asigna automáticamente el usuario logueado al crear el recurso
        serializer.save(creador=self.request.user)

```

---

### Paso 6: Resumen del Flujo de Petición en DRF

Cuando llega un request HTTP a un endpoint protegido:

```text
[Request HTTP con Header Authorization]
                 │
                 ▼
     [Autenticación (JWT)]
  ↳ ¿Token válido? ── NO ──► [401 Unauthorized]
                 │ SÍ
                 ▼
    [request.user = Usuario]
                 │
                 ▼
     [Verificación de Permisos]
  ↳ ¿Cumple permisos? ── NO ──► [403 Forbidden]
                 │ SÍ
                 ▼
      [Ejecución de la Vista]

```

---

### Paso 7: Ejercicio Práctico Guiado y Desafío de Clase

#### Consigna para los Alumnos:

Dado un sistema de Blog con un modelo `Articulo` (que posee un campo ForeignKey `autor` hacia `User`):

1. Proteger el endpoint `/api/articulos/` para que:
* Cualquier usuario pueda ver el listado de artículos.
* Solo usuarios autenticados mediante JWT puedan crear nuevos artículos.


2. Crear un permiso personalizado `IsAuthorOrAdmin` que garantice que:
* Un artículo individual solo pueda ser editado o eliminado por su autor original o por un usuario administrador (`is_staff=True`).


3. Probar los endpoints utilizando Postman, Insomnia o la Browsable API enviando el token en el header `Authorization: Bearer <token>`.

#### Solución sugerida para mostrar al finalizar:

```python
# permissions.py
from rest_framework import permissions

class IsAuthorOrAdmin(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Permitir si es staff/admin O si es el autor del artículo
        return request.user.is_staff or obj.autor == request.user

# views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from .models import Articulo
from .serializers import ArticuloSerializer
from .permissions import IsAuthorOrAdmin

class ArticuloDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Articulo.objects.all()
    serializer_class = ArticuloSerializer
    permission_classes = [IsAuthenticatedOrReadOnly, IsAuthorOrAdmin]

```