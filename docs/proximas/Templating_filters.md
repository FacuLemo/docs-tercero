Los **filtros en el sistema de plantillas de Django** son herramientas que permiten **transformar o formatear datos directamente en el HTML**, sin necesidad de escribir lógica compleja en la vista.

---
Django template language
para mejorar la escalabilidad del sistema, usamos url:
```django
{% url 'home' %}
```

# ¿Qué es un filtro en Django Template Language?

Un filtro es una función que se aplica a una variable dentro de una plantilla usando el operador `|`.

### Sintaxis básica:

```django
{{ variable|filtro:argumento }}
```

---

# ⚙️ ¿Cómo funcionan?

1. La vista envía datos al template (contexto).
2. El template muestra esos datos.
3. Los filtros modifican esos datos antes de renderizarlos.

Ejemplo simple:

```python
# views.py
def ejemplo(request):
    return render(request, "index.html", {"nombre": "juan"})
```

```django
<!-- index.html -->
{{ nombre|upper }}
```

Resultado:

```
JUAN
```

---

# Los filtros más útiles

## 1. texto y str

### `upper`, `lower` y `title`

```django
{{ "hola"|upper }}   → HOLA
{{ "HOLA"|lower }}   → hola
{{ "juan perez"|title }} → Juan Perez
```

### `linebreaks`

Convierte saltos de línea en `<p>` y `<br>`:

```
{{ texto|linebreaks }}
```

Ideal para textos largos guardados en la DB.

### `cut`

Elimina caracteres:

```django
{{ "hola mundo"|cut:" " }} → holamundo
```

### `truncatechars`

Recorta texto:

```django
{{ texto|truncatechars:10 }}
```

---

## 2. Números

### floatformat
```
{{ precio|floatformat:2 }}
```

---

## 3. Fechas

### `date`

```django
{{ fecha|date:"d/m/Y" }} → 22/04/2026
```

---

## 4. Listas

### `length`

```django
{{ lista|length }}
```

---

## 7. Valores por defecto

### `default`

```django
{{ nombre|default:"Anónimo" }}
```

---

#  Encadenar filtros

Puedes usar varios filtros seguidos:

```django
{{ nombre|lower|title }}
```

---

# 🧭 Resumen

* Los filtros transforman datos en templates
* Se aplican con `|`
* Pueden recibir argumentos
* Se pueden encadenar
* Puedes crear filtros personalizados

