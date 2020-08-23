========
Examples
========

The bulk views (and mixins) are very similar to Django REST Framework's own
generic views (and mixins)

.. code-block:: python

    from rest_framework_bulk import (
        BulkListSerializer,
        BulkSerializerMixin,
        ListBulkCreateUpdateDestroyAPIView,
    )

    class FooSerializer(BulkSerializerMixin, ModelSerializer):
        class Meta(object):
            model = FooModel
            # only necessary in DRF3
            list_serializer_class = BulkListSerializer

    class FooView(ListBulkCreateUpdateDestroyAPIView):
        queryset = FooModel.objects.all()
        serializer_class = FooSerializer

The above will allow to create the following queries:

* GET

::

    # list queryset
    GET

* POST

::

    # create single resource
    # json object in request data
    {"field":"value","field2":"value2"}

* POST

::

    # create multiple resources
    # json list of objects in data
    [{"field":"value","field2":"value2"}]

* PUT

::

    # update multiple resources (requires all fields)
    # json list of objects in data
    [{"field":"value","field2":"value2"}]

* PATCH

::

    # partial update multiple resources
    # json list of objects in data
    [{"field":"value"}]    

* DELETE (for more info, see :ref:`Notes`)

::

    # delete queryset
    # json list of ids of objects to be deleted
    [id1, id2, id3 ...]

Router
======

The bulk router can automatically map the bulk actions.

.. code-block:: python

    from rest_framework_bulk.routes import BulkRouter

    class UserViewSet(BulkModelViewSet):
        model = User

        def allow_bulk_destroy(self, qs, filtered):
            """Don't forget to fine-grain this method"""

    router = BulkRouter()
    router.register(r'users', UserViewSet)


Rest Framework V3
=================

Django REST Framework made many API changes which included major changes
in serializers. As a result, please note the following in order to use
DRF-bulk with DRF3:

* You must specify custom ``list_serializer_class`` if your view(set)
  will require update functionality (when using ``BulkUpdateModelMixin``)
* DRF3 removes read-only fields from ``serializer.validated_data``.
  As a result, it is impossible to correlate each ``validated_data``
  in ``ListSerializer`` with a model instance to update since ``validated_data``
  will be missing the model primary key since that is a read-only field.
  To deal with that, you must use ``BulkSerializerMixin`` mixin in your serializer
  class which will add the model primary key field back to the ``validated_data``.
  By default ``id`` field is used however you can customize that field
  by using ``update_lookup_field`` in the serializers ``Meta``.

.. code-block:: python

    class FooSerializer(BulkSerializerMixin, ModelSerializer):
        class Meta(object):
            model = FooModel
            list_serializer_class = BulkListSerializer
            update_lookup_field = 'slug'


.. _Notes:

Notes
=====

Most API urls have two URL levels for each resource:

1. ``url(r'foo/', ...)``
2. ``url(r'foo/(?P<pk>\d+)/', ...)``

The second url however is not applicable for bulk operations because
the url directly maps to a single resource. Therefore all bulk
generic views only apply to the first url.

There are multiple generic view classes in case only a certail
bulk functionality is required. For example ``ListBulkCreateAPIView``
will only do bulk operations for creating resources.
For a complete list of available generic view classes, please
take a look at the source code at ``generics.py`` as it is mostly
self-explanatory.

Most bulk operations are pretty safe in terms of how they operate,
that is you explicitly describe all requests. For example, if you
need to update 3 specific resources, you have to explicitly identify
those resources in the request's ``PUT`` or ``PATCH`` data.
The only exception to this is bulk delete. Consider a ``DELETE``
request to the first url. That can potentially delete all resources
without any special confirmation. To try to account for this, bulk delete
mixin allows to implement a hook to determine if the bulk delete
request should be allowed.

.. code-block:: python

    class FooView(BulkDestroyAPIView):
        def allow_bulk_destroy(self, qs, filtered):
            # custom logic here

            # default checks if the qs was filtered
            # qs comes from self.get_queryset()
            # filtered comes from self.filter_queryset(qs)
            return qs is not filtered

By default it checks if the queryset was filtered and if not will not
allow the bulk delete to complete. The logic here is that if the request
is filtered to only get certain resources, more attention was payed hence
the action is less likely to be accidental. On how to filter requests,
please refer to Django REST
`docs <http://www.django-rest-framework.org/api-guide/filtering>`_.
Either way, please use bulk deletes with extreme caution since they
can be dangerous.