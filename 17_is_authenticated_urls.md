# IS_AUTHENTICATED_ & Protecting All URLs

Pour vérifier qu'un utilisateur est connecté, il existe un moyen autre que 
de passer par le `ROLE_USER`.  
En effet, on peut aller dans `security.yaml` et renseigner dans `access_control` 
la ligne suivante :
```YAML
access_control:
        - { path: ^/account, roles: IS_AUTHENTICATED_FULLY }
```
La chaîne de caractères `IS_AUTHENTICATED_FULLY` ne fait que vérifier si 
l'utilisateur est connecté ou non. Ici, c'est donc strictement équivalent à faire
un `- { path: ^/account, roles: ROLE_USER }`.
 
Si on va sur la page /account et qu'on clique alors sur l'icône d'utilisateur
(qu'on appelle aussi l'icône Sécurité), on peut voir tout en bas de la page 
l'**Access Decision Log** que l'accès à la page est vérifié en premier lieu
via le `IS_AUTHENTICATED_FULLY` qu'on a mis dans `access_control`.  
Ensuite, une vérification du `ROLE_USER` est faite via l'AccountController,
puis une autre est faite via le template Twig, qui a aussi vérifié que notre
spacebar9 n'a pas le ROLE_ADMIN en 4ème ligne de l'Access decision log.  

On peut donc forcer la connexion sur toutes les pages d'un coup en modifiant
légèrement la ligne précédente dans `security.yaml` :
```YAML
access_control:
        - { path: ^/, roles: IS_AUTHENTICATED_FULLY }
```

Si on fait ça, puis qu'on se déconnecte via un `/logout` dans le navigateur, 
le navigateur nous retourne une erreur. En effet on a tellement restreint 
les accès qu'à présent, la page `/login` elle-même demande une authentification
préalable de l'utilisateur.  
Pour corriger ce point, on ajoute une ligne **avant** la précédente dans notre
partie `access_control` (on rappelle que Symfony lit les lignes du
`access_control` dans leur ordre d'apparition) :
```YAML
access_control:
        - { path: ^/login$, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/, roles: IS_AUTHENTICATED_FULLY }
```

La chaîne `IS_AUTHENTICATED_ANONYMOUSLY` signifie que toutes les sessions anonymes
ou non (les gens connectés aussi) ont accès ici à la page de connexion, ce qui 
résout notre souci.

On peut remplacer `IS_AUTHENTICATED_FULLY` par `IS_AUTHENTICATED_REMEMBERED` :
`IS_AUTHENTICATED_REMEMBERED` veut dire que la connexion par cookie "remember me"
passe aussi, alors que `IS_AUTHENTICATED_FULLY` nécessite obligatoirement un cookie
de session, pas juste un "REMEMBER_ME" :
```YAML
access_control:
        - { path: ^/login$, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/, roles: IS_AUTHENTICATED_REMEMBERED }
```

On peut même alors laisser certaines adresses en "REMEMBER_ME" mais forcer une
session non "REMEMBER ME" lors de certaines pages sensibles comme celle du
changement de mot de passe :
```YAML
access_control:
        - { path: ^/login$, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/change-password, roles: IS_AUTHENTICATED_FULLY }
        - { path: ^/, roles: IS_AUTHENTICATED_REMEMBERED }
```

Notons qu'on peut utiliser ces trois types de `IS_AUTHENTICATED_` dans un 
contrôleur ou en Twig.