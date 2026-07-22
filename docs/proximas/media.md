
#### 1. Introducción: Estáticos vs. Media (10 min)
Arranca con una analogía rápida para despejar la confusión clásica:
*   **Archivos Estáticos (`static`):** Es la "pintura y decoración" del sitio. CSS, JavaScript, logos, fuentes. Son archivos que *tú* (el desarrollador) pones en el proyecto.
*   **Archivos Multimedia (`media`):** Es el contenido que genera el usuario final. Imágenes de perfil, PDFs, fotos de productos.


#### 3. Archivos Media: Subiendo imágenes con ModelForms (45 min)
 Supongamos que en su CRUD están gestionando un catálogo de videojuegos; ahora van a agregarle la portada del juego.

*   **Paso 1: Instalación de dependencias.**  Django necesita la librería `Pillow` para manejar `ImageField`. (`pip install Pillow`).
*   **Paso 2: Configurar `settings.py` y `urls.py`.**
    *   Agregar `MEDIA_URL` y `MEDIA_ROOT` en los settings.
    ```python
        # La URL pública desde el navegador (ej: localhost:8000/media/foto.jpg)
    MEDIA_URL = '/media/'

    # La carpeta real en el disco duro donde se guardarán
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    ```
    *   **paso crítico** : añadir la ruta estática al final de `urls.py` para que Django sirva los archivos en desarrollo (`+ static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)`).
        ```python
        from django.conf import settings
        from django.conf.urls.static import static
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
        ```
    *   Luego hacer `makemigrations` y `migrate`.
*   **Paso 4: La magia del `ModelForm`.**
    *   Agregar el campo `portada` a los `fields = ...` del ModelForm. Cuando se listan los campos en su `ModelForm`, el formulario *ya* sabe que debe renderizar un `<input type="file">`. ¡No tienen que programar el input a mano!
*   **Paso 5: Actualizar la Vista y el Template (El punto de quiebre).**
    *   **Template de Creación/Edición:** Se debe agregar `enctype="multipart/form-data"` en la etiqueta `<form>`. Sin esto, el archivo no viaja al servidor.
    *   **Vista:** Ahora hay que recibir los archivos pasando `request.FILES` al instanciar el formulario en el método POST: `form = MiModeloForm(request.POST, request.FILES)`. en views.py:
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

### Errores Comunes

1.  **"No me sube la imagen pero no me da error":** 99% de las veces se olvidaron de poner `enctype="multipart/form-data"` en el HTML o se olvidaron de pasar `request.FILES` en la vista.
2.  **"Subí la imagen, veo el link en la base de datos, pero la imagen sale rota en el navegador":** Les falta agregar la configuración de `MEDIA_URL` en el archivo `urls.py` principal.
3.  **"Me da error al hacer la migración":** Se olvidaron de instalar `Pillow` antes de agregar el `ImageField`.

```