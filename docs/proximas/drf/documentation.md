
# Documentación de nuestra API – Los lineamientos entre Backend y Frontend


## El problema de la comunicación (El "Por qué" documentar la api)

Cuando desarrollamos un backend, rara vez somos los únicos que consumimos esa API. El código será utilizado por un equipo de frontend, una aplicación móvil, o incluso por otros sistemas (los llamados "clientes"). Si en un proyecto real el frontend necesita saber qué datos enviar para registrar a un paciente, no puede estar adivinando si el campo se llama `nombre` o `first_name`, ni qué pasa si falta el DNI.

En lugar de escribir un manual de Word que quedará desactualizado rápidamente, utilizamos el estándar OpenAPI 3 para generar una documentación viva que siempre refleje el estado real de nuestro código.

## Paso 1: Implementar `drf-spectacular`

**Instalar la dependencia**
Instalamos la librería (con `pip install drf-spectacular` o `uv add drf-spectacular`), y la agregamos a nuestras aplicaciones.

**Configuración inicial:**
Luego de instalarla y agregarla a `INSTALLED APPS`, tendremos que configurar el schema desde la constante de configuración de DRF:

```python
REST_FRAMEWORK = {
    # ...
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}
```

Adicionalmente podremos agregar o cambiar metadatos para el eschema general:
```python
# También en settings.py
SPECTACULAR_SETTINGS = {
    'TITLE': 'Your Project API',
    'DESCRIPTION': 'Your project description',
    'VERSION': '1.0.0',
    # ...
}
```

**Creando los endpoints (`urls.py`):**
Ahora podemos agregar los endpoints de documentación en urls.py de nuestro proyecto.

```python
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView, SpectacularRedocView
from django.urls import path

urlpatterns = [
    # El archivo JSON/YAML real. Entrar descargará el archivo.
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # La interfaz interactiva donde el frontend puede probar peticiones
    path('api/docs/swagger/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    # La interfaz de lectura, ideal para adjuntar en entregas formales
    path('api/docs/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]

```

*El por qué:* `Swagger UI` nos da un entorno de pruebas directamente en el navegador, evitando que el frontend tenga que configurar Postman (u otro cliente rest) desde cero. `ReDoc` organiza todo de forma muy limpia y profesional para su lectura.

## Paso 2: Enriquecer la documentación con `@extend_schema`

**Explicación:**
La autogeneración de DRF es genial, pero a veces necesitamos dar más contexto. Un endpoint de "crear turno" podría necesitar una explicación humana de cómo funciona por detrás, o documentar explícitamente qué errores puede devolver.

**Código de ejemplo (`views.py`):**

```python
from drf_spectacular.utils import extend_schema, OpenApiResponse
from rest_framework import viewsets

class PacienteViewSet(viewsets.ModelViewSet):
    queryset = Paciente.objects.all()
    serializer_class = PacienteSerializer

    @extend_schema(
        summary="Registra un nuevo paciente en el sistema",
        description="Este endpoint crea un paciente. Requiere que el DNI no esté registrado previamente.",
        responses={
            201: PacienteSerializer, #created
            400: OpenApiResponse(description="Error de validación (ej: DNI duplicado)")
        }
    )
    def create(self, request, *args, **kwargs):
        return super().create(request, *args, **kwargs) # No cambiamos la función interna del create

```

*El por qué:* Con esto, el panel de Swagger deja de ser solo una lista de URLs y se convierte en una verdadera guía de integración. Quien consuma la API sabrá exactamente qué esperar, reduciendo la fricción y las preguntas constantes al desarrollador backend.

