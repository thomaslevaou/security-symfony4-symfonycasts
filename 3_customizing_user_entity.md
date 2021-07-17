# Customizing the User Entity

Le système de sécurité de Symfony "n'accorde pas d'importance" à ce à quoi la classe User ressemble, tant qu'elle 
implémente méthodes de l'interface `UserInterface`.  

On peut notamment ajouter un champ "firstname" sans souci, et c'est ce qu'on va faire maintenant :
```
./bin/console make:entity (Classe User, champ firstName, le reste par défaut)
./bin/console make:migration (on peut bien voir dans le fichier de migrations que la colonne "roles" est de type
json, cf ci-dessous)
./bin/console doctrine:migrations:migrate
./bin/console make:fixtures (on va l'appeler UserFixture)
```


Dans la classe User, la propriété `roles` a l'annotation suivante :
```PHP
/**
 * @ORM\Column(type="json")
 */
private $roles = [];
```

L'annotation est là pour dire que la propriété est de type json : en base, ce sera soit une colonne de type
JSON si le SGBD le permet (par exemple sur MySQL 5.7 ou supérieur), soit une colonne de texte normale pour 
les versions de MySQL antérieures à 5.6 (qui sera alors encodé/décodé avec des `json_encode/json_decode`).  

Attention cependant, la version sur laquelle Doctrine se base est celle précisée dans le fichier 
`cnfig/packages/doctrine.yaml`, dans `doctrine: > dbal: > server_version:`. Attention à bien paramétrer 
ce fichier, notamment pour le rendre compatible avec la bdd de production (sous peine d'avoir une 
grosse erreur).  


La fixture `UserFixture` va être remplie avec le code suivant (avec un léger remaniement de BaseFixture, mais qui permettra
grosso modo de faire la même chose qu'avant) :
```PHP
<?php

namespace App\DataFixtures;

use Doctrine\Persistence\ObjectManager;

class UserFixture extends BaseFixture
{
    protected function loadData(ObjectManager $manager)
    {
        $this->createMany(10, 'main_users', function($i) {
           $user = new User();
           $user->setEmail(sprintf('spacebar%d@example.com', $i));
           $user->setFirstName($this->faker->firstName);

           return $user;
        });
        $manager->flush();
    }
}
```
Ce qui permet de charger correctement les fixtures via un `./bin/console doctrine:fixtures:load`.

Notons que pour faire marcher les annotations correctement, j'ai du réinstaller Gedmo :
``` 
composer require stof/doctrine-extensions-bundle
```
Cette Recipe a installé un nouveau fichier `stof_doctrine_extensions.yaml` dans `config/packages/`.  
Le contenu de ce fichier doit être modifié pour devenir le suivant :
```YAML
# Read the documentation: https://symfony.com/doc/current/bundles/StofDoctrineExtensionsBundle/index.html
# See the official DoctrineExtensions documentation for more details: https://github.com/Atlantic18/DoctrineExtensions/tree/master/doc/
stof_doctrine_extensions:
    default_locale: en_US
    orm:
        default:
            sluggable: true
            timestampable: true
```

Les nouveaux utilisateurs sont alors visibles en base via un `php bin/console doctrine:query:sql 'SELECT * FROM user'`.