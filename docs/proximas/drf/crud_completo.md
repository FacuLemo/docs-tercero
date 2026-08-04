Creamos un root artificial
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(["GET"])
def api_root(request):
    return Response({
        "tareas": request.build_absolute_uri("/tareas/")
    })
```

terminar crud de api_views.py
```python
from django.shortcuts import get_object_or_404

@api_view(["GET", "POST"])
def tareas(request):
    if request.method == "GET":
        tareas = Tarea.objects.all().select_related("persona").prefetch_related("categoria")
        serializer = TareaNestedSerializer(tareas, many=True)
        return Response(serializer.data,status=status.HTTP_200_OK)

    if request.method == "POST":
        serializer = TareaSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data,status=status.HTTP_201_CREATED)
        return Response(serializer.errors,status=status.HTTP_400_BAD_REQUEST)

@api_view(["GET", "PUT","DELETE"])
def tarea_detail(request, pk):

    tarea = get_object_or_404(Tarea, pk=pk)

    if request.method == "GET":
        serializer = TareaNestedSerializer(tarea)
        return Response(serializer.data)

    if request.method == "PUT": #partial=True para path en serializer
        serializer = TareaSerializer(tarea, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)

        return Response(serializer.errors,status=status.HTTP_400_BAD_REQUEST)

    tarea.delete()
    return Response(status=status.HTTP_204_NO_CONTENT)

```