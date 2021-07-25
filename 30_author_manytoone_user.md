# Author ManyToOne Relation to User

Dans notre entité `Article`, l'attribut `author` n'est qu'une string.  

Pour remédier à cela, on supprime l'attribut `$author` de l'entité, ainsi que ses
accesseurs en lecture et en écriture, avant de lancer la migration via un 
`./bin/console make:migration`.  

On peut alors reprendre l'édition de notre entité via un
`./bin/console make:entity`. On créé une propriété `author`, qui sera de type 
`relation` sur l'entité `User` en `ManyToOne` (plusieurs articles peuvent avoir
le même auteur, mais plusieurs utilisateurs ne peuvent pas être ici auteurs du 
même article) non nullable, la propriété sera aussi accessible côté User sous le 
nom `articles`, et sans orphanRemoval (supprimer un auteur ne supprimera pas 
ses articles, bien que ça me paraisse bizarre vu que les articles auront un
auteur qui n'existera pas alors que c'est un champ obligatoire, mais je verrai
plus tard).  
On peut alors refaire un `./bin/console make:migration`.  

Notons qu'on se retrouve dans une situation où on a oublié de faire la 
première migration. Quand ça arrive, le mieux est de supprimer le dernier fichier
de migration, exécuter la précédente `./bin/console doctrine:migrations:migrate`,
puis de refaire un `./bin/console make:migration` avant de refaire encore un
`./bin/console doctrine:migrations:migrate`.  

Le problème c'est que du coup la migration plante, car la colonne author_id
est obligatoire tandis que les lignes actuellement en base n'ont pas de valeur
associée à ce sujet.  

Dans une appli déjà partie en prod, le mieux est de d'abord rendre la propriété 
`author` facultative pour les articles, puis exécuter un script/requête qui 
associe le bon author_id à chaque requête, puis re-faire une migration où 
l'author_id devient obligatoire.  

Comme ici, l'appli n'a pas été déployée en production, on va supprimer les tables
de la base de données via un
`./bin/console doctrine:schema:drop --full-database --force`, puis relancer les 
migrations via un `./bin/console doctrine:migrations:migrate`. Comme ça a bugué
j'ai aussi ajouté toutes les migrations de SymfonyCasts du dossier
`src/Migrations` dans `migrations`, édité le `DROP TABLE migration_version` en
`DROP TABLE IF EXISTS migration_versions`, et effectué deux requêtes SQL
custom:
```
php bin/console doctrine:query:sql "drop table user"
php bin/console doctrine:query:sql "drop table migration_versions"
```

Ce qui a fait tourner ensuite correctement
`./bin/console doctrine:migrations:migrate`.  

Pour associer correctement des auteurs à des articles dans les fixtures, on va 
faire comme dans les fixtures des commentaires, c'est-à-dire utiliser des
références telles qu'elles ont été écrites dans `BaseFixture`, via un 
`$article->setAuthor($this->getRandomReference('main_users'))` dans 
`ArticleFixtures`. Dans cette même classe, on peut supprimer le tableau
d'auteurs écrit en dur dans `$articleAuthors`. On doit aussi précser les 
dépendances d'`ArticleFixtures` dans le `getDependencies()` à la fin
de `ArticleFixtures` :
```PHP
public function getDependencies()
{
    return [
        UserFixture::class,
        TagFixture::class,
    ];
}
```

Et c'est tout, on peut alors
exécuter le classique `./bin/console doctrine:fixtures:load` et 
les fixtures sont bien implémentées en base: les articles ont tous
un author_id aléatoire parmi les id d'user disponibles.