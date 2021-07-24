# API Auth: Do you Need it? And its Parts

Même si on a une API avec endpoints, il reste probable qu'on n'ait pas besoin
d'authentification par API dans notre appli. En effet si les endpoints ne 
sont utiles que pour notre propre JavaScript ou notre propre site, alors 
l'authentification par API via token est inutile. Dans ce genre de 
situation, un formulaire de connexion par identifiant / mot de passe suffit
largement.  

Rappelons que les requêtes AJAX sont possibles une fois authentifié car 
les requêtes envoient le cookie de session (du coup le back-end reconnaît
qui est connecté). D'ailleurs, si on veut que la connexion soit faite en 
AJAX, il suffit de modifier les retours dans `LoginFormAuthenticator` pour
qu'ils soient sous forme de JSON au lieu de redirections Symfony.  

On a besoin d'un système d'authentification par token quand on souhaite que
des trucs autres que notre propre JavaScript / site utilise l'API.  
Les deux principales parties du système d'authentification par token API
sont les suivantes :
1. Comment l'appli prend un token existant et connecte l'utilisateur.
2. Comment les token sont créés et distribués.  

On rappelle qu'un **token API** est une chaîne de caractères reliée d'une
manière ou d'une autre à un utilisateur. Le client place le token dans 
le header Authorization pour être connecté à l'API en tant que cet 
utilisateur.  

Pour associer un token à un utilisateur (1ère partie du système 
d'authentification par token API), on peut :
- utiliser la base de données pour associer directement un token à un
utilisateur (indirectement via jointure ou directement dans la même table);
- chiffrer un json d'informations utilisateurs pour que ce soit le token
associé (on parle alors de **JSON Web Token**).
  
Pou créer et distribuer les token (2ème partie du système d'authentification
par token API), on peut :
- Permettre à l'utilisateur de gérer ses propres token par une page Web
  (comme la page de clés SSH sur GitHub / GitLab): c'est super simple 
  techniquement, mais sans génération automatique c'est lourd à comprendre
  pour des utilisateurs non techos;
- Créer un endpoint API qui va générer un token à partir de l'identifiant
et du mot de passe : on a un script qui automatise la création de token,
mais si quelqu'un veut créer une appli mobile utilisant l'API, alors l'app
va demander les identifiants et mots de passe de l'utilisateur, ce qui 
n'est pas forcément tip top niveau sécurité;
- Faire de l'**OAuth2** : c'est une manière sécurisée de créer et récupérer
des token API pour nos utilisateurs, et permettant de créer des token même
depuis une appli tierce de manière plus sécurisée que la précédente, mais
elle est la manière la plus complexe des trois à mettre en place. 
    

Dans le cadre de ce cours, nous ne verrons que la 1ère partie à savoir 
comment l'appli prend un token existant et connecte l'utilisateur.