

#### 1. Introducción y Conceptos Básicos (15 min)
*   **La charla inicial:** Explícales que nunca, **jamás**, deben programar un sistema de login desde cero guardando contraseñas en texto plano. Háblales un poco de seguridad básica y cómo Django ya hace el trabajo sucio (hashing de contraseñas, manejo de sesiones y cookies) por ellos.
*   **El superusuario:** Recuérdales el `createsuperuser` que seguramente ya usaron para el panel de Admin. Ese mismo sistema es el que van a exponer ahora hacia el "frente" de la página.

#### 2. Login y Logout "Automágico" (30 min)
Django ya tiene vistas programadas para esto. No necesitan escribir la lógica de verificar contraseñas.

*   **Paso 1: Las URLs.** En el `urls.py` principal, agregar las rutas de autenticación de Django.
    ```python
    path('cuentas/', include('django.contrib.auth.urls')),
    ```
*   **Paso 2: Configurar las redirecciones.** En `settings.py`, decirle a Django a dónde ir cuando alguien entra o sale.
    ```python
    LOGIN_REDIRECT_URL = 'saludo' # O el nombre de la URL de su home
    LOGOUT_REDIRECT_URL = 'saludo'
    Incluir en Templates 'DIRS': [BASE_DIR / 'templates'], # Esto une la raíz con tu nueva carpeta
    ```
*   **Paso 3: El template obligatorio.** Explicarles que Django buscará el HTML del login en una ruta muy específica por defecto: `templates/registration/login.html`. Tienen que crear esa carpeta y ese archivo, y usar un formulario normal con `method="POST"` que renderice `{{ form.as_p }}`.


```html
<nav>
    <a href="{% url 'saludo' %}">Inicio</a>
    | 
    {% if user.is_authenticated %}
        <span>Hola, <strong>{{ user.username }}</strong></span> 
        <form action="{% url 'logout' %}" method="post" style="display:inline;">
            {% csrf_token %}
            <button type="submit">Cerrar Sesión</button>
        </form>
    {% else %}
        <a href="{% url 'login' %}">Iniciar Sesión</a>
    {% endif %}
</nav>
```
luego un login
```html
{% extends 'base.html' %}

{% block title %}Iniciar Sesión{% endblock %}

{% block content %}
<div class="login-container">
    <h2>Ingresar al Sistema</h2>
    
    <p>Por favor, introduce tus credenciales para gestionar tus tareas.</p>

    <form method="POST">
        {% csrf_token %}
        
        {{ form.as_p }}

        <button type="submit">Entrar</button>
    </form>

    <p>¿No tienes cuenta? <a href="{% url 'registro' %}">Regístrate aquí</a></p>
</div>
```


#### 3. Restringiendo el CRUD y la Vista (25 min)
Aquí es donde aplican el control sobre lo que ya construyeron.

*   **En las Vistas (Backend):** Presentar el decorador `@login_required`. Explicar que es como un "patovica" o guardia de seguridad que se pone arriba de la función. Si no estás logueado, te manda a la página de login.
    ```python
    from django.contrib.auth.decorators import login_required

    @login_required
    def crear_videojuego(request):
        # ... el código que ya tenían ...
    ```
*   **En los Templates (Frontend):** Mostrarles cómo usar `request.user.is_authenticated` para cambiar la barra de navegación o esconder los botones de edición.
    ```html
    {% if user.is_authenticated %}
        <p>Hola, {{ user.username }} | <a href="{% url 'logout' %}">Salir</a></p>
        <a href="{% url 'crear_videojuego' %}" class="btn">Agregar Juego</a>
    {% else %}
        <a href="{% url 'login' %}">Iniciar Sesión</a>
    {% endif %}
    ```

#### 4. Registro de Usuarios (50 min)
Ya pueden entrar y salir, pero necesitan poder crear cuentas nuevas sin entrar al panel de Admin.

*   **La magia del `UserCreationForm`:** Muestra cómo Django ya tiene un formulario prearmado que pide usuario, contraseña y confirmación de contraseña, y que además valida que no existan usuarios duplicados.
*   **La Vista de Registro:**
    ```python
    from django.shortcuts import render, redirect
    from django.contrib.auth.forms import UserCreationForm
    from django.contrib.auth import login

    def registro(request):
        if request.method == 'POST':
            form = UserCreationForm(request.POST)
            if form.is_valid():
                usuario = form.save()
                login(request, usuario) # Loguea al usuario automáticamente tras registrarse
                return redirect('lista_juegos')
        else:
            form = UserCreationForm()
        return render(request, 'registro.html', {'form': form})
    ```
*   Conectar esta nueva vista en sus `urls.py` y crear un template sencillo `registro.html` que sea casi idéntico al de login.

---

### ⚠️ Puntos de fricción comunes en esta clase

1.  **"Profe, hice el login pero me tira un error de *TemplateDoesNotExist*":** Es el error número uno. Pasa porque crean el `login.html` en la carpeta de templates de su app, en lugar de crear la subcarpeta exacta `registration/login.html`.
2.  **"El botón de Logout me da error 405 (Method Not Allowed)":** A partir de Django 5.0, la vista de Logout solo acepta peticiones POST por motivos de seguridad. Si tus alumnos están usando un simple `<a href="{% url 'logout' %}">` (que es una petición GET), les fallará. Tendrán que usar un pequeño formulario con un botón para hacer el logout.
3.  **Confusión entre `request.user` y el modelo de su base de datos:** A veces piensan que al crear un usuario automáticamente se asocia con los datos del CRUD.

¿Preferís que para la parte de registro y login enfoquemos el código usando Vistas Basadas en Clases (Class-Based Views) para ahorrar aún más código, o seguimos manteniendo Vistas Basadas en Funciones como venían trabajando en el CRUD?
```