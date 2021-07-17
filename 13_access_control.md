# access_control Authorization & Roles

Maintenant qu'on est pas trop mal niveau Authentification, on peut à présent se 
pencher sur l'**Authorization**. Tout le principe de l'Authorization consiste à 
savoir si un utilisateur peut ou ne peut pas avoir accès à quelque chose.
Par exemple, c'est via un système d'Authorization qu'on pourra déterminer les pages
qui auront besoin de se connecter pour être vues, ou de restreindre certaines parties
du site à des administrateurs uniquement.  

En fonction des situations, les deux manières de gérer l'Authorization sont
l'**access_control** et le **denying access**. On va commencer par voir 
l'access_control dans un premier temps.  

A la fin du fichier `security.yaml`, on peut voir une partie appelée
`access_control`. On peut la renseigner par exemple de la manière suivante :
```yaml
    access_control:
         - { path: ^/admin, roles: ROLE_ADMIN }
```
Ce qui impliquera que seuls les utilisateurs ayant un rôle ROLE_ADMIN pourront
accéder aux pages dont l'URL commence par `/admin`.  
En vérifiant la liste des URL actuelles du site (`./bin/console debug:router`),
on peut voir que quelques URL sont concernées par ce cas (`/admin/article/new` ou 
`/admin/comment`).  
Si on essaie d'y accéder quand on est connecté avec un compte, on se prend une 
erreur 403 "Access Denied". Alors que si on met "ROLE_USER" à la place de 
"ROLE_ADMIN" dans le yaml ci-dessus, notre utilisateur spacebar9@example.com
peut bien voir la page de commentaires.

En effet quand on cliquer sur l'icône d'utilisateur dans le Profiler, on peut
voir que son tableau de roles contient uniquement la valeur "ROLE_USER".  
Chaque utilisateur connecté va se voir attribuer un rôle (comme "ROLE_USER" par 
exemple). Tandis que certaines pages du site vont exiger certains rôles précis
pour être affichées.  

Le fait que notre utilisateur ait ce petit tableau de rôles est dû au code de 
la méthode `getRoles` de notre entité `User`:
```PHP
public function getRoles(): array
{
    $roles = $this->roles;
    // guarantee every user at least has ROLE_USER
    $roles[] = 'ROLE_USER';

    return array_unique($roles);
}
```

On rappelle que l'entité User dispose d'un attribut `$roles`, et donc que chaque
utilisateur a une colonne "roles" en base. Cette colonne étant pour le moment vide
pour tous les utilisateurs, la méthode `getRoles()` a ajouté uniquement le rôle
"ROLE_USER" à ce tableau vide.  
D'une manière générale, on doit s'assurer que la méthode `getRoles()`
retourne toujours au moins un rôle. Sinon, l'utilisateur devient un "zombie 
mort-vivant qui est 'en quelque sorte' connecté".  

On peut avoir autant de lignes dans la partie `access_control` de `security.yaml`
qu'on le souhaite. Notons quand même que Symfony commence par regarder la première
ligne dans cette partie, puis la suivante etc. jusqu'à s'arrêter quand il trouve un
access_control qui correspond à l'URL. Et ce, à chaque accès d'une page du site.  

J'imagine donc qu'il vaut mieux mettre les pages les moins restrictives en premier,
puis mettre les plus restrictives à la fin ? A voir sûrement dans les chapitres
suivants.




