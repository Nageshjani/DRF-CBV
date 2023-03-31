```python
INSTALLED_APPS = [
    'app',
    'rest_framework'
]
```

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=100)

    def __str__(self):
        return self.title
```


```python
from django.contrib import admin
from .models import Book

admin.site.register(Book)
```

## app/serializers.py
```python
from .models import Book
from rest_framework import serializers


class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model=Book
        fields='__all__'
```


```bash
python manage.py makemigrations
python manage.py migrate
```


```python

from django.contrib import admin
from django.urls import path
from app.views import BookListAPIView,BookDetailAPIView,BookDeleteAPIView,BookCreateAPIView,BookUpdateAPIView


urlpatterns = [
    path('admin/', admin.site.urls),
    path('',BookListAPIView.as_view(),name='books_list'),
    path('create/', BookCreateAPIView.as_view(), name='book_create'),
    path('detail/<int:pk>',BookDetailAPIView.as_view(), name='book_detail'),
    path('update/<int:pk>',BookUpdateAPIView.as_view(), name='book_update'),
    path('delete/<int:pk>',BookDeleteAPIView.as_view(), name='book_delete'),
]

```



```python

from rest_framework import generics
from .models import Book
from app.serializers import BookSerializer
from rest_framework.response import Response





#GET_ALL
class BookListAPIView(generics.ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())
        serializer = self.get_serializer(queryset, many=True)

        # Custom response
        data = {
            'books': serializer.data
        }

        return Response(data)

#GET ONE
class BookDetailAPIView(generics.RetrieveAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    lookup_field = 'pk'
    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)

        # Custom response
        data = {
            'book': serializer.data
        }

        return Response(data)

#DELETE
class BookDeleteAPIView(generics.DestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    lookup_field = 'pk'
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        self.perform_destroy(instance)

        # Custom response
        message = f"Book with ID {kwargs['pk']} deleted successfully."
        data = {
            'message': message
        }

        return Response(data)
    

#POST
class BookCreateAPIView(generics.CreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)

        # Custom response
        message = "Book created successfully."
        data = {
            'message': message,
            'book': serializer.data
        }

        headers = self.get_success_headers(serializer.data)
        return Response(data, status=201, headers=headers)

#PUT
class BookUpdateAPIView(generics.UpdateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    lookup_field = 'pk'
    def update(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response("done")


```