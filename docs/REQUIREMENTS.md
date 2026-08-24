FR-UC01-01 
Le système doit permettre à tout utilisateur, authentifié ou non, d'effectuer une recherche de designs.

FR-UC01-02 — Mot-clé
Le système doit permettre à l'utilisateur de saisir un ou plusieurs mots-clés pour effectuer une recherche.

FR-UC01-03 — Résultats
Le système doit afficher les designs correspondant aux mots-clés saisis par l'utilisateur.

FR-UC01-04 — Présentation des résultats
Chaque résultat doit être présenté sous forme d'un élément visuel contenant au minimum une image permettant d'identifier le design.

FR-UC01-05 — Résultats cliquables
Chaque résultat de recherche doit être cliquable.

FR-UC01-06 — Accès à l'article
Lorsque l'utilisateur sélectionne un résultat, le système doit ouvrir la page correspondant à l'article sélectionné.

FR-UC01-07 — Aucun résultat
Si aucun design ne correspond aux mots-clés, le système doit informer l'utilisateur qu'aucun résultat n'a été trouvé.


FR-UC02-01
Le système doit permettre à un visiteur de consulter un article public.

FR-UC02-02
Le système doit afficher l'image de l'article.

FR-UC02-03
Le système doit afficher sa description.

FR-UC02-04
Le système doit afficher les informations d'analyse disponibles.

FR-UC02-05
Le système doit afficher l'auteur de l'article.

FR-UC05-01 Le système doit permettre à l'utilisateur de soumettre une image (formats acceptés : JPEG, PNG, WEBP ; taille maximale : ex. 10 Mo).

FR-UC05-02 Le système doit déclencher l'analyse de manière asynchrone et afficher un indicateur de chargement (« Analyse en cours... ») à l'utilisateur.

FR-UC05-03 Si l'analyse réussit, le système doit stocker et afficher les résultats en distinguant clairement les observations factuelles des hypothèses.

FR-UC05-04 Si l'analyse échoue (timeout de l'API, image illisible ou non reconnue comme vêtement), le système doit capturer l'erreur, mettre à jour le statut de l'analyse à FAILED, et afficher un message clair à l'utilisateur lui permettant de réessayer avec une autre image.

FR-UC02-06
Un utilisateur authentifié doit pouvoir commenter l'article.

FR-UC02-07
Un utilisateur authentifié doit pouvoir signaler l'article.
