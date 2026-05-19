
#### 1. Introducción: Estáticos vs. Media (10 min)
Arranca con una analogía rápida para despejar la confusión clásica:
*   **Archivos Estáticos (`static`):** Es la "pintura y decoración" del sitio. CSS, JavaScript, logos, fuentes. Son archivos que *tú* (el desarrollador) pones en el proyecto.
*   **Archivos Multimedia (`media`):** Es el contenido que genera el usuario final. Imágenes de perfil, PDFs, fotos de productos.

#### 2. Archivos Estáticos: Dándole vida al CRUD (30 min)
Tus alumnos ya tienen un CRUD funcional, pero probablemente se vea muy plano.
*   **Configuración en `settings.py`:** Muestra cómo configurar `STATIC_URL` y la lista `STATICFILES_DIRS` para apuntar a una carpeta global en la raíz del proyecto.
*   **El template tag:** Enseña el uso de `{% load static %}` al principio de los templates de su CRUD.
*   **Aplicación Práctica:** Haz que vinculen una hoja de estilos `style.css`. Puedes proponerles aplicar una paleta de colores moderna para salir del típico blanco y negro. Por ejemplo, un fondo oscuro general (`#121212`), tarjetas en un gris claro (`#EDF2F4`), y botones de advertencia o "Eliminar" en colores vibrantes como un amarillo "Sunglow" o un rojo coral. Esto les dará una victoria visual rápida.

#### 3. Archivos Media: Subiendo imágenes con ModelForms (45 min)
Aquí es donde conectas con la clase anterior. Supongamos que en su CRUD están gestionando un catálogo de videojuegos; ahora van a agregarle la portada del juego.

*   **Paso 1: Instalación de dependencias.** Explicar que Django necesita la librería `Pillow` para manejar `ImageField`. (`pip install Pillow`).
*   **Paso 2: Configurar `settings.py` y `urls.py`.**
    *   Agregar `MEDIA_URL` y `MEDIA_ROOT` en los settings.
    ```python
        # La URL pública desde el navegador (ej: localhost:8000/media/foto.jpg)
    MEDIA_URL = '/media/'

    # La carpeta real en el disco duro donde se guardarán
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    ```
    *   El **paso crítico** que siempre se olvida: añadir la ruta estática al final de `urls.py` para que Django sirva los archivos en desarrollo (`+ static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)`).
        ```python
        from django.conf import settings
        if settings.DEBUG:
            urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
        
        #finalmente en models.py en el modelo agregar:
        portada = models.ImageField(upload_to='portadas/', null=True, blank=True)
        ```
*   **Paso 3: Actualizar el Modelo y aplicar Migraciones.**
    *   Agregar un campo `portada = models.ImageField(upload_to='portadas/', null=True, blank=True)` a su modelo.
        ```python
        #finalmente en models.py en el modelo agregar:
        portada = models.ImageField(upload_to='portadas/', null=True, blank=True)

        #De paso:

        class Meta:
            # Custom plural name for the Admin interface
            verbose_name_plural = "Categories"
            ordering = ['-id']
        ```
    *   Hacer `makemigrations` y `migrate`.
*   **Paso 4: La magia del `ModelForm`.**
    *   Mostrarles que, como usaron `fields = '__all__'` (o listaron los campos) en su `ModelForm`, el formulario *ya* sabe que debe renderizar un `<input type="file">`. ¡No tienen que programar el input a mano!
*   **Paso 5: Actualizar la Vista y el Template (El punto de quiebre).**
    *   **Template de Creación/Edición:** Enseñar el sagrado `enctype="multipart/form-data"` en la etiqueta `<form>`. Sin esto, el archivo no viaja al servidor.
    *   **Vista:** Mostrar cómo recibir los archivos pasando `request.FILES` al instanciar el formulario en el método POST: `form = MiModeloForm(request.POST, request.FILES)`.
        ```python
        if request.method == 'POST':
            # ATENCIÓN AQUÍ: Se debe pasar request.FILES
            form = VideojuegoForm(request.POST, request.FILES)
            if form.is_valid():
                form.save()
                return redirect('lista_juegos')
        ```
    *   **Template de Detalle/Lista:** Mostrar cómo renderizar la imagen subida validando primero que exista:
      ```html
      {% if videojuego.portada %}
          <img src="{{ videojuego.portada.url }}" alt="Portada">
      {% endif %}
      ```

### ⚠️ Puntos de fricción comunes (Para que estés preparado)

Como instructor, prepárate para escuchar estos errores al menos 3 o 4 veces durante la práctica:
1.  **"Profe, no me sube la imagen pero no me da error":** 99% de las veces se olvidaron de poner `enctype="multipart/form-data"` en el HTML o se olvidaron de pasar `request.FILES` en la vista.
2.  **"Profe, subí la imagen, veo el link en la base de datos, pero la imagen sale rota en el navegador":** Les falta agregar la configuración de `MEDIA_URL` en el archivo `urls.py` principal.
3.  **"Me da error al hacer la migración":** Se olvidaron de instalar `Pillow` antes de agregar el `ImageField`.

¿Te gustaría que profundicemos en el código exacto de la configuración de las `urls.py` para la carpeta media, o prefieres armar un pequeño boilerplate de CSS para darles a los chicos y que no pierdan tanto tiempo maquetando?
```