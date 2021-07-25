# Voters

On cherche à centraliser et rendre réutilisable la logique d'accès 
à la page d'édition écrite en fin du chapitre précédent.  

Pour ce faire, on commencer par remplacer la condition écrite
précédemment par celle-ci :
```PHP
if (!$this->isGranted('MANAGE', $article)) {
    throw $this->createAccessDeniedException('No access !');
}
```

En fait, à chaque exécution de la fonction `isGranted()`, Symfony
exécuter un **système de Voters**.  
Ce système a fonctionné jusqu'à présent de la manière suivante :
- Si la chaîne de caractères en paramètre de `isGranted` a commencé
par "ROLE_", alors le **RoleVoter** a vérifié si l'utilisateur avait
le rôle nécessaire ou non, et a retourné true ou false en fonction. 
Le **AuthenticatedVoter** s'est abstenu, et donc c'est **RoleVoter** 
qui a tranché;
- Si la chaîne de caractères a commencé par `IS_AUTHENTICATED_`, alors
le **AuthenticatedVoter** a vérifié si l'utilisateur était authentifié
anonymement/pleinement/par remember me, et a retourné true ou false en 
fonction. Le **RoleVoter** s'est abstenu, et donc c'est
l'**AuthenticatedVoter** qui a tranché.  
  
Mais maintenant, on va pouvoir ajouter notre Voter custom pour cette
nouvelle règle. En effet si on met juste "MANAGE" en paramètre de
`isGranted()`, alors les deux voteurs actuels s'abstiennent, et 
donc l'accès est par défaut refusé.  

Notons qu'on peut ajouter un deuxième paramètre à `isGranted`
pour permettre à certains voteurs de trancher sur certaines
décisions (ce qu'on va faire dans notre voteur custom avec
l'article).  

Notons aussi que les chaînes de caractères passées en premier
paramètre de `isGranted()` ("ROLE_USER",
"IS_AUTHENTICATED_FULLY", etc) sont appelées des **attributs
de permission**. Donc ici, notre nouveau voteur va se baser
sur notre nouvel attribut de permission "MANAGE" pour n'importe
quelle action sur l'article (édition, publication, suppression
par exemple).
