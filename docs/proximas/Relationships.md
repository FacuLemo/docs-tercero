# Relaciones modelos django

Tarea 

```python
from django.contrib.auth.models import User

class tarea(Models.Model)
    titulo = models.CharField(max_length=100)
    responsable = models.ForeignKey(User, on_delete=models.CASCADE, related_name="responsable")
    terminada = models.BooleanField(
        default=False,
        verbose_name="Tarea Finalizada",
        help_text="Marca la casilla si la tarea ha finalizado",
    )
```

```python
from django.contrib.auth.models import User

class Etiqueta(Models.Model)
    nombre = models.CharField(max_length=100)

class tarea(Models.Model)
    titulo = models.CharField(max_length=100)
    nombre = models.ManyToManyField(Etiqueta, blank=True)
   
```