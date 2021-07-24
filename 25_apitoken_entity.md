# ApiToken Entity

Dans le cadre de ce cours, nous allons créer un token et l'associer à 
un utilisateur directement en base de données.  

De ce fait, on va créer une entité pour ce token. On va donc entrer la commande
`./bin/console make:entity`, nommer notre nouvelle entité `ApiToken`, lui ajouter
un champ `token` qui sera une chaîne de caractères non nulle, un champ
`expiredAt` en tant que datetime non nul pour préciser sa date d'expiration, un
champ `user` qui décrira le user associé à ce token (un token n'a qu'un seul user
associé, mais un utilisateur peut avoir plusieurs token) de type relation à User
en ManyToOne qui ne pourra pas être nul (un user doit obligatoirement avoir un 
token non nul) dont l'entité User aura aussi une relation `apiTokens`
dans sa classe pour accéder plus rapidement à ses token en orphanRemoval à yes.  

On peut alors faire le classique `./bin/console make:migration`, qu'on peut alors
vérifier et lancer via un `./bin/console doctrine:migrations:migrate`.  

Ce cours ne couvre pas la création de token en OAuth2 ou autre. Nous allons 
juste créer les token dans les fixtures, comme si ceux-ci avaient déjà été 
créés auparavant par un système compatible avec les besoins de l'appli.  

Dans notre entité `ApiToken`, on va insister sur le fait que le User
est obligatoire en le précisant dans le constructeur de `ApiToken` :

```PHP
public function __construct(User $user)
    {
        $this->token = bin2hex(random_bytes(60));
        $this->user = $user;
        $this->expiresAt = new \DateTime('+1 hour');
    }
```

On souhaite alors rendre cette classe immutable: une fois instanciée,
on souhaite que cette classe ne puisse plus être modifiée. C'est 
pourquoi on a supprimé tous les accesseurs en écriture de la
classe.

Histoire de toujours garder des fixtures simples, on va gérer 
la création des fixtures des token dans `UserFixture`: 
```PHP
$this->createMany(10, 'main_users', function($i) use ($manager) {
   ...

   $apiToken1 = new ApiToken($user);
   $apiToken2 = new ApiToken($user);
   $manager->persist($apiToken1);
   $manager->persist($apiToken2);

   return $user;
});
```
On a ajouté `$manager` dans la fonction du 3ème argument de
`createMany` pour pouvoir l'utiliser quand on appelle `persist`.  
On a besoin d'appeler `persist` ici parce que le `persist` de 
la fonction `createMany` dans `BaseFixtures` ne s'occupe que 
de l'entité retournée par la fonction du 3ème argument, c'est-à-dire
ici `$user`.  

Bref on peut alors faire un `./bin/console doctrine:fixtures:load`.  
On peut ensuite voir les token créés avec succès en base via un 
`./bin/console doctrine:query:sql 'select * from api_token'`.