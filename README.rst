onyx
====

Documentation de conception du langage Onyx.

Le contenu est organisé par domaine afin de faciliter les décisions
indépendantes et les révisions ciblées.

Documentation
-------------

* `Vision <docs/vision.md>`_
* `Core syntax <docs/syntax.md>`_
* `Valeurs et types <docs/values-and-types.md>`_
* `Tableaux et tenseurs <docs/tensors.md>`_
* `Constructions du langage <docs/language-constructs.md>`_
* `Modèle objet <docs/object-model.md>`_
* `Mémoire et propriété <docs/memory-and-ownership.md>`_
* `Questions ouvertes <docs/open-questions.md>`_
* `Ordre de décision <docs/roadmap.md>`_
* `Exemples de conformité <docs/conformance-examples.md>`_

Les règles de `docs/open-questions.md` ne font pas encore partie de la
spécification stable.

Snippets
--------

Des programmes de test exercent la structure de la référence :

* `Algorithmes de tri <snippets/01_sorting.onx>`_
* `Scaffold micrograd <snippets/02_micrograd_scaffold.onx>`_
* `Coroutines (esquisse) <snippets/03_coroutines.onx>`_

Ces snippets illustrent les règles documentées et signalent, en
commentaires, les points qui dépendent de questions encore ouvertes.