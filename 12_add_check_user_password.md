# Adding & Checking the User's Password

Maintenant on va vraiment vérifier les mots de passe au lieu de mettre `true`
en dur dans notre méthode `checkCredentials()` de l'Authenticator.  

On rappelle que dans le chapitre 1, on a mis "non" quand la commande de création
d'utilisateur nous a demandé si Symfony devait gérer les mots de passe, pour des
raisons pédagogiques. De ce fait, notre classe `User` n'a pas encore d'attribut
du type "password". On va remédier à cela maintenant.  

Pour ce faire, on commence par entrer la commande `php bin/console make:entity`.  
En indiquant comme nom de classe "User", la console sait qu'on veut éditer cette
classe, et propose de lui ajouter un nouveau champ. C'est ce qu'on va faire en
ajoutant notre nouveau champ "password". Ce sera une string de 255 caractères 
maximum. Il ne pourra pas être null. Une fois ce champ ajouté, il suffit comme
souvent de taper `Entrée` au champ suivant pour quitter la console. Notons que 
la commande n'a pas généré ici de méthode `getPassword`, car elle existait déjà
dans notre entité `User`.  

On peut donc préparer la mise à jour de la base :
```
./bin/console make:migration
./bin/console doctrine:migrations:migrate
```
On rappelle que chaque `make:migration` doit être suivi d'une vérification
dans le nouveau fichier de migrations pour vérifier qu'il est ok.  

Maintenant, on peut bien vérifier dans notre classe `User` que l'attribut
`password` a été ajouté dans notre entité, ainsi qu'une nouvelle méthode 
`setPassword()`. On peut alors modifier le code de l'accesseur en lecture
pour qu'il tourne comme attendu :
```PHP
public function getPassword(): ?string
{
    return $this->password;
}
```

Bien entendu, le mot de passe ne sera pas écrit en dur, mais sera chiffré et salé.  
La plupart des encodeurs modernes incluant le salt en tant que partie du mot de
passe chiffré, nous utiliserons du salage sans avoir besoin d'utiliser la méthode
`getSalt()` de notre classe d'utilisateur.  

Afin de préciser l'algorithme de chiffrement de mots de passe que nous allons
utiliser, on va écrire une nouvelle partie `encoders:` juste en-dessous
de `security:` dans `security.yaml`:
```yaml
security:
    encoders:
        App\Entity\User:
            algorithm: auto
```
Le fait d'avoir écrit `auto` (disponible à partir de Symfony 4.3) 
implique que le meilleur algorithme disponible dans le "système"
(d'exploitation ?) sera employé. Comme je suis en Symfony 4.4.25
(vérifiable via un `./bin/console --version`), je peux m'en servir.  

A l'heure actuelle, les deux meilleurs algorithmes de chiffrement 
de mots de passe sont **bcrypt** et **argon2i**. L'algo argon2i 
est un peu plus sécurisé que bcrypt, mais n'est disponible qu'à partir de
PHP 7.2 ou en installant une extension appelée Sodium.  

Par contre, changer l'algorithme de chiffrement plus tard dans le temps sera
bien relou dans tous les cas. Pour les algos de chiffrement, qu'on appelle 
aussi **Encoders**, on peut aussi configurer l'option `cost:` dans le fichier yaml.
Un algorithme avec un cost élevé permettra des chiffrements plus sécurisés, mais 
consommera plus de CPU.  

En tout cas grâce à cette config, Symfony peut maintenant chiffrer les mots de passe
et vérifier si ceux-ci sont valides.  

On va donc à présent remplir les fixtures avec des mots de passe pour chaque 
utilisateur. Pour chiffrer les mots de passe des utilisateurs que nous allons
créer, on va faire appel au Service Symfony `UserPasswordEncoderInterface`.  

Ainsi, les méthodes de notre classe `UserFixture` ressemblent à présent à ceci:
```PHP
public function __construct(UserPasswordEncoderInterface $passwordEncoder)
{
    $this->passwordEncoder = $passwordEncoder;
}

protected function loadData(ObjectManager $manager)
{
    $this->createMany(10, 'main_users', function($i) {
       $user = new User();
       $user->setEmail(sprintf('spacebar%d@example.com', $i));
       $user->setFirstName($this->faker->firstName);
       $user->setPassword($this->passwordEncoder->encodePassword(
           $user,
           'engage'
       ));

       return $user;
    });
    $manager->flush();
}
```

Tous les mots de passe de nos utilisateurs seront "engage".  
Si la méthode `encodePassword` du `passwordEncoder` a besoin du `$user` en premier
paramètre, c'est pour récupérer l'algorithme de chiffrement à utiliser, qui est
précisé dans `security.yaml`.  

On peut donc à présent recharger les fixtures :
`./bin/console doctrine:fixtures:load`

Et donc en allant jeter un oeil en base (`./bin/console doctrine:query:sql`),
on peut voir les mots de passes chiffrés pour chaque utilisateur. Nos algos de 
chiffrement ont créé un SALT pour chaque utilisateur, qui fait partie du mot
de passe directement en base.  

Après avoir fait tout ça, il ne reste qu'à ajouter notre 
`UserPasswordEncoderInterface` comme nouveau paramètre du constructeur de 
`LoginFormAuthenticator`, et de faire appel à sa méthode `isPasswordValid()` 
dans `checkCredentials()`:
```PHP
public function checkCredentials($credentials, UserInterface $user)
{
    return $this->passwordEncoder->isPasswordValid($user, $credentials['password']);
}
```



