# Clase 4: Templates HTML

### 1. Estructura de Carpetas

Ya podemos renderizar texto plano en las vistas haciendo que las mismas retornen un `HttpResponse()`. Ahora, haremos que las vistas retornen un HTML con los datos que nosotros necesitemos.

Primero necesitamos crear las plantillas que se van a usar para cada vista. Las vamos a guardar dentro de nuestra aplicación, en `templates/'nombreapp'/'template'.html`

En nuestro caso, dentro de la carpeta de la app `holamundo`, crearemos:

```text
holamundo/
├── migrations/
├── templates/           <-- 1. Crear carpeta 'templates'
│   └── holamundo/       <-- 2. Crear carpeta con el MISMO nombre de la app
│       └── index.html   <-- 3. Crear el archivo HTML aquí
├── views.py
└── ...

```

> "¿Por qué `templates/holamundo/`? ¿No es redundante?"
> **Respuesta:** Django junta todos los templates de todas las apps en una sola pila. Si tienes dos apps con un archivo `index.html`, Django no sabrá cuál usar. Al ponerle el nombre de la app como subcarpeta, evitamos colisiones (Namespacing).

---

### 2. El Archivo HTML (`index.html`)

Vamos a escribir un HTML básico, pero con un bloque disponible para datos dinámicos.

*Archivo: `holamundo/templates/holamundo/index.html*`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi Web Django</title>
</head>
<body>
    <h1>Bienvenido a mi sitio</h1>
    
    <p>Hola, {{ nombre_usuario }}!</p> 
    
    <p>Hoy estamos aprendiendo a usar Templates.</p>
</body>
</html>

```

> **Explicación rápida:** `{{ variable }}` es como un marcador. Django buscará ese nombre y lo reemplazará por el valor real antes de enviar la página al navegador.

---

### 3. Conectando la Vista (`views.py`)

Ahora modificamos la vista para que use el template html en lugar de texto plano.

*Archivo: `holamundo/views.py*`

```python
from django.shortcuts import render # <--- Ahora usamos este!
# (Ya no necesitamos HttpResponse para esto, aunque se puede dejar importado)

def saludo(request):
    # 1. Preparamos los datos (El Contexto)
    datos = {
        'nombre_usuario': 'Estudiante de Django' 
    }

    # 2. Renderizamos
    # render(request, 'ruta_del_template', contexto)
    return render(request, 'holamundo/index.html', datos)

```

> El contexto que se le pase será un diccionario, que buscará coincidencias de claves en el html y ubicará allí los valores. 

---

### 4. Verificación

1. Asegurar que el servidor corre: `python manage.py runserver`
2. Refrescar el navegador: `http://127.0.0.1:8000/hola/` (o la ruta que hayas definido).

**Deberían ver:** Un título grande y el texto "Hola, Estudiante de Django!".

---


### Errores Comunes (Troubleshooting)

1. **TemplateDoesNotExist:**
* *Causa:* Escribieron mal la ruta en el `render`.
* *Solución:* Verificar que en `render` diga `'holamundo/index.html'` y que las carpetas existan físicamente tal cual.
* *Ojo:* A veces hay que detener y reiniciar el servidor si crearon carpetas nuevas, para que Django las detecte.


2. **No aparece el nombre (sale vacío):**
* *Causa:* La clave del diccionario en `views.py` no coincide *exactamente* con lo que pusieron entre `{{ }}` en el HTML. (Ej: `nombre` vs `nombre_usuario`).


---