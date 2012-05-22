.. _api:

API
===

.. module:: requests

Cette partie de la documentation présente toutes les interfaces possibles de
Requests. Pour certaines parties, Requests dépend de librairies externes,
nous documentons les plus importantes ici et fournissons des liens vers
les documentations externes.



Interface Principale
--------------------

Toutes les fonctionnalités de Requests sont accessibles via ces 7 méthodes.
Elles retournent toutes une instance de l'objet :class:`Response <Response>`.


.. autofunction:: request

---------------------


.. autoclass:: Response
   :inherited-members:

---------------------

.. autofunction:: head
.. autofunction:: get
.. autofunction:: post
.. autofunction:: put
.. autofunction:: patch
.. autofunction:: delete


-----------------

.. autofunction:: session



Exceptions
~~~~~~~~~~

.. module:: requests

.. autoexception:: RequestException
.. autoexception:: ConnectionError
.. autoexception:: HTTPError
.. autoexception:: URLRequired
.. autoexception:: TooManyRedirects


.. _configurations:

Configurations
--------------

.. automodule:: requests.defaults


.. _async:

Async
-----

.. module:: requests.async


.. autofunction:: map
.. autofunction:: request
.. autofunction:: head
.. autofunction:: get
.. autofunction:: post
.. autofunction:: put
.. autofunction:: patch
.. autofunction:: delete



Utilitaires
-----------

Ces fonctions sont utilisées en internes, mais peuvent être utiles 
en dehors de Requests.

.. module:: requests.utils

Status Code Lookup
~~~~~~~~~~~~~~~~~~

.. autofunction:: requests.codes

::

    >>> requests.codes['temporary_redirect']
    307

    >>> requests.codes.teapot
    418

    >>> requests.codes['\o/']
    200

Cookies
~~~~~~~

.. autofunction:: dict_from_cookiejar
.. autofunction:: cookiejar_from_dict
.. autofunction:: add_dict_to_cookiejar


Encodages
~~~~~~~~~

.. autofunction:: get_encodings_from_content
.. autofunction:: get_encoding_from_headers
.. autofunction:: get_unicode_from_response
.. autofunction:: decode_gzip


Internes
--------

Ces éléments sont des composants internes de Requests, et ne doivent jamais
être vus par l'utilisateur final (développeur). Cette partie de la documentation
existe uniquement pour ceux qui étendent les fonctionnalités de Requests.


Classes
~~~~~~~

.. autoclass:: requests.Response
   :inherited-members:

.. autoclass:: requests.Request
   :inherited-members:

.. _sessionapi:

.. autoclass:: requests.Session
   :inherited-members:

