

# Clase 5: Lógica (Bucles/Condicionales) con Jinja2 y Estilos 

**Objetivo de la clase:**

1. Pasar estructuras de datos complejas (Listas/Booleanos) desde la vista.
2. Usar `{% for %}` para recorrer listas.
3. Usar `{% if %}` para mostrar u ocultar elementos.
4. (Opcional/Si da tiempo) Agregar un archivo CSS básico.

---

### Parte 1: Mejorando la Vista (`views.py`)

Vamos a simular que traemos datos de una base de datos. Modificamos la función `saludo`.

*Archivo: `holamundo/views.py*`

```python
def saludo(request):
    # Simulamos datos
    lista_tareas = ['Aprender Python', 'Instalar Django', 'Crear primera App']
    
    datos = {
        'nombre_usuario': 'Pepito',
        'tareas': lista_tareas,    # Pasamos la lista
        'tiene_tareas': True       # Un booleano para probar el IF
    }

    return render(request, 'holamundo/index.html', datos)

```

> **Tip Docente:** Explica que el diccionario de contexto puede llevar cualquier cosa: listas, números, otros diccionarios, objetos, etc.

---

### Parte 2: El Template con "Poderes" (`index.html`)

Aquí introducimos la diferencia entre `{{ variable }}` (imprimir) y `{% tag %}` (lógica).

*Archivo: `holamundo/templates/holamundo/index.html*`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Lista de Tareas</title>
</head>
<body>
    <h1>Bienvenido, {{ nombre_usuario }}</h1>

    {% if tiene_tareas %}
        <p>Tienes tareas pendientes:</p>
        
        <ul>
            {% for tarea in tareas %}
                <li>{{ tarea }}</li>
            {% endfor %}
        </ul>
        
    {% else %}
        <p>¡Felicidades! Estás libre por hoy.</p>
    {% endif %}
    
</body>
</html>

```

> **Conceptos a explicar:**
> 1. **Etiquetas de bloque:** Los `{% %}` ejecutan código pero no imprimen nada en pantalla.
> 2. **Cierre de bloques:** En Python usamos indentación. En HTML/Django Template, como no hay indentación obligatoria, **debemos cerrar** los bloques explícitamente con `{% endif %}` y `{% endfor %}`.
> 
> 

---

### Parte 3: Archivos Estáticos (CSS)

Ahora que funciona, vamos a hacer que no se vea tan "plano".

#### 3.1. Estructura de carpetas

Al igual que con los templates, necesitamos una estructura específica dentro de la App.

```text
holamundo/
├── static/              <-- 1. Crear carpeta 'static'
│   └── holamundo/       <-- 2. Crear subcarpeta con nombre de la app
│       └── estilos.css  <-- 3. Crear archivo CSS

```

#### 3.2. Crear el CSS básico

*Archivo: `holamundo/static/holamundo/estilos.css*`

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    padding: 20px;
}

h1 {
    color: #2c3e50;
}

li {
    color: #e67e22;
    font-weight: bold;
}

```

#### 3.3. Conectar el CSS al HTML

Volvemos al `index.html`. Hay que hacer dos cosas: cargar la librería estática y enlazar el archivo.

```html
{% load static %}

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="{% static 'holamundo/estilos.css' %}">
    <title>Lista de Tareas</title>
</head>

```

> **Tip Docente:** Explica que `{% static 'ruta' %}` es mágico. Django busca esa ruta y genera la URL correcta automáticamente, sin importar si estamos en local o en un servidor real.

---

### Errores Comunes (Troubleshooting)

1. **TemplateSyntaxError:**
* *Causa:* Se olvidaron de cerrar un bloque (falta `{% endif %}` o `{% endfor %}`).


2. **El CSS no carga (pantalla sigue fea):**
* *Causa 1:* No pusieron `{% load static %}` en la **primera línea** del HTML.
* *Causa 2:* El navegador guardó caché. Intentar `Ctrl + F5`.
* *Causa 3:* Crearon la carpeta `static` *mientras* el servidor corría.
* *Solución:* **Reiniciar el servidor** (`Ctrl+C` y `runserver` de nuevo). Django a veces no ve carpetas nuevas de estáticos hasta que reinicias.


3. **No se ve nada en la lista:**
* *Causa:* La lista en `views.py` está vacía o la variable `tiene_tareas` es `False`.



---

### Resumen del Flujo de la Clase

1. Modificar `views.py` agregando una lista.
2. Modificar `index.html` usando `{% for %}`. **(Probar aquí)**.
3. Modificar `views.py` agregando booleano y `index.html` con `{% if %}`. **(Probar aquí)**.
4. Crear estructura `static` y archivo `.css`.
5. Cargar estáticos en el HTML. **(Resultado final)**.

¿Te parece bien esta estructura? Si la clase es muy larga, puedes dejar la parte de CSS para la siguiente y enfocarte bien en el `for` y `if`.