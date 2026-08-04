

# Clase 2: Serializadores y Validaciones en DRF

## Objetivos de la clase

1. Comprender la función bidireccional de los serializadores en Django REST Framework (DRF): serialización y deserialización.
2. Implementar `ModelSerializer` utilizando el modelo `Tarea` para mapear objetos a JSON.
3. Aplicar validaciones personalizadas a nivel de campo (`validate_<campo>`) y a nivel de objeto global (`validate`).
4. Manejar correctamente la estructura de errores y respuestas HTTP en la creación y actualización de recursos.

---

## Paso 1: ¿Qué es un Serializador y para qué sirve?

En Django tradicional, los formularios (`forms.Form`) se encargan de validar HTML y parsear datos del navegador. En DRF, los **Serializadores (`serializers.Serializer`)** cumplen un rol similar, pero adaptado a APIs.

Tienen dos responsabilidades principales:

1. **Serialización (Lectura - GET):** Convierten instancias complejas de Python (como QuerySets o modelos de Django) en tipos de datos nativos (diccionarios) que luego se transforman en **JSON**.
2. **Deserialización (Escritura - POST/PUT):** Toman datos en formato JSON enviados por el cliente, los **validan** según reglas de negocio y los convierten en objetos de Python o registros en la base de datos.

---

## Paso 2: Definir el Modelo de Base de Datos

Utilizaremos el modelo `Tarea` (y su relación con `Categoria` y el usuario) en nuestra app `tareas`.

Edita el archivo `tareas/models.py`:

```python
# tareas/models.py

from django.db import models
from django.conf import settings

class Categoria(models.Model):
    nombre = models.CharField(max_length=50)

    def __str__(self):
        return self.nombre


class Tarea(models.Model):
    titulo = models.CharField(max_length=100)
    completada = models.BooleanField(default=False)
    timestamp = models.DateTimeField(auto_now=True)
    prioridad = models.IntegerField(
        default=1, 
        help_text="1-Sin prioridad, 5-Prioridad máxima"
    )
    persona = models.ForeignKey(
        settings.AUTH_USER_MODEL, 
        on_delete=models.CASCADE, 
        related_name="tareas"
    )
    categorias = models.ManyToManyField(
        Categoria, 
        related_name="tareas"
    )
    imagen = models.ImageField(upload_to="tareas/", null=True, blank=True)

    def __str__(self):
        return f"Tarea {self.titulo}, completada: {self.completada}, Responsable: {self.persona}"

```

Ejecuta las migraciones en la terminal para crear las tablas correspondientes:

```bash
python manage.py makemigrations
python manage.py migrate

```

---

## Paso 3: Crear el primer `ModelSerializer`

**`ModelSerializer`** inspecciona el modelo de Django y genera automáticamente los campos, además de incluir implementaciones predeterminadas para los métodos `.create()` y `.update()`.

Crea un archivo llamado `tareas/serializers.py`:

```python
# tareas/serializers.py

from rest_framework import serializers
from .models import Tarea

class TareaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tarea
        fields = [
            'id', 
            'titulo', 
            'completada', 
            'prioridad', 
            'persona', 
            'categorias', 
            'imagen', 
            'timestamp'
        ]
        read_only_fields = ['id', 'timestamp']

```

> **Nota para los alumnos:** `read_only_fields` asegura que campos generados o gestionados automáticamente como `id` y `timestamp` se incluyan al responder una consulta (`GET`), pero sean ignorados si el cliente intenta enviarlos por `POST` o `PUT`.

---

## Paso 4: Agregar Validaciones Personalizadas

DRF permite definir reglas de validación personalizadas dentro de la misma clase del serializador.

### 4.1. Validación a nivel de campo (`validate_<nombre_campo>`)

Se ejecuta para comprobar las reglas específicas de un solo campo.

### 4.2. Validación a nivel de objeto (`validate`)

Se ejecuta después de validar los campos individuales y permite comparar múltiples campos entre sí.

Actualiza `tareas/serializers.py` incorporando ambas validaciones:

```python
# tareas/serializers.py

from rest_framework import serializers
from .models import Tarea

class TareaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tarea
        fields = [
            'id', 
            'titulo', 
            'completada', 
            'prioridad', 
            'persona', 
            'categorias', 
            'imagen', 
            'timestamp'
        ]
        read_only_fields = ['id', 'timestamp']

    # 1. Validación individual: El título debe tener al menos 3 caracteres
    def validate_titulo(self, value):
        if len(value.strip()) < 3:
            raise serializers.ValidationError("El título de la tarea debe tener al menos 3 caracteres.")
        return value

    # 2. Validación individual: La prioridad debe estar en el rango de 1 a 5
    def validate_prioridad(self, value):
        if value < 1 or value > 5:
            raise serializers.ValidationError("La prioridad debe ser un número entero entre 1 y 5.")
        return value

    # 3. Validación global: Comparación entre campos según reglas de negocio
    def validate(self, data):
        prioridad = data.get('prioridad', 1)
        titulo = data.get('titulo', '')

        # Si es de máxima prioridad (5), el título debe ser descriptivo (mínimo 10 caracteres)
        if prioridad == 5 and len(titulo.strip()) < 10:
            raise serializers.ValidationError({
                "titulo": "Las tareas con prioridad máxima (5) requieren un título descriptivo de al menos 10 caracteres."
            })

        return data

```

---

## Paso 5: Conectar el Serializador en las Vistas (`views.py`)

Ahora utilizaremos el serializador para listar tareas existentes (`GET`) y crear nuevas tareas con validación previa (`POST`).

Edita `tareas/views.py`:

```python
# tareas/views.py

from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

from .models import Tarea
from .serializers import TareaSerializer

@api_view(['GET', 'POST'])
def lista_tareas(request):
    """
    GET: Lista todas las tareas registradas.
    POST: Crea una nueva tarea validando los datos recibidos.
    """
    if request.method == 'GET':
        tareas = Tarea.objects.all()
        # many=True indica que serializamos una lista de objetos (QuerySet)
        serializer = TareaSerializer(tareas, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)

    elif request.method == 'POST':
        # Pasamos los datos recibidos en el cuerpo JSON
        serializer = TareaSerializer(data=request.data)
        
        # is_valid() ejecuta las validaciones del modelo y las personalizadas
        if serializer.is_valid():
            serializer.save()  # Persiste la nueva Tarea en la BD
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        
        # Si falla, serializer.errors retorna el detalle exacto del error de validación
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

---

## Paso 6: Configurar las Rutas (`urls.py`)

Asegúrate de registrar la vista en `tareas/urls.py` y de incluir la app en el `urls.py` principal del proyecto.

```python
# tareas/urls.py

from django.urls import path
from .views import lista_tareas

urlpatterns = [
    path('tareas/', lista_tareas, name='lista_tareas'),
]

```


En `config/urls.py`:

```python
# config/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('tareas.urls')),
]

```

---

## Paso 7: Pruebas en la *Browsable API*

1. Levanta el servidor:
```bash
python manage.py runserver

```


2. Ingresa en el navegador a `[http://127.0.0.1:8000/api/tareas/](http://127.0.0.1:8000/api/tareas/)`
3. Realiza una petición `POST` enviando un JSON con datos inválidos para corroborar los mensajes de error:
```json
{
    "titulo": "Ir",
    "prioridad": 10,
    "persona": 1
}

```


**Resultado esperado:** HTTP `400 Bad Request` indicando los errores en `prioridad` y `titulo`.
4. Realiza un `POST` válido:
```json
{
    "titulo": "Preparar examen de DRF",
    "prioridad": 5,
    "persona": 1,
    "categorias": []
}

```


**Resultado esperado:** HTTP `201 Created` con la tarea creada correctamente y su ID asignado.

---
```python
urlpatterns = [
    path("api/", include("rest_framework.urls"))
]
```