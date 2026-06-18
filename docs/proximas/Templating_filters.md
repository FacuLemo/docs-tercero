Los **filtros en el sistema de plantillas de Django** son herramientas que permiten **transformar o formatear datos directamente en el HTML**, sin necesidad de escribir lógica compleja en la vista.

---

# 🧩 ¿Qué es un filtro en Django?

Un filtro es una función que se aplica a una variable dentro de una plantilla usando el operador `|`.

### Sintaxis básica:

```django
{{ variable|filtro }}
```

También pueden recibir argumentos:

```django
{{ variable|filtro:argumento }}
```

---

# ⚙️ ¿Cómo funcionan?

1. La vista envía datos al template (contexto).
2. El template muestra esos datos.
3. Los filtros modifican esos datos antes de renderizarlos.

👉 Ejemplo simple:

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

# Más filtros 

## 1. 🔠 Formateo de texto

### `upper` y `lower`

```django
{{ "hola"|upper }}   → HOLA
{{ "HOLA"|lower }}   → hola
```

### `title`

```django
{{ "juan perez"|title }} → Juan Perez
```

---

### linebreaks

Convierte saltos de línea en <p> y <br>:
```
{{ texto|linebreaks }}
```
Ideal para textos largos guardados en la DB.

## 2. ✂️ Manipulación de strings

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

## 3. 🔢 Números

### `add`

```django
{{ 5|add:3 }} → 8
```

### floatformat
```
{{ precio|floatformat:2 }}
```

---

## 4. 📅 Fechas

### `date`

```django
{{ fecha|date:"d/m/Y" }}
```

Resultado:

```
22/04/2026
```

---

## 5. 📦 Listas

### `length`

```django
{{ lista|length }}
```

### `join`

```django
{{ lista|join:", " }}
```

---

## 7. 🔄 Valores por defecto

### `default`

```django
{{ nombre|default:"Anónimo" }}
```

---

# 🔗 Encadenar filtros

Puedes usar varios filtros seguidos:

```django
{{ nombre|lower|title }}
```

---

# 🧠 Crear filtros personalizados

Si los filtros existentes no alcanzan, puedes crear los tuyos.

### Paso 1: Crear archivo

```
app/
 └── templatetags/
      └── custom_filters.py
```

### Paso 2: Definir filtro

```python
from django import template

register = template.Library()

@register.filter
def multiplicar(valor, arg):
    return valor * arg
```

### Paso 3: Usarlo en template

```django
{% load custom_filters %}

{{ 5|multiplicar:3 }}  → 15
```

---

# 🚨 Buenas prácticas

* ❌ No pongas lógica compleja en templates
* ✅ Usa filtros solo para presentación
* ✅ Mantén el código limpio y legible
* ❌ No abuses de `safe` (riesgo XSS)

---

# 🧭 Resumen

* Los filtros transforman datos en templates
* Se aplican con `|`
* Pueden recibir argumentos
* Se pueden encadenar
* Puedes crear filtros personalizados
