.. Requests documentation master file, created by
   sphinx-quickstart on Sun Feb 13 23:54:25 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Requests: HTTP pour les Humains™
================================

Release v\ |version|. (:ref:`Installation <install>`)


.. image:: https://pepy.tech/badge/requests
    :target: https://pepy.tech/project/requests
    
.. image:: https://img.shields.io/pypi/l/requests.svg
    :target: https://pypi.org/project/requests/

.. image:: https://img.shields.io/pypi/wheel/requests.svg
    :target: https://pypi.org/project/requests/

.. image:: https://img.shields.io/pypi/pyversions/requests.svg
    :target: https://pypi.org/project/requests/

**Requests** est une librairie HTTP élégante et simple pour Python, créée pour les êtres humains.

-------------------

**Voici le pouvoir de Requests**::

    >>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
    >>> r.status_code
    200
    >>> r.headers['content-type']
    'application/json; charset=utf8'
    >>> r.encoding
    'utf-8'
    >>> r.text
    u'{"type":"User"...'
    >>> r.json()
    {u'private_gists': 419, u'total_private_repos': 77, ...}

Voir `le même code, sans Requests <https://gist.github.com/973705>`_.

**Requests** vous permet d'envoyer des requêtes HTTP/1.1 extrêmement facilement.
Il n'y a pas besoin d'ajouter manuellement des chaînes de requête
à vos URLs, ou d'encoder par formulaire vos données POST. Le maitien en version
et la mise en commun des connexions sont à 100% automatiques, grâce à
`urllib3 <https://github.com/shazow/urllib3>`_. 

Fonctionnalités
---------------

Requests est prêt pour le web d'aujourd'hui.

- Maintien en vie & mise en commun des connections
- Gestion de domaines internationaux et URLs
- Sessions avec persistance des cookies
- Vérification SSL de type navigateur
- Décodage automatique de contenu
- Authentication Basic/Digest
- Gestion élégante des cookies clé/valeur
- Décompression automatique
- Unicode Response Bodies
- Support des proxy HTTP(S)
- Téléversement de fichiers en plusieurs parties
- Téléchargements en streaming
- Délais de connexion
- Requêtes fractionnées
- Support  ``.netrc`` 

Requests supporte officiellement Python 2.7 & 3.4-3.7, and s'exécute bien sur Pypy.


Guide utilisateur
-----------------

Cette partie de la documentation commence avec des informations de fond sur Requests, puis nous présentons étape par étape les instructions pour tirer le meilleur parti de Requests.

.. toctree::
   :maxdepth: 2

   user/intro
   user/install
   user/quickstart
   user/advanced
   user/authentication


Guide de la communauté
-----------------------

Cette partie de la documentation détaille l'ecosystème de Requests et de sa communauté

.. toctree::
   :maxdepth: 2

   community/sponsors
   community/recommended
   community/faq
   community/out-there
   community/support
   community/vulnerabilities
   community/updates
   community/release-process

Documentation de l'API
----------------------

Si vous cherchez des informations sur une fonction, une classe ou une méthode
spécifique, c'est dans cette partie de la documentation que vous devez regarder.

.. toctree::
   :maxdepth: 2

   api


Guide du contributeur
---------------------

Si vous voulez contribuer au projet, cette partie de la documentation est pour
vous.

.. toctree::
   :maxdepth: 3

   dev/philosophy
   dev/internals
   dev/todo
   dev/authors

Il n'y a plus de guides. Vous êtes maintenant sans aide.
Bonne chance.