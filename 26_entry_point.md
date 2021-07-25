# Entry Point: Helping Users Authenticate

Maintenant que nous avons relié des token aux User en base, on va faire en sorte 
que quand une requête API envoie un token valide, on va lire ce token et 
authentifier la requête par l'utilisateur qui possède ce token.  

Du coup, on va avoir besoin d'un second Authenticator. Comme vu dans le chapitre
5, on va faire `./bin/console make:auth`. On va choisir de faire un Empty
Authenticator, qu'on va appeler `ApiTokenAuthenticator`. On doit ensuite choisir
l'entry point (la page de redirection si l'utilisateur n'a pas accès à la page
qu'il souhaite), ce sera ici le `LoginFormAuthenticator`.  

Une fois la commande exécutée, on peut voir que notre nouvelle classe
`ApiTokenAuthenticator` a été ajoutée dans le dossier `Security`, et que le 
fichier `security.yaml` a été mis à jour:  le nouvel authenticator a été ajouté
dans la partie `firewalls > main > guard > authenticators`, et l'entry point a 
été précisé dans l'`entry_point`.  
L'entry point est codé dans la classe parente du `LoginFormAuthenticator` ici,
dans la méthode `start()` de `AbstractFormLoginAuthenticator`.  

L'entry point doit être unique : si un utilisateur souhaite accéder à une 
page restreinte, il n'a en général fourni aucune information de connexion
donc Symfony ne peut pas savoir si l'utilisateur a tenté ou devrait se 
connecter par token ou par couple identifiant / mot de passe. C'est 
pourquoi ici quelque soit la méthode de connexion employée, l'utilisateur
sera redirigé sur le formulaire de connexion de l'appli.  

Si on le souhaite à l'avenir, on peut tout à fait surcharger la méthode
`start()` dans le `LoginFormAuthenticator` pour un peu mieux gérer les 
conditions de redirection, comme par exemple retourner une réponse
de type API (un json) si l'utilisateur appelle une URL commençant 
par API.