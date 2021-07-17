# Role Hierarchy

On va chercher ici à avoir une gestion des rôles un peu plus fournie que
juste la séparation binaire user/admin qu'on a actuellement.  

On peut par exemple ajouter un rôle par type d'utilisateur qui utilisera le 
site, par exemple, ROLE_EDITOR, ROLE_HUMAN_RESOURCES, etc.  

Mais le plus explicite est de nommer les rôles en fonction des accès 
qu'ils donnent sur le site, comme ROLE_ADMIN_ARTICLE pour notre 
`AdminArticleController` ou ROLE_ADMIN_COMMENT pour
`CommentAdminController`.  
C'est ce qu'on va faire ici: ainsi on va appliquer ces rôles dans les
annotations au-dessus de chacun des deux contrôleurs concernés, ainsi
que remplacer `ROLE_ADMIN` par `ROLE_ADMIN_ARTICLE` dans `base.html.twig`,
à l'endroit où on vérifiait si l'utilisateur avait bien ce rôle avant 
d'afficher le bouton "Create Post".  

Du coup maintenant dans l'état actuel de nos fixtures, le lien
http://localhost:8000/admin/comment est devenu inaccessible.  
Plutôt que d'ajouter le(s) rôle(s) concernés aux utilisateurs qu'on veut
dans les fixtures, on va faire en sorte que nos utilisateurs ayant un 
rôle `ROLE_ADMIN` aient aussi automatiquement accès aux deux nouveaux
rôles qu'on vient de créer.  

Et pour ce faire, on va appliquer la fonctionnalité de Symfony appelée 
`role_hierarchy`. Il suffit pour cela d'aller dans `security.yaml` et 
d'ajouter dans la clé `security` le contenu suivant :
```YAML
    role_hierarchy:
        ROLE_ADMIN: [ROLE_ADMIN_COMMENT, ROLE_ADMIN_ARTICLE]
```

Les pages de créations d'article ou d'administration de commentaires 
deviennent alors accessibles à tout utilisateur ayant le rôle 
`ROLE_ADMIN`.  

On peut bien entendu utiliser la `role_hierarchy` pour faire toutes les 
combinaisons de droits possibles sur les utilisateurs de notre appli.