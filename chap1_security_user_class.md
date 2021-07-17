# Security & the User Class

Comment faire marcher le code du cours :
- Suppression de toutes les occurrences de slack (le service ne doit faire qu'afficher
  des logs): dans le fichier de Service, et dans le `config/nexy_slack.yaml`
- `compose remove nexylan/slack-bundle`
- Suppression de toutes les occurrences de HttpLugbundle : dans le `bundles.php` et 
suppression du contenu du fichier .yaml associé
- `composer require doctrine/doctrine-cache-bundle`
- Remplacement des `RegistryInterface` par des `ManagerRegistry` avec le bon "use"
associé dans les repository.
  
Et voilà ça fait marcher les choses.  

Dans cette partie, on va chercher à authentifier des utilisateurs sur notre site.  

Pour ce faire, on va utiliser une classe **User**.  

Et pour l'utiliser, on va commencer par entrer la commande 
`composer update symfony/maker-bundle` pour s'assurer qu'on est au mois sur la 
version 1.7 de MakerBundle.  

Ensuite, on va installer **Security**, via un `composer require security`.
Ce composant Symfony a installé un fichier `config/packages/security.yaml` via 
sa Recipe.

On entre alors la commande `./bin/console make:user`. La classe va s'appeler
User (le nom par défaut). A la question nous demandant si on souhaite stocker 
les données de l'utilisateur en base de données. En général c'est pratique: 
même si les données sont stockées sur un autre serveur (LDAP ou SSO), c'est toujours
utile d'avoir des données sur les utilisateurs dans notre base de données.  
On ne met "non" que si on est sûr qu'on n'aura pas besoin de stocker des informations
sur les utilisateurs en base de données.  
Ensuite, on doit sélectionner la propriété qui va être "unique" pour cet utilisateur
et permettra de l'identifier sur une interface (le nom d'utilisateur ou l'e-mail
par exemple). Ici on va considérer que ce sera l'e-mail.  

Puis on doit indiquer si l'appli doit vérifier des mots de passe. Parce que dans
des applis par authentification par token, les utilisateurs n'utilisent parfois pas 
de mot de passe. Ou parfois les données d'utilisateurs sont envoyées à un autre
serveur, et donc ce n'est pas à l'appli Symfony de vérifier le mot de passe. 
Pour l'instant, on va mettre "no" pour des raisons pédagogiques (on reviendra dessus
plus tard).  


Notre appli a donc créé une entité `User`, un repository `UserRepository` et 
mis à jour le fichier de configuration `security.yaml`. On va voir les changements 
apportés dans les chapitres suivants.