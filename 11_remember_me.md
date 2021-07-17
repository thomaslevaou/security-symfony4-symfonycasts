# Adding Remember Me

Pour faire tourner la case à cocher "Remember Me", on va commencer par remplacer la
value "remember-me" de l'input par un `name="_remember_me"` (que Symfony ira
chercher) dans le formulaire Twig:
```HTML
<input type="checkbox" name="_remember_me"> Remember me
```

Puis dans le `security.yaml`, dans la partie `main:` on va ajouter le contenu 
suivant :
```yaml
remember_me:
    secret: '%kernel.secret%'
    lifetime: 2592000
```
2592000 c'est 1 mois en secondes.
Grâce à Symfony, dès qu'un utilisateur coche une case dont l'attribut `name` vaut
`_remember_me`, alors un cookie "Remember me" va être mis en place et re-connecter
tout seul l'utilisateur si sa session expire.  

L'option `secret` est un secret de chiffrement qui est une "signature" des données
du cookie.  
On rappelle que `kernel.secret` est un paramètre Symfony. Et que pour voir la liste 
de tous les paramètres Symfony existant, on peut entrer la commande 
`php bin/console debug:container --parameters` (et on peut voir qu'il y a pas mal
de paramètres).  
Les paramètres les plus importants sont ceux démarrant par `kernel`. On peut voir
que le paramètre `kernel` a pour valeur `%env(APP_SECRET)%`, et donc à la valeur
de la variable d'environnement `APP_SECRET`, qu'on peut voir dans le fichier `.env`.

Si on affiche l'inspecteur dans le navigateur au niveau de notre page de connexion,
on rappelle que les cookies sont accessibles dans Stockage > cookies sur Firefox,
et Application > Cookies sur Chrome.  
Au seul affichage de la page, il n'y a qu'un seul cookie appelé PHPSESSID.  

Mais après une connexion où on a coché "Remember Me", un deuxième cookie appelé 
`REMEMBERME` apparaît dans le navigateur.  
Une fois connecté, si on supprime le cookie PHPSESSID dans l'inspecteur de cookies
du navigateur et qu'on recharge la page, on est toujours connecté
(un cookie de session vide peut potentiellement alors être créé). Si on passe la 
souris sur notre identifiant "spacebar9@example.com" du Profiler, on peut voir 
que sa Token class est alors `RememberMeToken` (alors qu'avant c'était
`PostAuthenticationGuardToken`).
