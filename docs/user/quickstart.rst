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


Faire une requête GET
---------------------

Lancer une requête standard avec Request est très simple.

Récupérons la timeline publique de github ::

    r = requests.get('https://github.com/timeline.json')

Nous récupérons un objet :class:`Response` appelé ``r``. Celui-ci fournit
toutes les informations dont nous avons besoin.

Typiquement, vous aimeriez envoyer des informations quand la query string (URL).
Pour faire cela, il suffit de passer un dictionnaire avec l'argument `params`.
Votre dictionnaire de données sera automatiquement encodé lorsque la requête 
sera faite::

    >>> payload = {'key1': 'value1', 'key2': 'value2'}
    >>> r = requests.get("http://httpbin.org/get", params=payload)
    >>> print r.text
    {
      "origin": "179.13.100.4",
      "args": {
        "key2": "value2",
        "key1": "value1"
      },
      "url": "http://httpbin.org/get",
      "headers": {
        "Connections": "keep-alive",
        "Content-Length": "",
        "Accept-Encoding": "identity, deflate, compress, gzip",
        "Accept": "*/*",
        "User-Agent": "python-requests/0.11.0",
        "Host": httpbin.org",
        "Content-Type": ""
      },
    }



Contenu de la réponse
---------------------

Nous pouvons lire le contenu de la réponse du serveur::

    >>> r.text
    '[{"repository":{"open_issues":0,"url":"https://github.com/...

Requests décode automatiquement le contenu envoyé par le serveur. La plupart des
caractères unicode sont décodés de manière transparente.

Quand vous créez une requête, l'attribut ``r.encoding`` est défini par rapport 
aux en-têtes HTTP (headers). Requests utilisera ensuite cet encodage quand vous 
accéderez à ``r.text``. Si ``r.encoding`` est ``None``, Requests tentera de 
deviner l'encodage du corps de la réponse. Vous pouvez également forcer 
``r.encoding`` à l'encodage de votre choix, et ce charset sera alors utilisé 
pour le décodage de la réponse.


Contenu de réponse binaire
--------------------------

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


Contenu de réponse brut
-----------------------

Dans de rares cas, si vous avez besoin d'accéder au contenu brut de la 
réponse serveur, vous pouvez y accéder avec ``r.raw``::

    >>> r.raw
    <requests.packages.urllib3.response.HTTPResponse object at 0x101194810>

    >>> r.raw.read(10)
    '\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'



Faire une requête POST
----------------------

Les requêtes POST sont aussi simples::

    r = requests.post("http://httpbin.org/post")


Typiquement, vous aimeriez envoyer des données encodées comme un formulaire HTML.
Pour cela, passez simplement un dictionnaire avec l'argument `data`. Votre
dictionnaire de données sera automatiquement encodé comme un formulaire au moment
de la requête::

    >>> payload = {'key1': 'value1', 'key2': 'value2'}
    >>> r = requests.post("http://httpbin.org/post", data=payload)
    >>> print r.text
    {
      "origin": "179.13.100.4",
      "files": {},
      "form": {
        "key2": "value2",
        "key1": "value1"
      },
      "url": "http://httpbin.org/post",
      "args": {},
      "headers": {
        "Content-Length": "23",
        "Accept-Encoding": "identity, deflate, compress, gzip",
        "Accept": "*/*",
        "User-Agent": "python-requests/0.8.0",
        "Host": "127.0.0.1:7077",
        "Content-Type": "application/x-www-form-urlencoded"
      },
      "data": ""
    }

Dans certains cas, vous ne souhaitez pas que les données soit encodées. 
Si vous passez une chaîne de caractères ``string`` à la place d'un object 
``dict``, les données seront postées directement.

Par exemple, l'API GitHub v3 accepte les requêtes POST/PATCH avec des données 
JSON::

    url = 'https://api.github.com/some/endpoint'
    payload = {'some': 'data'}

    r = requests.post(url, data=json.dumps(payload))


En-têtes personnalisées
-----------------------

Si vous souhaitez ajouter des en-têtes HTTP personnalisées (headers) à une 
requête, il vous suffit de passer un object ``dict`` dans le paramètre ``headers``.

Par exemple, nous n'avions pas spécifié de content-type dans notre exemple 
précédent::

    url = 'https://api.github.com/some/endpoint'
    payload = {'some': 'data'}
    headers = {'content-type': 'application/json'}

    r = requests.post(url, data=json.dumps(payload), headers=headers)


POST avec des fichiers MultiPart
--------------------------------

Requests simplifie l'upload de fichiers encodés en MultiPart::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'report.xls': open('report.xls', 'rb')}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      "origin": "179.13.100.4",
      "files": {
        "report.xls": "<censored...binary...data>"
      },
      "form": {},
      "url": "http://httpbin.org/post",
      "args": {},
      "headers": {
        "Content-Length": "3196",
        "Accept-Encoding": "identity, deflate, compress, gzip",
        "Accept": "*/*",
        "User-Agent": "python-requests/0.8.0",
        "Host": "httpbin.org:80",
        "Content-Type": "multipart/form-data; boundary=127.0.0.1.502.21746.1321131593.786.1"
      },
      "data": ""
    }

Forcer le nom du fichier explicitement::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': ('report.xls', open('report.xls', 'rb'))}

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      "origin": "179.13.100.4",
      "files": {
        "file": "<censored...binary...data>"
      },
      "form": {},
      "url": "http://httpbin.org/post",
      "args": {},
      "headers": {
        "Content-Length": "3196",
        "Accept-Encoding": "identity, deflate, compress, gzip",
        "Accept": "*/*",
        "User-Agent": "python-requests/0.8.0",
        "Host": "httpbin.org:80",
        "Content-Type": "multipart/form-data; boundary=127.0.0.1.502.21746.1321131593.786.1"
      },
      "data": ""
    }

Envoyer des chaînes de caractères en tant que fichier ::

    >>> url = 'http://httpbin.org/post'
    >>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')} 

    >>> r = requests.post(url, files=files)
    >>> r.text
    {
      "origin": "179.13.100.4",
      "files": {
        "file": "some,data,to,send\\nanother,row,to,send\\n"
      },
      "form": {},
      "url": "http://httpbin.org/post",
      "args": {},
      "headers": {
         "Content-Length": "216",
         "Accept-Encoding": "identity, deflate, compress, gzip",
         "Connection": "keep-alive",
         "Accept": "*/*",
         "User-Agent": "python-requests/0.11.1",
         "Host": "httpbin.org", 
         "Content-Type": "multipart/form-data; boundary=127.0.0.1.502.41433.1335385481.788.1"
      }, 
      "json": null,
      "data": ""
    }


Codes de retour des réponses (status)
-------------------------------------

Nous pouvons vérifier le code de retour d'une réponse::

    >>> r.status_code
    200

Requests fournit également un code de statut interne pour faciliter
les verifications :

    >>> r.status_code == requests.codes.ok
    True

Si nous faisons une mauvaise requête (code de retour autre que 200), nous
pouvons lever une exception avec :class:`Response.raise_for_status()`::

    >>> _r = requests.get('http://httpbin.org/status/404')
    >>> _r.status_code
    404

    >>> _r.raise_for_status()
    Traceback (most recent call last):
      File "requests/models.py", line 394, in raise_for_status
        raise self.error
    urllib2.HTTPError: HTTP Error 404: NOT FOUND

Mais si notre ``status_code`` est ``200``, lorsqu'on l'appelle::

    >>> r.raise_for_status()
    None

Tout va bien.


En-têtes des réponses
---------------------

On peut accéder aux en-têtes HTTP (headers) de la réponse du serveur via
une simple dictionnaire Python::

    >>> r.headers
    {
        'status': '200 OK',
        'content-encoding': 'gzip',
        'transfer-encoding': 'chunked',
        'connection': 'close',
        'server': 'nginx/1.0.4',
        'x-runtime': '148ms',
        'etag': '"e1ca502697e5c9317743dc078f67693f"',
        'content-type': 'application/json; charset=utf-8'
    }

Ce dictionnaire est cependant particulier : Il est spécifique aux en-têtes HTTP.
En effet, selon la `RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html>`_, 
les en-têtes HTTP ne doivent pas être sensibles à la casse.

Donc, nous pouvons accéder aux en-têtes quelque soit la casse utilisée::

    >>> r.headers['Content-Type']
    'application/json; charset=utf-8'

    >>> r.headers.get('content-type')
    'application/json; charset=utf-8'

Si l'en-tête n'existe pas dans la Response, la valeur par défaut est ``None``::

    >>> r.headers['X-Random']
    None


Cookies
-------

Si la résponse contient des Cookies, vous pouvez y accéder rapidement::

    >>> url = 'http://httpbin.org/cookies/set/requests-is/formidable'
    >>> r = requests.get(url)

    >>> r.cookies['requests-is']
    'formidable'

Pour envoyer vos propres cookies au serveur, vous pouvez utiliser le
paramètre ``cookies``::

    >>> url = 'http://httpbin.org/cookies'
    >>> cookies = dict(cookies_are='working')

    >>> r = requests.get(url, cookies=cookies)
    >>> r.text
    '{"cookies": {"cookies_are": "working"}}'


Authentification basique
------------------------

La plupart des services web nécessitent une authentification. Il y a
différents types d'authentification, mais la plus commune est 
l'authentification HTTP basique.

Utiliser l'authentification basique avec Requests est extrêmement simple::

    >>> from requests.auth import HTTPBasicAuth
    >>> requests.get('https://api.github.com/user', auth=HTTPBasicAuth('user', 'pass'))
    <Response [200]>

Comme l'authentification HTTP basique est le standard le plus répandu, Requests 
fournit un raccourci pour cette méthode d'authentification::

    >>> requests.get('https://api.github.com/user', auth=('user', 'pass'))
    <Response [200]>

Fournir de cette manière un tuple d'authentification au paramètre `auth` 
équivaut à utiliser l'exemple ``HTTPBasicAuth`` ci-dessus.


Authentification Digest 
-----------------------

Une autre form populaire de protection des web services est l'autthentification Digest::

    >>> from requests.auth import HTTPDigestAuth
    >>> url = 'http://httpbin.org/digest-auth/auth/user/pass'
    >>> requests.get(url, auth=HTTPDigestAuth('user', 'pass'))
    <Response [200]>


Authentification OAuth
----------------------

Le projet `requests-oauth <http://pypi.python.org/pypi/requests-oauth>`_ de Miguel Araujo fournit une interface
simple pour établir des connexions OAuth. La documentation et des exemples peuvent être trouvées sur `git repository <https://github.com/maraujop/requests-oauth>`_.


Redirections et Historique
--------------------------

Requests effectue automatiquement les redirections en utilisant des méthodes 
idempotentes, qui ne modifient pas la requête.

GutHub redirige tout le traffic HTTP vers HTTPS. Regardons ce qu'il se passe::

    >>> r = requests.get('http://github.com')
    >>> r.url
    'https://github.com/'
    >>> r.status_code
    200
    >>> r.history
    [<Response [301]>]

La liste :class:`Response.history` contient la liste des objets 
:class:`Request` qui ont été crées pour compléter la requête.

Si vous utilisez les methodes GET, HEAD, or OPTIONS, vous pouvez désactiver la 
gestion des redirections avec le paramètre ``allow_redirects``::

    >>> r = requests.get('http://github.com', allow_redirects=False)
    >>> r.status_code
    301
    >>> r.history
    []

Si vous utilisez POST, PUT, PATCH, *&c*, vous pouvez aussi autoriser explicitement
les redirections::

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
    requests.exceptions.Timeout: Request timed out.

.. admonition:: Note

    ``timeout`` affecte uniquement le processus de connection, pas le temps de téléchargment
    des réponses.


Erreurs et Exceptions
---------------------

Dans le cas de problèmes de réseau (e.g. erreurs DNS, connexions refusées, etc),
Requests lévera une exception :class:`ConnectionError`.

Dans les rares cas de réponses HTTP invalides, Requests lévera une exception 
:class:`HTTPError`.

Si une requête dépasse le temps d'attente, une exception :class:`Timeout` est levée.

Si une requête dépasse le nombre maximum de redirections possibles, une exception 
:class:`TooManyRedirects` est levée.

Toutes les exceptions levées par Requests héritent de :class:`requests.exceptions.RequestException`.

Vous pouvez vous réferer à :ref:`Configuration API Docs <configurations>` si vous souhaitez toujours lever 
des exceptions :class:`HTTPError` avec l'option ``danger_mode``, ou laisser Requests attraper la majorité 
des :class:`requests.exceptions.RequestException` avec l'option ``safe_mode``.

-----------------------

Prêt pour aller plus loin ? Visitez la section :ref:`avancée <advanced>`.
