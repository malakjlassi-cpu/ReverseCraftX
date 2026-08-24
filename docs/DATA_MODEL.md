Data Model — ReverseCraft
1. User

L'utilisateur représente toute personne utilisant ReverseCraft.

Attributes:
id
nom
prenom
Relations

Un User peut :

effectuer plusieurs recherches ;
consulter plusieurs articles ;
publier plusieurs articles s'il est authentifié ;
ajouter plusieurs commentaires ;
signaler plusieurs articles ;
enregistrer plusieurs articles ;
modifier son profil.
Authentification

On distingue deux états :

Visiteur / utilisateur non authentifié
Utilisateur authentifié

Un utilisateur doit être authentifié pour effectuer certaines actions nécessitant une identité, notamment :

publier un design ;
commenter un article ;
signaler un article ;
enregistrer un article ;
modifier son profil.
2. Search

Une Search représente une recherche effectuée par un utilisateur.

Attributes
id
mot_cle
date
Relations
Une recherche peut être effectuée par un utilisateur.
Un utilisateur peut effectuer plusieurs recherches.
Une recherche peut retourner plusieurs articles.
Un même article peut apparaître dans plusieurs recherches.
User 1 ─────── N Search
Search N ───── N Article

La relation entre Search et Article est many-to-many :

Une recherche peut retourner plusieurs articles, et un même article peut apparaître dans plusieurs recherches différentes.

3. Design

Un Design représente le design analysé ou décrit par ReverseCraft.

Attributes
id
nom
Relations
Un design peut être associé à plusieurs articles.
Un article correspond à un seul design.
Design 1 ─────── N Article
4. Article

Un Article représente une publication visible dans la bibliothèque de ReverseCraft.

Attributes
id
date_ajout
Relations
Auteur

Un article est publié par un utilisateur authentifié.

User 1 ─────── N Article

Un utilisateur peut donc publier plusieurs articles, tandis qu'un article possède un seul auteur.

Design
Design 1 ─────── N Article

Un design peut être présenté dans plusieurs articles.

Images

Un article peut contenir plusieurs images.

Cela permet notamment à un article de contenir l'image originale ainsi que plusieurs versions d'un même design.

Article 1 ─────── N Image
Commentaires

Un article peut recevoir plusieurs commentaires.

Article 1 ─────── N Comment
Articles enregistrés

Un article peut être enregistré par plusieurs utilisateurs.

User N ─────── N Article
       saves

La relation sera représentée par une entité associative SavedArticle.

5. Image

Une Image représente une image associée à un article.

Attributes
id
url / chemin
date_ajout
Relations
Une image appartient à un seul article.
Un article peut avoir plusieurs images.
Une image peut être analysée par ReverseCraft.
Article 1 ─────── N Image

6. Analysis

Une Analysis représente l'analyse produite par ReverseCraft à partir d'une image.

Attributes
id
composants
materiaux_probables
structure
hypotheses
etapes_fabrication
date_creation
Relations

Une analyse est associée à une image.

Image 1 ─────── N Analysis

La cardinalité exacte entre Image et Analysis devra être confirmée lorsque les Use Cases liés à l'analyse seront détaillés.

Pour le moment, on considère qu'une image peut avoir une ou plusieurs analyses.

7. Comment

Un Comment représente un commentaire publié sur un article.

Attributes
id
contenu
date_saisie
date_modification
Relations
Un commentaire appartient à un seul article.
Un article peut avoir plusieurs commentaires.
Un commentaire est écrit par un utilisateur authentifié.
Un utilisateur peut écrire plusieurs commentaires.
User 1 ─────── N Comment
Article 1 ──── N Comment
8. SavedArticle

SavedArticle représente l'enregistrement d'un article par un utilisateur.

Un article enregistré n'est pas un type particulier d'article et n'est donc pas une relation d'héritage.

Il s'agit d'une relation entre User et Article.

Attributes
user_id
article_id
date_enregistrement
Relations
Un utilisateur peut enregistrer plusieurs articles.
Un article peut être enregistré par plusieurs utilisateurs.
User 1 ─────── N SavedArticle N ─────── 1 Article
9. Relations principales

Le modèle actuel peut être résumé ainsi :

                         ┌──────────────┐
                         │     User     │
                         └──────┬───────┘
                                │
              ┌─────────────────┼──────────────────┐
              │                 │                  │
             1:N               1:N                1:N
              │                 │                  │
              ▼                 ▼                  ▼
           Search            Article            Comment
              │                 │
              │                 │
              │                 ├──── 1:N ────> Image
              │                 │                  │
              │                 │                  │
              │                 │                  ▼
              │                 │              Analysis
              │                 │
              │                 N:1
              │                 │
              │                 ▼
              │              Design
              │
              └──────── N:N ──────────────> Article


              User N ─────── N Article
                     saves


10. Relations importantes
User → Search
User 1 ─────── N Search

Un utilisateur peut effectuer plusieurs recherches.

Search → Article
Search N ─────── N Article

Une recherche peut retourner plusieurs articles et un article peut apparaître dans plusieurs recherches.

User → Article
User 1 ─────── N Article

Un utilisateur peut publier plusieurs articles.

Design → Article
Design 1 ─────── N Article

Un design peut être présenté dans plusieurs articles.

Article → Image
Article 1 ─────── N Image

Un article peut contenir plusieurs images, notamment pour représenter différentes versions du design.

Image → Analysis
Image 1 ─────── N Analysis

Une image peut être analysée par ReverseCraft.

User → Comment
User 1 ─────── N Comment

Un utilisateur peut écrire plusieurs commentaires.

Article → Comment
Article 1 ─────── N Comment

Un article peut recevoir plusieurs commentaires.

User → Article — SavedArticle
User N ─────── N Article
       saves

Un utilisateur peut enregistrer plusieurs articles et un article peut être enregistré par plusieurs utilisateurs.
