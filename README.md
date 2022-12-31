# Django Rest Framework Views

__APIView__ class is a base for all the views that you might choose to use in your DRF application. 
- Extending __APIView__ offers the most freedom, but it also leaves a lot more work for you. It's a great choice if you need to have control over every aspect of the view or if you have very complicated views.
- With the __generic view__ classes, you can develop faster and still have quite a bit of control over the API endpoints.
- With __ModelViewSets__, you can get an API stood up with five lines of codes





To quickly see the differences between them, let's look at example of all three in action achieving the same thing.

In the following we wants to do the below actions by different views:

- Listing all items
- Creating a new item
- Retrieving, updating, and deleting a single item

## APIViews 
[Read more in DRF documentation part 1](https://www.django-rest-framework.org/api-guide/views/)

[Read more in DRF documentation part 2](https://www.django-rest-framework.org/tutorial/2-requests-and-responses/#pulling-it-all-together)


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
#### Class-based Views
Here's a __Class-based View__ that allows users to delete all the items at once:
```angular2html
from rest_framework.permissions import IsAuthenticated
from rest_framework.renderers import JSONRenderer
from rest_framework.response import Response
from rest_framework.views import APIView

class ItemsNotDone(APIView):

    permission_classes = [IsAuthenticated]  # policy attribute
    renderer_classes = [JSONRenderer]       # policy attribute

    def get(self, request):

        user_count = Item.objects.filter(done=False).count()
        content = {'not_done': user_count}

        return Response(content)

```


#### Function-based Views
If you're writing a view in the form of a function, you'll need to use the __@api_view__ decorator.
```angular2html
from rest_framework.decorators import api_view, permission_classes, renderer_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.renderers import JSONRenderer
from rest_framework.response import Response

@api_view(['GET'])
@permission_classes([IsAuthenticated])  # policy decorator
@renderer_classes([JSONRenderer])       # policy decorator
def items_not_done(request):
    user_count = Item.objects.filter(done=False).count()
    content = {'not_done': user_count}

    return Response(content)

```



## Generic Views
[Read more in DRF documentation part 1](https://www.django-rest-framework.org/api-guide/generic-views/)

[Read more in DRF documentation part 2](https://www.django-rest-framework.org/tutorial/3-class-based-views/)


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
__ViewSet__ classes remove the need for additional lines of code and, when coupled with __routers__, help keep your URLs consistent. The most significant advantage of ViewSets is that the URL construction is handled automatically (with a router class).


[Read more in DRF documentation part 1](https://www.django-rest-framework.org/api-guide/viewsets/)

[Read more in DRF documentation part 2](https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/)


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
There are four types of __ViewSets__, from the most basic to the most powerful:

1- ViewSet

2- GenericViewSet

3- ReadOnlyModelViewSet

4- ModelViewSet



__NOTE:__ This article is under development and will be improved ...
