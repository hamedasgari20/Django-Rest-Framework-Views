# Django Rest Framework Views

To quickly see the differences between them, let's look at example of all three in action achieving the same thing.

In the following we wants to do the below actions by different views:

- Listing all items
- Creating a new item
- Retrieving, updating, and deleting a single item

## APIViews
Here's how you can accomplish this by extending __APIView__:
```
class ItemsList(APIView):

    def get(self, request, format=None):
        items = Item.objects.all()
        serializer = ItemSerializer(items, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = ItemSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class ItemDetail(APIView):

    def get(self, request, pk, format=None):
        item = get_object_or_404(Item.objects.all(), pk=pk)
        serializer = ItemSerializer(item)

        return Response(serializer.data)

    def put(self, request, pk, format=None):
        item = get_object_or_404(Item.objects.all(), pk=pk)
        serializer = ItemSerializer(item, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        item = get_object_or_404(Item.objects.all(), pk=pk)
        item.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
Its ```url.py```file:
```angular2html
from django.urls import path
from views import ItemsList, ItemDetail

urlpatterns = [
    path('api-view', ItemsList.as_view()),
    path('api-view/<pk>', ItemDetail.as_view()),
]

```
## Generic Views
Here's how you do the same thing with concrete __generic views__:
```angular2html
class ItemsListGeneric(ListCreateAPIView):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer


class ItemDetailGeneric(RetrieveUpdateDestroyAPIView):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer

```
Its ```url.py```file:
```angular2html
from django.urls import path,
from views import ItemsListGeneric, ItemDetailGeneric

urlpatterns = [
    path('generic-view', ItemsListGeneric.as_view()),
    path('generic-view/<pk>', ItemDetailGeneric.as_view()),
]

```
## ViewSets
And here are the lines that you need with __ModelViewSet__:
```angular2html
class ItemsViewSet(ModelViewSet):
    serializer_class = ItemSerializer
    queryset = Item.objects.all()

```
Its ```url.py```file:
```angular2html
from django.urls import path, include
from rest_framework import routers
from views import ItemsViewSet

router = routers.DefaultRouter()
router.register(r'viewset', ItemsViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```


