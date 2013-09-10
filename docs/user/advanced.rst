.. _advanced:

Utilisation avancée
===================

Ce document traite de quelques fonctionnalités avancées de Requests.


Objets Session
--------------

L'objet Session vous permet de conserver des paramètres entre plusieurs
requêtes. Il permet également de conserver les cookies entre toutes les 
requêtes de la même instance Session.

Un objet Session a toutes les methodes de l'API Requests principale.

Pour conserver des cookies entre les requêtes::

    s = requests.Session()

    s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
    r = s.get("http://httpbin.org/cookies")

    print r.text
    # '{"cookies": {"sessioncookie": "123456789"}}'


Les Sessions peuvent aussi être utilisées pour fournir des valeurs par défaut
aux requêtes. Ceci est fait en fournissant des propriétés à l'objet Session::

    s = requests.Session()
    s.auth = ('user', 'pass')
    s.headers.update({'x-test': 'true'})

    # both 'x-test' and 'x-test2' are sent
    s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})


Tous les dictionnaires que vous passez aux methodes de requête sont fusionnés
avec les valeur de session déja définies. Les paramètres de la méthode 
surchargent les paramètres de session.

.. admonition:: Supprimer une valeur d'un paramètre

    Parfois vous voudrez supprimer des paramètres de session lors de vos
    requêtes. Pour cela, il suffit d'envoyer lors de l'appel de la méthode
    un dictionnaire dont les clés seraient les paramètres a supprimer et les
    valeurs seraient ``None``. Ces paramètres seront alors automatiquement
    omis.

Toutes les valeurs contenues dans la session sont directement accessibles.
Pour en savoir plus, cf :ref:`Session API Docs <sessionapi>`.


Objets Request et Response
--------------------------

Lorsqu'un appel à requests.* est effectué, vous faites deux choses. Premièrement,
vous construisez un object ``Request`` qui va être envoyé au serveur pour récupérer
ou intérroger des ressources. En second, un object ``Response`` est généré une
fois que ``requests`` reçoit une réponse du servuer. L'objet ``Response`` contient
toutes les informations retournées par le serveur mais aussi l'object ``Request``
que vous avz crée initialement. Voici une requête simple pour obtenir des
informations depuis les serveurs Wikipedia::

    >>> r = requests.get('http://en.wikipedia.org/wiki/Monty_Python')

Si nous voulons accéder aux en-têtes renvoyées par le serveur, nous faisons::

    >>> r.headers
    {'content-length': '56170', 'x-content-type-options': 'nosniff', 'x-cache':
    'HIT from cp1006.eqiad.wmnet, MISS from cp1010.eqiad.wmnet', 'content-encoding':
    'gzip', 'age': '3080', 'content-language': 'en', 'vary': 'Accept-Encoding,Cookie',
    'server': 'Apache', 'last-modified': 'Wed, 13 Jun 2012 01:33:50 GMT',
    'connection': 'close', 'cache-control': 'private, s-maxage=0, max-age=0,
    must-revalidate', 'date': 'Thu, 14 Jun 2012 12:59:39 GMT', 'content-type':
    'text/html; charset=UTF-8', 'x-cache-lookup': 'HIT from cp1006.eqiad.wmnet:3128,
    MISS from cp1010.eqiad.wmnet:80'}

Toutefois, si nous souhaitons récupérer les en-têtes que nous avions envoyées au
serveur, nous accédons simplement à la requête, et aux en-têtes de la requête::

    >>> r.request.headers
    {'Accept-Encoding': 'identity, deflate, compress, gzip',
    'Accept': '*/*', 'User-Agent': 'python-requests/1.2.0'}

Prepared Requests
-----------------

Whenever you receive a :class:`Response <requests.models.Response>` object
from an API call or a Session call, the ``request`` attribute is actually the
``PreparedRequest`` that was used. In some cases you may wish to do some extra
work to the body or headers (or anything else really) before sending a
request. The simple recipe for this is the following::

    from requests import Request, Session

    s = Session()
    prepped = Request('GET',  # or any other method, 'POST', 'PUT', etc.
                      url,
                      data=data
                      headers=headers
                      # ...
                      ).prepare()
    # do something with prepped.body
    # do something with prepped.headers
    resp = s.send(prepped,
                  stream=stream,
                  verify=verify,
                  proxies=proxies,
                  cert=cert,
                  timeout=timeout,
                  # etc.
                  )
    print(resp.status_code)

Since you are not doing anything special with the ``Request`` object, you
prepare it immediately and modified the ``PreparedRequest`` object. You then
send that with the other parameters you would have sent to ``requests.*`` or
``Sesssion.*``.

Verifications certificats SSL
-----------------------------

Requests peut vérifier les certificats SSL sur les requêtes HTTPS, comme n'importe quel navigateur web. Pour vérifier le certificat d'un serveur, vous pouvez utiliser l'argument ``verify``::

    >>> requests.get('https://kennethreitz.com', verify=True)
    requests.exceptions.SSLError: hostname 'kennethreitz.com' doesn't match either of '*.herokuapp.com', 'herokuapp.com'

SSL n'est pas configuré sur ce domaine, donc cela génère une
erreur. Parfait. Par contre, GitHub en a un::

    >>> requests.get('https://github.com', verify=True)
    <Response [200]>

Vous pouvez aussi passer au paramètre ``verify`` le chemin vers un fichier ``CA_BUNDLE`` pour les certificats privés. Vous pouvez également définir la variable d'environnement ``REQUESTS_CA_BUNDLE``.

Requests can also ignore verifying the SSL certificate if you set ``verify`` to False.

::

    >>> requests.get('https://kennethreitz.com', verify=False)
    <Response [200]>

By default, ``verify`` is set to True. Option ``verify`` only applies to host certs.

You can also specify a local cert to use as client side certificate, as a single file (containing the private key and the certificate) or as a tuple of both file's path::

    >>> requests.get('https://kennethreitz.com', cert=('/path/server.crt', '/path/key'))
    <Response [200]>

If you specify a wrong path or an invalid cert::

    >>> requests.get('https://kennethreitz.com', cert='/wrong_path/server.pem')
    SSLError: [Errno 336265225] _ssl.c:347: error:140B0009:SSL routines:SSL_CTX_use_PrivateKey_file:PEM lib


Workflow du contenu des réponses
--------------------------------

Par défaut, lorsque vous effectuez une requête, le corps de la réponse n'est pas
téléchargé automatiquement. Les en-têtes sont téléchargés, mais le contenu
lui-même n'est téléchargé que lorsque vous accédez à l'attribut
:class:`Response.content`::

    tarball_url = 'https://github.com/kennethreitz/requests/tarball/master'
    r = requests.get(tarball_url)

A ce point, seulement les en-têtes de la réponse ont été téléchargé et la
connection est toujours ouverte.::

    if int(r.headers['content-length']) < TOO_LING:
        content = r.content
        ...

You can further control the workflow by use of the :class:`Response.iter_content` and :class:`Response.iter_lines` methods, or reading from the underlying urllib3 :class:`urllib3.HTTPResponse` at :class:`Response.raw`.

Keep-Alive
----------

Bonne nouvelle - grâce à urllib3, le keep-alive est 100% automatique pendant une session! Toutes les requêtes que vous ferez à travers une session réutiliseront automatiquement la connexion appropriée!

A noter que les connexions ne sont libérées pour réutilisation seulement lorsque les données ont été lues. Faites attention à bien mettre ``prefetch`` à ``True`` ou toujours accéder à la propriété ``content`` de l'object ``Response``.


Streaming Uploads
-----------------

Requests supports streaming uploads, which allow you to send large streams or files without reading them into memory. To stream and upload, simply provide a file-like object for your body::

    with open('massive-body') as f:
        requests.post('http://some.url/streamed', data=f)

Chunk-Encoded Requests
----------------------

Requests also supports Chunked transfer encoding for outgoing and incoming requests. To send a chunk-encoded request, simply provide a generator (or any iterator without a length) for your body::


    def gen():
        yield 'hi'
        yield 'there'

    requests.post('http://some.url/chunked', data=gen())


Hooks d'évenements
------------------

Requests dispose d'un système de 'hooks' que vous pouvez utiliser pour
manipuler des portions du processus de requêtage ou signaler des évènements.

Hooks disponibles:

``response``:
    La réponse générée après une requête.


Vous pouvez assigner une fonction de hook par requête, en passant au 
paramètre ``hooks`` de la Request un dictionnaire de hooks 
``{hook_name: callback_function}``::

    hooks=dict(response=print_url)

La fonction ``callback_function`` recevra un bloc de données en premier 
argument.

::

    def print_url(r, *args, **kwargs):
        print(r.url)

Si une exception apparait lors de l'éxecution du callback, un warning est
affiché.

Si le callback renvoie une valeur, on suppose que cela remplace les données
qui lui ont été passées. Si la fonction ne renvoie rien, alors rien n'est
affecté.

Affichons quelques arguments a la volée::

    >>> requests.get('http://httpbin.org', hooks=dict(response=print_url))
    http://httpbin.org
    <Response [200]>


Authentification personnalisée
------------------------------

Requests vous permet de spécifier vos propres mécanismes d'authentification.

N'importe quel 'callable' à qui l'on passe l'argument ``auth`` pour une méthode
de requête a l'opportunité de modifier la requête avant de la dispatcher.

Les implémentations d'authentification doivent hériter de la classe
``requests.auth.AuthBase``, et sont très faciles à définir. Request fournit
deux modèles communs d'authentification dans ``requests.auth``: ``HTTPBasicAuth``
et ``HTTPDigestAuth``.

Admettons que nous ayons un webservice qui réponde uniquement si le header ``X-Pizza``
est présent et défini avec un certain mot de passe. Peu de chance que cela arrive,
mais voyons voir ce que cela pourrait donner.

::

    from requests.auth import AuthBase

    class PizzaAuth(AuthBase):
        """Attache l'authentification HTTP Pizza à un object Request."""
        def __init__(self, username):
            # setup any auth-related data here
            self.username = username

        def __call__(self, r):
            # modify and return the request
            r.headers['X-Pizza'] = self.username
            return r

On peut alors faire une requête qui utilise notre authentification Pizza::

    >>> requests.get('http://pizzabin.org/admin', auth=PizzaAuth('kenneth'))
    <Response [200]>

.. _streaming-requests

Requête en streaming
--------------------

Avec la méthode ``requests.Response.iter_lines()`` vous pouvez facilement itérer sur des
API en streaming comme par exemple la `Twitter Streaming API <https://dev.twitter.com/docs/streaming-api>`_.

Pour utiliser la Twitter Streaming API et pister le mot-clé "requests"::

    import json
    import requests

    r = requests.get('http://httpbin.org/stream/20', stream=True)

    for line in r.iter_lines():

        # filtre toutes les lignes vides (keep-alive)
        if line: 
            print json.loads(line)


Proxys
-------

Si vous avez besoin d'utiliser un proxy, vous pouvez configurer individuellement
les requêtes avec l'argument ``proxies`` dans toutes les méthodes::

    import requests

    proxies = {
      "http": "http://10.10.1.10:3128",
      "https": "http://10.10.1.10:1080",
    }

    requests.get("http://example.org", proxies=proxies)

Vous pouvez aussi définir des proxys avec les variables d'environnement
``HTTP_PROXY`` et ``HTTPS_PROXY``.

::

    $ export HTTP_PROXY="http://10.10.1.10:3128"
    $ export HTTPS_PROXY="http://10.10.1.10:1080"
    $ python
    >>> import requests
    >>> requests.get("http://example.org")

To use HTTP Basic Auth with your proxy, use the `http://user:password@host/` syntax::

    proxies = {
        "http": "http://user:pass@10.10.1.10:3128/",
    }

Compliance
----------

Requests is intended to be compliant with all relevant specifications and
RFCs where that compliance will not cause difficulties for users. This
attention to the specification can lead to some behaviour that may seem
unusual to those not familiar with the relevant specification.

Encodings
^^^^^^^^^

When you receive a response, Requests makes a guess at the encoding to use for
decoding the response when you call the ``Response.text`` method. Requests
will first check for an encoding in the HTTP header, and if none is present,
will use `charade <http://pypi.python.org/pypi/charade>`_ to attempt to guess
the encoding.

The only time Requests will not do this is if no explicit charset is present
in the HTTP headers **and** the ``Content-Type`` header contains ``text``. In
this situation,
`RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.7.1>`_
specifies that the default charset must be ``ISO-8859-1``. Requests follows
the specification in this case. If you require a different encoding, you can
manually set the ``Response.encoding`` property, or use the raw
``Response.content``.

HTTP Verbs
----------

Requests provides access to almost the full range of HTTP verbs: GET, OPTIONS,
HEAD, POST, PUT, PATCH and DELETE. The following provides detailed examples of
using these various verbs in Requests, using the GitHub API.

We will begin with the verb most commonly used: GET. HTTP GET is an idempotent
method that returns a resource from a given URL. As a result, it is the verb
you ought to use when attempting to retrieve data from a web location. An
example usage would be attempting to get information about a specific commit
from GitHub. Suppose we wanted commit ``a050faf`` on Requests. We would get it
like so::

    >>> import requests
    >>> r = requests.get('https://api.github.com/repos/kennethreitz/requests/git/commits/a050faf084662f3a352dd1a941f2c7c9f886d4ad')

We should confirm that GitHub responded correctly. If it has, we want to work
out what type of content it is. Do this like so::

    >>> if (r.status_code == requests.codes.ok):
    ...     print r.headers['content-type']
    ...
    application/json; charset=utf-8

So, GitHub returns JSON. That's great, we can use the ``r.json`` method to
parse it into Python objects.

::

    >>> commit_data = r.json()
    >>> print commit_data.keys()
    [u'committer', u'author', u'url', u'tree', u'sha', u'parents', u'message']
    >>> print commit_data[u'committer']
    {u'date': u'2012-05-10T11:10:50-07:00', u'email': u'me@kennethreitz.com', u'name': u'Kenneth Reitz'}
    >>> print commit_data[u'message']
    makin' history

So far, so simple. Well, let's investigate the GitHub API a little bit. Now,
we could look at the documentation, but we might have a little more fun if we
use Requests instead. We can take advantage of the Requests OPTIONS verb to
see what kinds of HTTP methods are supported on the url we just used.

::

    >>> verbs = requests.options(r.url)
    >>> verbs.status_code
    500

Uh, what? That's unhelpful! Turns out GitHub, like many API providers, don't
actually implement the OPTIONS method. This is an annoying oversight, but it's
OK, we can just use the boring documentation. If GitHub had correctly
implemented OPTIONS, however, they should return the allowed methods in the
headers, e.g.

::

    >>> verbs = requests.options('http://a-good-website.com/api/cats')
    >>> print verbs.headers['allow']
    GET,HEAD,POST,OPTIONS

Turning to the documentation, we see that the only other method allowed for
commits is POST, which creates a new commit. As we're using the Requests repo,
we should probably avoid making ham-handed POSTS to it. Instead, let's play
with the Issues feature of GitHub.

This documentation was added in response to Issue #482. Given that this issue
already exists, we will use it as an example. Let's start by getting it.

::

    >>> r = requests.get('https://api.github.com/repos/kennethreitz/requests/issues/482')
    >>> r.status_code
    200
    >>> issue = json.loads(r.text)
    >>> print issue[u'title']
    Feature any http verb in docs
    >>> print issue[u'comments']
    3

Cool, we have three comments. Let's take a look at the last of them.

::

    >>> r = requests.get(r.url + u'/comments')
    >>> r.status_code
    200
    >>> comments = r.json()
    >>> print comments[0].keys()
    [u'body', u'url', u'created_at', u'updated_at', u'user', u'id']
    >>> print comments[2][u'body']
    Probably in the "advanced" section

Well, that seems like a silly place. Let's post a comment telling the poster
that he's silly. Who is the poster, anyway?

::

    >>> print comments[2][u'user'][u'login']
    kennethreitz

OK, so let's tell this Kenneth guy that we think this example should go in the
quickstart guide instead. According to the GitHub API doc, the way to do this
is to POST to the thread. Let's do it.

::

    >>> body = json.dumps({u"body": u"Sounds great! I'll get right on it!"})
    >>> url = u"https://api.github.com/repos/kennethreitz/requests/issues/482/comments"
    >>> r = requests.post(url=url, data=body)
    >>> r.status_code
    404

Huh, that's weird. We probably need to authenticate. That'll be a pain, right?
Wrong. Requests makes it easy to use many forms of authentication, including
the very common Basic Auth.

::

    >>> from requests.auth import HTTPBasicAuth
    >>> auth = HTTPBasicAuth('fake@example.com', 'not_a_real_password')
    >>> r = requests.post(url=url, data=body, auth=auth)
    >>> r.status_code
    201
    >>> content = r.json()
    >>> print content[u'body']
    Sounds great! I'll get right on it.

Brilliant. Oh, wait, no! I meant to add that it would take me a while, because
I had to go feed my cat. If only I could edit this comment! Happily, GitHub
allows us to use another HTTP verb, PATCH, to edit this comment. Let's do
that.

::

    >>> print content[u"id"]
    5804413
    >>> body = json.dumps({u"body": u"Sounds great! I'll get right on it once I feed my cat."})
    >>> url = u"https://api.github.com/repos/kennethreitz/requests/issues/comments/5804413"
    >>> r = requests.patch(url=url, data=body, auth=auth)
    >>> r.status_code
    200

Excellent. Now, just to torture this Kenneth guy, I've decided to let him
sweat and not tell him that I'm working on this. That means I want to delete
this comment. GitHub lets us delete comments using the incredibly aptly named
DELETE method. Let's get rid of it.

::

    >>> r = requests.delete(url=url, auth=auth)
    >>> r.status_code
    204
    >>> r.headers['status']
    '204 No Content'

Excellent. All gone. The last thing I want to know is how much of my ratelimit
I've used. Let's find out. GitHub sends that information in the headers, so
rather than download the whole page I'll send a HEAD request to get the
headers.

::

    >>> r = requests.head(url=url, auth=auth)
    >>> print r.headers
    ...
    'x-ratelimit-remaining': '4995'
    'x-ratelimit-limit': '5000'
    ...

Excellent. Time to write a Python program that abuses the GitHub API in all
kinds of exciting ways, 4995 more times.

Link Headers
------------

Many HTTP APIs feature Link headers. They make APIs more self describing and discoverable.

GitHub uses these for `pagination <http://developer.github.com/v3/#pagination>`_ in their API, for example::

    >>> url = 'https://api.github.com/users/kennethreitz/repos?page=1&per_page=10'
    >>> r = requests.head(url=url)
    >>> r.headers['link']
    '<https://api.github.com/users/kennethreitz/repos?page=2&per_page=10>; rel="next", <https://api.github.com/users/kennethreitz/repos?page=6&per_page=10>; rel="last"'

Requests will automatically parse these link headers and make them easily consumable::

    >>> r.links["next"]
    {'url': 'https://api.github.com/users/kennethreitz/repos?page=2&per_page=10', 'rel': 'next'}

    >>> r.links["last"]
    {'url': 'https://api.github.com/users/kennethreitz/repos?page=7&per_page=10', 'rel': 'last'}

Transport Adapters
------------------

As of v1.0.0, Requests has moved to a modular internal design. Part of the
reason this was done was to implement Transport Adapters, originally
`described here`_. Transport Adapters provide a mechanism to define interaction
methods for an HTTP service. In particular, they allow you to apply per-service
configuration.

Requests ships with a single Transport Adapter, the
:class:`HTTPAdapter <requests.adapters.HTTPAdapter>`. This adapter provides the
default Requests interaction with HTTP and HTTPS using the powerful `urllib3`_
library. Whenever a Requests :class:`Session <Session>` is initialized, one of
these is attached to the :class:`Session <Session>` object for HTTP, and one
for HTTPS.

Requests enables users to create and use their own Transport Adapters that
provide specific functionality. Once created, a Transport Adapter can be
mounted to a Session object, along with an indication of which web services
it should apply to.

::

    >>> s = requests.Session()
    >>> s.mount('http://www.github.com', MyAdapter())

The mount call registers a specific instance of a Transport Adapter to a
prefix. Once mounted, any HTTP request made using that session whose URL starts
with the given prefix will use the given Transport Adapter.

Implementing a Transport Adapter is beyond the scope of this documentation, but
a good start would be to subclass the ``requests.adapters.BaseAdapter`` class.

.. _`described here`: http://kennethreitz.org/exposures/the-future-of-python-http
.. _`urllib3`: https://github.com/shazow/urllib3

Blocking Or Non-Blocking?
-------------------------

With the default Transport Adapter in place, Requests does not provide any kind
of non-blocking IO. The ``Response.content`` property will block until the
entire response has been downloaded. If you require more granularity, the
streaming features of the library (see :ref:`streaming-requests`) allow you to
retrieve smaller quantities of the response at a time. However, these calls
will still block.

If you are concerned about the use of blocking IO, there are lots of projects
out there that combine Requests with one of Python's asynchronicity frameworks.
Two excellent examples are `grequests`_ and `requests-futures`_.

.. _`grequests`: https://github.com/kennethreitz/grequests
.. _`requests-futures`: https://github.com/ross/requests-futures
