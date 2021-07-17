# Dynamic Roles

Pour expliquer ce qui va suivre, on va créer une nouvelle page de compte 
utilisateur.  
On commence donc par faire un `./bin/console make:controller` en appelant notre
nouveau contrôleur `AccountController`.  
On lui fait faire un contrôleur assez basique :
```PHP
class AccountController extends AbstractController
{
    /**
     * @Route("/account", name="app_account")
     */
    public function index(): Response
    {
        return $this->render('account/index.html.twig', [
        ]);
    }
}
```

Idem pour le rendu Twig :
```HTML
{% extends 'base.html.twig' %}

{% block title %}Manage Account!{% endblock %}

{% block body %}
    <h1>Manage your Account</h1>
{% endblock %}
```

Et on va à présent vouloir rendre cette page accessible uniquement pour les
utilisateurs connectés.  

On a déjà vu qu'on pouvait restreindre l'accès aux utilisateurs ayant le rôle
`ROLE_USER` via une annotation au-dessus du contrôleur :
```PHP
/**
 * @IsGranted("ROLE_USER")
 */
class AccountController extends AbstractController
{
...
}
```

Avant de voir une autre manière de faire, on va ajouter des rôles à nos
utilisateurs, de sorte à en avoir au moins un ayant pour rôle 
ROLE_ADMIN et ainsi rendre la page /admin/comment accessible pour 
au moins un utilisateur.  
Pour ce faire, il suffit de créer un deuxième groupe d'utilisateurs
dans la méthode `loadData()` de `UserFixture`, qui fera cette fois
appel à la méthode `setRoles()` de l'entité `User`:
```PHP
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

    $this->createMany(3, 'admin_users', function($i) {
        $user = new User();
        $user->setEmail(sprintf('admin%d@thespacebar.com', $i));
        $user->setFirstName($this->faker->firstName);
        $user->setRoles(['ROLE_ADMIN']);
        $user->setPassword($this->passwordEncoder->encodePassword(
            $user,
            'engage'
        ));

        return $user;
    });
    $manager->flush();
}
```

Puis de faire un `./bin/console doctrine:fixtures:load` comme d'habitude.  
La page de commentaires devient alors accessible uniquement pour ces trois
administrateurs que nous venons de créer.  

On va alors remettre le menu de droite comme on attendrait d'un site normal :
si l'utilisateur est connecté, un menu dropdown, sinon : un lien "Login".  
Pour ce faire, il suffit de remplacer l'ancien code du bouton "Login"
et du gros bloc qu'on avait commenté dans `base.html.twig`, par 
celui ci-dessous :
```HTML
<ul class="navbar-nav ml-auto">
    {% if is_granted('ROLE_USER') %}
        <li class="nav-item dropdown" style="margin-right: 75px;">
            <a class="nav-link dropdown-toggle" href="http://example.com" id="navbarDropdownMenuLink" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
              <img class="nav-profile-img rounded-circle" src="{{ asset('images/astronaut-profile.png') }}">
            </a>
            <div class="dropdown-menu" aria-labelledby="navbarDropdownMenuLink">
                <a class="dropdown-item" href="{{ path('app_account') }}">Profile</a>
                {% if is_granted('ROLE_ADMIN') %}
                    <a class="dropdown-item" href="#{{ path('admin_article_new') }}">Create Post</a>
                {% endif %}
                <a class="dropdown-item" href="{{ path('app_logout') }}">Logout</a>
            </div>
        </li>
    {% else %}
        <li class="nav-item">
            <a style="color: #fff;" class="nav-link" href="{{ path('app_login') }}">Login</a>
        </li>
    {% endif %}
</ul>
```
Et voilà. Notons que le lien "Create Post" ne sera visible que par les admins,
et qu'on a précisé ce nouveau nom `admin_article_new` dans `AdminArticleController`.
Et qu'on peut utiliser la fonction Twig `is_granted()` pour vérifier les accès
d'un utilisateur en Twig.