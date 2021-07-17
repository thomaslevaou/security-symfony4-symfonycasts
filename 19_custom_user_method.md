# Custom User Method

Dans le HTML & CSS qu'on a ajouté dans la vue Twig sur notre page /account,
on a ajouté un champ "Twitter". Jusqu'à présent laissé vide, on va maintenant le 
remplir. Pour ce faire, on va mettre à jour notre entité :
`./bin/console make:entity`. Après avoir entré "User", on indique notre nouveau
champ souhaité `twitterUsername`, qui est une string de 255 caractères max 
nullable. On peut ensuite `./bin/console make:migration`, vérifier que tout est ok
dans le fichier, puis `./bin/console doctrine:migrations:migrate`.  

On met ensuite à jour les fixtures, en ajoutant juste ce if :
```PHP
if ($this->faker->boolean) {
   $user->setTwitterUsername($this->faker->userName);
}
```
Dans le premier `createMany()` de `loadData()`. On peut alors les charger via
un `./bin/console doctrine:fixtures:load`.  
Et ainsi afficher le nom d'utilisateur Twitter dans `account/index.html.twig`:
```HTML
{% if app.user.twitterUsername %}
    <h4 class="white">
        <i class="fa fa-twitter"></i> {{ app.user.twitterUsername }}</h4>
{% endif %}
```

Notons qu'après un reload de fixtures, l'id de notre compte avec lequel on était 
connectés dans le navigateur a été supprimé. De ce fait, on a été déconnecté de 
notre session utilisateur.  
Une fois reconnecté avec un utilisateur qui a un twitterUsername, on peut bien le 
voir sur l'interface.  


Maintenant ce qui nous intéresse va être de remplacer **proprement** l'image
de profil par l'avatar de l'utilisateur. Il suffirait en soi de copier l'URL
dans `base.html.twig` et d'y ajouter à la fin un paramètre GET "size=100*100".  
Mais comme ici on fait les choses proprement et qu'on ne veut pas copier
la même URL à deux endroits différents, on va créer notre propre méthode 
`getAvatarUrl()` dans l'entité `User`. Cette méthode contiendra le code suivant :
```PHP
public function getAvatarUrl(int $size = null): string
{
     $url = "https://robohash.org/".$this->getEmail();

     if ($size) {
         $url .= sprintf('?size=%dx%d', $size, $size);
     }
     
     return $url;
}
```

On peut alors récupérer l'avatar pour la page account de cette manière :
```HTML
<img src="{{ app.user.avatarUrl }}" class="img-responsive thumbnail">
```
(Oui, même si l'attribut `$avatarUrl` n'existe pas, Twig ne fait qu'appeler
`getAvatarUrl` donc ça passe)

Et appeler l'avatar pour la barre "profil" de droite de cette manière :
```HTML
<img class="nav-profile-img rounded-circle" src="{{ app.user.avatarUrl(100) }}">
```
(Oui, on peut mettre un paramètre comme ça entre parenthèses et ça passe)