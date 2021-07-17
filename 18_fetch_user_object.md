# Fetch the User Object

Dans `AccountController`, on souhaite récupérer l'utilisateur actuellement 
connecté pour en afficher ses données. Pour ce faire on va faire appel à la 
fonction `$this->getUser()` dans ce contrôleur, qui donne tout l'entité User
associée à l'utilisateur connecté.  
On peut par exemple s'en servir pour afficher l'e-mail de l'utilisateur 
connecté dans les logs debug du Profiler :
```PHP
public function index(LoggerInterface $logger): Response
    {
//        dd($this->getUser());
        $logger->debug('Checking account page for '.$this->getUser()->getEmail());
        return $this->render('account/index.html.twig', [
        ]);
    }
```


On peut cependant remarquer que la méthode `getUser()` est codée dans un
`ControllerTrait`, qui ne sait lui-même pas par défaut le type du retour 
de cette méthode. Comme c'est pas très propre, on va créer nous-mêmes notre propre
`BaseController`, qui va être codé comme ci-dessous :

```PHP
abstract class BaseController extends AbstractController
{
    protected function getUser(): User
    {
        return parent::getUser();
    }
}
```
La seule différence avec sa classe parente étant qu'on précise juste ici que 
le `getUser()` retourne un objet de type `User`.
Et faire hériter notre `AccountController` de ce `BaseController`.  

Comme vu sur Trëmma, il peut être fréquence de créer notre propre BaseController
pour centraliser des méthodes régulièrement utilisées dans notre application 
Symfony.  

Pour appeler des données de l'utilisateur en Twig, on peut utiliser `app.user` :
```HTML
<h1>Manage your Account {{ app.user.firstName }}</h1>
```
On rappelle que si on écrit `.firstName` en Twig, ce dernier appelle la méthode
`getFirstName()` de notre entité `User`, et ainsi de suite pour tous les autres
attributs de cette entité.  

On peut ainsi par exemple ajouter plus de html/css pour avoir une page plus 
réaliste, et la compléter avec des données utilisateurs (ce qu'on a fait ici).