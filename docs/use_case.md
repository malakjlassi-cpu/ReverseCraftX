UC-01 — Rechercher des designs
UC-02 — Consulter un article
UC-03 — Créer un compte / s'authentifier
UC-04 — Publier un design
UC-05 — Analyser une image
UC-06 — Enregistrer un article
UC-07 — Consulter ses articles enregistrés
UC-08 — Ajouter un commentaire
UC-09 — Modifier son profil


UC-01 — Rechercher des designs

Acteur :
Visiteur / utilisateur authentifié

But :
Trouver rapidement des designs publiés dans ReverseCraft.

Précondition :
Aucune authentification nécessaire.

Input :
Mot-clé.

Exemple :
"robe longue"
"robe satin"
"robe mariage"

Scénario principal :
1. L'utilisateur ouvre la recherche.
2. Il saisit un mot-clé.
3. Le système recherche les articles publics correspondants.
4. Le système affiche les résultats.
5. L'utilisateur sélectionne un article.

Output :
Liste d'articles correspondant à la recherche.

Cas particulier :
Aucun résultat → le système informe l'utilisateur qu'aucun article correspondant n'a été trouvé.


UC-02 — Consulter un article

Acteur :
Visiteur / utilisateur authentifié

But :
Consulter les détails d'un article trouvé dans les résultats de recherche.

Préconditions :

L'article existe.
L'article est public.
L'utilisateur a accès aux résultats de recherche.
Aucune authentification n'est nécessaire pour consulter l'article.

Input :

Article sélectionné depuis les résultats de recherche.

Scénario principal :

L'utilisateur effectue une recherche.
Le système affiche les articles correspondants.
L'utilisateur sélectionne un article.
Le système ouvre la page de l'article.
Le système affiche les détails disponibles :
image ;
titre ;
description ;
analyse ReverseCraft ;
auteur ;
commentaires éventuels.
L'utilisateur peut consulter le contenu de l'article.

Actions possibles après consultation :

Visiteur : consulter uniquement.
Utilisateur authentifié : ajouter un commentaire, enregistrer l'article, éventuellement signaler l'article.

Cas particuliers :

Article introuvable → le système affiche un message indiquant que l'article n'existe plus.
Article supprimé → le système informe l'utilisateur que l'article n'est plus disponible.
Image indisponible → les autres informations de l'article restent accessibles si possible.
Erreur de chargement → le système affiche un message et permet de réessayer.
