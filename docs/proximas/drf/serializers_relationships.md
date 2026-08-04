
# Clase 3: Relaciones en Serializadores y `SerializerMethodField`

## Objetivos de la clase

1. Comprender las diferentes formas de representar relaciones (`ForeignKey` y `ManyToManyField`) en Django REST Framework.
2. Implementar `PrimaryKeyRelatedField` y `StringRelatedField` para controlar cómo se visualizan los objetos relacionados.
3. Crear y utilizar **Serializadores Anidados** (*Nested Serializers*) para devolver estructuras JSON complejas.
4. Utilizar `SerializerMethodField` para calcular y retornar información dinámica que no existe explícitamente en la base de datos.

---

## Paso 1: Tipos de Representación de Relaciones en DRF

En la clase anterior vimos que por defecto `ModelSerializer` representa las relaciones (`persona` y `categorias`) mediante **IDs clave primaria** (`PrimaryKeyRelatedField`).

Aunque enviar un ID (`persona: 1`) es eficiente para crear o actualizar registros (escritura), suele ser insuficiente cuando un cliente consume la API para mostrar información en pantalla (lectura).

DRF nos ofrece distintas alternativas para manejar relaciones:

* **`PrimaryKeyRelatedField`:** Representa la relación solo mediante su ID (comportamiento por defecto).
* **`StringRelatedField`:** Muestra la representación textual del objeto relacionado (lo que devuelve su método `__str__`).
* **Serializadores Anidados (*Nested Serializers*):** Incluye todo el objeto relacionado serializado dentro del JSON.
* **`SerializerMethodField`:** Permite ejecutar un método personalizado para generar un valor calculado al vuelo.

---

## Paso 2: Serializador de Categoría y `StringRelatedField`

Primero, vamos a crear un serializador para el modelo `Categoria` y a modificar el de `Tarea` para observar cómo cambia la respuesta cuando aplicamos `StringRelatedField`.

Edita `tareas/serializers.py`:

```python
# tareas/serializers.py

from rest_framework import serializers
from django.contrib.auth.models import User
from .models import Tarea, Categoria


# 1. Serializador básico para Categoria
class CategoriaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Categoria
        fields = ['id', 'nombre']


# 2. Serializador de Tarea usando StringRelatedField
class TareaStringRelatedSerializer(serializers.ModelSerializer):
    # Devuelve el __str__ del usuario (ej: "admin") en lugar de su ID
    persona = serializers.StringRelatedField(read_only=True)
    
    # Devuelve una lista de los __str__ de las categorías (ej: ["Estudios", "Urgente"])
    categorias = serializers.StringRelatedField(many=True, read_only=True)

    class Meta:
        model = Tarea
        fields = [
            'id', 
            'titulo', 
            'completada', 
            'prioridad', 
            'persona', 
            'categorias', 
            'timestamp'
        ]

```

### Ejemplo de respuesta JSON obtenida:

```json
{
    "id": 1,
    "titulo": "Preparar examen de DRF",
    "completada": false,
    "prioridad": 5,
    "persona": "facundo",
    "categorias": [
        "Estudios",
        "Programación"
    ],
    "timestamp": "2026-07-28T18:30:00Z"
}

```

---

## Paso 3: Serializadores Anidados (*Nested Serializers*)

Si la aplicación cliente necesita acceder a todos los detalles de las categorías asignadas sin realizar peticiones HTTP adicionales, podemos **anidar** `CategoriaSerializer` dentro de `TareaSerializer`.

Actualiza `tareas/serializers.py`:

```python
# tareas/serializers.py

# Serializador para mostrar la persona asignada con más detalle
class UsuarioDetalleSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']


class TareaDetalleSerializer(serializers.ModelSerializer):
    # Anidamos el serializador de usuario (relación uno a muchos)
    persona = UsuarioDetalleSerializer(read_only=True)
    
    # Anidamos el serializador de categoría con many=True (relación muchos a muchos)
    categorias = CategoriaSerializer(many=True, read_only=True)

    class Meta:
        model = Tarea
        fields = [
            'id', 
            'titulo', 
            'completada', 
            'prioridad', 
            'persona', 
            'categorias', 
            'timestamp'
        ]

```

### Ejemplo de respuesta JSON anidada:

```json
{
    "id": 1,
    "titulo": "Preparar examen de DRF",
    "completada": false,
    "prioridad": 5,
    "persona": {
        "id": 1,
        "username": "facundo",
        "email": "facundo@ejemplo.com"
    },
    "categorias": [
        {
            "id": 1,
            "nombre": "Estudios"
        },
        {
            "id": 2,
            "nombre": "Programación"
        }
    ],
    "timestamp": "2026-07-28T18:30:00Z"
}

```

> **Consejo pedagógico:** Explicar a los alumnos que los serializadores anidados marcados con `read_only=True` son excelentes para **consultas (`GET`)**, pero no aceptan IDs numéricos al intentar crear registros (`POST`). Por esta razón, se suele utilizar un serializador "liviano" (con IDs) para la creación/edición y uno "completo" (con objetos anidados) para la lectura.

---

## Paso 4: Campos Calculados con `SerializerMethodField`

`SerializerMethodField` es un campo de solo lectura que obtiene su valor invocando a un método en la misma clase del serializador.

**Regla de nombres en DRF:** El método asociador debe llamarse obligatoriamente `get_<nombre_del_campo>(self, obj)`. El parámetro `obj` representa la instancia del modelo que se está procesando actualmente (en este caso, un objeto `Tarea`).

Agreguemos dos campos calculados al `TareaDetalleSerializer`:

1. `prioridad_etiqueta`: Convierte el valor entero de prioridad (`1` a `5`) en una descripción legible (ej: `"Baja"`, `"Alta"`, `"Máxima"`).
2. `cantidad_categorias`: Devuelve la cantidad total de categorías asociadas a la tarea.

```python
# tareas/serializers.py

class TareaDetalleSerializer(serializers.ModelSerializer):
    persona = UsuarioDetalleSerializer(read_only=True)
    categorias = CategoriaSerializer(many=True, read_only=True)
    
    # Definimos los campos calculados
    prioridad_etiqueta = serializers.SerializerMethodField()
    cantidad_categorias = serializers.SerializerMethodField()

    class Meta:
        model = Tarea
        fields = [
            'id', 
            'titulo', 
            'completada', 
            'prioridad', 
            'prioridad_etiqueta',
            'persona', 
            'categorias', 
            'cantidad_categorias',
            'timestamp'
        ]

    # Método para calcular 'prioridad_etiqueta'
    def get_prioridad_etiqueta(self, obj):
        mapeo_prioridades = {
            1: "Sin prioridad",
            2: "Baja",
            3: "Media",
            4: "Alta",
            5: "Prioridad Máxima"
        }
        # obj es la instancia actual de Tarea
        return mapeo_prioridades.get(obj.prioridad, "Desconocida")

    # Método para calcular 'cantidad_categorias'
    def get_cantidad_categorias(self, obj):
        return obj.categorias.count()

```

---

## Paso 5: Implementación en la Vista (`views.py`)

Para integrar ambos flujos (creación con IDs vs. lectura con relaciones enriquecidas), podemos seleccionar dinámicamente qué serializador usar en la vista.

Edita `tareas/views.py`:

```python
# tareas/views.py

from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

from .models import Tarea
from .serializers import TareaSerializer, TareaDetalleSerializer

@api_view(['GET', 'POST'])
def lista_tareas(request):
    """
    GET: Devuelve la lista de tareas con objetos anidados y campos calculados.
    POST: Recibe datos con IDs de persona y categorías para crear la tarea.
    """
    if request.method == 'GET':
        tareas = Tarea.objects.all().select_related('persona').prefetch_related('categorias')
        # Usamos TareaDetalleSerializer para enviar información rica al cliente
        serializer = TareaDetalleSerializer(tareas, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)

    elif request.method == 'POST':
        # Usamos TareaSerializer (el de la Clase 2) que acepta IDs en persona y categorias
        serializer = TareaSerializer(data=request.data)
        
        if serializer.is_valid():
            nueva_tarea = serializer.save()
            # Respondemos con el serializador de detalle para que el cliente reciba la tarea completa recién creada
            respuesta_serializer = TareaDetalleSerializer(nueva_tarea)
            return Response(respuesta_serializer.data, status=status.HTTP_201_CREATED)
        
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

> **Nota de optimización:** Observa el uso de `.select_related('persona')` y `.prefetch_related('categorias')` en el `GET`. Evita el problema de consultas $N+1$ en la base de datos al serializar listas de objetos con relaciones.

---

## Paso 6: Pruebas en la *Browsable API*

1. Accede a `[http://127.0.0.1:8000/api/tareas/](http://127.0.0.1:8000/api/tareas/)`.
2. Realiza una petición `GET` y verifica que los campos `prioridad_etiqueta`, `cantidad_categorias`, `persona` y `categorias` devuelvan objetos formateados y no simples enteros.
3. Envía una petición `POST` con IDs en la sección inferior:
```json
{
    "titulo": "Implementar JWT en Django",
    "prioridad": 4,
    "persona": 1,
    "categorias": [1, 2]
}

```


4. Comprueba que el código de respuesta sea `201 Created` y que el JSON resultante responda inmediatamente con la información del usuario y los objetos de las categorías anidadas.

---
