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

    s = requests.session()

    s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
    r = s.get("http://httpbin.org/cookies")

    print r.text
    # '{"cookies": {"sessioncookie": "123456789"}}'


Les Sessions peuvent aussi être utilisées pour fournir des valeurs par défaut
aux requêtes::

    headers = {'x-test': 'true'}
    auth = ('user', 'pass')

    with requests.session(auth=auth, headers=headers) as c:

        # 'x-test' et 'x-test2' sont envoyés
        c.get('http://httpbin.org/headers', headers={'x-test2': 'true'})


Tous les dictionnaires que vous passez aux methodes de requête sont fusionnés avec les valeur de session déja définies. Les paramètres de la méthode surchargent les paramètres de session.

.. admonition:: Supprimer une valeur d'un paramètre

    Parfois vous voudrez supprimer des paramètres de session lors de vos requêtes. Pour cela, il suffit d'envoyer lors de l'appel de la méthodeun dictionnaire dont les clés seraient les paramètres a supprimer et les valeurs seraient ``None``. Ces paramètres seront alors automatiquement omis.

Toutes les valeurs contenues dans la session sont directement accessibles. Pour en savoir plus, cf :ref:`Session API Docs <sessionapi>`.

Verifications certificats SSL
-----------------------------

Requests peut vérifier les certificats SSL sur les requêtes HTTPS, comme n'importe quel navigateur web. Pour vérifier le certificat d'un serveur, vous pouvez utiliser l'argument ``verify``::

    >>> requests.get('https://kennethreitz.com', verify=True)
    requests.exceptions.SSLError: hostname 'kennethreitz.com' doesn't match either of '*.herokuapp.com', 'herokuapp.com'

SSL n'est pas configuré sur ce domaine, donc cela génère une erreur. Parfait. Par contre, GitHub en a un::

    >>> requests.get('https://github.com', verify=True)
    <Response [200]>

Vous pouvez aussi passer au paramètre ``verify`` le chemin vers un fichier ``CA_BUNDLE`` pour les certificats privés. Vous pouvez également définir la variable d'environnement ``REQUESTS_CA_BUNDLE``.


Workflow du contenu des réponses
--------------------------------

Par défaut, lorsque vous effectuez une requête, le corps de la réponse n'est pas téléchargé automatiquement. Les en-têtes sont téléchargés, mais le contenu lui-même n'est téléchargé que lorsque vous accédez à l'attribut  :class:`Response.content`.

Exemple::

    tarball_url = 'https://github.com/kennethreitz/requests/tarball/master'
    r = requests.get(tarball_url)

La requête a été effectuée, et la connexion est toujours ouverte. Le corps de la réponse n' a pas encore  été téléchargé.::

    r.content

Le contenu est téléchargé et mis en cache à ce moment-là.

Vous pouvez modifier ce comportement par défaut avec le paramètre ``prefetch``::

    r = requests.get(tarball_url, prefetch=True)
    # Appel bloquant jusqu'à reception du corps de la réponse


Configurer Requests
--------------------

Vous pouvez avoir envie de configurer une requête pour personnaliser son comportement.
Pour faire cela vous pouvez passer un dictionnaire ``config`` à une requête ou une session.
Pour en savoir plus, cf :ref:`Configuration API Docs <configurations>` to learn more.


Keep-Alive
----------

Bonne nouvelle - grâce à urllib3, le keep-alive est 100% automatique pendant une session! Toutes les requêtes que vous ferez à travers une session réutiliseront automatiquement la connexion appropriée!

A noter que les connexions ne sont libérées pour réutilisation seulement lorsque les données ont été lues. Faites attention à bien mettre ``prefetch`` à ``True`` ou toujours accéder à la propriété ``content`` de l'object ``Response``.

Si vous souhaitez désactiver le keep-alive, vous pouvez définir l'attribut de configuration ``keep_alive`` à ``False``::

    s = requests.session()
    s.config['keep_alive'] = False


Requêtes asynchrones
--------------------

Requests dispose d'un support de première classe pour les requêtes concurrentes, 
grâce à gevent. ceci vous permet d'envoyer des paquets de requêtes HTTP en même temps.

Premièrement, importez le module async. Bien sûr, il vous faut le module `gevent <http://pypi.python.org/pypi/gevent>`_::


    from requests import async

Le module ``async`` a exactement la même API que ``requests``, sauf qu'il 
n'envoie pas les requêtes immédiatement. A la place, il renvoie un objet 
``Request``.

On peut facilement créer une liste d'objets ``Request``::

    urls = [
        'http://python-requests.org',
        'http://httpbin.org',
        'http://python-guide.org',
        'http://kennethreitz.com'
    ]

    rs = [async.get(u) for u in urls]

Maintenant nous avons une liste d'objets ``Request``, prêts à être envoyés. Nous 
pourrions envoyer les requêtes une à une, avec ``Request.send()``, mais 
cela prendrait un peu de temps. Au lieu de ca, nous allons les envoyer toutes
d'un coup avec ``async.map()``. De cette façon, cela garantira également 
l'execution du hook ``response``, comme suit. ::

    >>> responses = async.map(rs)
    >>> responses
    [<Response [200]>, <Response [200]>, <Response [200]>, <Response [200]>]

.. admonition:: Throttling

    La fonction ``map`` prend également un paramètre ``size``, qui permet de spécifier le nombre de connexions simultannées.

        async.map(rs, size=5)


Hooks d'évenements
------------------

Requests dispose d'un système de 'hooks' que vous pouvez utiliser pour
manipuler des portions du processus de requêtage ou signaler des évènements.

Hooks disponibles:

``args``:
    Un dictionnaire d'arguments prêts à être envoyés à Request().

``pre_request``:
    L'objet Request, juste avant d'être envoyé.

``post_request``:
    L'objet Request, juste après avoir été envoyé.

``response``:
    La réponse générée après une requête.


Vous pouvez assigner une fonction de hook par requête, en passant au 
paramètre ``hooks`` de la Request un dictionnaire de hooks 
``{hook_name: callback_function}``::

    hooks=dict(args=print_url)

La fonction ``callback_function`` recevra un bloc de données en premier 
argument.

::

    def print_url(args):
        print args['url']

Si une exception apparait lors de l'éxecution du callback, un warning est
affiché.

Si le callback renvoie une valeur, on suppose que cela remplace les données
qui lui ont été passées. Si la fonction ne renvoie rien, alors rien n'est
affecté.

Affichons quelques arguments a la volée::

    >>> requests.get('http://httpbin.org', hooks=dict(args=print_url))
    http://httpbin.org
    <Response [200]>

Cette fois-ci, modifions les arguments avec un nouveau callback::

    def hack_headers(args):
        if args.get('headers') is None:
            args['headers'] = dict()

        args['headers'].update({'X-Testing': 'True'})

        return args

    hooks = dict(args=hack_headers)
    headers = dict(yo=dawg)

Et essayons::

    >>> requests.get('http://httpbin.org/headers', hooks=hooks, headers=headers)
    {
        "headers": {
            "Content-Length": "",
            "Accept-Encoding": "gzip",
            "Yo": "dawg",
            "X-Forwarded-For": "::ffff:24.127.96.129",
            "Connection": "close",
            "User-Agent": "python-requests.org",
            "Host": "httpbin.org",
            "X-Testing": "True",
            "X-Forwarded-Protocol": "",
            "Content-Type": ""
        }
    }


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


Requête en streaming
--------------------

Avec la méthode ``requests.Response.iter_lines()`` vous pouvez facilement itérer sur des
API en streaming comme par exemple la `Twitter Streaming API <https://dev.twitter.com/docs/streaming-api>`_.

Pour utiliser la Twitter Streaming API et pister le mot-clé "requests":

::

    import requests
    import json

    r = requests.post('https://stream.twitter.com/1/statuses/filter.json',
        data={'track': 'requests'}, auth=('username', 'password'))

    for line in r.iter_lines():
        if line: # filtre les lignes vides (keep-alive)
            print json.loads(line)


Logging verbeux
---------------

Si vous voulez avoir une bonne vision des requêtes HTTP qui sont envoyées
par votre application, vous pouvez activer le logging verbeux.

Pour cela, configurez Requests avec un stream où ecrire les logs::

    >>> my_config = {'verbose': sys.stderr}
    >>> requests.get('http://httpbin.org/headers', config=my_config)
    2011-08-17T03:04:23.380175   GET   http://httpbin.org/headers
    <Response [200]>

Proxys
-------

Si vous avez besoin d'utiliser un proxy, vous pouvez configurer individuellement
les requêtes avec l'argument ``proxies`` dans toutes les méthodes.

::

    import requests

    proxies = {
      "http": "10.10.1.10:3128"
      "https": "10.10.1.10:1080"
    }

    requests.get("http://example.org", proxies=proxies)

Vous pouvez aussi définir des proxys avec les variables d'environnement
``HTTP_PROXY`` et ``HTTPS_PROXY``.

::

    $ export HTTP_PROXY="10.10.1.10:3128"
    $ export HTTPS_PROXY="10.10.1.10:1080"
    $ python
    >>> import requests
    >>> requests.get("http://example.org")
