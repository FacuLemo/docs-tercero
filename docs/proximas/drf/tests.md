# Clase 10: Testing Automatizado – Asegurando la Calidad del Software

**Objetivos de la clase:**

* Entender la diferencia entre pruebas manuales y pruebas automatizadas (Regresión).
* Escribir pruebas de integración utilizando `APITestCase` de Django REST Framework.


* Validar tanto el "camino feliz" (respuestas exitosas) como los casos de error y seguridad.

---

## Paso 1: El miedo a romper código (El "Por qué" del Testing)

**Explicación para la clase:**
Todos hemos modificado una línea de código para arreglar un detalle y, sin querer, roto una funcionalidad en otra parte del sistema. Si dependemos de probar todo a mano, a medida que el proyecto crece, es imposible verificar cada endpoint, cada permiso y cada filtro antes de entregar el código. Las pruebas automatizadas son scripts que actúan como "clientes fantasma" consumiendo nuestra API miles de veces en segundos para asegurarnos de que todo sigue funcionando. Es el estándar absoluto para las Prácticas Profesionalizantes y la industria.

## Paso 2: El entorno de pruebas y el patrón Arrange-Act-Assert

**Explicación:**
Utilizaremos `APITestCase`. Esta clase no toca nuestra base de datos real; Django crea una base de datos temporal vacía, ejecuta las pruebas y luego la destruye. El patrón de trabajo siempre es: Preparar (Arrange), Actuar (Act) y Afirmar (Assert).

**Código de ejemplo (`tests.py`):**

```python
from rest_framework.test import APITestCase
from rest_framework import status
from django.contrib.auth.models import User
from .models import Paciente

class PacienteTests(APITestCase):
    def setUp(self):
        # Arrange (Preparar): Creamos datos iniciales en la DB temporal
        self.usuario = User.objects.create_user(username='medico', password='123')
        self.paciente = Paciente.objects.create(nombre="Juan", dni="12345678")
        self.url = '/api/pacientes/'

```

## Paso 3: Probando el "Camino Feliz" (Happy Path)

**Explicación:**
Vamos a simular lo que pasaría si todo sale bien: un usuario autenticado pide la lista de datos.

**Código de ejemplo (Añadiendo a la clase anterior):**

```python
    def test_listar_pacientes_autenticado(self):
        # 1. Arrange: Autenticamos al cliente de prueba
        self.client.force_authenticate(user=self.usuario)
        
        # 2. Act: Simulamos un GET
        response = self.client.get(self.url)
        
        # 3. Assert: Verificamos los resultados
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data), 1)

```

*El por qué:* Si el día de mañana alguien cambia la URL o el Serializador por error, este test fallará al ejecutar `python manage.py test`, avisándonos del problema antes de que llegue a producción.

## Paso 4: Probando la Seguridad y los Errores (Edge Cases)

**Explicación:**
Un buen backend no solo funciona bien cuando recibe lo esperado, sino que se defiende correctamente ante datos inválidos o accesos no autorizados.

**Código de ejemplo:**

```python
    def test_crear_paciente_sin_autenticacion(self):
        # Act: Intentamos hacer POST sin iniciar sesión
        data = {'nombre': 'Ana', 'dni': '87654321'}
        response = self.client.post(self.url, data)
        
        # Assert: La API DEBE rechazar la petición con un 401
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

    def test_crear_paciente_datos_invalidos(self):
        self.client.force_authenticate(user=self.usuario)
        
        # Act: Enviamos un POST sin el campo obligatorio 'dni'
        data = {'nombre': 'Ana'}
        response = self.client.post(self.url, data)
        
        # Assert: La API DEBE devolver un 400 Bad Request
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

```

*El por qué:* Aquí demostramos que nuestras políticas de permisos (`IsAuthenticated`) y las validaciones del `ModelSerializer` realmente están funcionando como un escudo protector. Obligar a los alumnos a pensar en cómo "romper" su propia API los convierte en desarrolladores mucho más analíticos.