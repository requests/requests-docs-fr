.. _quickstart:

Quickstart
==========

.. module:: requests.models

Impatient de commencer? Cette page vous donne une bonne introduction pour démarrer
avec Requests. Ceci implique que vous ayez déja Requests installé. Si ce n'est pas
la cas, suivez la section  :ref:`Installation <install>`.

Premièrement, vérifier que:

* Requests est :ref:`installé <install>`
* Requests est :ref:`à jour <updates>`


Commençons avec des exemples et des cas simples.


Créer une requête
-----------------

Créer une requête standard avec Request est très simple.

Commençons par import le module Requests::

    >>> import requests

Maintenant, essayons de récupérer une page web. Pour cette exemple, récupérons
la timeline publique de github ::

    >>> r = requests.get('https://github.com/timeline.json')

Nous récupérons alors un objet :class:`Response` appelé ``r``. Celui-ci contient
toutes les informations dont nous avons besoin.

L'API simple de Requests permet d'effectuer toute sorte de requête HTTP très 
simplement. Par exemple, pour faire une requete HTTP POST::

    >>> r = requests.post("http://httpbin.org/post")

Pratique, non? Et pour les autres types de requêtes: PUT, DELETE, HEAD et
OPTIONS ? C'est tout aussi simple::

    >>> r = requests.put("http://httpbin.org/put")
    >>> r = requests.delete("http://httpbin.org/delete")
    >>> r = requests.head("http://httpbin.org/get")
    >>> r = requests.options("http://httpbin.org/get")

Jusqu'ici tout va bien, et c'est juste un aperçu de ce que Requests peut faire.


Passer des paramètres dans les URLs
-----------------------------------

Il est fréquent d'avoir besoin de passer des données dans les URLs sous forme
de paramètres. En construisant l'URL à la main, ces données devraient être 
fournies sous forme de clé/valeur dans l'URL après un point d'interrogation,
par exemple ``httpbin.org/get?key=val``. Requests vous permet de fournir ces
arguments sous forme de dictionnaire, en utilisant l'argument ``params``. Par
exemple, si vous souhaitez passer ``key1=value1`` et ``key2=value2`` à
``httpbin.org/get``, vous pouvez utiliser le code suivant::

    >>> payload = {'key1': 'value1', 'key2': 'value2'}
    >>> r = requests.get("http://httpbin.org/get", params=payload)

Vous pouvez constater que l'URL a été correctement encodée en imprimant l'URL::

    >>> print r.url
    http://httpbin.org/get?key2=value2&key1=value1

Note that any dictionary key whose value is ``None`` will not be added to the
URL's query string.


Contenu de la réponse
---------------------

Nous pouvons lire le contenu de la réponse du serveur. Pour reprendre l'exemple
de la timeline GitHub::

    >>> import requests
    >>> r = requests.get('https://github.com/timeline.json')
    >>> r.text
    u'[{"repository":{"open_issues":0,"url":"https://github.com/...

Lorsque vous effectuez une requête, Requests devine l'encodage de la réponse en
fonction des en-têtes HTTP. Le texte décodé selon cet encodage est alors
accesible via ``r.text``. Pour consulter l'encoding que Requests a utilisé, et
le modifier, utilisez la propriété ``r.encoding``::

    >>> r.encoding
    'utf-8'
    >>> r.encoding = 'ISO-8859-1'

Lorsque vous modifiez cette propriété, Requests l'utilise la nouvelle valeur
quand vous accédez à ``r.text``.

Requests sait également utiliser des encodages personnalisés si jamais vous en
avez besoin. Si vous avez crée votre propre encodage et l'avez enregistré avec
le module ``codecs``, utilisez simplement le nom du codec comme valeur de
``r.encoding`` et Requests gérera le décodage pour vous.


Réponse binaire
---------------

Pour les requêtes non-texte (fichiers binaires), vous pouvez accéder au 
contenu de la réponse sous forme de bytes::

    >>> r.content
    b'[{"repository":{"open_issues":0,"url":"https://github.com/...

Les réponse avec le header transfer-encodings à ``gzip`` et ``deflate`` sont 
automatiquement décodés pour vous.

Par exemple, pour créer une image à partir de données recues par une requête, vous
pouvez utiliser le code suivant:

    >>> from PIL import Image
    >>> from StringIO import StringIO
    >>> i = Image.open(StringIO(r.content))


Réponse JSON
------------

Si vous devez travailler avec des données JSON, Requests dispose dun décodeur intégré::

    >>> import requests
    >>> r = requests.get('https://github.com/timeline.json')
    >>> r.json()
    [{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...

Si jamais le décodage JSON échoue, ``r.json`` soulève une exception. Par exemple,
si la réponse est un 401 (Non authorisé), ``r.json`` soulève ``ValueError: No
JSON object could be decoded``

Réponse brute
-------------

Dans de rares cas, si vous avez besoin d'accéder au contenu brut de la 
réponse serveur, vous pouvez y accéder avec ``r.raw``. Si vous voulez faire
cela, soyez sûr d'avoir indiqué ``stream=True`` dans votre requête initiale. 
Une fois que vous l'avez fait, vous pouvez faire ceci::

    >>> r = requests.get('https://github.com/timeline.json', stream=True)
    >>> r.raw
    <requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
    >>> r.raw.read(10)
    '\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'


En-têtes personnalisées
-----------------------

Si vous souhaitez ajouter des headers HTTP à une requête, passez simpement un
object de type ``dict`` au pramètre ``headers``.

Par exemple, pour spécifier un content-type dans l'exemple précédent::

    >>> import json
    >>> url = 'https://api.github.com/some/endpoint'
    >>> payload = {'some': 'data'}
    >>> headers = {'content-type': 'application/json'}


Requêtes POST avancées
----------------------

Typiquement, vous avec besoin d'envoyer des données encodées comme
par exemple un formulaire HTML. Pour cela, on passe simplement un
dictionnaire avec l'argument `data`. Votre dictionnaire de données
sera automatiquement encodé comme un formulaire au moment de la requête::

    >>> payload = {'key1': 'value1', 'key2': 'value2'}
    >>> r = requests.post("http://httpbin.org/post", data=payload)
    >>> print r.text
    {
      ...
      "form": {
        "key2": "value2",
        "key1": "value1"
      },
      ...
    }

Dans certains cas, vous ne souhaitez pas que les données soit encodées. 
Si vous passez une chaîne de caractères ``string`` à la place d'un objet 
``dict``, les données seront postées directement.

Par exemple, l'API GitHub v3 accepte les requêtes POST/PATCH avec des données 
JSON::

    >>> import json
    >>> url = 'https://api.github.com/some/endpoint'
    >>> payload = {'some': 'data'}

    >>> r = requests.post(url, data=json.dumps(payload))


POST avec des fichiers Multipart
--------------------------------

Requests simplifie l'upload de fichiers encodés en MultiPart::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': open('report.xls', 'rb')}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      ...
      "files": {
        "file": "<censored...binary...data>"
      },
      ...
    }

Pour forcer le nom du fichier explicitement::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': ('report.xls', open('report.xls', 'rb'))}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      ...
      "files": {
        "file": "<censored...binary...data>"
      },
      ...
    }

Vous pouvez également envoyer des chaînes de caractères en tant que fichier ::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      ...
      "files": {
        "file": "some,data,to,send\\nanother,row,to,send\\n"
      },
      ...
    }


Codes de retour des réponses (status)
-------------------------------------

Nous pouvons vérifier le code de retour d'une réponse::

    >>> r = requests.get('http://httpbin.org/get')
    >>> r.status_code
    200

Requests fournit également un code de statut interne pour faciliter
les vérifications :

    >>> r.status_code == requests.codes.ok
    True

<<<<<<< HEAD
Si nous faisons une mauvaise requête (une erreur client 4XX ou une erreur
serveur), nous pouvons lever une exception avec
:class:`Response.raise_for_status()`::

    >>> bad_r = requests.get('http://httpbin.org/status/404')
    >>> bad_r.status_code
    404

    >>> bad_r.raise_for_status()
    Traceback (most recent call last):
      File "requests/models.py", line 832, in raise_for_status
        raise http_error
    requests.exceptions.HTTPError: 404 Client Error


Mais comme notre ``status_code`` pour ``r`` était ``200``, lorsque l'on
appele ``raise_for_status()`` nous obtenons::

    >>> r.raise_for_status()
    None

Tout va bien.


En-têtes des réponses
---------------------

On peut accéder aux en-têtes HTTP (headers) de la réponse du serveur via
une simple dictionnaire Python::

    >>> r.headers
    {
        'content-encoding': 'gzip',
        'transfer-encoding': 'chunked',
        'connection': 'close',
        'server': 'nginx/1.0.4',
        'x-runtime': '148ms',
        'etag': '"e1ca502697e5c9317743dc078f67693f"',
        'content-type': 'application/json'
    }

Ce dictionnaire est cependant particulier : Il est spécifique aux en-têtes HTTP.
En effet, selon la `RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html>`_, 
les en-têtes HTTP ne doivent pas être sensibles à la casse.

Donc, nous pouvons accéder aux en-têtes quelque soit la casse utilisée::

    >>> r.headers['Content-Type']
    'application/json'

    >>> r.headers.get('content-type')
    'application/json'


Cookies
-------

Si la résponse contient des Cookies, vous pouvez y accéder rapidement::

    >>> url = 'http://example.com/some/cookie/setting/url'
    >>> r = requests.get(url)

    >>> r.cookies['example_cookie_name']
    'example_cookie_value'

Pour envoyer vos propres cookies au serveur, vous pouvez utiliser le
paramètre ``cookies``::

    >>> url = 'http://httpbin.org/cookies'
    >>> cookies = dict(cookies_are='working')

    >>> r = requests.get(url, cookies=cookies)
    >>> r.text
    '{"cookies": {"cookies_are": "working"}}'

Redirections et Historique
--------------------------

Requests effectue automatiquement les redirections lorsque vous utilisez les
méthodes GET et OPTIONS.

GitHub redirige toutes les requêtes HTTP vers HTTPS. Nous pouvons utiliser la
méthode ``history`` de l'objet Response pour tracer les redirections. Regardons
ce qu'il se passe pour Github::

    >>> r = requests.get('http://github.com')
    >>> r.url
    'https://github.com/'
    >>> r.status_code
    200
    >>> r.history
    [<Response [301]>]

La liste :class:`Response.history` contient la liste des objets 
:class:`Request` qui ont été crées pour compléter la requête. Cette liste est triée de la plus ancienne à la plus récente.
du plus ancien au plus récent.

Si vous utilisez les methodes GET ou OPTIONS, vous pouvez désactiver la 
gestion des redirections avec le paramètre ``allow_redirects``::

    >>> r = requests.get('http://github.com', allow_redirects=False)
    >>> r.status_code
    301
    >>> r.history
    []

Si vous utilisez POST, PUT, PATCH, DELETE ou HEAD vous pouvez également
autoriser explicitement les redirections::

    >>> r = requests.post('http://github.com', allow_redirects=True)
    >>> r.url
    'https://github.com/'
    >>> r.history
    [<Response [301]>]


Timeouts
--------

Vous pouvez demander à Requests d'arrêter d'attendre après un certains nombre de secondes 
avec le paramètre ``timeout``::

    >>> requests.get('http://github.com', timeout=0.001)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)


.. admonition:: Note

    ``timeout`` n'est pas une limite de temps sur le téléchargement de la
    réponse entière, mais plutot, lève une exception si le serveur n'a pas
    commencé à répondre pour le ``timeout`` specifié. 

Erreurs et Exceptions
---------------------

Dans le cas de problèmes de réseau (e.g. erreurs DNS, connexions refusées, etc),
Requests lévera une exception :class:`~requests.exceptions.ConnectionError`.

Dans les rares cas de réponses HTTP invalides, Requests lévera une exception 
:class:`~requests.exceptions.HTTPError`.

Si une requête dépasse le temps d'attente, une exception :class:`~requests.exceptions.timeout` est levée.

Si une requête dépasse le nombre maximum de redirections possibles configuré,
une exception :class:`~requests.exceptions.TooManyRedirects` est levée.

Toutes les exceptions levées par Requests héritent de
:class:`requests.exceptions.RequestException`.

-----------------------

Prêt pour aller plus loin ? Visitez la section :ref:`avancée <advanced>`.
