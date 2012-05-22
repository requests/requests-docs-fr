.. Requests documentation master file, created by
   sphinx-quickstart on Sun Feb 13 23:54:25 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Requests: HTTP pour les humains
===============================

Release v\ |version|. (:ref:`Installation <install>`)

Requests est une librairie HTTP :ref:`sous licence ISC<isc>`, écrite en Python,
pour les êtres humains.

Le module **urllib2** de la librairie standard fournit toutes les
fonctionnalités dont vous avez besoin, mais son API est complètement **moisie**.
Il a été crée dans un autre époque - lorsque le web était autre chose,
et demande une *énorme* quantité de de travail (voir des remplacements de
méthodes) pour achever les plus simples tâches.

Ca ne doit pas se passer comme ca. Pas en Python.

::

    >>> r = requests.get('https://api.github.com', auth=('user', 'pass'))
    >>> r.status_code
    204
    >>> r.headers['content-type']
    'application/json'
    >>> r.text
    ...

Voir `le même code, sans Requests <https://gist.github.com/973705>`_.

Requests reprend tout les travaux autour de Python HTTP/1.1 - et rend
l'intégration avec des webservices très facile. Pas besoin d'ajouter des
querystrings à vos URLs manuellement, ou d'encoder vous-même vos datas pour
les POST. Les Keep-alive et le groupement des connexions HTTP sont 100%
automatisés, grâce à `urllib3 <https://github.com/shazow/urllib3>`_,
qui est directement intégré à Requests.

Témoignages
------------

`Heroku <http://heroku.com>`_, `PayPal <https://www.paypal.com/>`_,
`Transifex <https://www.transifex.net/>`_,
`Native Instruments <http://www.native-instruments.com/>`_, `The Washington Post <http://www.washingtonpost.com/>`_,
`Twitter, Inc <http://twitter.com>`_,
`Readability <http://readability.com>`_, and
Federal US Institutions
utilisent Requests en interne. Le module a été installé plus de 100,000 fois via PyPI.

**Armin Ronacher**
    Requests est l'exemple parfait de ce que peut être une belle API grâce
    à la bonne dose d'abstraction.

**Matt DeBoard**
    Je vais me tatouer le module Python requests de @kennethreitz
    sur le corps, d'une façon ou d'une autre. partout.

**Daniel Greenfeld**
    Je viens de passer de 1200 lignes de code spaghetti à 10 lignes de code
    grâce au module requests de @kennethreitz. Aujourd'hui est un grand jour.

**Kenny Meyers**
    HTTP avec Python: si vous avez un doute, ou pas, utilisez Requests.
    Beau, simple, pythonic.


Fonctionnalités
---------------

Requests est prêt pour le web d'aujourd'hui

- Gestion domaines et URLS internationales
- Keep-Alive & Groupement de connections (Pooling)
- Sessions et Cookies persistants
- Verification SSL
- Authentifications Basic/Digest ou personnalisées
- Gestion élégante des Cookies clé/valeur
- Décompression automatique
- Corps des réponses en unicode
- Upload de fichiers multiparts
- Timeouts de connexion
- supprt de ``.netrc``
- Thread-safe.


Guide utilisateur
-----------------

Cette partie de la documentation commence avec des informations de fond sur Requests, puis nous présentons étape par étape les instructions pour tirer le meilleur parti de Requests.

.. toctree::
   :maxdepth: 2

   user/intro
   user/install
   user/quickstart
   user/advanced


Guide de la communauté
-----------------------

Cette partie de la documentation détaille l'ecosystème de Requests et de sa communauté

.. toctree::
   :maxdepth: 1

   community/faq
   community/out-there.rst
   community/support
   community/updates

Documentation de l'API
----------------------

Si vous cherchez des informations sur une fonction, une classe ou une méthode spécifique, c'est dans cette partie de la documentation que vous devez regarder.

.. toctree::
   :maxdepth: 2

   api


Guide du développeur
--------------------

Si vous voulez contribuer au projet, cette partie de la documentation est pour vous

.. toctree::
   :maxdepth: 1

   dev/internals
   dev/todo
   dev/authors
